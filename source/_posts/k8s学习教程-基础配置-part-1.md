---
title: k8s学习教程(基础配置)--part 1
date: 2023-11-05 20:25:42
cover: https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230115202715679.png
categories:
  - Computer Science
  - Development
  - Operations
tags:
  - Experience
  - Computer Science
  - Development
  - k8s
  - Ops
---
# k8s学习教程（一、初始配置）

<img src="https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230115202715679.png" alt="image-20230115202715679" style="zoom: 33%;" />

## 1 minikube 启动过程

在启动之前，需要下载docker！！！

### 1.1 minikube start

启动指令

```shell
# 启动指令
# 先设置一下k8s版本   注意：大坑  不明原因
sudo usermod -aG docker lucky && newgrp docker
minikube config set kubernetes-version v1.23.3
minikube start --image-mirror-country='cn'
```

需要切换一个用户 同时需要用ssh工具重新连接这个新用户，不可以su 新用户，这样才可以启动dashboard

```shell
minikube dashboard
# 设置minikube dashboard的ip和端口为本机，以便外网访问 --address 为 ip
kubectl proxy --port=7999 --address='202.120.87.115' --accept-hosts='^.*' &
```

### 1.2 minikube stop

```
minikube stop 
minikube delete
minikube start
```

## 2 minikube 集群部署初体验

### 2.1 kubectl

常用指令

```shell
# 通过 yaml配置文件 部署nginx集群
kubectl apply -f nginx.yaml
# 通过created方式创建并部署nginx集群
kubectl create deployment nginx --image=nginx
# 以nodeport方式暴露端口 80 -> 8080
kubectl expose deployment nginx --port=8080 --target-port=80 --type="NodePort"
# port-forward 端口转发且允许任意ip访问，即可在别处访问 8080 -> 8080
kubectl port-forward --address 0.0.0.0 -n default service/nginx 8080:8080
```

nginx.yaml文件内容

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

```

查看pod和service指令

```shell
kubectl get pods
kubectl get services
# 查看pod详细信息
kubectl describe deployment nginx
#查看service相信信息
kubectl describe service nginx
```





