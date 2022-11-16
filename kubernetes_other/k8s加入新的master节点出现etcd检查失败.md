---
layout: post
cid: 6
title: k8s加入新的master节点出现etcd检查失败
slug: 6
date: 2021/12/30 16:58:02
updated: 2021/12/30 16:58:02
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/822b3cb8ad29468fbab442636897ca91~tplv-k3u1fbpfcp-zoom-1.image)

 **背景：**  

 **昨天在建立好新的集群后，出现了新的问题，其中的一台master节点无法正常工作。虽然可以正常使用，但是就出现了单点故障，今天在修复时出现了etcd健康检查自检没通过。**

 **Yesterday, after a new cluster was established, a new problem a problem occurred, and one of the master nodes did not work properly. Although can be used normally, but there is a single point of failure, today in the repair of the etcd health check self-test failed.**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d331969e33c44e378170cce9b8c803dd~tplv-k3u1fbpfcp-zoom-1.image)

**对加入集群中时，出现如下报错：**

**When you join a cluster, the following error occurs**  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d6b5acc83f54005847ad3a4eb70338e~tplv-k3u1fbpfcp-zoom-1.image)

  

    **提示 etcd 监控检查失败，查看一下Kubernetes 集群中的 kubeadm 配置信息。**

 **Prompt the etcd monitoring check to fail and review the kubeadm configuration information in the Kubernetes cluster.**

  

* * *

```
\[root@master-01 ~\]# kubectl describe configmaps kubeadm-config -n kube-system
----
apiEndpoints:
  master-01:
    advertiseAddress: 10.0.0.11
    bindPort: 6443
  master-02:
    advertiseAddress: 10.0.0.12
    bindPort: 6443
  master-03:
    advertiseAddress: 10.0.0.13
    bindPort: 6443
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterStatus

Events:  <none>

```

  

  **因为集群搭建的时候，etcd是镜像的方式，在master02上面出现问题后，进行剔除完成后，etcd还是在存储在每个master上面，所以重新添加的时候会得知健康检查失败。**

 **Because when the cluster is built, etcd is mirrored, after the problem on master02, after the cull is completed, etcd is still stored on top of each master, so when you add again, you will learn that the health check failed.**

  

* * *

 **这时就需要进入容器内部进行手动删除这个etcd了，首先获取集群中的etcd pod列表看一下，并进入内部给一个sh窗口。**  

 **At this point you need to go inside the container to manually delete this etcd, first get the list of etcd pods in the cluster to see, and go inside to give a sh window**

  

```
\[root@master-01 ~\]# kubectl get pods -n kube-system | grep etcd
\[root@master-01 ~\]# kubectl exec -it etcd-master-03 sh -n kube-system

```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/693f737c0d5343d6a3f622380144f2ce~tplv-k3u1fbpfcp-zoom-1.image)

  

    **进入容器后，执行如下操作**：

    **After entering the container, do the following**  
  

```
\## 配置环境
$ export ETCDCTL_API=3
$ alias etcdctl='etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key'

## 查看 etcd 集群成员列表
$ etcdctl member list

## 删除 etcd 集群成员 master-02
$ etcdctl member remove 

## 再次查看 etcd 集群成员列表
$ etcdctl member list

## 退出容器
$ exit
```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de5f4d3222304d2998cf66d5e13d878f~tplv-k3u1fbpfcp-zoom-1.image)

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69580aae7635438787b6e2f7ed18d513~tplv-k3u1fbpfcp-zoom-1.image)

  

**查看列表并删除已不存在的master**

**View the list and remove the master that no longer exists**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a800ec4b901346e6b1bcda214dea07d7~tplv-k3u1fbpfcp-zoom-1.image)

* * *

  

**再次进行加入master，即可成功。**

**Join master again and you'll be successful**

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5560a9a6ea24fa09d7ac83e72d50220~tplv-k3u1fbpfcp-zoom-1.image)

  

* * *

* * *

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23cb342ecf694be6a7b3317b62995d9d~tplv-k3u1fbpfcp-zoom-1.image)

高新科技园