---
layout: post
cid: 31
title: 学习docker看此文足以
slug: 31
date: 2021/12/30 17:07:00
updated: 2022/03/25 15:49:46
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: 
keywords: 
mode: default
thumb: 
video: 
---


### 什么是 Docker

Docker 最初是 dotCloud 公司创始人  在法国期间发起的一个公司内部项目，它是基于 dotCloud 公司多年云服务技术的一次革新，并于 ，主要项目代码在  上进行维护。Docker 项目后来还加入了 Linux 基金会，并成立推动 。

Docker 自开源后受到广泛的关注和讨论，至今其  已经超过 5 万 7 千个星标和一万多个 fork。甚至由于 Docker 项目的火爆，在 2013 年底，。Docker 最初是在 Ubuntu 12.04 上开发实现的；Red Hat 则从 RHEL 6.5 开始对 Docker 进行支持；Google 也在其 PaaS 产品中广泛应用 Docker。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d8004eb674b46c4a7d0f62143504ea4~tplv-k3u1fbpfcp-zoom-1.image)  

### 为什么要用 Docker

作为一种新兴的虚拟化方式，Docker 跟传统的虚拟化方式相比具有众多的优势。

**更高效的利用系统资源**

由于容器不需要进行硬件虚拟以及运行完整操作系统等额外开销，Docker 对系统资源的利用率更高。无论是应用执行速度、内存损耗或者文件存储速度，都要比传统虚拟机技术更高效。因此，相比虚拟机技术，一个相同配置的主机，往往可以运行更多数量的应用。

**更快速的启动时间**

传统的虚拟机技术启动应用服务往往需要数分钟，而 Docker 容器应用，由于直接运行于宿主内核，无需启动完整的操作系统，因此可以做到秒级、甚至毫秒级的启动时间。大大的节约了开发、测试、部署的时间。

**一致的运行环境**

开发过程中一个常见的问题是环境一致性问题。由于开发环境、测试环境、生产环境不一致，导致有些 bug 并未在开发过程中被发现。而 Docker 的镜像提供了除内核外完整的运行时环境，确保了应用运行环境一致性，从而不会再出现 「这段代码在我机器上没问题啊」 这类问题。

**持续交付和部署**

对开发和运维（）人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。

使用 Docker 可以通过定制应用镜像来实现持续集成、持续交付、部署。开发人员可以通过  来进行镜像构建，并结合  系统进行集成测试，而运维人员则可以直接在生产环境中快速部署该镜像，甚至结合  系统进行自动部署。

而且使用  使镜像构建透明化，不仅仅开发团队可以理解应用运行环境，也方便运维团队理解应用运行所需条件，帮助更好的生产环境中部署该镜像。

**更轻松的迁移**

由于 Docker 确保了执行环境的一致性，使得应用的迁移更加容易。Docker 可以在很多平台上运行，无论是物理机、虚拟机、公有云、私有云，甚至是笔记本，其运行结果是一致的。因此用户可以很轻易的将在一个平台上运行的应用，迁移到另一个平台上，而不用担心运行环境的变化导致应用无法正常运行的情况。

**更轻松的维护和扩展**

Docker 使用的分层存储以及镜像的技术，使得应用重复部分的复用更为容易，也使得应用的维护更新更加简单，基于基础镜像进一步扩展镜像也变得非常简单。此外，Docker 团队同各个开源项目团队一起维护了一大批高质量的 ，既可以直接在生产环境使用，又可以作为基础进一步定制，大大的降低了应用服务的镜像制作成本。

  

### docker一键安装

  

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

  

### Docker命令实战

#### 常用命令

  

  

### 基础实战

#### 1、镜像

  

##### 下载最新版镜像

```
root@hello:~# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
7d63c13d9b9b: Pull complete 
5cb019b641b5: Pull complete 
d477de77abf8: Pull complete 
c60e7d4c1c30: Pull complete 
365a49996569: Pull complete 
039c6e901970: Pull complete 
Digest: sha256:168a6a2be5c65d4aafa7a78ca98ff8b110fe44c6ca41e7ccb4314ed481e32288
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

  

##### 查看本地镜像

  

```
root@hello:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx       latest   e9ce56a96f8e   8 hours ago   141MB
root@hello:~
```

##### 删除镜像

```
root@hello:~# docker images

REPOSITORY  TAG    IMAGE ID    CREATED    SIZE

nginx     latest   e9ce56a96f8e  8 hours ago  141MB

root@hello:~# 

root@hello:~# docker rmi e9ce56a96f8e

Untagged: nginx:latest

Untagged: nginx@sha256:168a6a2be5c65d4aafa7a78ca98ff8b110fe44c6ca41e7ccb4314ed481e32288

Deleted: sha256:e9ce56a96f8e0e9f75051f258a595d1257bd6bb91913b79455ea77e67e686c5c

