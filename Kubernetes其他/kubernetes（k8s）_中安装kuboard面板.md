---
layout: post
cid: 68
title: kubernetes（k8s） 中安装kuboard面板
slug: 68
date: 2021/12/30 17:15:06
updated: 2021/12/30 17:15:06
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


  

kubernetes（k8s） 中安装kuboard面板

  

  

01

—

  

背景及安装

  

Kuboard 是一款专为 Kubernetes 设计的免费管理界面，兼容 Kubernetes 版本 1.13 及以上。Kuboard 每周发布一个 beta 版本，最长每月发布一个正式版本，经过两年的不断迭代和优化，已经具备多集群管理、权限管理、监控套件、日志套件等丰富的功能。

  

  

删除之前的版本

  

```
docker stop $(docker ps -a | grep "eipwork/kuboard" | awk '{print $1 }')
docker rm $(docker ps -a | grep "eipwork/kuboard" | awk '{print $1 }')
```

  

安装最新版

  

```
  sudo docker run -d \
  --restart=unless-stopped \
  --name=kuboard \
  -p 80:80/tcp \
  -p 10081:10081/udp \
  -p 10081:10081/tcp \
  -e KUBOARD_ENDPOINT="http://192.168.1.12:80" \
  -e KUBOARD_AGENT_SERVER_UDP_PORT="10081" \
  -e KUBOARD_AGENT_SERVER_TCP_PORT="10081" \
  -v /root/kuboard-data:/data \
  eipwork/kuboard:v3.3.0.3
```

  

  

在浏览器输入 http://192.168.1.12 即可访问 Kuboard 的界面，登录方式：

  

用户名：admin

密 码：Kuboard123

  

**注：可以在 https://hub.docker.com/r/eipwork/kuboard/tags 中查看最新版本号**

  

  

02

—  

  

访问登录

  

登录：

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e495a4efb9249dc945f699b42ad43d8~tplv-k3u1fbpfcp-zoom-1.image)

  

添加集群，使用Agent方式添加

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8be9ff7010d4ea6b1045125f2ef95a3~tplv-k3u1fbpfcp-zoom-1.image)

  

  

  

获得命令

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/806a45c3f4614b269c283da374d171d2~tplv-k3u1fbpfcp-zoom-1.image)

  

  

在集群master上执行命令

  

```
[root@hello ~]# curl -k 'http://192.168.1.12:80/kuboard-api/cluster/cby/kind/KubernetesCluster/cby/resource/installAgentToKubernetes?token=zyrsQqY6Krsy3gvWUNHK2kvKWHmJZneL' > kuboard-agent.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  5775    0  5775    0     0   433k      0 --:--:-- --:--:-- --:--:--  433k
[root@hello ~]# 
[root@hello ~]# kubectl apply -f ./kuboard-agent.yaml
namespace/kuboard unchanged
serviceaccount/kuboard-admin created
clusterrolebinding.rbac.authorization.k8s.io/kuboard-admin-crb created
serviceaccount/kuboard-viewer created
clusterrolebinding.rbac.authorization.k8s.io/kuboard-viewer-crb created
deployment.apps/kuboard-agent-soxwal created
deployment.apps/kuboard-agent-soxwal-2 created
[root@hello ~]#
```

  

  

稍作等待即可在首页看到

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae5a340f4c484f1cae1cdcff08d48ef3~tplv-k3u1fbpfcp-zoom-1.image)

  
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