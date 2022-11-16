---
layout: post
cid: 163
title: 使用kubeadm快速启用一个集群
slug: 163
date: 2022/05/06 17:50:00
updated: 2022/06/01 15:14:34
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


使用kubeadm快速启用一个集群
=================

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c81f8393b0041668964c4ce4bece096~tplv-k3u1fbpfcp-zoom-1.image)

CentOS 配置YUM源
=============

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=kubernetes
baseurl=https://mirrors.ustc.edu.cn/kubernetes/yum/repos/kubernetes-el7-$basearch
enabled=1
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet


# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
sudo systemctl enable --now kubelet

```

Ubuntu 配置APT源
=============

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl

```

配置containerd
============

```
wget https://github.com/containerd/containerd/releases/download/v1.6.4/cri-containerd-cni-1.6.4-linux-amd64.tar.gz

#解压
tar -C / -xzf cri-containerd-cni-1.6.4-linux-amd64.tar.gz

#创建服务启动文件
cat > /etc/systemd/system/containerd.service <<EOF
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd
Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
EOF


mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml

sed -i "s#SystemdCgroup\ \=\ false#SystemdCgroup\ \=\ true#g" /etc/containerd/config.toml
sed -i "s#k8s.gcr.io#registry.cn-hangzhou.aliyuncs.com/chenby#g" /etc/containerd/config.toml

systemctl daemon-reload
systemctl enable --now containerd


```

配置基础环境
======

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
fs.may_detach_mounts = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720


net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384


net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
net.ipv6.conf.all.forwarding = 1
EOF


modprobe br_netfilter

sudo sysctl --system


hostnamectl set-hostname k8s-master01
hostnamectl set-hostname k8s-node01
hostnamectl set-hostname k8s-node02


sed -ri 's/.*swap.*/#&/' /etc/fstab
swapoff -a && sysctl -w vm.swappiness=0

cat /etc/fstab


cat > /etc/hosts <<EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.1.31 k8s-master01
192.168.1.32 k8s-node01
192.168.1.33 k8s-node01
EOF

```

初始化安装
=====

```
root@k8s-master01:~# kubeadm config images
missing subcommand; "images" is not meant to be run on its own
To see the stack trace of this error execute with --v=5 or higher
root@k8s-master01:~# kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.24.0
k8s.gcr.io/kube-controller-manager:v1.24.0
k8s.gcr.io/kube-scheduler:v1.24.0
k8s.gcr.io/kube-proxy:v1.24.0
k8s.gcr.io/pause:3.7
k8s.gcr.io/etcd:3.5.3-0
k8s.gcr.io/coredns/coredns:v1.8.6
root@k8s-master01:~# 
root@k8s-master01:~# 
root@k8s-master01:~# 
root@k8s-master01:~# kubeadm init  --image-repository registry.cn-hangzhou.aliyuncs.com/chenby
[init] Using Kubernetes version: v1.24.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.1.31]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master01 localhost] and IPs [192.168.1.31 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master01 localhost] and IPs [192.168.1.31 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 9.502219 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master01 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-master01 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: nsiavq.637f6t76cbtwckq9
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.31:6443 --token nsiavq.637f6t76cbtwckq9 \
        --discovery-token-ca-cert-hash sha256:963b47c1d46199eb28c2813c893fcd201cfaa32cfdfd521f6bc78a70c13878c4 
root@k8s-master01:~# 
root@k8s-master01:~#   mkdir -p $HOME/.kube
root@k8s-master01:~#   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@k8s-master01:~#   sudo chown $(id -u):$(id -g) $HOME/.kube/config
root@k8s-master01:~# 




root@k8s-node01:~# kubeadm join 192.168.1.31:6443 --token nsiavq.637f6t76cbtwckq9 \
>         --discovery-token-ca-cert-hash sha256:963b47c1d46199eb28c2813c893fcd201cfaa32cfdfd521f6bc78a70c13878c4
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

root@k8s-node01:~# 


root@k8s-node02:~# kubeadm join 192.168.1.31:6443 --token nsiavq.637f6t76cbtwckq9 \
>         --discovery-token-ca-cert-hash sha256:963b47c1d46199eb28c2813c893fcd201cfaa32cfdfd521f6bc78a70c13878c4
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

root@k8s-node02:~# 

```

验证
==

```
root@k8s-master01:~# kubectl  get node
NAME           STATUS   ROLES           AGE   VERSION
k8s-master01   Ready    control-plane   86s   v1.24.0
k8s-node01     Ready    <none>          42s   v1.24.0
k8s-node02     Ready    <none>          37s   v1.24.0
root@k8s-master01:~# 
root@k8s-master01:~# 
root@k8s-master01:~# kubectl  get pod -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-bc77466fc-jxkpv                1/1     Running   0          83s
kube-system   coredns-bc77466fc-nrc9l                1/1     Running   0          83s
kube-system   etcd-k8s-master01                      1/1     Running   0          87s
kube-system   kube-apiserver-k8s-master01            1/1     Running   0          89s
kube-system   kube-controller-manager-k8s-master01   1/1     Running   0          87s
kube-system   kube-proxy-2lgrn                       1/1     Running   0          83s
kube-system   kube-proxy-69p9r                       1/1     Running   0          47s
kube-system   kube-proxy-g58m2                       1/1     Running   0          42s
kube-system   kube-scheduler-k8s-master01            1/1     Running   0          87s
root@k8s-master01:~# 

```

> https://www.oiox.cn/
> 
> https://www.chenby.cn/
> 
> https://blog.oiox.cn/
> 
> https://cby-chen.github.io/
> 
> https://blog.csdn.net/qq_33921750
> 
> https://my.oschina.net/u/3981543
> 
> https://www.zhihu.com/people/chen-bu-yun-2
> 
> https://segmentfault.com/u/hppyvyv6/articles
> 
> https://juejin.cn/user/3315782802482007
> 
> https://cloud.tencent.com/developer/column/93230
> 
> https://www.jianshu.com/u/0f894314ae2c
> 
> https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/
> 
> CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、今日头条、个人博客、全网可搜《小陈运维》
> 
> 文章主要发布于微信：《Linux运维交流社区》
