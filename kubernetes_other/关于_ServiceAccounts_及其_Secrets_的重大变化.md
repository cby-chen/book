---
layout: post
cid: 226
title: 关于 ServiceAccounts 及其 Secrets 的重大变化
slug: 226
date: 2022/05/25 15:20:28
updated: 2022/05/25 15:22:48
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: 关于 ServiceAccounts 及其 Secrets 的重大变化
keywords: Kubernetes,k8s,二进制,
mode: default
thumb: 
video: 
---


# 关于 ServiceAccounts 及其 Secrets 的重大变化



kubernetes v1.24.0 更新之后进行创建 ServiceAccount 不会自动生成 Secret 需要对其手动创建



## 创建 ServiceAccount

```
cat<<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cby
  namespace: default
EOF
```



## 查看 ServiceAccount

```
root@cby:~# kubectl get serviceaccounts cby
NAME                     SECRETS   AGE
cby                      0         9s
```



## 查看 ServiceAccount 详细详细，没有对 Token 进行创建

```
root@cby:~# kubectl describe serviceaccounts cby
Name:                cby
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>
root@cby:~# 

root@cby:~# kubectl get secrets
No resources found in default namespace.
root@cby:~#
```



## 创建 Secret 资源并与 ServiceAccount 关联

```
cat<<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: cby
  annotations:
    kubernetes.io/service-account.name: "cby"
EOF
```



## 再次查看 ServiceAccount 已对 Secret 关联

```
root@cby:~# kubectl describe serviceaccounts cby
Name:                cby
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              cby
Events:              <none>
root@cby:~# 
```



## 查看 Secret 详细详细

```
root@cby:~# kubectl get secrets cby 
NAME   TYPE                                  DATA   AGE
cby    kubernetes.io/service-account-token   3      35s
root@cby:~# 

root@cby:~# kubectl describe secrets cby 
Name:         cby
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: cby
              kubernetes.io/service-account.uid: c6629b84-1c08-483d-9a12-c2930ac0a2fe

Type:  kubernetes.io/service-account-token

Data
====

ca.crt:     1363 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjRwMk02VU9leXU3N3lraUN6UVQ4R3I3Smw3eFhYdEVMX1Z2aTFjU2luSVEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImNieSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJjYnkiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJjNjYyOWI4NC0xYzA4LTQ4M2QtOWExMi1jMjkzMGFjMGEyZmUiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpjYnkifQ.r0nHVPO-QY-1p0fwKx0p0AfkiCGpTZ8vGzE8ioDtih5cAP1ew3ABnrj01EqeIEn8vhz29i0NHtZfh5XtYttqjU6o_b1IGFtkW5uIwlxYX2gtmm9njsL2NM7YM6lM0BDfQXvYrpKUuWLQUR-8i79h-GH9WFydmEwnthdxit7uSMJIZuyZP0X0ebxWUg1GGHsqNPy514zXEyvTZh8vs4fVl5ROJbKzFuSuQ1TntXMDncHSf8DSJ7iHUZ0pD757ysHvFKH9l6IbGrt8GUvxWxjMvnNjclLozKgfLXQEOVei39VrPU5DtsPp9DU8C04Gn4TWFW_WsyEWM14lGsQEGD-2QA
root@cby:~# 
```



## 删除 ServiceAccount 随之 Secret 一并自动删除

```
root@cby:~# kubectl delete serviceaccounts cby 
serviceaccount "cby" deleted
root@cby:~#

root@cby:~# kubectl get serviceaccounts
root@cby:~# kubectl get secret
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