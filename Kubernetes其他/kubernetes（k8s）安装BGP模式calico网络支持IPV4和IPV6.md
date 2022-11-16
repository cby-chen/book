---
layout: post
cid: 264
title: kubernetes（k8s）安装BGP模式calico网络支持IPV4和IPV6
slug: 264
date: 2022/06/16 16:47:56
updated: 2022/06/16 16:47:56
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: kubernetes（k8s）安装BGP模式calico网络支持IPV4和IPV6
keywords: Kubernetes,k8s,二进制,
mode: default
thumb: 
video: 
---


kubernetes（k8s）安装BGP模式calico网络支持IPV4和IPV6
=========================================

BGP是互联网上一个核心的去中心化自治路由协议，它通过维护IP路由表或“前缀”表来实现自治系统AS之间的可达性，属于矢量路由协议。不过，考虑到并非所有的网络都能支持BGP，以及Calico控制平面的设计要求物理网络必须是二层网络，以确保 vRouter间均直接可达，路由不能够将物理设备当作下一跳等原因，为了支持三层网络，Calico还推出了IP-in-IP叠加的模型，它也使用Overlay的方式来传输数据。IPIP的包头非常小，而且也是内置在内核中，因此理论上它的速度要比VxLAN快一点 ，但安全性更差。Calico 3.x的默认配置使用的是IPIP类型的传输方案而非BGP。

Calico的系统架构如图所示：

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3887ac0fa7f4151a5133451ebcb46c0~tplv-k3u1fbpfcp-zoom-1.image)

Calico 主要由 Felix、etcd、BGP client 以及 BGP Route Reflector 组成

1.  Felix，Calico Agent，跑在每台需要运行 Workload 的节点上，主要负责配置路由及 ACLs 等信息来确保 Endpoint 的连通状态；
    
2.  etcd，分布式键值存储，主要负责网络元数据一致性，确保 Calico 网络状态的准确性；
    
3.  BGP Client（BIRD）, 主要负责把 Felix 写入 Kernel 的路由信息分发到当前 Calico 网络，确保 Workload 间的通信的有效性；
    
4.  BGP Route Reflector（BIRD），大规模部署时使用，摒弃所有节点互联的 mesh 模式，通过一个或者多个 BGP Route Reflector 来完成集中式的路由分发。
    
5.  calico/calico-ipam，主要用作 Kubernetes 的 CNI 插件
    

### 配置NetworkManager防止干扰calico

```
[root@k8s-master01 ~]# vim /etc/NetworkManager/conf.d/calico.conf
[root@k8s-master01 ~]# cat /etc/NetworkManager/conf.d/calico.conf
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:tunl*
[root@k8s-master01 ~]# 
```



### 下载官方最新calico配置文件

```
[root@k8s-master01 ~]# curl https://projectcalico.docs.tigera.io/manifests/calico-typha.yaml -o calico.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  228k  100  228k    0     0  83974      0  0:00:02  0:00:02 --:--:-- 83974
[root@k8s-master01 ~]# 
```



### 修改calico配置得以支持IPV6

```
[root@k8s-master01 ~]# cp calico.yaml calico-ipv6.yaml
[root@k8s-master01 ~]# vim calico-ipv6.yaml
# calico-config ConfigMap处
    "ipam": {
        "type": "calico-ipam",
        "assign_ipv4": "true",
        "assign_ipv6": "true"
    },
    
    - name: CLUSTER_TYPE
      value: "k8s,bgp"

    - name: IP
      value: "autodetect"

    - name: IP6
      value: "autodetect"

    - name: CALICO_IPV4POOL_CIDR
      value: "172.16.0.0/16"

    - name: CALICO_IPV6POOL_CIDR
      value: "fc00::/48"

    - name: FELIX_IPV6SUPPORT
      value: "true"
```



### 修改calico配置得以支持IPV4

```
[root@k8s-master01 ~]# grep "IPV4POOL_CIDR" calico.yaml  -A 1
            - name: CALICO_IPV4POOL_CIDR
              value: "172.16.0.0/12"
[root@k8s-master01 ~]# kubectl apply -f calico.yaml
```



### 查看POD

```
[root@k8s-master01 ~]# kubectl  get pod -A -w
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-56cdb7c587-l8h5k   1/1     Running   0          66s
kube-system   calico-node-b2mpq                          1/1     Running   0          66s
kube-system   calico-node-jlk89                          1/1     Running   0          66s
kube-system   calico-node-nqdc4                          1/1     Running   0          66s
kube-system   calico-node-pjrcn                          1/1     Running   0          66s
kube-system   calico-node-w4gfm                          1/1     Running   0          66s
kube-system   calico-typha-6775694657-vk4ds              1/1     Running   0          66s
```

> **关于**
>
> https://www.oiox.cn/
>
> https://www.oiox.cn/index.php/start-page.html
>
> **CSDN、GitHub、知乎、开源中国、思否、掘金、简书、华为云、阿里云、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客**
>
> **全网可搜《小陈运维》**
>
> **文章主要发布于微信：《Linux运维交流社区》**