Deleted: sha256:6e5a463ea9608e4712465e1c575b2932dde96f99fa2c2fc31a5bacbe69c725cb

Deleted: sha256:a12cc243b903b34c8137e57160d206d6c1ee76a1ab6011a1cebdceb8b6ff8768

Deleted: sha256:a562e4589c72b0706526e13eed9c4f037ab5d1f50eb4529b38670abe353248f2

Deleted: sha256:fd67efaafabe1a0b146e9f7d958de79ec8fcec9aa6ee13ca3052b4acd8a3b81a

Deleted: sha256:c3967df88e47f739c3048492985aafaafecd5806de6c6870cbd76997fc0c68b0

Deleted: sha256:e8b689711f21f9301c40bf2131ce1a1905c3aa09def1de5ec43cf0adf652576e

root@hello:~# 

root@hello:~# docker images

REPOSITORY  TAG    IMAGE ID  CREATED  SIZE

root@hello:~#
```

  

##### 下载指定版本镜像

```
root@hello:~# docker pull nginx:1.20.1
1.20.1: Pulling from library/nginx
b380bbd43752: Pull complete 
83acae5e2daa: Pull complete 
33715b419f9b: Pull complete 
eb08b4d557d8: Pull complete 
74d5bdecd955: Pull complete 
0820d7f25141: Pull complete 
Digest: sha256:a98c2360dcfe44e9987ed09d59421bb654cb6c4abe50a92ec9c912f252461483
Status: Downloaded newer image for nginx:1.20.1
docker.io/library/nginx:1.20.1
root@hello:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        1.20.1    c8d03f6b8b91   5 weeks ago   133MB
root@hello:~#
```

  

#### 2、容器

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

【docker run  设置项  镜像名  】 镜像启动运行的命令（镜像里面默认有的，一般不会写）

# -d：后台运行

# --restart=always: 开机自启

# -p 主机端口：容器端口

root@hello:~# docker run --name=myningx -d --restart=always -p 88:80 nginx 
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
7d63c13d9b9b: Pull complete 
5cb019b641b5: Pull complete 
d477de77abf8: Pull complete 
c60e7d4c1c30: Pull complete 
365a49996569: Pull complete 
039c6e901970: Pull complete 
Digest: sha256:168a6a2be5c65d4aafa7a78ca98ff8b110fe44c6ca41e7ccb4314ed481e32288
Status: Downloaded newer image for nginx:latest
15db0ba492cf2b86714e3e29723d413b97e64cc2ee361d4109f4216b2e0cba60
root@hello:~# 
root@hello:~# curl -I 127.0.0.1:88
HTTP/1.1 200 OK
Server: nginx/1.21.4
Date: Wed, 17 Nov 2021 02:03:13 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 02 Nov 2021 14:49:22 GMT
Connection: keep-alive
ETag: "61814ff2-267"
Accept-Ranges: bytes

root@hello:~#
```

  

##### 查看当前运行的容器

```
root@hello:~# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                               NAMES
15db0ba492cf   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:88->80/tcp, :::88->80/tcp   myningx
root@hello:~#
```

  

##### 停止容器

```
root@hello:~# docker stop 15db0ba492cf
15db0ba492cf
root@hello:~# 
root@hello:~# docker  ps 
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@hello:~#
```

  

##### 查看所有容器

```
root@hello:~# docker  ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                      PORTS     NAMES
15db0ba492cf   nginx     "/docker-entrypoint.…"   2 minutes ago   Exited (0) 12 seconds ago             myningx
root@hello:~#
```

  

##### 启动容器

```
root@hello:~# docker start 15db0ba492cf
15db0ba492cf
root@hello:~# docker  ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                               NAMES
15db0ba492cf   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 3 seconds   0.0.0.0:88->80/tcp, :::88->80/tcp   myningx
root@hello:~#
```

  

##### 删除容器，在运行中无法删除

```
root@hello:~# docker rm 15db0ba492cf
Error response from daemon: You cannot remove a running container 15db0ba492cf2b86714e3e29723d413b97e64cc2ee361d4109f4216b2e0cba60. Stop the container before attempting removal or force remove
```

  

##### 强制删除容器

```
root@hello:~# docker rm -f 15db0ba492cf
15db0ba492cf
root@hello:~#
```

  

#### 3、进入容器操作容器

  

```
root@hello:~# docker exec -it b1d72657b /bin/bash
root@b1d72657b272:/# 
root@b1d72657b272:/#
```

  

#### 4、修改容器内容

