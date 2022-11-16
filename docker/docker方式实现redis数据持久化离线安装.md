---
layout: post
cid: 135
title: docker方式实现redis数据持久化离线安装
slug: 135
date: 2022/04/06 10:35:34
updated: 2022/04/06 10:36:45
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


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fb9968de62f47f18d2e931adf7d32e2~tplv-k3u1fbpfcp-zoom-1.image)

  

保存镜像  

  

```
root@hello:~# docker pull redis:latest
latest: Pulling from library/redis
a2abf6c4d29d: Already exists 
c7a4e4382001: Pull complete 
4044b9ba67c9: Pull complete 
c8388a79482f: Pull complete 
413c8bb60be2: Pull complete 
1abfd3011519: Pull complete 
Digest: sha256:db485f2e245b5b3329fdc7eff4eb00f913e09d8feb9ca720788059fdc2ed8339
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
root@hello:~# 
root@hello:~# docker save > redis.tar redis:latest 
root@hello:~# 
root@hello:~# ll redis.tar 
-rw-r--r-- 1 root root 116304384 Mar 30 07:30 redis.tar
root@hello:~#
```

  

导入镜像

```
root@hello:~# docker load -i redis.tar 
2edcec3590a4: Loading layer [==================================================>]  83.86MB/83.86MB
9b24afeb7c2f: Loading layer [==================================================>]  338.4kB/338.4kB
4b8e2801e0f9: Loading layer [==================================================>]  4.274MB/4.274MB
529cdb636f61: Loading layer [==================================================>]   27.8MB/27.8MB
9975392591f2: Loading layer [==================================================>]  2.048kB/2.048kB
8e5669d83291: Loading layer [==================================================>]  3.584kB/3.584kB
Loaded image: redis:latest
```

  

创建目录，修改配置

```
root@hello:~# mkdir /data/redis -p
root@hello:~# mkdir /data/redis/data -p


root@hello:~# cp -p redis.conf /data/redis/
root@hello:~# vim /data/redis/redis.conf
```

  

修改redis.conf配置文件：

主要配置的如下：

  

 bind 127.0.0.1 #注释掉这部分，使redis可以外部访问

 daemonize no#用守护线程的方式启动

 requirepass thinekr #给redis设置密码

 appendonly yes #redis持久化　　默认是no

 tcp-keepalive 300 #防止出现远程主机强迫关闭了一个现有的连接的错误 默认是300

  

 启动容器

```
 root@hello:~# docker run -p 6379:6379 --name redis -v /data/redis/redis.conf:/etc/redis/redis.conf  -v /data/redis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
a59d3137a3ade6ec05588e0895d2265aff0e81746ec1847553fef6bd4df59348
root@hello:~#
```

  

测试

```
root@hello:~# sudo docker logs redis
1:C 30 Mar 2022 07:36:59.712 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 30 Mar 2022 07:36:59.712 # Redis version=6.2.6, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 30 Mar 2022 07:36:59.712 # Configuration loaded
1:M 30 Mar 2022 07:36:59.713 * monotonic clock: POSIX clock_gettime
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 6.2.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                  
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           https://redis.io       
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               


1:M 30 Mar 2022 07:36:59.714 # Server initialized
1:M 30 Mar 2022 07:36:59.714 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 30 Mar 2022 07:36:59.715 * Ready to accept connections
root@hello:~#




root@hello:~# redis-cli -h 3.7.191.194 -p 6379
3.7.191.194:6379> auth thinker
OK
3.7.191.194:6379> ping
PONG
3.7.191.194:6379>
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