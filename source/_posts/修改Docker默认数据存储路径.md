---
title: Ubuntu下修改docker默认数据存储路径
date: 2023-10-30 21:56:29
tags: Docker
---
# `Ubuntu`下修改Docker默认存储路径

事件起因：发现一个大问题，docker虽然用着很舒服，但是用的不好就会存在一个大问题，比如你没钱购买磁盘空间大的服务器，但是docker的数据日志是一直在增长变大，非常占空间，所以我们可以挂载一个新的磁盘用来当作docker的数据日志目录，并且可以把日志数据全部迁移到这个新的磁盘中。

## 步骤如下：

### 1、查看docker默认存储目录

```shell
[root@thinkcenter ~]# docker info
...
Docker Root Dir: /var/lib/docker
...
```

### 2、停止docker服务

```shell
systemctl stop docker
```

### 3、修改存储路径

```shell
vim /usr/lib/systemd/system/docker.service

#在EXECStart的后面增加 --data-root /lucky/docker (/lucky/docker是自己创建的新存储位置)
 
ExecStart=/usr/bin/dockerd  -H fd:// --containerd=/run/containerd/containerd.sock  --data-root=/lucky/docker

```

#### 注意：`ubuntu`下是data-root，在`cent-os`下是graph

### 4、迁移docker已经存在的数据

```shell
[root@thinkcenter ~]# rsync -avz /var/lib/docker/ /lucky/docker/    # 同步数据
[root@thinkcenter ~]# ll /lucky/docker/        # 确认同步后的数据
```

### 5、配置生效并重启docker

```shell
[root@localhost docker]# systemctl daemon-reload
[root@localhost docker]# systemctl start docker
```

### 6、查看修改后的目录是否改变

```shell
[root@thinkcenter ~]# docker info
...
Docker Root Dir: /lucky/docker
...
```

### 7、查看原docker容器是否运行正常

```shell
docker ps
docker ps -a
```

### 8、放心删除原数据

```shell
[root@harbor ~]# rm -rf /var/lib/docker/      # 删除原根目录
```

## 注意：`ubuntu`下执行`apt update / apt upgrade`之后docker的目录会变回默认存储目录！！！



参考：https://blog.csdn.net/m0_46600592/article/details/129490458