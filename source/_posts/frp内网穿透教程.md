---
title: frp内网穿透教程
date: 2023-10-18 10:44:41
cover: https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230218213701529.png
tags: Guidance
---


# frp搭建内网穿透教程

<img src="https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230218213701529.png" alt="image-20230218213701529" style="zoom:50%;" />

我们用过服务器的同学都知道ssh，就是一个远程连接服务器的协议，如果是对于实验室的电脑则需要在同一局域网下才可以连接。用过阿里云服务器或者腾讯云的同学也知道，这种公网ip的服务器可以直接不受局域网影响即可访问。

所以可能就有同学想问是否可以把实验室的服务器也弄成公网。答案是：有

- 花生壳（不推荐）

![image-20230218212009006](https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230218212009006.png)

- frp（推荐）

1、前提需要一台公网服务器，比如阿里云用来当作frp的服务器端，实验室的电脑则作为客户端。（可借用师兄或者某台作为服务器端）

frp仓库：https://github.com/fatedier/frp

下载后解压 `tar -zxvf ***.tar.gz` 再进入该目录

`vim frps.ini` 编辑服务器端配置 for example

```ini
[common]
bind_port = 7000
dashboard_port = 7500
token = 123456789
dashboard_user = lucky
dashboard_pwd = 111111111   #该三部分自己设置
```

2、运行服务器端

```shell
nohup ./frps -c frps.ini & #建议nohup保持云服务器不关机
```

3、客户端

下载过程以及解压过程如上所述

不同之处：

`vim frpc.ini`

```ini
[common]
server_addr = ***.***.**.** #公网ip
server_port = 7000 #服务器端口
token = 123456789

[ssh]
type = tcp
local_ip = 127.0.0.1 #本地ip
local_port = 22 #需要映射的端口
remote_port = 222 #映射到公网服务器的哪个端口
```

4、运行

```shell
nohup ./frpc -c frpc.ini &
```

5、测试连接即可成功





参考：

https://blog.csdn.net/weixin_49764009/article/details/122018688