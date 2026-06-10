---
title: 基于Docker和vsftpd协议搭建FTP服务器
date: 2023-10-16 16:37:40
cover: https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230416164142324-16816345069691.png
categories:
  - Computer Science
  - Development
  - Operations
tags:
  - Handbook
  - Computer Science
  - Development
  - docker
  - Ops
---
## FTP内置服务器搭建

注意端口映射2020 -> 20；2121 -> 2121（如果你的20和21端口被占用的话）

```yml
version: '3'
services:
  vsftpd:
    image: fauria/vsftpd
    restart: always
    container_name: ftp_for_kirin
    ports:
      - "2020:20"
      - "2121:21"
      - "21100-21110:21100-21110"
    volumes:
      - ./files:/home/vsftpd
    environment:
      - FTP_USER=admin
      - FTP_PASS=admin
      - PASV_ADDRESS=127.0.0.1
      - PASV_MIN_PORT=21100
      - PASV_MAX_PORT=21110
      - LOG_STDOUT=true
      - LOG_STDERR=true
      - LOG_FILE=/var/log/vsftpd.log
```

文件存储目录在宿主机的`/work/docker/ftp/files/kirin/`

用户名和密码自己设置即可

`PASV_ADDRESS`为自己的宿主机`ip`

测试成功与否：`MobaXTerm`或者`xftp`这类终端工具，可以连接上并且上传文件正常即搭建成功。

![image-20230416164014862](https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230416164014862.png)