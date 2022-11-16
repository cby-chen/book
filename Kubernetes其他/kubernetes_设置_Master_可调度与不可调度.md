---
layout: post
cid: 244
title: kubernetes 设置 Master 可调度与不可调度
slug: 244
date: 2022/06/10 09:26:29
updated: 2022/06/10 09:28:35
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: kubernetes 设置 Master 可调度与不可调度
keywords: 小陈运维,小陈,小陈博客,文档,
mode: default
thumb: 
video: 
---




# kubernetes 设置 Master 可调度与不可调度



## 语法

kubectl taint node [node] key=value[effect]

[effect] 可取值: [ NoSchedule | PreferNoSchedule | NoExecute ]

NoSchedule: 一定不能被调度

PreferNoSchedule: 尽量不要调度

NoExecute: 不仅不会调度, 还会驱逐Node上已有的Pod



## 取消污点

```
取消污点
[root@k8s-master01 ~]# kubectl taint node k8s-master node-role.kubernetes.io/master-
```



## 设置污点

```
# 设置为一定不能被调度

[root@k8s-master01 ~]# kubectl taint node k8s-master01 node-role.kubernetes.io/master="":NoSchedule
node/k8s-master01 tainted
[root@k8s-master01 ~]# kubectl taint node k8s-master02 node-role.kubernetes.io/master="":NoSchedule
node/k8s-master02 tainted
[root@k8s-master01 ~]# kubectl taint node k8s-master03 node-role.kubernetes.io/master="":NoSchedule
node/k8s-master03 tainted
[root@k8s-master01 ~]# 

# 查看污点
[root@k8s-master01 ~]# kubectl  describe  node | grep Ta
Taints:             node-role.kubernetes.io/master:NoSchedule
Taints:             node-role.kubernetes.io/master:NoSchedule
Taints:             node-role.kubernetes.io/master:NoSchedule
Taints:             <none>
Taints:             <none>
Taints:             <none>
Taints:             <none>
Taints:             <none>
[root@k8s-master01 ~]# 
```



## 查看验证

```
# 查看已经调度到maser上的pod没有被驱逐
[root@k8s-master01 ~]# kubectl  get pod -o wide
NAMESPACE              NAME                                         READY   STATUS    RESTARTS       AGE   IP               NODE           NOMINATED NODE   READINESS GATES
default                hostname-test-cby-58d85dccdb-7zgjj           1/1     Running   1 (2d1h ago)   19d   172.25.244.195   k8s-master01   <none>           <none>
default                hostname-test-cby-58d85dccdb-8t7zv           1/1     Running   1 (2d1h ago)   19d   172.25.244.196   k8s-master01   <none>           <none>
default                hostname-test-cby-58d85dccdb-9bqsq           1/1     Running   1 (2d1h ago)   19d   172.25.92.74     k8s-master02   <none>           <none>
default                hostname-test-cby-58d85dccdb-jj2ml           1/1     Running   1 (2d1h ago)   19d   172.17.125.3     k8s-node01     <none>           <none>
default                hostname-test-cby-58d85dccdb-k96zl           1/1     Running   1 (2d1h ago)   19d   172.18.195.3     k8s-master03   <none>           <none>
default                hostname-test-cby-58d85dccdb-lng8b           1/1     Running   1 (2d1h ago)   19d   172.29.115.131   k8s-node04     <none>           <none>
default                hostname-test-cby-58d85dccdb-lsrbg           1/1     Running   1 (2d1h ago)   19d   172.25.214.195   k8s-node03     <none>           <none>
default                hostname-test-cby-58d85dccdb-mlv24           1/1     Running   1 (2d1h ago)   19d   172.17.54.131    k8s-node05     <none>           <none>
default                hostname-test-cby-58d85dccdb-p5vc8           1/1     Running   1 (2d1h ago)   19d   172.27.14.195    k8s-node02     <none>           <none>
default                hostname-test-cby-58d85dccdb-z6ptf           1/1     Running   1 (2d1h ago)   19d   172.25.214.196   k8s-node03   
  <none>           <none>
[root@k8s-master01 ~]# 
```



## 设置污点

```
# 设置为不仅不会调度, 还会驱逐Node上已有的Pod
[root@k8s-master01 ~]# kubectl taint node k8s-master03 node-role.kubernetes.io/master="":NoExecute
node/k8s-master03 tainted
[root@k8s-master01 ~]# kubectl taint node k8s-master02 node-role.kubernetes.io/master="":NoExecute
node/k8s-master02 tainted
[root@k8s-master01 ~]# kubectl taint node k8s-master01 node-role.kubernetes.io/master="":NoExecute
node/k8s-master01 tainted

# 查看污点
[root@k8s-master01 ~]# kubectl  describe  node | grep Ta
Taints:             node-role.kubernetes.io/master:NoExecute
Taints:             node-role.kubernetes.io/master:NoExecute
Taints:             node-role.kubernetes.io/master:NoExecute
Taints:             <none>
Taints:             <none>
Taints:             <none>
Taints:             <none>
Taints:             <none>
[root@k8s-master01 ~]# 
```



## 查看验证

```
# 查看已经调度到master节点的pod已进行驱逐
[root@k8s-master01 ~]# kubectl  get pod  -A -o wide
NAMESPACE              NAME                                         READY   STATUS              RESTARTS       AGE   IP               NODE           NOMINATED NODE   READINESS GATES
default                mysql-0                                      2/2     Running             0              34m   172.27.14.206    k8s-node02     <none>           <none>
default                mysql-1                                      2/2     Running             0              34m   172.17.125.11    k8s-node01     <none>           <none>
default                mysql-2                                      2/2     Terminating         0              34m   172.18.195.10    k8s-master03   <none>           <none>
[root@k8s-master01 ~]# 
```



> https://www.oiox.cn/
> https://www.chenby.cn/
> https://cby-chen.github.io/
> https://blog.csdn.net/qq_33921750
> https://my.oschina.net/u/3981543
> https://www.zhihu.com/people/chen-bu-yun-2
> https://segmentfault.com/u/hppyvyv6/articles
> https://juejin.cn/user/3315782802482007
> https://cloud.tencent.com/developer/column/93230
> https://www.jianshu.com/u/0f894314ae2c
> https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/
>
> CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、今日头条、个人博客、全网可搜《小陈运维》
>
> 文章主要发布于微信：《Linux运维交流社区》