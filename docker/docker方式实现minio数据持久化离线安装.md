---
layout: post
cid: 137
title: docker方式实现minio数据持久化离线安装
slug: 137
date: 2022/04/08 14:21:02
updated: 2022/04/08 14:21:02
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


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1942175b9f7c4b2285f931f31896ba71~tplv-k3u1fbpfcp-zoom-1.image)

  

保存镜像

```
root@hello:~# docker pull minio/minio
Using default tag: latest
latest: Pulling from minio/minio
d46336f50433: Pull complete 
be961ec68663: Pull complete 
44173c602141: Pull complete 
a9809a6a679b: Pull complete 
df29d4a76971: Pull complete 
2b5a8853d302: Pull complete 
84f01ee8dfc1: Pull complete 
Digest: sha256:d786220feef7d8fe0239d41b5d74501dc824f6e7dd0e5a05749c502fff225bf3
Status: Downloaded newer image for minio/minio:latest
docker.io/minio/minio:latest
root@hello:~#
root@hello:~# docker save > minio.tar minio/minio
root@hello:~# ll minio.tar
-rw-r--r-- 1 root root 415240704 Mar 30 07:03 minio.tar
root@hello:~#
```

  

导入镜像

```
root@hello:~# docker load -i minio.tar 
744c86b54390: Loading layer [==================================================>]  104.1MB/104.1MB
1323ffbff4dd: Loading layer [==================================================>]  20.48kB/20.48kB
9a5123a464dc: Loading layer [==================================================>]  3.584kB/3.584kB
9e9eecfbe95d: Loading layer [==================================================>]  3.584kB/3.584kB
6088fcbd6a76: Loading layer [==================================================>]  1.724MB/1.724MB
678ce496e457: Loading layer [==================================================>]  36.86kB/36.86kB
50f383b04a07: Loading layer [==================================================>]  309.3MB/309.3MB
Loaded image: minio/minio:latest
root@hello:~#
```

  

创建目录

```
root@hello:~# mkdir /data/config -p
root@hello:~# mkdir /data/data -p
root@hello:~#
```

  

启动容器

```
root@hello:~# docker run -itd -p 9000:9000 --name minio -p 9001:9001 -e "MINIO_ACCESS_KEY=minio" -e "MINIO_SECRET_KEY=minio@123" -v /data/data:/data -v /data/config:/root/.minio minio/minio server /data --address '0.0.0.0:9000'  --console-address '0.0.0.0:9001'
5c69e875ce561ac311a85708594072eca8c1b4740773d83045f256d316efc06c
root@hello:~# docker ps | grep minio
5c69e875ce56   minio/minio             "/usr/bin/docker-ent…"   9 seconds ago   Up 8 seconds   0.0.0.0:9000-9001->9000-9001/tcp, :::9000-9001->9000-9001/tcp   minio
root@hello:~#
```

  

访问账号密码

url：http://ip:9001/login

user：minio

password：minio@123

  

  

  

  

  

  

https://www.oiox.cn/

https://www.chenby.cn/

https://cby-chen.github.io/

https://weibo.com/u/5982474121

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

https://www.jianshu.com/u/0f894314ae2c

https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/

CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客、全网可搜《小陈运维》
