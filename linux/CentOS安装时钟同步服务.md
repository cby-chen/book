---
layout: post
cid: 89
title: CentOS安装时钟同步服务
slug: 89
date: 2022/02/15 12:37:00
updated: 2022/03/25 15:37:17
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


  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc6cf2866ac64e0698e289c141aa34a7~tplv-k3u1fbpfcp-zoom-1.image)

  

  

使用chrony用于时间同步

  

```
yum install chrony -y

vim /etc/chrony.conf

cat /etc/chrony.conf | grep -v  "^#" | grep -v "^$"
pool ntp.aliyun.com iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
allow 10.0.0.0/8
local stratum 10
keyfile /etc/chrony.keys
leapsectz right/UTC
logdir /var/log/chrony


systemctl restart chronyd
systemctl enable chronyd
```

  

客户端安装并配置

  

```
yum install chrony -y

vim /etc/chrony.conf

cat /etc/chrony.conf | grep -v  "^#" | grep -v "^$"
pool 10.0.0.20 iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
keyfile /etc/chrony.keys
leapsectz right/UTC
logdir /var/log/chrony

systemctl restart chronyd ; systemctl enable chronyd
```

  

使用客户端进行验证

  

```
chronyc sources -v


  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* master01                      3   6    17     7  -2539ns[  -29us] +/-   16ms


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