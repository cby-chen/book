---
layout: post
cid: 55
title: kubernetes核心实战（五）--- StatefulSets
slug: 55
date: 2021/12/30 17:12:38
updated: 2021/12/30 17:12:38
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


#### 7、StatefulSets

StatefulSet 是用来管理有状态应用的工作负载 API 对象。

StatefulSet 用来管理 Deployment 和扩展一组 Pod，并且能为这些 Pod 提供序号和唯一性保证。

和 Deployment 相同的是，StatefulSet 管理了基于相同容器定义的一组 Pod。但和 Deployment 不同的是，StatefulSet 为它们的每个 Pod 维护了一个固定的 ID。这些 Pod 是基于相同的声明来创建的，但是不能相互替换：无论怎么调度，每个 Pod 都有一个永久不变的 ID。

StatefulSet 和其他控制器使用相同的工作模式。你在 StatefulSet 对象 中定义你期望的状态，然后 StatefulSet 的 控制器 就会通过各种更新来达到那种你想要的状态。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/825bfff1644b4702a283bfcc940eebf0~tplv-k3u1fbpfcp-zoom-1.image)

  

###### 使用 StatefulSets

StatefulSets 对于需要满足以下一个或多个需求的应用程序很有价值：

稳定的、唯一的网络标识符。稳定的、持久的存储。有序的、优雅的部署和缩放。有序的、自动的滚动更新。在上面，稳定意味着 Pod 调度或重调度的整个过程是有持久性的。如果应用程序不需要任何稳定的标识符或有序的部署、删除或伸缩，则应该使用由一组无状态的副本控制器提供的工作负载来部署应用程序，比如 Deployment 或者 ReplicaSet 可能更适用于您的无状态应用部署需要。

###### 限制

给定 Pod 的存储必须由 PersistentVolume 驱动 基于所请求的 storage class 来提供，或者由管理员预先提供。删除或者收缩 StatefulSet 并不会删除它关联的存储卷。这样做是为了保证数据安全，它通常比自动清除 StatefulSet 所有相关的资源更有价值。StatefulSet 当前需要无头服务 来负责 Pod 的网络标识。您需要负责创建此服务。当删除 StatefulSets 时，StatefulSet 不提供任何终止 Pod 的保证。为了实现 StatefulSet 中的 Pod 可以有序和优雅的终止，可以在删除之前将 StatefulSet 缩放为 0。在默认 Pod 管理策略(OrderedReady) 时使用 滚动更新，可能进入需要 人工干预 才能修复的损坏状态。

  

###### 示例：

```
[root@k8s-master-node1 ~/yaml/test]# vim statefulsets.yaml
[root@k8s-master-node1 ~/yaml/test]# cat statefulsets.yaml 
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pvc-0
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pvc-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pvc-2
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
      volumes:
        - name: www
          persistentVolumeClaim:
            claimName: nginx-pvc-0
      volumes:
        - name: www
          persistentVolumeClaim:
            claimName: nginx-pvc-1
      volumes:
        - name: www
          persistentVolumeClaim:
            claimName: nginx-pvc-2

[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 创建statefulsets

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  apply -f statefulsets.yaml 
service/nginx created
statefulset.apps/web created
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看pod

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  get pod
NAME                                     READY   STATUS    RESTARTS   AGE
ingress-demo-app-694bf5d965-8rh7f        1/1     Running   0          67m
ingress-demo-app-694bf5d965-swkpb        1/1     Running   0          67m
nfs-client-provisioner-dc5789f74-5bznq   1/1     Running   0          52m
web-0                                    1/1     Running   0          93s
web-1                                    1/1     Running   0          85s
web-2                                    1/1     Running   0          66s
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看statefulsets

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  get statefulsets.apps -o wide
NAME   READY   AGE    CONTAINERS   IMAGES
web    3/3     113s   nginx        nginx
[root@k8s-master-node1 ~/yaml/test]#
```

  

注意：前提是解决kubernetes动态分配pv，参考文档：https://cloud.tencent.com/developer/article/1902519

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad0194a22f5c42cfb18977a7fd5be1c2~tplv-k3u1fbpfcp-zoom-1.image)  

> **关于**
>
> https://www.oiox.cn/
>
> https://www.oiox.cn/index.php/start-page.html
>
> **CSDN、GitHub、51CTO、知乎、开源中国、思否、掘金、简书、华为云、阿里云、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客**
>
> **全网可搜《小陈运维》**
>
> **文章主要发布于微信**