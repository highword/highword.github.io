---
title: HTTPs证书验证 代码级过程
date: 2025-09-28 16:21:04
cover: https://mypersonaldata.oss-cn-shanghai.aliyuncs.com/img/202509272145639.png
categories:
  - Development
  - Baseline
tags:
  - Computer Science
  - Development
  - network
  - security
  - Tutorial
---

### 1. 浏览器出厂时预装了“根公钥池”

- Windows：注册表 `HKLM\SOFTWARE\Microsoft\SystemCertificates\ROOT\Certificates\`
    
- macOS：钥匙串 “System Roots”
    
- Firefox：自己维护 `cert9.db`  
    每条记录里最重要的两个字段：
    
    复制
    
    ```
    Subject : CN=DigiCert Global Root G3  
    Subject Public Key Info :  
       Algorithm=RSA (1.2.840.113549.1.1.1)  
       Modulus = 0x9E...（2048-bit 整数）  
    ```
    

浏览器就是把这段 modulus 当作“官方公钥”来用，**攻击者无法篡改**。

---

### 2. 服务器发来的证书链（以 TLS 1.2/1.3 为例）

抓包可以看到三条证书：

复制

```
Certificate[0] : 服务器叶子证书（*.example.com）  
Certificate[1] : 中间 CA “DigiCert TLS Hybrid ECC SHA384 2020 CA1”  
Certificate[2] : 根 CA “DigiCert Global Root G3”（可选，浏览器一般本地已有）
```

每条证书都是一段 DER 编码的 X.509 结构，核心字段：

复制

```
tbsCertificate      // 待签名部分（包含了公钥、域名、有效期等）
signatureAlgorithm  // 例如 1.2.840.113549.1.1.11 = sha256WithRSAEncryption  
signatureValue      // 对 tbsCertificate 的哈希再用上级私钥加密后的字节串
```

---

### 3. 浏览器先验证“根指纹”

把 Certificate[2] 的 `Subject Key Identifier` 与本地根库比对：

复制

```
本地根库 SKI : 0x75E6...  
证书 SKI      : 0x75E6...  → 匹配，说明这条根在信任列表里。
```

如果 SKI 对不上，直接弹 `SEC_ERROR_UNKNOWN_ISSUER`，后续都不用做。

---

### 4. 取出“根公钥”准备验签

从本地根记录里拿到 RSA 公钥参数：

复制

```
modulus  N = 0x9E2F...（256 B）  
exponent e = 0x010001
```

浏览器现在手里有了**可信的公钥**。

---

### 5. 对中间 CA 证书做签名验证

中间 CA 证书的 `signatureValue` 是 256 B 的大整数：

`sig = 0x4B1A...`

浏览器用根公钥对它做 **RSA 解密**（严格讲是“公钥运算”）：

复制

```
RSA_Verify(sig, N, e) → 得到 256 B 填充+哈希块  
按 PKCS#1 v1.5 结构解开：
   0x00 0x01 0xFF ... 0x00 || ASN.1 metadata || 32 B SHA-256 hash
```

取出 32 B 哈希：

`hash_from_sig = 0x3C7D...`

同时浏览器自己对 `tbsCertificate` 字段做独立哈希：

`hash_calc = SHA256(tbsCertificate) = 0x3C7D...`

**只有这两个哈希逐位相等，才认为“根 CA 确实用私钥签过这条中间证书”。**  
你的伪造签名因为**没有根 CA 私钥**，解密后的哈希必然对不上，这一步就失败。

---

### 6. 叶子证书再用中间 CA 的公钥重复一次

中间 CA 的公钥在上一张证书里的 `Subject Public Key Info` 字段：

复制

```
Algorithm = id-ecPublicKey (1.2.840.10045.2.1)  
namedCurve = secp384r1 (1.3.132.0.34)  
pubKey = 0x04B6...（97 B 未压缩点）
```

浏览器用 **ECDSA 签名验证**（SHA-384 哈希 + secp384r1 曲线）再跑一次，流程与 RSA 类似，只是算法换成 ECC。  
伪造者如果换成自己的 ECC 公钥，这一步验签同样会失败。

---

### 7. 关键字段额外校验

即使签名碰巧对得上（现实中不可能），浏览器还会硬编码检查：

- **有效期** `notBefore / notAfter`
    
- **密钥用法** `KeyUsage=keyCertSign, cRLSign`（中间 CA 必须置位）
    
- **基本约束** `BasicConstraints CA=true, pathLen=0`
    
- **名称约束** `NameConstraints permitted=*.example.com`
    
- **吊销状态** OCSP 或 CRL：  
    浏览器会实时发 OCSP 请求 `http://ocsp.digicert.com`，得到 `certStatus=good` 才继续。  
    你把域名改成自己的，OCSP 返回 `unknown` 或超时，也会被拒。


### 一句话总结

浏览器**用本地已知的根公钥去“试解密”每一层签名**，只有解密出的哈希与自己独立算出的哈希**逐位相等**才继续；  
伪造者因为没有受信任 CA 的**私钥**，解密结果必然对不上，于是**在第一步 RSA/ECDSA 验证就失败**，整条信任链立刻断裂。
