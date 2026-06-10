---
title: k8s初体验-minikube突破隔断访问限制(以nginx为例)
date: 2023-11-10 09:08:25
cover: https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230310101006602.png
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
# minikube部署对外访问配置

我们现在大家都知道了 minikube集群相当于k8s的一个虚拟机。

同时pod每次消亡重建之后都会分配一个虚拟的ip，这个ip可供集群内部访问，但是如果要对外访问呢？

我们可以给他暴露ip出来，但是如果pod宕机后重建ip就发生了变化，所以不能对pod进行ip的暴露。

**那应该对什么进行暴露呢?**

我们首先需要了解service，简而言之，为了解决pod动态变化的虚拟ip，因此service 相当于某组pod的外部访问接口。因此我们对service为单位进行ip的暴露即可。这同时也解决了在minikube中一开始学习的时候必须要进入minikube之后才可以访问pod的问题。

<img src="https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20200408194716912-16783627155621.png" alt="From 黑马" style="zoom:67%;" />

```shell
lucky@thinkcentre:/work/k8s$ kubectl expose deploy nginx --name=svc-nginx1 --type=ClusterIP --port=80 --target-port=80 -n dev
service/svc-nginx1 exposed
lucky@thinkcentre:/work/k8s$ kubectl get svc svc-nginx1 -n dev -o wide
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
svc-nginx1   ClusterIP   10.111.120.200   <none>        80/TCP    30s   run=nginx
lucky@thinkcentre:/work/k8s$ curl 10.111.120.200:80
^C
lucky@thinkcentre:/work/k8s$ minikube ssh
Last login: Thu Mar  9 10:23:21 2023 from 192.168.49.1
docker@minikube:~$ curl 10.111.120.200:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
docker@minikube:~$

```

上面暴露端口的方式是clusterIp，因此还是只是集群内部可以访问，只不过相比较上一节minikube外部无法访问pod来说没有实质性的改变，改变的只不过是访问svc从而访问高可用pod。因此我们想要minikube外部可以访问的话，就需要将type修改为NodePort

```shell
kubectl expose deploy nginx --name=svc-nginx2 --port=80 --target-port=80 --type=NodePort -n dev
# port-forward 端口转发且允许任意ip访问
kubectl port-forward --address 0.0.0.0 -n dev service/svc-nginx2 80:80
```



执行上述命令后 即可在同一网段内访问nginx，不受minikube限制，如果想让外网访问，则需要对80端口做一个内网穿透，参考我之前的文章即可。

**其实从port-forward可以看出本质是端口转发到主机而已**
<img src="https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230310090645549.png" alt="image-20230310090645549" style="zoom:50%;" />

