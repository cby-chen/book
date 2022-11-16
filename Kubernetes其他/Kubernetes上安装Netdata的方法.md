---
layout: post
cid: 141
title: 在Kubernetes上安装Netdata的方法
slug: 141
date: 2022/04/20 10:40:05
updated: 2022/04/20 10:41:09
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


介绍
==

Netdata可用于监视kubernetes集群并显示有关集群的信息，包括节点内存使用率、CPU、网络等，简单的说，Netdata仪表板可让您全面了解Kubernetes集群，包括在每个节点上运行的服务和Pod。

安装HELM
======

```
root@hello:~# curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
root@hello:~# sudo apt-get install apt-transport-https --yes
root@hello:~# echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
root@hello:~# sudo apt-get update
root@hello:~# sudo apt-get install helm

```

添加源并安装
======

```
root@hello:~# helm repo add netdata https://netdata.github.io/helmchart/
"netdata" has been added to your repositories


root@hello:~# helm install netdata netdata/netdata
W0420 09:20:51.993046 1306427 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
W0420 09:20:52.298158 1306427 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
NAME: netdata
LAST DEPLOYED: Wed Apr 20 09:20:50 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. netdata will be available on http://netdata.k8s.local/, on the exposed port of your ingress controller

In a production environment, you 
 You can get that port via `kubectl get services`. e.g. in the following example, the http exposed port is 31737, the https one is 30069.
 The hostname netdata.k8s.local will need to be added to /etc/hosts, so that it resolves to the exposed IP. That IP depends on how your cluster is set up: 
        - When no load balancer is available (e.g. with minikube), you get the IP shown on `kubectl cluster-info`
        - In a production environment, the command `kubectl get services` will show the IP under the EXTERNAL-IP column

The port can be retrieved in both cases from `kubectl get services`

NAME                                         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
exiled-tapir-nginx-ingress-controller        LoadBalancer   10.98.132.169    <pending>     80:31737/TCP,443:30069/TCP   11h


root@hello:~# helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
netdata default         1               2022-04-20 09:20:50.947921117 +0800 CST deployed        netdata-3.7.15  v1.33.1    

```

查看POD
=====

```
root@hello:~# kubectl  get pod 
NAME                                      READY   STATUS    RESTARTS      AGE
netdata-child-2h65n                       2/2     Running   0             77s
netdata-child-dfv82                       2/2     Running   0             77s
netdata-child-h6fw6                       2/2     Running   0             77s
netdata-child-lc9fd                       2/2     Running   0             77s
netdata-child-nh566                       2/2     Running   0             77s
netdata-child-ns2p2                       2/2     Running   0             77s
netdata-child-v74x5                       2/2     Running   0             77s
netdata-child-xjlrv                       2/2     Running   0             77s
netdata-parent-57bf6bf47d-vc6fq           1/1     Running   0             77s

```

添加SVC使外部即可访问
============

```
root@hello:~# kubectl  get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP     18d
netdata      ClusterIP   10.102.160.106   <none>        19999/TCP   3m39s


root@hello:~# kubectl expose  deployment netdata-parent --type="NodePort" --port 19999
service/netdata-parent exposed


root@hello:~# kubectl  get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP           18d
netdata          ClusterIP   10.102.160.106   <none>        19999/TCP         3m43s
netdata-parent   NodePort    10.100.122.173   <none>        19999:30518/TCP   2s
root@hello:~# 

通过http://<yourmaster-IP>:30518  访问浏览器中的netdata仪表板
```

  

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a8de81edea4462aac070499223de171~tplv-k3u1fbpfcp-zoom-1.image)

  

点击左侧可以查看具体每一台机器的信息  

https://www.oiox.cn/  
https://www.chenby.cn/  
https://cby-chen.github.io/  
https://blog.csdn.net/qq_33921750  
https://my.oschina.net/u/3981543  
https://www.zhihu.com/people/chen-bu-yun-2  
https://segmentfault.com/u/hppyvyv6/articles  
https://juejin.cn/user/3315782802482007  
https://cloud.tencent.com/developer/column/93230  
https://www.jianshu.com/u/0f894314ae2c  
https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/  
CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、今日头条、个人博客、全网可搜《小陈运维》  
