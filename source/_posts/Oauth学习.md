---
title: Oauth学习总结
date: 2023-11-07 09:59:04
cover: https://mypersonaldata.oss-cn-shanghai.aliyuncs.com/img/202509282131309.png
categories:
  - Computer Science
  - Development
tags:
  - Experience
  - Computer Science
  - Development
  - security
  - network
---





# 什么是OAuth?

OAuth 不是一个API或者服务，而是一个验证授权(Authorization)的开放标准，所有人都有基于这个标准实现自己的OAuth。

更具体来说，OAuth是一个标准，app可以用来实现`secure delegated access`. OAuth基于HTTPS，以及APIs，Service应用使用`access token`来进行身份验证。

OAuth主要有OAuth 1.0a和OAuth 2.0两个版本，并且二者完全不同，且不兼容。OAuth2.0 是目前广泛使用的版本，我们多数谈论OAuth时，为OAuth2.0。
为什么要有OAuth?

-----------

在OAuth之前，`HTTP Basic Authentication`, 即用户输入用户名，密码的形式进行验证, 这种形式是不安全的。OAuth的出现就是为了解决访问资源的安全性以及灵活性。OAuth使得第三方应用对资源的访问更加安全。
OAuth 中心组件

----------

OAuth 主要下面中心组件构成 (Central Components), 接下来会依次介绍如下这些组件。

* Scopes and Consent
* Actors
* Clients
* Tokens
* Authorization Server
* Flows

### OAuth Scopes

Scopes即Authorizaion时的一些请求权限，即与access token绑定在一起的一组权限。OAuth Scopes将授权策略（Authorization policy decision）与授权执行分离开来。并会很明确的表示OAuth Scopes将会获得的权限范围。

### OAuth Actors

OAuth的流程中，主要有如下四个角色。其关系如下图所示：

* Resource Owner: 用户拥有资源服务器上面的数据。例如：我是一名Facebook的用户，我拥有我的Facebook 个人简介的信息。
* Resource Server: 存储用户信息的API Service
* Client: 想要访问用户的客户端
* Authorization Server: OAuth的主要引擎，授权服务器，获取token。

