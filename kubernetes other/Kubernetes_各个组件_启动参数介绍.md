---
layout: post
cid: 102
title: Kubernetes 各个组件 启动参数介绍
slug: 102
date: 2022/03/08 17:58:14
updated: 2022/03/08 17:58:59
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


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d859edb86c542e894815f35edc810b7~tplv-k3u1fbpfcp-zoom-1.image)  

**kube-controller-manager**

  

Kubernetes 控制器管理器是一个守护进程，内嵌随 Kubernetes 一起发布的核心控制回路。在机器人和自动化的应用中，控制回路是一个永不休止的循环，用于调节系统状态。在 Kubernetes 中，每个控制器是一个控制回路，通过 API 服务器监视集群的共享状态， 并尝试进行更改以将当前状态转为期望状态。目前，Kubernetes 自带的控制器例子包括副本控制器、节点控制器、命名空间控制器和服务账号控制器等。

  
  

```
cat > /usr/lib/systemd/system/kube-controller-manager.service << EOF


[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
After=network.target


[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
--v=2 \
--logtostderr=true \
--address=127.0.0.1 \
--root-ca-file=/etc/kubernetes/pki/ca.pem \
--cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem \
--cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem \
--service-account-private-key-file=/etc/kubernetes/pki/sa.key \
--kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \
--leader-elect=true \
--use-service-account-credentials=true \
--node-monitor-grace-period=40s \
--node-monitor-period=5s \
--pod-eviction-timeout=2m0s \
--controllers=*,bootstrapsigner,tokencleaner \
--allocate-node-cidrs=true \
--cluster-cidr=172.16.0.0/12 \
--requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \
--node-cidr-mask-size=24


Restart=always
RestartSec=10s


[Install]
WantedBy=multi-user.target


EOF


-v, --v int
日志级别详细程度取值。


--logtostderr     默认值：true
将日志写出到标准错误输出（stderr）而不是写入到日志文件。


--root-ca-file string
如果此标志非空，则在服务账号的令牌 Secret 中会包含此根证书机构。所指定标志值必须是一个合法的 PEM 编码的 CA 证书包。


--cluster-signing-cert-file string
包含 PEM 编码格式的 X509 CA 证书的文件名。该证书用来发放集群范围的证书。如果设置了此标志，则不能指定更具体的--cluster-signing-* 标志。


--cluster-signing-key-file string
包含 PEM 编码的 RSA 或 ECDSA 私钥的文件名。该私钥用来对集群范围证书签名。若指定了此选项，则不可再设置 --cluster-signing-* 参数。


--kubeconfig string
指向 kubeconfig 文件的路径。该文件中包含主控节点位置以及鉴权凭据信息。


--leader-elect     默认值：true
在执行主循环之前，启动领导选举（Leader Election）客户端，并尝试获得领导者身份。在运行多副本组件时启用此标志有助于提高可用性。


--use-service-account-credentials
当此标志为 true 时，为每个控制器单独使用服务账号凭据。


--node-monitor-grace-period duration     默认值：40s
在将一个 Node 标记为不健康之前允许其无响应的时长上限。必须比 kubelet 的 nodeStatusUpdateFrequency 大 N 倍；这里 N 指的是 kubelet 发送节点状态的重试次数。


--node-monitor-period duration     默认值：5s
节点控制器对节点状态进行同步的重复周期。


--pod-eviction-timeout duration     默认值：5m0s
在失效的节点上删除 Pods 时为其预留的宽限期。


--controllers strings     默认值：[*]
要启用的控制器列表。\* 表示启用所有默认启用的控制器；foo 启用名为 foo 的控制器；-foo 表示禁用名为 foo 的控制器。
控制器的全集：attachdetach、bootstrapsigner、cloud-node-lifecycle、clusterrole-aggregation、cronjob、csrapproving、csrcleaner、csrsigning、daemonset、deployment、disruption、endpoint、endpointslice、endpointslicemirroring、ephemeral-volume、garbagecollector、horizontalpodautoscaling、job、namespace、nodeipam、nodelifecycle、persistentvolume-binder、persistentvolume-expander、podgc、pv-protection、pvc-protection、replicaset、replicationcontroller、resourcequota、root-ca-cert-publisher、route、service、serviceaccount、serviceaccount-token、statefulset、tokencleaner、ttl、ttl-after-finished
默认禁用的控制器有：bootstrapsigner 和 tokencleaner。


--allocate-node-cidrs
基于云驱动来为 Pod 分配和设置子网掩码。


--requestheader-client-ca-file string
根证书包文件名。在信任通过 --requestheader-username-headers 所指定的任何用户名之前，要使用这里的证书来检查请求中的客户证书。警告：一般不要依赖对请求所作的鉴权结果。


--node-cidr-mask-size int32
集群中节点 CIDR 的掩码长度。对 IPv4 而言默认为 24；对 IPv6 而言默认为 64。


--node-cidr-mask-size-ipv4 int32
在双堆栈（同时支持 IPv4 和 IPv6）的集群中，节点 IPV4 CIDR 掩码长度。默认为 24。


--node-cidr-mask-size-ipv6 int32
在双堆栈（同时支持 IPv4 和 IPv6）的集群中，节点 IPv6 CIDR 掩码长度。默认为 64。
```

  

  