```
root@hello:~# docker exec -it b1d72657b /bin/bash
root@b1d72657b272:/# echo "123" > /usr/share/nginx/html/index.html 
root@b1d72657b272:/# 
root@b1d72657b272:/# echo "cby" > /usr/share/nginx/html/index.html 

root@hello:~# curl 127.0.0.1:88
123
root@hello:~# 

root@hello:~# docker exec -it b1d72657b /bin/bash
root@b1d72657b272:/# echo "cby" > /usr/share/nginx/html/index.html 

root@hello:~# curl 127.0.0.1:88
cby
root@hello:~#
```

  

#### 5、挂载外部数据

```
root@hello:~# docker run --name=myningx -d --restart=always -p 88:80 -v /data/html:/usr/share/nginx/html/ nginx  

e3788cdd7be695fe9a1bebd7306c131d6380da215a416d19c162c609b8f108ae

root@hello:~# 

root@hello:~# 

root@hello:~# curl 127.0.0.1:88

<html>

<head><title>403 Forbidden</title></head>

<body>

<center><h1>403 Forbidden</h1></center>

<hr><center>nginx/1.21.4</center>

</body>

</html>

root@hello:~# 

root@hello:~# echo "cby" > /data/html/index.html

root@hello:~# 

root@hello:~# 

root@hello:~# curl 127.0.0.1:88

cby

root@hello:~#
```

  

#### 6、将运行中的容器构建为镜像

```
root@hello:~# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                               NAMES
e3788cdd7be6   nginx     "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   0.0.0.0:88->80/tcp, :::88->80/tcp   myningx
root@hello:~# 
root@hello:~# docker commit -a "cby" -m "my app" e3788cdd7be6 myapp:v1.0
sha256:07a7b54c914c79dfbd402029a3d144201235eca72a4f26c92e2ec7780c485226
root@hello:~# 
root@hello:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
myapp        v1.0      07a7b54c914c   4 seconds ago   141MB
nginx        latest    e9ce56a96f8e   8 hours ago     141MB
root@hello:~#
```

  

#### 7、镜像保存与导入

```
root@hello:~# docker save -o cby.tar myapp:v1.0
root@hello:~# ll cby.tar 
-rw------- 1 root root 145910784 Nov 17 02:21 cby.tar
root@hello:~# 
root@hello:~# docker load -i cby.tar 
Loaded image: myapp:v1.0
root@hello:~# 
root@hello:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
myapp        v1.0      07a7b54c914c   3 minutes ago   141MB
nginx        latest    e9ce56a96f8e   8 hours ago     141MB
nginx        1.20.1    c8d03f6b8b91   5 weeks ago     133MB
root@hello:~#
```

  

#### 8、推送到DockerHub，并在其他主机上可拉去该镜像

```
root@hello:~# docker tag myapp:v1.0 chenbuyun/myapp:v1.0
root@hello:~# 
root@hello:~# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: chenbuyun
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
root@hello:~# docker push chenbuyun/myapp:v1.0
The push refers to repository [docker.io/chenbuyun/myapp]
799aefeaf6b1: Pushed 
fd688ba2259e: Mounted from library/nginx 
c731fe3d8126: Mounted from library/nginx 
3b1690d8cd86: Mounted from library/nginx 
03f105433dc8: Mounted from library/nginx 
bd7b2912e0ab: Mounted from library/nginx 
e8b689711f21: Mounted from library/nginx 
v1.0: digest: sha256:f085a533e36cccd27a21fe4de7c87f652fe9346e1ed86e3d82856d5d4434c0a0 size: 1777
root@hello:~# 
root@hello:~# docker logout
Removing login credentials for https://index.docker.io/v1/
root@hello:~# 
root@hello:~# docker pull chenbuyun/myapp:v1.0 
v1.0: Pulling from chenbuyun/myapp
Digest: sha256:f085a533e36cccd27a21fe4de7c87f652fe9346e1ed86e3d82856d5d4434c0a0
Status: Downloaded newer image for chenbuyun/myapp:v1.0
docker.io/chenbuyun/myapp:v1.0
root@hello:~# 
root@hello:~# docker images
REPOSITORY        TAG       IMAGE ID       CREATED         SIZE
chenbuyun/myapp   v1.0      07a7b54c914c   9 minutes ago   141MB
myapp             v1.0      07a7b54c914c   9 minutes ago   141MB
nginx             latest    e9ce56a96f8e   8 hours ago     141MB
nginx             1.20.1    c8d03f6b8b91   5 weeks ago     133MB
root@hello:~#
```

  

以上仅为常用命令，更多docker相关知识可在：https://www.runoob.com/docker/docker-tutorial.html

> **关于**
>
> https://www.oiox.cn/
>
> https://www.oiox.cn/index.php/start-page.html
>
> **CSDN、GitHub、51CTO、知乎、开源中国、思否、掘金、简书、华为云、阿里云、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客**
>
> **全网可搜《小陈运维》**
>
> **文章主要发布于微信公众号**