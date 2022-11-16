---
layout: post
cid: 138
title: docker方式实现postgres数据持久化离线安装
slug: 138
date: 2022/04/12 14:46:07
updated: 2022/04/12 14:48:01
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


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52545252ef3c4d689a27881538efc483~tplv-k3u1fbpfcp-zoom-1.image)

保存镜像
====

```
root@hello:~# docker pull postgres
Using default tag: latest
latest: Pulling from library/postgres
a2abf6c4d29d: Already exists 
e1769f49f910: Pull complete 
33a59cfee47c: Pull complete 
461b2090c345: Pull complete 
8ed8ab6290ac: Pull complete 
495e42c822a0: Pull complete 
18e858c71c58: Pull complete 
594792c80d5f: Pull complete 
794976979956: Pull complete 
eb5e1a73c3ca: Pull complete 
6d6360292cba: Pull complete 
131e916e1a28: Pull complete 
757a73507e2e: Pull complete 
Digest: sha256:f329d076a8806c0ce014ce5e554ca70f4ae9407a16bb03baa7fef287ee6371f1
Status: Downloaded newer image for postgres:latest
docker.io/library/postgres:latest
root@hello:~# 
root@hello:~# docker save > postgres.tar postgres:latest 
root@hello:~# ll postgres.tar
-rw-r--r-- 1 root root 381950976 Mar 30 08:04 postgres.tar
root@hello:~#

```

导入镜像
====

```
root@hello:~# docker load -i postgres.tar 
7ab4f6ae3ff7: Loading layer [==================================================>]  10.18MB/10.18MB
db8b35906c8d: Loading layer [==================================================>]    340kB/340kB
f9f2c722c092: Loading layer [==================================================>]   4.19MB/4.19MB
75be6af37d28: Loading layer [==================================================>]   25.7MB/25.7MB
15dd9dd29d12: Loading layer [==================================================>]  1.682MB/1.682MB
1d5d2439ed88: Loading layer [==================================================>]  2.048kB/2.048kB
920ba1e03a88: Loading layer [==================================================>]  6.656kB/6.656kB
eb96dca5c689: Loading layer [==================================================>]  255.8MB/255.8MB
3acb2bfab7b0: Loading layer [==================================================>]  66.56kB/66.56kB
140aef27609a: Loading layer [==================================================>]  2.048kB/2.048kB
c06253083edb: Loading layer [==================================================>]  3.584kB/3.584kB
e7b07b473569: Loading layer [==================================================>]  15.36kB/15.36kB
Loaded image: postgres:latest

```

启动容器
====

```
root@hello:~# mkdir /data/postgres -p

root@hello:~# docker run --name postgres -e POSTGRES_PASSWORD=thinker -p 5432:5432 -v /data/postgres:/var/lib/postgresql/data -d postgres
ae30b561a607210d4cbb42f5cc344898341124feeb1a2e5fe68031ec1a46b5b4

root@hello:~# docker ps | grep postgres 
ae30b561a607   postgres             "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp                       postgres

```

访问测试
====

```
root@hello:~# docker exec -it ae30b561a607 bash
root@ae30b561a607:/# su postgres
postgres@ae30b561a607:/$ psql
psql (14.1 (Debian 14.1-1.pgdg110+1))
Type "help" for help.

postgres-# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)

postgres-#

```

  

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
