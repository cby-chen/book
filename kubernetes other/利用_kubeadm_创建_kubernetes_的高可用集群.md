---
layout: post
cid: 3
title: 利用 kubeadm 创建 kubernetes 的高可用集群
slug: 3
date: 2021/12/30 16:56:25
updated: 2021/12/30 16:56:25
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


**引言**：

kubeadm提供了两种不同的高可用方案。

  

    堆叠方案：etcd服务和控制平面被部署在同样的节点中，对基础设施的要求较低，对故障的应对能力也较低

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f5cf8d9a83145b8900eb99ae15498d2~tplv-k3u1fbpfcp-zoom-1.image)

堆叠方案

    最小三个Master（也称工作平面），因为Etcd使用RAFT算法选主，节点数量需要为2n+1个。

  

    外置etcd方案：etcd和控制平面被分离，需要更多的硬件，也有更好的保障能力

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eba922ac2e764f6bb181581e87ebf1c3~tplv-k3u1fbpfcp-zoom-1.image)

  

外置etcd方案

  

  

**一、资源环境**

  


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41a7d4791c9d42f29a380bd2e61f31b0~tplv-k3u1fbpfcp-watermark.image)

  

    下面采用的是kubeadm的堆叠方案搭建k8s集群，也就是说如果3台Master宕了2台时，集群将不可用，可能收到如下错误信息"Error from server: etcdserver: request timed out"。

  

**二、系统设置（所有主机）**

   设置主机名  

```
hostnamectl set-hostname master-\*
hostnamectl set-hostname node-\*
```

  

    设置静态IP  

```
\[root@localhost ~\]# vim /etc/sysconfig/network-scripts/ifcfg-ens18 
\[root@localhost ~\]# 
\[root@localhost ~\]# 
\[root@localhost ~\]# 
\[root@localhost ~\]# cat /etc/sysconfig/network-scripts/ifcfg-ens18
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
IPADDR=10.0.0.11
NETMASK=255.0.0.0
GATEWAY=10.0.0.1
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens18
UUID=555fe27b-19eb-4958-aca7-c9c71365432f
DEVICE=ens18
ONBOOT=yes
\[root@localhost ~\]# reboot
```

  

    配置主机名  

```
\[root@master-01 ~\]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.0.0.11 master-01
10.0.0.12 master-02
10.0.0.13 master-03
10.0.0.14 node-01
10.0.0.15 master-01
10.0.0.16 master-01
```

  

    安装依赖  

```
\[root@node-01 ~\]# yum update -y
Repository AppStream is listed more than once in the configuration
Repository extras is listed more than once in the configuration
Repository PowerTools is listed more than once in the configuration
Repository centosplus is listed more than once in the configuration
Last metadata expiration check: 0:19:42 ago on Sat 28 Nov 2020 04:25:04 PM CST.
Dependencies resolved.
Nothing to do.
Complete!

\[root@node-01 ~\]# yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp bind-utils
...
```

  

    关闭防火墙、swap、selinux

```
\[root@master-01 ~\]# systemctl stop firewalld && systemctl disable firewalld
Removed /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
\[root@master-01 ~\]# swapoff -a
\[root@master-01 ~\]# iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT
\[root@master-01 ~\]# sed -i '/swap/s/^\\(.\*\\)$/#\\1/g' /etc/fstab
\[root@master-01 ~\]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Mon Nov 23 08:19:33 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=46ea6159-eda5-4931-ae11-73095cf284c1 /boot                   ext4    defaults        1 2
#/dev/mapper/cl-swap     swap                    swap    defaults        0 0
\[root@master-01 ~\]# setenforce 0
\[root@master-01 ~\]# vim /etc/sysconfig/selinux
\[root@master-01 ~\]# cat /etc/sysconfig/selinux

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

```

  

高新科技园

广东省深圳市南山区科文路4附近

   系统参数设置  