![](https://pic2.zhimg.com/80/v2-7f724c320f76f07e7667ca9efafd0041_1440w.webp)



### OAuth Tokens

* Access token: 即客户端用来请求Resource Server(API). Access tokens通常是short-lived短暂的。access token是short-lived, 因此没有必要对它做revoke, 只需要等待access token过期即可。
* Refresh token: 当access token过期之后refresh token可以用来获取新的access token。refresh token是long-lived。refresh token可以被revoke。

Token从Authorization server上的不同的endpoint获取。主要两个endpoint为`authorize endpoint`和`token endpoint`. authorize endpoint主要用来获得来自用户的许可和授权(consent and authorization)，并将用户的授权信息传递给`token endpoint`。token endpoint对用户的授权信息，处理之后返回`access token`和`refresh token`。 当access token过期之后，可以使用refresh token去请求token endpoint获取新的token。（开发者在开发endpoint时，需要维护token的状态，refresh token rotate）

![](https://pic4.zhimg.com/80/v2-4c08d08ea39dfcf1d4b89e50812c7103_1440w.webp)



OAuth有两个流程，1.获取Authorization，2. 获取Token。这两个流程发送在不同的channel，Authorization发生在Front Channel（发生在用户浏览器）而Token发生在Back Channel。

* Front Channel: 客户端通过浏览器发送Authorization请求，由浏览器重定向到Authorization Server上的Authorization Endpoint，由Authorization Server返回对话框，并询问“是否允许这个应用获取如下权限”。Authorization通过结束后通过浏览器重定向到回调URL（Callback URL）。
* Back Channel: 获取Token之后，token应有由客户端应用程序使用，并与资源服务器（Resource Service）进行交互。

![](https://pic2.zhimg.com/80/v2-f2ac72dd2848265ac1b2deeab5cbec75_1440w.webp)



下面就以实际的OAuth authorization code模式结合HTTP请求来说明Front Channel和Back Channel。

### Front channel

![](https://pic3.zhimg.com/80/v2-fdbfa42035c5f9aebe1d71b37803337e_1440w.webp)



```java
Request:
    GET https://accounts.google.com/o/oauth2/auth?scope=gmail.insert gmail.send
    &redirect_uri=https://app.example.com/oauth2/callback
    &response_type=code&client_id=812741506391
    &state=af0ifjsldkj
```

GET请求，指定了redirect_uri, 完成authorization之后，需要重定向到哪里。 response_type表明是用哪种OAuth flow进行验证。State为安全标志位，类似于XRSF，更多XRSF可以了解Cross-Site-Request-Forgery (跨站请求伪造)。

```java
Response:
    HTTP/1.1 302 Found
    Location: https://app.example.com/oauth2/callback?
    code=MsCeLvIaQm6bTrgtp7&state=af0ifjsldkj
```

返回的code即表明，已经获得授权authorization grant. state用来保证不是伪造的请求，和request传入的保持一致。

### Back channel

![](https://pic3.zhimg.com/80/v2-9c0226376719764fd9b534bdcb8e2872_1440w.webp)



```js
Request:
    POST /oauth2/v3/token HTTP/1.1
    Host: www.googleapis.com
    Content-Type: application/x-www-form-urlencoded
    code=MsCeLvIaQm6bTrgtp7&client_id=812741506391&client_secret={client_secret}&redirect_uri=https://app.example.com/oauth2/callback&grant_type=authorization_code
```

请求的参数，code即为上一步front channel所返回的code。

```js
Response:
    {
      "access_token": "2YotnFZFEjr1zCsicMWpAA",
      "token_type": "Bearer",
      "expires_in": 3600,
      "refresh_token": "tGzv3JOkF0XG5Qx2TlKWIA"
    }
```

当获取到access token之后，就可以在Authorization header中使用token，进行对资源服务器的请求访问
    curl -H "Authorization: Bearer 2YotnFZFEjr1zCsicMWpAA" \
      https://www.googleapis.com/gmail/v1/users/1444587525/messages

### OAuth Flows

* implicit flow: 也称之为 2 Legged OAuth 所有OAuth的过程都在浏览器中完成，且access token通过authorization request (front channel only) 直接返回。不支持refresh token。安全性不高。
* Authorization code: 也称之为 3 Legged OAuth。使用front channel和back channel。front channel负责authorization code grant。back channel负责将authorization code换成（exchange）access token以及refresh token。
* Client Credential flow: 对于server-to-server的场景。通常使用这种模式。在这种模式下要保证client secret不会被泄露。
* Resource Owner Password Flow：类似于直接用户名，密码的模式，不推荐使用。

下图即为Authorization code模式下的主要流程图

![](https://pic2.zhimg.com/80/v2-a0efc677da376bf02b6b3c57d57d30e9_1440w.webp)
安全性建议
-----

* 使用CSRF token。state参数保证整个流程的完整性
* 重定向URL（redirect URIs）要在白名单内
* 通过client ID将authorization grant和token request确保在同一个client上发生
* 对于保密的client（confidential client），确保client secret不被泄露。不要将secret随代码一起发布

OpenID Connect
--------------

OpenID Connect 是在OAuth2.0 协议基础上增加了身份验证层 （identity layer）。OAuth 2.0 定义了通过access token去获取请求资源的机制，但是没有定义提供用户身份信息的标准方法。OpenID Connect作为OAuth2.0的扩展，实现了Authentication的流程。OpenID Connect根据用户的 `id_token` 来验证用户，并获取用户的基本信息。

`id_token`通常是JWT（Json Web Token），JWT有三部分组成，header，body，signature。header主要用来声明使用的算法，声明claim在body中，并且签名在signature中。OpenID Connection 在OAuth2.0 的基础上额外增加了UserInfo的Endpoint。`id_token`作为访问UserInfo Endpoint的凭证来获取用户的基本信息（profile，email，phone），并验证用户。



![](https://pic4.zhimg.com/80/v2-faea5f70a2a96217d7a627b50a364deb_1440w.webp)



OpenID Connect流程主要涉及如下几个步骤：

1. 发现获取OIDC metadata
2. 执行OAuth流程，获取`id_token`和`access_token`。例如：在 `Authorization code`模式下即为通过code来换取`id_token`和`access_token`。
3. 获取JWT签名（signature key）并且可选的动态的注册客户端应用
4. 基于日期签名来本地验证JWT `id_token`，或者将`id_token`发给后端backend进行验证
5. 根据`id_token`通过UserInfo Endpoint获取用户信息，根据`access_token`获取用户其他资源信息

总结
--

OAuth2.0 不是一个Authentication Protocol， 而是一个Authorization framework，授予应用对API的访问权限（delegate access to APIs）。OAuth设定了对于API访问的scope的权限，以及支持多种授权方式，以及使用场景。OAuth提供了更好的安全性以及便利，简化了软件系统的复杂性。





