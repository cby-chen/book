---
layout: post
cid: 52
title: kubernetes核心实战（二）---Pod+ReplicaSet
slug: 52
date: 2021/12/30 17:12:00
updated: 2022/05/19 15:56:32
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


#### 3、pod

Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e8a8942405543b1b5db2621d244407a~tplv-k3u1fbpfcp-zoom-1.image)

Pod （就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） 容器；这些容器共享存储、网络、以及怎样运行这些容器的声明。Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。Pod 所建模的是特定于应用的“逻辑主机”，其中包含一个或多个应用容器， 这些容器是相对紧密的耦合在一起的。在非云环境中，在相同的物理机或虚拟机上运行的应用类似于 在同一逻辑主机上运行的云应用。

  

除了应用容器，Pod 还可以包含在 Pod 启动期间运行的 Init 容器。你也可以在集群中支持临时性容器 的情况外，为调试的目的注入临时性容器。

  

##### 使用 Pod

通常你不需要直接创建 Pod，甚至单实例 Pod。相反，你会使用诸如 Deployment 或 Job 这类工作负载资源 来创建 Pod。如果 Pod 需要跟踪状态， 可以考虑 StatefulSet 资源。

  

Kubernetes 集群中的 Pod 主要有**两种**用法：

  

运行单个容器的 Pod。"每个 Pod 一个容器"模型是最常见的 Kubernetes 用例；在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。

  

运行多个协同工作的容器的 Pod。Pod 可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序。这些位于同一位置的容器可能形成单个内聚的服务单元 —— 一个容器将文件从共享卷提供给公众， 而另一个单独的“挂斗”（sidecar）容器则刷新或更新这些文件。Pod 将这些容器和存储资源打包为一个可管理的实体。

  

说明：将多个并置、同管的容器组织到一个 Pod 中是一种相对高级的使用场景。只有在一些场景中，容器之间紧密关联时你才应该使用这种模式。

每个 Pod 都旨在运行给定应用程序的单个实例。如果希望横向扩展应用程序（例如，运行多个实例 以提供更多的资源），则应该使用多个 Pod，每个实例使用一个 Pod。在 Kubernetes 中，这通常被称为 副本（Replication）。通常使用一种工作负载资源及其控制器 来创建和管理一组 Pod 副本。

  

Pod 怎样管理多个容器

Pod 被设计成支持形成内聚服务单元的多个协作过程（形式为容器）。Pod 中的容器被自动安排到集群中的同一物理机或虚拟机上，并可以一起进行调度。容器之间可以共享资源和依赖、彼此通信、协调何时以及何种方式终止自身。

  

例如，你可能有一个容器，为共享卷中的文件提供 Web 服务器支持，以及一个单独的 “sidecar（挂斗）”容器负责从远端更新这些文件，如下图所示：

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b95fa02d2c3140cc982206fb9988e4f6~tplv-k3u1fbpfcp-zoom-1.image)

#### 4、ReplicaSet

ReplicaSet 的目的是维护一组在任何时候都处于运行状态的 Pod 副本的稳定集合。因此，它通常用来保证给定数量的、完全相同的 Pod 的可用性。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0048a17c108e438283d95a482ac437a4~tplv-k3u1fbpfcp-zoom-1.image)

ReplicaSet 的工作原理

RepicaSet 是通过一组字段来定义的，包括一个用来识别可获得的 Pod 的集合的选择算符，一个用来标明应该维护的副本个数的数值，一个用来指定应该创建新 Pod 以满足副本个数条件时要使用的 Pod 模板等等。每个 ReplicaSet 都通过根据需要创建和 删除 Pod 以使得副本个数达到期望值，进而实现其存在价值。当 ReplicaSet 需要创建 新的 Pod 时，会使用所提供的 Pod 模板。

  

ReplicaSet 通过 Pod 上的 metadata.ownerReferences 字段连接到附属 Pod，该字段给出当前对象的属主资源。ReplicaSet 所获得的 Pod 都在其 ownerReferences 字段中包含了属主 ReplicaSet 的标识信息。正是通过这一连接，ReplicaSet 知道它所维护的 Pod 集合的状态， 并据此计划其操作行为。

  

ReplicaSet 使用其选择算符来辨识要获得的 Pod 集合。如果某个 Pod 没有 OwnerReference 或者其 OwnerReference 不是一个 控制器，且其匹配到 某 ReplicaSet 的选择算符，则该 Pod 立即被此 ReplicaSet 获得。

  

示例：

```
[root@k8s-master-node1 ~/yaml/test]# vim pod.yaml [root@k8s-master-node1 ~/yaml/test]# cat pod.yaml apiVersion: apps/v1kind: ReplicaSetmetadata:  name: frontend  labels:    app: guestbook    tier: frontendspec:  # modify replicas according to your case  replicas: 3  selector:    matchLabels:      tier: frontend  template:    metadata:      labels:        tier: frontend    spec:      containers:      - name: nginx        image: nginx[root@k8s-master-node1 ~/yaml/test]#
```

  

##### 创建

```
[root@k8s-master-node1 ~/yaml/test]# kubectl apply -f pod.yaml replicaset.apps/frontend created
```

  

##### 查看

```
[root@k8s-master-node1 ~/yaml/test]# kubectl get podNAME                                     READY   STATUS    RESTARTS     AGEfrontend-8zxxw                           1/1     Running   0            2m26sfrontend-l22df                           1/1     Running   0            2m26sfrontend-qnhkr                           1/1     Running   0            2m26s[root@k8s-master-node1 ~/yaml/test]#
```

  

##### 删除

```
[root@k8s-master-node1 ~/yaml/test]# kubectl delete -f pod.yaml replicaset.apps "frontend" deleted[root@k8s-master-node1 ~/yaml/test]#
```

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcb34e25e7bc44959c9255c2b5dc0555~tplv-k3u1fbpfcp-zoom-1.image)  

  

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

知乎、CSDN、开源中国、思否、掘金、哔哩哔哩、腾讯云

  

```