```
\# 开启ipvs模块
\[root@master-01 ~\]#  cat > /etc/sysconfig/modules/ipvs.modules <<EOF
> #!/bin/bash
> modprobe -- ip_vs
> modprobe -- ip_vs_rr
> modprobe -- ip_vs_wrr
> modprobe -- ip_vs_sh
> modprobe -- nf_conntrack_ipv4
> modprobe br_netfilter
> EOF

# 生效文件
\[root@master-01 ~\]# chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
ip_vs_sh               16384  0
ip_vs_wrr              16384  0
ip_vs_rr               16384  0
ip_vs                 172032  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_defrag_ipv6         20480  2 nf_conntrack_ipv6,ip_vs
nf_conntrack_ipv4      16384  1
nf_defrag_ipv4         16384  1 nf_conntrack_ipv4
nf_conntrack          155648  7 nf_conntrack_ipv6,nf_conntrack_ipv4,nf_nat,nft_ct,nf_nat_ipv6,nf_nat_ipv4,ip_vs
libcrc32c              16384  4 nf_conntrack,nf_nat,xfs,ip_vs

# 制作配置文件
\[root@master-01 ~\]# cat > /etc/sysctl.d/kubernetes.conf <<EOF
> net.bridge.bridge-nf-call-iptables=1
> net.bridge.bridge-nf-call-ip6tables=1
> net.ipv4.ip_forward=1
> net.ipv4.tcp_tw_recycle=1 # 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
> net.ipv4.tcp_keepalive_time=600 # 超过这个时间没有数据传输，就开始发送存活探测包
> net.ipv4.tcp_keepalive_intvl=15 # keepalive探测包的发送间隔
> net.ipv4.tcp_keepalive_probes=3 # 如果对方不予应答，探测包的发送次数
> vm.swappiness=0 # 禁止使用 swap 空间，只有当系统 OOM 时才允许使用它
> vm.overcommit_memory=1 # 不检查物理内存是否够用
> vm.panic_on_oom=0 # 开启 OOM
> fs.inotify.max_user_instances=8192
> fs.inotify.max_user_watches=1048576
> fs.file-max=52706963
> fs.nr_open=52706963
> net.ipv6.conf.all.disable_ipv6=1
> net.netfilter.nf_conntrack_max=2310720
> EOF

# 生效配置文件
\[root@master-01 ~\]# sysctl -p /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
sysctl: cannot stat /proc/sys/net/ipv4/tcp_tw_recycle: No such file or directory
net.ipv4.tcp_keepalive_time = 600 # 超过这个时间没有数据传输，就开始发送存活探测包
net.ipv4.tcp_keepalive_intvl = 15 # keepalive探测包的发送间隔
net.ipv4.tcp_keepalive_probes = 3 # 如果对方不予应答，探测包的发送次数
vm.swappiness = 0 # 禁止使用 swap 空间，只有当系统 OOM 时才允许使用它
vm.overcommit_memory = 1 # 不检查物理内存是否够用
vm.panic_on_oom = 0 # 开启 OOM
fs.inotify.max_user_instances = 8192
fs.inotify.max_user_watches = 1048576
fs.file-max = 52706963
fs.nr_open = 52706963
net.ipv6.conf.all.disable_ipv6 = 1
net.netfilter.nf_conntrack_max = 2310720

# 调整系统 TimeZone
\[root@master-01 ~\]# timedatectl set-timezone Asia/Shanghai

# 将当前的 UTC 时间写入硬件时钟
\[root@master-01 ~\]#  timedatectl set-local-rtc 0

# 重启依赖于系统时间的服务
\[root@master-01 ~\]# systemctl restart rsyslog && systemctl restart crond

# 关闭无关的服务
\[root@master-01 ~\]#  systemctl stop postfix && systemctl disable postfix
Failed to stop postfix.service: Unit postfix.service not loaded.
# 设置 rsyslogd 和 systemd journald
\[root@master-01 ~\]# mkdir /var/log/journal
\[root@master-01 ~\]# mkdir /etc/systemd/journald.conf.d

\[root@master-01 ~\]# cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
> \[Journal\]
> # 持久化保存到磁盘
> Storage=persistent
> 
> # 压缩历史日志
> Compress=yes
> 
> SyncIntervalSec=5m
> RateLimitInterval=30s
> RateLimitBurst=1000
> 
> # 最大占用空间 10G
> SystemMaxUse=10G
> 
> # 单日志文件最大 200M
> SystemMaxFileSize=200M
> 
> # 日志保存时间 2 周
> MaxRetentionSec=2week
> 
> # 不将日志转发到 syslog
> ForwardToSyslog=no
> EOF
\[root@master-01 ~\]# 

```

  

  

