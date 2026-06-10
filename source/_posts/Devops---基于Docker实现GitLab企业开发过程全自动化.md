---
title: Devops---基于Docker实现GitLab企业开发过程全自动化
date: 2023-11-04 20:48:41
cover: https://mypersonaldata.oss-cn-shanghai.aliyuncs.com/img/202509282133057.png
categories:
  - Computer Science
  - Development
  - Operations
tags:
  - Handbook
  - Computer Science
  - Development
  - docker
  - DevOps
  - Ops

---
# Devops---基于Docker实现GitLab企业开发过程全自动化

## 搭建该流程化过程起因

原项目gitlab仓库所在私服磁盘被docker数据占满，无法访问，容器寄掉了。于是自己动手在自己的私服上不想浪费那1TB的磁盘，于是从0开始完整搭建了gitlab仓库（企业用的比较多）、gitlab-runner（实现Devops的关键）、部署的全过程。下图是GitLab官网的介绍：[The DevSecOps Platform | GitLab](https://about.gitlab.com/)

![image-20230404213900259](https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230404213900259.png)

## 一、搭建gitlab仓库

首先介绍一下如何搭建gitlab仓库。

```shell
# 搜索gitlab版本 我一般都用最新版latest
docker search gitlab/gitlab-ce
# 拉取镜像
docker pull gitlab/gitlab-ce
```

我没用docker run的命令来启动gitlab，因为我怕忘记命令，所以用的docker-compose.yml文件启动方式。

```yaml
version: '3.1'
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    container_name: gitlab
    restart: always
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://ip:8929' # 此处ip替换为你当前服务器/虚拟机的ip
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    ports:
      - '8929:8929'
      - '2224:2224'
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'
```

```shell
# 启动容器
docker-compose up -d
```

注意：刚启动容器初始化比较慢，可能一直访问不了或者报502错误，不要紧张，等一会儿就行了。可以通过下面的命令来查看容器日志。

```shell
docker logs -f [容器id/容器名]
```

![image-20230404210036687](https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230404210036687.png)

gitlab默认用户名是root，密码需要我们进入gitlab容器内查看

```
docker exec -it gitlab cat /etc/gitlab/initial_root_password
```

![image-20230404210100141](https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230404210100141.png)

登录之后就可以在设置里面修改密码。

之后就可以完成你想进行的操作了：repo、ci/cd等等。

## 二、搭建gitlab-runner



同样 docker安装gitlab-runner

```shell
docker search gitlab/gitlab-runner
```

拉取gitlab-runner镜像

```shell
docker pull gitlab/gitlab-runner
```

docker-compose.yml启动容器：

```yaml
version: '3.1'
services:
    runner-guanfang:
      image: gitlab/gitlab-runner:latest
      container_name: gitlab-runner
      restart: always
      privileged: true
      volumes:        #务必保证 /home/runner/config有写的权限否则容器启动会失败
        - ./config:/etc/gitlab-runner  #容器与宿主机runner配置文件挂载，防止容器重启或者recreate数据丢失，这个很重要。
        - /var/run/docker.sock:/var/run/docker.sock #用于runner容器共享宿主机的docker，不然在runner容器里起容器端口挂载就会有问题了。解决Docker in Docker的问题。
```

启动容器：

```shell
docker-compose up -d
```

gitlab-runner信息注册关联gitlab仓库

```shell
# 第一个gitlab-runner是容器名，第二个是image
docker exec -it gitlab-runner gitlab-runner register
```

记下来就是注册信息 需要注意：最后选择docker版本的时候建议：`docker:latest`

![img](https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/479335d429e84e679aa410435d5bb080.png)

上图是你的gitlab仓库的某个项目的CI/CD信息，可以看到URL和token，这是注册时需要的信息。注意注册时会有tags标签，这个在gitlab-ci.yml中需要指定，所以尽量不要乱起名字，需要保持一致，后续也可以在gitlab中修改。

### gitlab-runner/config关键配置信息修改

在挂载的数据卷目录下：比如gitlab-runner/config下有个config.toml文件，修改她。

```shell
vim config.toml
```

基本配置如下：

```toml
concurrent = 1
check_interval = 0
shutdown_timeout = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "kirinea's gitlab-rnner"
  url = "http://ip:8929/"   # gitlab的启动URL
  id = 1
  token = "************"    # gitlab仓库ci/cd下runner 的 token
  token_obtained_at = 2023-04-03T06:04:11Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"
  [runners.cache]
    MaxUploadedArchiveSize = 0
  [runners.docker]
    privileged = false
    tls_verify = false
    image = "docker:latest"    # Docker in Docker 就按这个来准没错
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock", "/usr/bin/docker:/usr/bin/docker", "/root/.m2:/root/.m2", "/root/.ssh:/root/.ssh", "/root/.docker/:/root/.docker/", "/root/.npm/:/root/.npm/"] #前提本机需要有安装maven、npm、打开ssh、docker等
    shm_size = 0
```

好了 ，接下来就可以编写你项目的`gitlab-ci.yml`文件了。

参考：[Docker安装Gitlab和Gitlab-Runner并实现项目的CICD-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/719968)

## 三、结合CI/CD执行build、deploy等自动化流程

![image-20230404212424737](https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230404212424737.png)

![image-20230404212457942](https://luckyblob.oss-cn-shanghai.aliyuncs.com/postimgs/image-20230404212457942.png)

## gitlab-ci.yml 和 Dockerfile 写法注意

我们可以把ci/cd当作一个大的脚本服务，里面可以打包镜像，执行docker build上传镜像，这里建议私有库，比如阿里云。

流程一般如下：

```shell
docker build
docker tag
docker push
docker rmi
```

在gitlab-ci.yml一般分为多个stage：build、test、deploy、cleanup

根据自己需求编写script。

一般build阶段如上所述，deploy阶段通过类似于`ssh ip bash ***.sh`的脚本（前提是将runner所在服务器的ssh公钥放入目标部署服务器的可信公钥） ，这个脚本一般是

```shell
docker pull、
docker-compose up -d
等等等等启动容器以及微服务注册中心等工具
```

**以上就是大致的整个过程 非常有意思和有意义**