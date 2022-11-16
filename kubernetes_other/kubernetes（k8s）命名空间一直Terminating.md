---
layout: post
cid: 132
title: kubernetes（k8s）命名空间一直Terminating
slug: 132
date: 2022/04/01 14:14:19
updated: 2022/04/01 14:15:23
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


  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/874e30765f58470b92478a60b674a941~tplv-k3u1fbpfcp-zoom-1.image)

  

  

```
root@hello:~# kubectl  get ns
NAME              STATUS        AGE
auth              Terminating   34m
default           Active        23h
kube-node-lease   Active        23h
kube-public       Active        23h
kube-system       Active        23h
```

  

新开命令行窗口打开proxy

```
root@hello:~# kubectl proxy
Starting to serve on 127.0.0.1:8001
```

  

回到刚才窗口 将 terminating 状态的命名空间信息导出到 json 文件：

```
root@hello:~# kubectl get namespace auth -o json >tmp.json
```

  

修改json文件中的 finalizers，将其设置为空

```
root@hello:~# vi tmp.json 
root@hello:~# cat tmp.json | grep finalizers
        "finalizers": []
```

  

在 temp.json 文件所在位置调下面的接口

```
root@hello:~# curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://127.0.0.1:8001/api/v1/namespaces/auth/finalize
```

\*auth 改为需要删除的 terminating 状态的命名空间的名字

  

  

验证

```
root@hello:~# kubectl  get ns
NAME              STATUS        AGE
default           Active        23h
kube-node-lease   Active        23h
kube-public       Active        23h
kube-system       Active        23h
root@hello:~#
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