**三、安装docker  
**

```
\[root@master-02 ~\]# wget https://download.docker.com/linux/centos/8/x86_64/stable/Packages/containerd.io-1.3.7-3.1.el8.x86_64.rpm
--2020-11-28 17:47:12--  https://download.docker.com/linux/centos/8/x86_64/stable/Packages/containerd.io-1.3.7-3.1.el8.x86_64.rpm
Resolving download.docker.com (download.docker.com)... 99.84.206.7, 99.84.206.109, 99.84.206.25, ...
Connecting to download.docker.com (download.docker.com)|99.84.206.7|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 30388860 (29M) \[binary/octet-stream\]
Saving to: ‘containerd.io-1.3.7-3.1.el8.x86_64.rpm’

containerd.io-1.3.7-3.1 100%\[===============================>\]  28.98M   188KB/s    in 3m 15s  

2020-11-28 17:50:27 (153 KB/s) - ‘containerd.io-1.3.7-3.1.el8.x86_64.rpm’ saved \[30388860/30388860\]

\[root@node-02 ~\]# yum install ./containerd.io-1.3.7-3.1.el8.x86_64.rpm 
Repository AppStream is listed more than once in the configuration
Repository extras is listed more than once in the configuration
...

\[root@node-01 ~\]# sudo yum -y install docker-ce
Repository AppStream is listed more than once in the configuration
Repository extras is listed more than once in the configuration
Repository PowerTools is listed more than once in the configuration
...

\[root@master-01 ~\]# systemctl start docker && systemctl enable docker
```

  

**四、安装必要工具，在主节点安装kubectl即可，其他节点无需进行安装**kubectl****

```
\[root@master-01 ~\]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
\[kubernetes\]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

\[root@master-01 ~\]# yum install -y kubelet kubeadm kubectl
\[root@master-01 ~\]# systemctl enable kubelet && systemctl start kubelet
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e28f4bbb0fbc49dda0a53de9cf81adb3~tplv-k3u1fbpfcp-zoom-1.image)

**五、安装LVS和keepalived**  

  

```
\[root@vip ~\]# yum -y install keepalived

# 备份并编辑
\[root@vip ~\]# cp /etc/keepalived/keepalived.conf{,.back}
\[root@vip ~\]# vim /etc/keepalived/keepalived.conf 


\[root@vip ~\]# echo "" > /etc/keepalived/keepalived.conf
\[root@vip ~\]# vim /etc/keepalived/keepalived.conf 
\[root@vip ~\]# systemctl enable keepalived && service keepalived start
Created symlink /etc/systemd/system/multi-user.target.wants/keepalived.service → /usr/lib/systemd/system/keepalived.service.
Redirecting to /bin/systemctl start keepalived.service
\[root@vip ~\]# 
\[root@vip ~\]# 
\[root@vip ~\]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id keepalived-master
}

vrrp_instance vip_1 {
  state MASTER
  ! 注意这是网卡名称，使用ip a命令查看自己的局域网网卡名称
  interface ens18
  ! keepalived主备router_id必须一致
  virtual_router_id 88
  ! 优先级，keepalived主节点优先级要比备节点高
  priority 100
  advert_int 3
  ! 配置虚拟ip地址
  virtual_ipaddress {
    10.0.0.99
  }
}

virtual_server 10.0.0.99 6443 {
  delay_loop 6
  lb_algo rr
  lb_kind DR
  persistence_timeout 0
  protocol TCP
    
  real_server 10.0.0.12 6443 {
    weight 1
    TCP_CHECK {
      connect_timeout 10
      nb_get_retry 3
      delay_before_retry 3
      connect_port 6443
    }
  }
  real_server 10.0.0.13 6443 {
    weight 1
    TCP_CHECK {
      connect_timeout 10
      nb_get_retry 3
      delay_before_retry 3
      connect_port 6443
    }
  }
  real_server 10.0.0.11 6443 {
    weight 1
    TCP_CHECK {
      connect_timeout 10
      nb_get_retry 3
      delay_before_retry 3
      connect_port 6443
    }
  }
}