**kube-scheduler**

  

Kubernetes 调度器是一个控制面进程，负责将 Pods 指派到节点上。调度器基于约束和可用资源为调度队列中每个 Pod 确定其可合法放置的节点。调度器之后对所有合法的节点进行排序，将 Pod 绑定到一个合适的节点。在同一个集群中可以使用多个不同的调度器；kube-scheduler 是其参考实现。参阅调度 以获得关于调度和 kube-scheduler 组件的更多信息。

  
  

```
cat > /usr/lib/systemd/system/kube-scheduler.service << EOF


[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
After=network.target


[Service]
ExecStart=/usr/local/bin/kube-scheduler \
--v=2 \
--logtostderr=true \
--address=127.0.0.1 \
--leader-elect=true \
--kubeconfig=/etc/kubernetes/scheduler.kubeconfig


Restart=always
RestartSec=10s


[Install]
WantedBy=multi-user.target


--logtostderr     默认值：true
日志记录到标准错误输出而不是文件。


--leader-elect     默认值：true
在执行主循环之前，开始领导者选举并选出领导者。使用多副本来实现高可用性时，可启用此标志。


--kubeconfig string
已弃用: 包含鉴权和主节点位置信息的 kubeconfig 文件的路径。如果 --config 指定了一个配置文件，那么这个参数将被忽略。
```

  
  

**kubelet**

  

kubelet 是在每个 Node 节点上运行的主要 “节点代理”。它可以使用以下之一向 apiserver 注册：主机名（hostname）；覆盖主机名的参数；某云驱动的特定逻辑。

  

kubelet 是基于 PodSpec 来工作的。每个 PodSpec 是一个描述 Pod 的 YAML 或 JSON 对象。kubelet 接受通过各种机制（主要是通过 apiserver）提供的一组 PodSpec，并确保这些 PodSpec 中描述的容器处于运行状态且运行状况良好。kubelet 不管理不是由 Kubernetes 创建的容器。

  

除了来自 apiserver 的 PodSpec 之外，还可以通过以下三种方式将容器清单（manifest）提供给 kubelet。

  

文件（File）：利用命令行参数传递路径。kubelet 周期性地监视此路径下的文件是否有更新。监视周期默认为 20s，且可通过参数进行配置。

  

HTTP 端点（HTTP endpoint）：利用命令行参数指定 HTTP 端点。此端点的监视周期默认为 20 秒，也可以使用参数进行配置。

  

HTTP 服务器（HTTP server）：kubelet 还可以侦听 HTTP 并响应简单的 API （目前没有完整规范）来提交新的清单。

  
  
  
  

