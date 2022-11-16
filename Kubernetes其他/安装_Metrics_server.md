---
layout: post
cid: 238
title: 安装 Metrics server
slug: 238
date: 2022/06/03 15:16:57
updated: 2022/06/03 15:16:57
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


#安装 Metrics server

Metrics Server 是 Kubernetes 内置自动缩放管道的可扩展、高效的容器资源指标来源。

Metrics Server 从 Kubelets 收集资源指标，并通过Metrics API在 Kubernetes apiserver 中公开它们，以供 Horizo​​ntal Pod Autoscaler和Vertical Pod Autoscaler使用。Metrics API 也可以通过 访问kubectl top，从而更容易调试自动缩放管道。

## 单机版 

```
单机版 
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

查看镜像地址
grep -rn image components.yaml 
140:        image: k8s.gcr.io/metrics-server/metrics-server:v0.6.1
141:        imagePullPolicy: IfNotPresent

设置镜像地址为阿里云
sed -i "s#k8s.gcr.io/metrics-server#registry.cn-hangzhou.aliyuncs.com/chenby#g" components.yaml

查看镜像地址已更新
grep -rn image components.yaml 
140:        image: registry.cn-hangzhou.aliyuncs.com/chenby/metrics-server:v0.6.1
141:        imagePullPolicy: IfNotPresent

args添加tls证书配置选项
vim components.yaml

添加"- --kubelet-insecure-tls"

例：
	    args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        image: registry.cn-hangzhou.aliyuncs.com/chenby/metrics-server:v0.6.1

执行配置
kubectl apply -f components.yaml 
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created


```



## 高可用版本

```
高可用版本
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability.yaml

查看镜像地址
grep -rn image high-availability.yaml 
150:        image: k8s.gcr.io/metrics-server/metrics-server:v0.6.1
151:        imagePullPolicy: IfNotPresent

设置镜像地址为阿里云
sed -i "s#k8s.gcr.io/metrics-server#registry.cn-hangzhou.aliyuncs.com/chenby#g" high-availability.yaml

查看镜像地址已更新
grep -rn image high-availability.yaml 
150:        image: registry.cn-hangzhou.aliyuncs.com/chenby/metrics-server:v0.6.1
151:        imagePullPolicy: IfNotPresent

args添加tls证书配置选项
vim high-availability.yaml

添加"- --kubelet-insecure-tls"

例：
	    args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        image: registry.cn-hangzhou.aliyuncs.com/chenby/metrics-server:v0.6.1

执行配置
kubectl apply -f high-availability.yaml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
Warning: policy/v1beta1 PodDisruptionBudget is deprecated in v1.21+, unavailable in v1.25+; use policy/v1 PodDisruptionBudget
poddisruptionbudget.policy/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```



## 验证

```
查看metrics资源
kubectl  get pod -n kube-system | grep metrics
metrics-server-65fb95948b-2bcht            1/1     Running   0             32s
metrics-server-65fb95948b-vqp5s            1/1     Running   0             32s

查看node资源情况
kubectl  top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
k8s-master01   127m         1%     2439Mi          64%       
k8s-node01     50m          0%     1825Mi          23%       
k8s-node02     53m          0%     1264Mi          16%   

查看pod资源情况
kubectl  top pod 
NAME                      CPU(cores)   MEMORY(bytes)   
chenby-57479d5997-44926   0m           10Mi            
chenby-57479d5997-tbpqc   0m           11Mi            
chenby-57479d5997-w8cp2   0m           6Mi 

```





> https://www.oiox.cn/
> https:/blog.oiox.cn/
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