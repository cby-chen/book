---
layout: post
cid: 203
title: kubernetes （k8s） v1.24.0 安装dashboard面板
slug: 203
date: 2022/05/17 12:30:00
updated: 2022/05/17 16:53:50
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


kubernetes （k8s） v1.24.0 安装dashboard面板
======================================

介绍
==

v1.24.0 使用之前的安装方式，在安装过程中会有一些异常，此文档已修复已知问题。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7147040ee6a44e2289da8e961a380509~tplv-k3u1fbpfcp-zoom-1.image)

下载所需配置
======

```
root@k8s-master01:~# wget https://raw.githubusercontent.com/cby-chen/Kubernetes/main/yaml/dashboard.yaml
root@k8s-master01:~# 

root@k8s-master01:~# kubectl apply -f dashboard.yaml
namespace/kubernetes-dashboard unchanged
serviceaccount/kubernetes-dashboard unchanged
service/kubernetes-dashboard configured
secret/kubernetes-dashboard-certs unchanged
secret/kubernetes-dashboard-csrf configured
Warning: resource secrets/kubernetes-dashboard-key-holder is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
secret/kubernetes-dashboard-key-holder configured
configmap/kubernetes-dashboard-settings unchanged
role.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
deployment.apps/kubernetes-dashboard configured
service/dashboard-metrics-scraper unchanged
deployment.apps/dashboard-metrics-scraper unchanged


root@k8s-master01:~# wget https://raw.githubusercontent.com/cby-chen/Kubernetes/main/yaml/dashboard-user.yaml


root@k8s-master01:~# kubectl  apply -f dashboard-user.yaml
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
root@k8s-master01:~# 

```

修改为nodePort
===========

```
root@k8s-master01:~# kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
service/kubernetes-dashboard edited


root@k8s-master01:~# kubectl get svc kubernetes-dashboard -n kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.96.221.8   <none>        443:32721/TCP   74s
root@k8s-master01:~#

```

创建token
=======

```
root@k8s-master01:~# kubectl -n kubernetes-dashboard create token admin-user
eyJhbGciOiJSUzI1NiIsImtpZCI6IlV6b3NRbDRiTll4VEl1a1VGbU53M2Y2X044Wjdfa21mQ0dfYk5BWktHRjAifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjUyNzYzMjUzLCJpYXQiOjE2NTI3NTk2NTMsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiNDYxYjc4MDItNTgzMS00MTNmLTg2M2ItODdlZWVkOTI3MTdiIn19LCJuYmYiOjE2NTI3NTk2NTMsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.nFF729zlDxz4Ed3fcVk5BE8Akc6jod6akf2rksVGJHmfurY7NO1nHP4EekrMx1FRa2JfoPOHTdxcWDVaQAymDC4vgP5aW5RCEOURUY6YdTQUxleRiX-Bgp3eNRHNOcPvdedGm0w7M7gnZqCwy4tsgyiXkIM7zZpvCqdCA1vGJxf_UIck4R8Izua5NSacnG25miIvAmxNzOAEHDD_jDIDHnPVi3iVZzrjBkDwG6spYx_yJbbLy1XbJCYMMH44X4ajuQulV_NS-aiIHj_-PbxfrBRAJCVTZ8L3zD14BraeAAHFqSoiLXohmYHLLjshtraVu4XcvehJDfnRMi8Y4b6sqA


https://192.168.1.31:32721/

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
> CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、今日头条、个人博客、全网可搜《小陈运维》  
> 文章主要发布于微信：《Linux运维交流社区》

