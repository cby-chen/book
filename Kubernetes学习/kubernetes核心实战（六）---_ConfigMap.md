---
layout: post
cid: 56
title: kubernetes核心实战（六）--- ConfigMap
slug: 56
date: 2021/12/30 17:12:00
updated: 2022/05/19 15:55:35
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


#### 8、ConfigMap

抽取应用配置，并且可以自动更新

###### 创建配置文件  

```
[root@k8s-master-node1 ~/yaml/test]# vim configmap.yaml
[root@k8s-master-node1 ~/yaml/test]# cat configmap.yaml 
apiVersion: v1
data:   
  redis.conf: |
    appendonly yes
kind: ConfigMap
metadata:
  name: redis-conf
  namespace: default
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl apply -f configmap.yaml 
configmap/redis-conf created
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看配置

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  get configmaps 
NAME               DATA   AGE
kube-root-ca.crt   1      110m
redis-conf         1      18s
[root@k8s-master-node1 ~/yaml/test]#
```

  

  

#### 9、DaemonSet

DaemonSet 确保全部（或者某些）节点上运行一个 Pod 的副本。当有节点加入集群时， 也会为他们新增一个 Pod 。当有节点从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。

DaemonSet 的一些典型用法：

在每个节点上运行集群存守护进程在每个节点上运行日志收集守护进程在每个节点上运行监控守护进程一种简单的用法是为每种类型的守护进程在所有的节点上都启动一个 DaemonSet。一个稍微复杂的用法是为同一种守护进程部署多个 DaemonSet；每个具有不同的标志， 并且对不同硬件类型具有不同的内存、CPU 要求。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f346eb21df342eb90fae6fc2f45f178~tplv-k3u1fbpfcp-zoom-1.image)

###### 创建

```
[root@k8s-master-node1 ~/yaml/test]# vim daemonset.yaml
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# cat daemonset.yaml 
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: redis-app
  labels:
    k8s-app: redis-app
spec:
  selector:
    matchLabels:
      name: fluentd-redis
  template:
    metadata:
      labels:
        name: fluentd-redis
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-redis
        image: redis
        command:
          - redis-server
          - "/redis-master/redis.conf"  #指的是redis容器内部的位置
        ports:
        - containerPort: 6379
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: data
          mountPath: /data
        - name: config
          mountPath: /redis-master
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: data
        emptyDir: {}
      - name: config
        configMap: 
          name: redis-conf
          items:
          - key: redis.conf
            path: redis.conf
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  apply -f daemonset.yaml 
daemonset.apps/redis-app created
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看

```
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  get pod
NAME                                     READY   STATUS    RESTARTS   AGE
ingress-demo-app-694bf5d965-8rh7f        1/1     Running   0          130m
ingress-demo-app-694bf5d965-swkpb        1/1     Running   0          130m
nfs-client-provisioner-dc5789f74-5bznq   1/1     Running   0          114m
redis-app-86g4q                          1/1     Running   0          28s
redis-app-rt92n                          1/1     Running   0          28s
redis-app-vkzft                          1/1     Running   0          28s
web-0                                    1/1     Running   0          64m
web-1                                    1/1     Running   0          63m
web-2                                    1/1     Running   0          63m
[root@k8s-master-node1 ~/yaml/test]# kubectl  get daemonsets.apps 
NAME        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
redis-app   3         3         3       3            3           <none>          38s
[root@k8s-master-node1 ~/yaml/test]#
```

  

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
> **文章主要发布于微信公众号**