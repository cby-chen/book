---
layout: post
cid: 149
title: 使用Kubernetes快速启用一个静态页面
slug: 149
date: 2022/04/28 09:39:00
updated: 2022/04/29 15:47:06
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


使用Kubernetes快速启用一个静态页面
======================

将html静态页面放置在nfs目录下，通过Deployment启动时挂在到nginx页面目录即可

查看yaml内容
========

```
root@hello:~# cat cby.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chenby
spec:
  replicas: 3
  selector:
    matchLabels:
      app: chenby
  template:
    metadata:
      labels:
        app: chenby
    spec:
      containers:
      - name: chenby
        image: nginx
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
        volumeMounts:
        - name: cby-nfs
          mountPath: /usr/share/nginx/html/
      volumes:
      - name: cby-nfs
        nfs:
          server: 192.168.1.123
          path: /cby-3/nfs/html

---
apiVersion: v1
kind: Service
metadata:
  name: chenby
spec:
  type: NodePort
  selector:
    app: chenby
  ports:
  - port: 80
    targetPort: 80

```

查看验证
====

```
root@hello:~# kubectl  get deployments.apps  chenby  -o wide
NAME     READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES   SELECTOR
chenby   3/3     3            3           4m44s   chenby       nginx    app=chenby
root@hello:~# 


root@hello:~# kubectl  get pod -o wide | grep chenby
chenby-77b57649c7-qv2ps                  1/1     Running   0          5m2s   172.17.125.19    k8s-node01     <none>           <none>
chenby-77b57649c7-rx98c                  1/1     Running   0          5m2s   172.25.214.207   k8s-node03     <none>           <none>
chenby-77b57649c7-tx2dz                  1/1     Running   0          5m2s   172.25.244.209   k8s-master01   <none>           <none>



root@hello:~# kubectl  get svc -o wide | grep chenby
chenby                NodePort    10.109.222.0    <none>        80:30971/TCP   5m8s   app=chenby
root@hello:~# 

```

演示
==

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05b307afd22544ef86046268b812fae4~tplv-k3u1fbpfcp-zoom-1.image)

在线体验：  

https://www.oiox.cn/

https://www.chenby.cn/

https://blog.oiox.cn/

  

> https://www.oiox.cn/
> 
> https://www.chenby.cn/
> 
> https://blog.oiox.cn/
> 
> https://cby-chen.github.io/
> 
> https://blog.csdn.net/qq_33921750
> 
> https://my.oschina.net/u/3981543
> 
> https://www.zhihu.com/people/chen-bu-yun-2
> 
> https://segmentfault.com/u/hppyvyv6/articles
> 
> https://juejin.cn/user/3315782802482007
> 
> https://cloud.tencent.com/developer/column/93230
> 
> https://www.jianshu.com/u/0f894314ae2c
> 
> https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/
> 
> CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、今日头条、个人博客、全网可搜《小陈运维》
> 
> 文章主要发布于微信：《Linux运维交流社区》