```

  

添加本地回环  

```
\[root@master-01 rs\]# vim /opt/rs/rs.sh 
\[root@master-01 rs\]# cat /opt/rs/rs.sh
#!/bin/bash
# 虚拟ip
vip=10.0.0.99
# 停止以前的lo:0
ifconfig lo:0 down
echo "1" > /proc/sys/net/ipv4/ip_forward
echo "0" > /proc/sys/net/ipv4/conf/all/arp_announce
# 启动一个回环地址并绑定给vip
ifconfig lo:0 $vip broadcast $vip netmask 255.0.0.0 up
route add -host $vip dev lo:0
echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
# ens33是主网卡名
echo "1" >/proc/sys/net/ipv4/conf/ens18/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/ens18/arp_announce

# 脚本不可以的话，使用命令吧
ifconfig lo:0 10.0.0.99 broadcast 10.0.0.99 netmask 255.255.255.255 up
route add -host 10.0.0.99 dev lo:0

# 设置开机自启
\[root@vip ~\]# echo '/opt/rs/rs.sh'  >> /etc/rc.d/rc.local
\[root@vip ~\]# chmod +x /etc/rc.d/rc.local

```

  

**keepalived** backup 设置

```
\[root@vip ~\]# 
\[root@vip ~\]# vim /etc/keepalived/keepalived.conf 
\[root@vip ~\]# systemctl enable keepalived && service keepalived start
Created symlink /etc/systemd/system/multi-user.target.wants/keepalived.service → /usr/lib/systemd/system/keepalived.service.
Redirecting to /bin/systemctl start keepalived.service
\[root@vip ~\]# 
\[root@vip ~\]# 
\[root@vip ~\]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   router_id keepalived-master
}

vrrp_instance vip_1 {
  state BACKUP
  ! 注意这是网卡名称，使用ip a命令查看自己的局域网网卡名称
  interface ens18
  ! keepalived主备router_id必须一致
  virtual_router_id 88
  ! 优先级，keepalived主节点优先级要比备节点高
  priority 99 
  advert_int 3
  ! 配置虚拟ip地址
  virtual_ipaddress {
    10.0.0.99
  }
}

virtual_server 10.0.0.99 6443 {
  delay_loop 6
  lb_algo rr
  lb_kind DR
  persistence_timeout 0
  protocol TCP
    
  real_server 10.0.0.12 6443 {
    weight 1
    TCP_CHECK {
      connect_timeout 10
      nb_get_retry 3
      delay_before_retry 3
      connect_port 6443
    }
  }
  real_server 10.0.0.13 6443 {
    weight 1
    TCP_CHECK {
      connect_timeout 10
      nb_get_retry 3
      delay_before_retry 3
      connect_port 6443
    }
  }
  real_server 10.0.0.11 6443 {
    weight 1
    TCP_CHECK {
      connect_timeout 10
      nb_get_retry 3
      delay_before_retry 3
      connect_port 6443
    }
  }
}

```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed6583742cb34b1fa59ab9b2dba73f27~tplv-k3u1fbpfcp-zoom-1.image)

**六、kubeadm搭建集群（区分节点）**  

  

master-01  

```
\[root@master-01 ~\]# cd /opt/kubernetes/
\[root@master-01 kubernetes\]# 
\[root@master-01 kubernetes\]# cat kubeadm-config.yaml 
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
# k8s的版本号，必须跟安装的Kubeadm版本等保持一致，否则启动报错
kubernetesVersion: v1.19.4
# docker镜像仓库地址，k8s.gcr.io需要翻墙才可以下载镜像，这里使用镜像服务器下载http://mirror.azure.cn/help/gcr-proxy-cache.html
# imageRepository: k8s.gcr.io/google_containers
# 集群名称
clusterName: kubernetes
# apiServer的集群访问地址，填写vip地址即可 #
controlPlaneEndpoint: "10.0.0.99:6443"
networking:
  # pod的网段
  podSubnet: 10.10.0.0/16
  serviceSubnet: 10.96.0.0/12
  dnsDomain: cluster.local
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
# kube-proxy模式指定为ipvs，需要提前在节点上安装ipvs的依赖并开启相关模块
mode: ipvs