```
cat >  /etc/systemd/system/kubelet.service.d/10-kubelet.conf << EOF
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig --kubeconfig=/etc/kubernetes/kubelet.kubeconfig"
Environment="KUBELET_SYSTEM_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin --container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock --cgroup-driver=systemd"
Environment="KUBELET_CONFIG_ARGS=--config=/etc/kubernetes/kubelet-conf.yml"
Environment="KUBELET_EXTRA_ARGS=--node-labels=node.kubernetes.io/node='' "
ExecStart=
ExecStart=/usr/local/bin/kubelet \$KUBELET_KUBECONFIG_ARGS \$KUBELET_CONFIG_ARGS \$KUBELET_SYSTEM_ARGS \$KUBELET_EXTRA_ARGS


EOF


--bootstrap-kubeconfig string
某 kubeconfig 文件的路径，该文件将用于获取 kubelet 的客户端证书。如果 --kubeconfig 所指定的文件不存在，则使用引导所用 kubeconfig 从 API 服务器请求客户端证书。成功后，将引用生成的客户端证书和密钥的 kubeconfig 写入 --kubeconfig 所指定的路径。客户端证书和密钥文件将存储在 --cert-dir 所指的目录。


--kubeconfig string
kubeconfig 配置文件的路径，指定如何连接到 API 服务器。提供 --kubeconfig 将启用 API 服务器模式，而省略 --kubeconfig 将启用独立模式。


--network-plugin string
<警告：alpha 特性> 设置 kubelet/Pod 生命周期中各种事件调用的网络插件的名称。仅当容器运行环境设置为 docker 时，此特定于 docker 的参数才有效。


--cni-conf-dir string     默认值：/etc/cni/net.d
<警告：alpha 特性> 此值为某目录的全路径名。kubelet 将在其中搜索 CNI 配置文件。仅当容器运行环境设置为 docker 时，此特定于 docker 的参数才有效。


--cni-bin-dir string     默认值：/opt/cni/bin
<警告：alpha 特性> 此值为以逗号分隔的完整路径列表。kubelet 将在所指定路径中搜索 CNI 插件的可执行文件。仅当容器运行环境设置为 docker 时，此特定于 docker 的参数才有效。


--container-runtime string     默认值：docker
要使用的容器运行时。目前支持 docker、remote。


--runtime-request-timeout duration     默认值：2m0s
设置除了长时间运行的请求（包括 pull、logs、exec 和 attach 等操作）之外的其他运行时请求的超时时间。到达超时时间时，请求会被取消，抛出一个错误并会等待重试。已弃用：应在 --config 所给的配置文件中进行设置。


--container-runtime-endpoint string     默认值：unix:///var/run/dockershim.sock
[实验性特性] 远程运行时服务的端点。目前支持 Linux 系统上的 UNIX 套接字和 Windows 系统上的 npipe 和 TCP 端点。例如：unix:///var/run/dockershim.sock、 npipe:////./pipe/dockershim。


--cgroup-driver string     默认值：cgroupfs
kubelet 用来操作本机 cgroup 时使用的驱动程序。支持的选项包括 cgroupfs 和 systemd。已弃用：应在 --config 所给的配置文件中进行设置。
```

  
  

**kube-proxy**

  

Kubernetes 网络代理在每个节点上运行。网络代理反映了每个节点上 Kubernetes API 中定义的服务，并且可以执行简单的 TCP、UDP 和 SCTP 流转发，或者在一组后端进行 循环 TCP、UDP 和 SCTP 转发。当前可通过 Docker-links-compatible 环境变量找到服务集群 IP 和端口， 这些环境变量指定了服务代理打开的端口。有一个可选的插件，可以为这些集群 IP 提供集群 DNS。用户必须使用 apiserver API 创建服务才能配置代理。

  
  

```
cat >  /usr/lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes
After=network.target


[Service]
ExecStart=/usr/local/bin/kube-proxy \
--config=/etc/kubernetes/kube-proxy.yaml \
--v=2


Restart=always
RestartSec=10s


[Install]
WantedBy=multi-user.target


EOF


--config string
配置文件的路径。
```
  

https://www.oiox.cn/

https://www.chenby.cn/

https://cby-chen.github.io/

https://weibo.com/u/5982474121

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

https://www.jianshu.com/u/0f894314ae2c

https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/

CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客、全网可搜《小陈运维》
