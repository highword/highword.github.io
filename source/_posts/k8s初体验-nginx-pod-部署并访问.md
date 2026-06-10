---
title: k8s初体验---nginx(pod)部署并访问
date: 2023-11-09 18:26:19
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
# k8s初学习---基于k8s部署nginx

创建命名空间dev

```shell
kubectl create ns dev
```

运行nginx

```shell
kubectl run nginx --image=nginx:1.17.1 --port=80 --namespace=dev
```

![image-20230308112157646](https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230308112157646.png)



查看某个命名空间下的pod的信息 根据pod-name

```shell
kubectl get pod nginx -n dev
```

试试访问pod

```shell
curl 172.17.0.5:80
```

<img src="https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230309160542890.png" alt="image-20230309160542890" style="zoom:50%;" />

很尴尬 不能访问 果然教程不帮你踩坑。

问了chatgpt，给出几种方案。我试了第一种

首先 我用`minikube ssh`指令之后 进入minikube内部，再访问居然成功了。

<img src="https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230309160735086.png" alt="image-20230309160735086" style="zoom:50%;" />

接下来我继续问chatGPT这是为什么 ？

<img src="https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230309161439775.png" alt="image-20230309161439775" style="zoom:50%;" />

但是实际上我测试了一下都是正常的 chatgpt并没有帮我解决？

接下来就问了师兄，师兄一语中的，师兄总是那么神。

<img src="https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230309161606272.png" alt="image-20230309161606272" style="zoom:50%;" />

所以chatGPT其实有的时候也并不是那么神啦！！！

<img src="https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230309161656417.png" alt="image-20230309161656417" style="zoom:50%;" />

到此结束！