# 拉去镜像
\[root@master-01 kubernetes\]# kubeadm config images pull
W1128 20:33:21.822265    4536 configset.go:348\] WARNING: kubeadm cannot validate component configs for API groups \[kubelet.config.k8s.io kubeproxy.config.k8s.io\]
\[config/images\] Pulled k8s.gcr.io/kube-apiserver:v1.19.4
\[config/images\] Pulled k8s.gcr.io/kube-controller-manager:v1.19.4
\[config/images\] Pulled k8s.gcr.io/kube-scheduler:v1.19.4
\[config/images\] Pulled k8s.gcr.io/kube-proxy:v1.19.4
\[config/images\] Pulled k8s.gcr.io/pause:3.2
\[config/images\] Pulled k8s.gcr.io/etcd:3.4.13-0

# 记得：
\[root@master-01 kubernetes\]# swapoff -a && kubeadm reset  && systemctl daemon-reload && systemctl restart kubelet  && iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

# 初始化
\[root@master-01 kubernetes\]# kubeadm init --config=kubeadm-config.yaml  --upload-certs
...
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f \[podnetwork\].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 10.0.0.99:6443 --token dtkoyq.8ciqez70nj1ysdix \\
    --discovery-token-ca-cert-hash sha256:f65ee972a9e9d0b8784f7db583a9cdf9865253459aa96a9b3529be2517570155 \\
    --control-plane --certificate-key 0dc20030f8dfdede8cbb3b0906eda1a3a140e91f7e6ebb6eac1ad02ac65389d3

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.0.99:6443 --token dtkoyq.8ciqez70nj1ysdix \\
    --discovery-token-ca-cert-hash sha256:f65ee972a9e9d0b8784f7db583a9cdf9865253459aa96a9b3529be2517570155 

# 安装网络组件
\[root@master-01 kubernetes\]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
\[root@master-01 kubernetes\]# 
\[root@master-01 kubernetes\]# 
\[root@master-01 kubernetes\]# 
\[root@master-01 kubernetes\]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                READY   STATUS     RESTARTS   AGE
kube-system   coredns-f9fd979d6-2hs76             0/1     Pending    0          5m18s
kube-system   coredns-f9fd979d6-5j4w8             0/1     Pending    0          5m18s
kube-system   etcd-master-01                      1/1     Running    0          5m29s
kube-system   kube-apiserver-master-01            1/1     Running    0          5m30s
kube-system   kube-controller-manager-master-01   1/1     Running    0          5m30s
kube-system   kube-flannel-ds-grhh6               0/1     Init:0/1   0          5s
kube-system   kube-proxy-pl74w                    1/1     Running    0          5m18s
kube-system   kube-scheduler-master-01            1/1     Running    0          5m30s

```

  

  

配置master-02和03

```
# 加入master组
\[root@master-03 ~\]#   kubeadm join 10.0.0.99:6443 --token dtkoyq.8ciqez70nj1ysdix     --discovery-token-ca-cert-hash sha256:f65ee972a9e9d0b8784f7db583a9cdf9865253459aa96a9b3529be2517570155     --control-plane --certificate-key 0dc20030f8dfdede8cbb3b0906eda1a3a140e91f7e6ebb6eac1ad02ac65389d3
...
等到pull镜像比较慢，耐心等待一下
...

# 加入完成后
\[root@master-03 ~\]#  mkdir -p $HOME/.kube
\[root@master-03 ~\]#  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
\[root@master-03 ~\]#  sudo chown $(id -u):$(id -g) $HOME/.kube/config
\[root@master-01 kubernetes\]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                READY   STATUS              RESTARTS   AGE
kube-system   coredns-f9fd979d6-2hs76             1/1     Running             0          45m
kube-system   coredns-f9fd979d6-5j4w8             1/1     Running             0          45m
kube-system   etcd-master-01                      1/1     Running             0          45m
kube-system   etcd-master-02                      1/1     Running             0          17m
kube-system   etcd-master-03                      0/1     Running             0          49s
kube-system   kube-apiserver-master-01            1/1     Running             0          45m
kube-system   kube-apiserver-master-02            1/1     Running             0          17m
kube-system   kube-apiserver-master-03            1/1     Running             0          51s
kube-system   kube-controller-manager-master-01   1/1     Running             1          45m
kube-system   kube-controller-manager-master-02   1/1     Running             0          17m
kube-system   kube-controller-manager-master-03   0/1     Running             0          51s
kube-system   kube-flannel-ds-76vcb               0/1     Init:0/1            0          17s
kube-system   kube-flannel-ds-8tqlh               1/1     Running             0          17m
kube-system   kube-flannel-ds-fq8kz               0/1     Init:0/1            0          17s
kube-system   kube-flannel-ds-grhh6               1/1     Running             0          40m
kube-system   kube-flannel-ds-hqj25               1/1     Running             0          52s
kube-system   kube-flannel-ds-rlg4z               0/1     Init:0/1            0          17s
kube-system   kube-proxy-8kf2r                    1/1     Running             0          17m
kube-system   kube-proxy-9n6p4                    0/1     ContainerCreating   0          17s
kube-system   kube-proxy-9xdrl                    1/1     Running             0          52s
kube-system   kube-proxy-pl74w                    1/1     Running             0          45m
kube-system   kube-proxy-vtm97                    0/1     ContainerCreating   0          17s
kube-system   kube-proxy-wdrpx                    0/1     ContainerCreating   0          17s
kube-system   kube-scheduler-master-01            1/1     Running             1          45m
kube-system   kube-scheduler-master-02            1/1     Running             0          17m
kube-system   kube-scheduler-master-03            0/1     Running             0          51s
```

  

node节点进行加入  

```
\[root@node-01 ~\]# kubeadm join 10.0.0.99:6443 --token dtkoyq.8ciqez70nj1ysdix \\
>     --discovery-token-ca-cert-hash sha256:f65ee972a9e9d0b8784f7db583a9cdf9865253459aa96a9b3529be2517570155 

\[root@node-01 ~\]#  mkdir -p $HOME/.kube
\[root@node-01 ~\]#  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
\[root@node-01 ~\]#  sudo chown $(id -u):$(id -g) $HOME/.kube/config

\[root@master-01 kubernetes\]# kubectl get nodes
NAME        STATUS   ROLES    AGE    VERSION
master-01   Ready    master   46m    v1.19.4
master-02   Ready    master   18m    v1.19.4
master-03   Ready    master   107s   v1.19.4
node-01     Ready    <none>   72s    v1.19.4
node-02     Ready    <none>   72s    v1.19.4
node-03     Ready    <none>   72s    v1.19.4

```

  

  

至此，高可用集群已部署完毕。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ea372e66c104e20b191f545339f151d~tplv-k3u1fbpfcp-zoom-1.image)

  

**七、部署Dashboard管理k8s集群**

  

```
\[root@master-01 ~\]# wget -P /etc/kubernetes/addons https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc1/aio/deploy/recommended.yaml && cd /etc/kubernetes/addons

\[root@master-01 addons\]# kubectl apply -f recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created


```

  

部署管理员  

```
\[root@master-01 addons\]# cat <<E0F > dashboard-adminuser.yaml
> apiVersion: v1
> kind: ServiceAccount
> metadata:
>   name: admin-user
>   namespace: kubernetes-dashboard
> 
> ---
> 
> apiVersion: rbac.authorization.k8s.io/v1
> kind: ClusterRoleBinding
> metadata:
>   name: admin-user
> roleRef:
>   apiGroup: rbac.authorization.k8s.io
>   kind: ClusterRole
>   name: cluster-admin
> subjects:
> - kind: ServiceAccount
>   name: admin-user
>   namespace: kubernetes-dashboard
> E0F
\[root@master-01 addons\]# kubectl apply -f dashboard-adminuser.yaml
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created


查看token
\[root@master-01 addons\]# kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}') | grep -E '^token' | awk '{print $2}'

```

  

  

  

  

高新科技园

广东省深圳市南山区科文路4附近