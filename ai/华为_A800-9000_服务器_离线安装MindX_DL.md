---
layout: post
cid: 12
title: 华为 A800-9000 服务器 离线安装MindX DL
slug: 12
date: 2021/12/30 17:01:10
updated: 2021/12/30 17:01:10
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


    MindX DL（昇腾深度学习组件）是支持 Atlas 800 训练服务器、Atlas 800 推理服务器的深度学习组件参考设计，提供昇腾 AI 处理器资源管理和监控、昇腾 AI 处理器优化调度、分布式训练集合通信配置生成等基础功能，快速使能合作伙伴进行深度学习平台开发。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16e4b9f7084840538d445b2d1d907fa5~tplv-k3u1fbpfcp-zoom-1.image)

  

    操作系统使用的是Ubuntu-1804，CPU是华为自研ARM架构。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d03504939ec94b649d970864ee829a47~tplv-k3u1fbpfcp-zoom-1.image)

  

  

一、安装前准备

  

1.  配置apt网络源  
    
      
    

```
hello@ubuntu:/etc/apt$ sudo cp sources.list~ sources.list

hello@ubuntu:/etc/apt$ cat sources.list

# 

# deb cdrom:[Ubuntu-Server 18.04.5 LTS _Bionic Beaver_ - Release arm64 (20200810)]/ bionic main restricted

#deb cdrom:[Ubuntu-Server 18.04.5 LTS _Bionic Beaver_ - Release arm64 (20200810)]/ bionic main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic main restricted
# deb-src http://cn.ports.ubuntu.com/ubuntu-ports/ bionic main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted
# deb-src http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic universe
# deb-src http://cn.ports.ubuntu.com/ubuntu-ports/ bionic universe
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-updates universe
# deb-src http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu 
## team, and may not be under a free licence. Please satisfy yourself as to 
## your rights to use the software. Also, please note that software in 
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic multiverse
# deb-src http://cn.ports.ubuntu.com/ubuntu-ports/ bionic multiverse
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-updates multiverse
# deb-src http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-backports main restricted universe multiverse
# deb-src http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu bionic partner
# deb-src http://archive.canonical.com/ubuntu bionic partner

deb http://ports.ubuntu.com/ubuntu-ports bionic-security main restricted
# deb-src http://ports.ubuntu.com/ubuntu-ports bionic-security main restricted
deb http://ports.ubuntu.com/ubuntu-ports bionic-security universe
# deb-src http://ports.ubuntu.com/ubuntu-ports bionic-security universe
deb http://ports.ubuntu.com/ubuntu-ports bionic-security multiverse
# deb-src http://ports.ubuntu.com/ubuntu-ports bionic-security multiverse

```

  

    2.配置kubernetes网络源

  

```
root@ubuntu:~/123/offline-pkg-arm64# cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
> deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
> EOF

```

  

    3.创建目录并下载基础包  

  

```
root@ubuntu:~/123# mkdir offline-pkg-arm64
root@ubuntu:~/123# cd offline-pkg-arm64/
root@ubuntu:~/123/offline-pkg-arm64# sudo apt update
root@ubuntu:~/123/offline-pkg-arm64# apt-get download conntrack cri-tools haveged keyutils libhavege1 libltdl7 libnfsidmap2 libtirpc-dev libtirpc1 nfs-common nfs-kernel-server rpcbind socat sshpass
root@ubuntu:~/123/offline-pkg-arm64# wget --no-check-certificate https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/arm64/docker-ce_18.06.3~ce~3-0~ubuntu_arm64.deb
root@ubuntu:~/123/offline-pkg-arm64# apt-get download kubelet=1.17.3-00 kubeadm=1.17.3-00 kubectl=1.17.3-00 kubernetes-cni=0.8.6-00
```

  

  

    4.下载docker镜像并导出保存  

  

```
root@ubuntu:~/123# mkdir docker_images
root@ubuntu:~/123# cd docker_images/
root@ubuntu:~/123/docker_images# docker pull calico/node:v3.11.3
root@ubuntu:~/123/docker_images# docker save -o calico-node_arm64.tar.gz calico/node:v3.11.3
root@ubuntu:~/123/docker_images# docker pull calico/pod2daemon-flexvol:v3.11.3
root@ubuntu:~/123/docker_images# docker save -o calico-pod2daemon-flexvol_arm64.tar.gz calico/pod2daemon-flexvol:v3.11.3
root@ubuntu:~/123/docker_images# docker pull calico/cni:v3.11.3
root@ubuntu:~/123/docker_images# docker save -o calico-cni_arm64.tar.gz calico/cni:v3.11.3
root@ubuntu:~/123/docker_images# docker pull calico/kube-controllers:v3.11.3
root@ubuntu:~/123/docker_images# docker save -o calico-kube-controllers_arm64.tar.gz calico/kube-controllers:v3.11.3
root@ubuntu:~/123/docker_images# docker pull coredns/coredns:1.6.5
root@ubuntu:~/123/docker_images# docker save -o coredns_arm64.tar.gz coredns/coredns:1.6.5
root@ubuntu:~/123/docker_images# docker pull cruse/etcd-arm64:3.4.3-0
root@ubuntu:~/123/docker_images# docker save -o etcd_arm64.tar.gz cruse/etcd-arm64:3.4.3-0
root@ubuntu:~/123/docker_images# docker pull cruse/kube-apiserver-arm64:v1.17.3
root@ubuntu:~/123/docker_images# docker save -o kube-apiserver_arm64.tar.gz cruse/kube-apiserver-arm64:v1.17.3
root@ubuntu:~/123/docker_images# docker pull cruse/kube-controller-manager-arm64:v1.17.3
root@ubuntu:~/123/docker_images# docker save -o kube-controller-manager_arm64.tar.gz  cruse/kube-controller-manager-arm64:v1.17.3
root@ubuntu:~/123/docker_images# docker pull cruse/kube-proxy-arm64:v1.17.3-beta.0
root@ubuntu:~/123/docker_images# docker save -o kube-proxy_arm64.tar.gz cruse/kube-proxy-arm64:v1.17.3-beta.0
root@ubuntu:~/123/docker_images# docker pull cruse/kube-scheduler-arm64:v1.17.3-beta.0
root@ubuntu:~/123/docker_images# docker save -o kube-scheduler_arm64.tar.gz cruse/kube-scheduler-arm64:v1.17.3-beta.0
root@ubuntu:~/123/docker_images# docker pull cruse/pause-arm64:3.1
root@ubuntu:~/123/docker_images# docker save -o pause_arm64.tar.gz cruse/pause-arm64:3.1
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# docker login -u 15648907522 -p RtZOXgmpYAQd5cj93uFCabNXUWB7wOftGw4pFdcal4XZH4bf06hvFxTOrYtr1nRao ascendhub.huawei.com
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# docker pull ascendhub.huawei.com/public-ascendhub/vc-controller-manager_arm64:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker pull ascendhub.huawei.com/public-ascendhub/vc-scheduler_arm64:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker pull ascendhub.huawei.com/public-ascendhub/vc-webhook-manager_arm64:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker pull ascendhub.huawei.com/public-ascendhub/vc-webhook-manager-base_arm64:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker pull ascendhub.huawei.com/public-ascendhub/hccl-controller_arm64:v20.2.0
root@ubuntu:~/123/docker_images# docker pull ascendhub.huawei.com/public-ascendhub/ascend-k8sdeviceplugin_arm64:v20.2.0
root@ubuntu:~/123/docker_images# docker pull ascendhub.huawei.com/public-ascendhub/cadvisor_arm64:v0.34.0-r40
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# docker tag ascendhub.huawei.com/public-ascendhub/vc-controller-manager_arm64:v1.0.1-r40 volcanosh/vc-controller-manager:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker tag ascendhub.huawei.com/public-ascendhub/vc-scheduler_arm64:v1.0.1-r40 volcanosh/vc-scheduler:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker tag ascendhub.huawei.com/public-ascendhub/vc-webhook-manager_arm64:v1.0.1-r40 volcanosh/vc-webhook-manager:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker tag ascendhub.huawei.com/public-ascendhub/vc-webhook-manager-base_arm64:v1.0.1-r40 volcanosh/vc-webhook-manager-base:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker tag ascendhub.huawei.com/public-ascendhub/hccl-controller_arm64:v20.2.0 hccl-controller:v20.2.0
root@ubuntu:~/123/docker_images# docker tag ascendhub.huawei.com/public-ascendhub/ascend-k8sdeviceplugin_arm64:v20.2.0 ascend-k8sdeviceplugin:v20.2.0
root@ubuntu:~/123/docker_images# docker tag ascendhub.huawei.com/public-ascendhub/cadvisor_arm64:v0.34.0-r40 google/cadvisor:v0.34.0-r40
root@ubuntu:~/123/docker_images# docker rmi ascendhub.huawei.com/public-ascendhub/vc-controller-manager_arm64:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker rmi ascendhub.huawei.com/public-ascendhub/vc-scheduler_arm64:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker rmi ascendhub.huawei.com/public-ascendhub/vc-webhook-manager_arm64:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker rmi ascendhub.huawei.com/public-ascendhub/vc-webhook-manager-base_arm64:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker rmi ascendhub.huawei.com/public-ascendhub/hccl-controller_arm64:v20.2.0
root@ubuntu:~/123/docker_images# docker rmi ascendhub.huawei.com/public-ascendhub/ascend-k8sdeviceplugin_arm64:v20.2.0
root@ubuntu:~/123/docker_images# docker rmi ascendhub.huawei.com/public-ascendhub/cadvisor_arm64:v0.34.0-r40
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# 
root@ubuntu:~/123/docker_images# docker save -o Ascend-K8sDevicePlugin-v20.2.0-arm64-Docker.tar.gz ascend-k8sdeviceplugin:v20.2.0
root@ubuntu:~/123/docker_images# docker save -o hccl-controller-v20.2.0-arm64.tar.gz hccl-controller:v20.2.0
root@ubuntu:~/123/docker_images# docker save -o huawei-cadvisor-v0.34.0-r40-arm64.tar.gz google/cadvisor:v0.34.0-r40
root@ubuntu:~/123/docker_images# docker save -o vc-controller-manager-v1.0.1-r40-arm64.tar.gz volcanosh/vc-controller-manager:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker save -o vc-scheduler-v1.0.1-r40-arm64.tar.gz volcanosh/vc-scheduler:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker save -o vc-webhook-manager-base-v1.0.1-r40-arm64.tar.gz volcanosh/vc-webhook-manager-base:v1.0.1-r40
root@ubuntu:~/123/docker_images# docker save -o vc-webhook-manager-v1.0.1-r40-arm64.tar.gz volcanosh/vc-webhook-manager:v1.0.1-r40

```

  

**注\* 其中部分镜像是需要在华为hub里面进行获取权限后进行下载**  

https://support.huaweicloud.com/usermanual-mindxdl202/atlasmindx_03_0047.html

  

    5.完成后的目录  

  

```
root@ubuntu:~/123# tree
.
├── docker_images
│   ├── Ascend-K8sDevicePlugin-v20.2.0-arm64-Docker.tar.gz
│   ├── calico-cni_arm64.tar.gz
│   ├── calico-kube-controllers_arm64.tar.gz
│   ├── calico-node_arm64.tar.gz
│   ├── calico-pod2daemon-flexvol_arm64.tar.gz
│   ├── coredns_arm64.tar.gz
│   ├── etcd_arm64.tar.gz
│   ├── hccl-controller-v20.2.0-arm64.tar.gz
│   ├── huawei-cadvisor-v0.34.0-r40-arm64.tar.gz
│   ├── kube-apiserver_arm64.tar.gz
│   ├── kube-controller-manager_arm64.tar.gz
│   ├── kube-proxy_arm64.tar.gz
│   ├── kube-scheduler_arm64.tar.gz
│   ├── pause_arm64.tar.gz
│   ├── vc-controller-manager-v1.0.1-r40-arm64.tar.gz
│   ├── vc-scheduler-v1.0.1-r40-arm64.tar.gz
│   ├── vc-webhook-manager-base-v1.0.1-r40-arm64.tar.gz
│   └── vc-webhook-manager-v1.0.1-r40-arm64.tar.gz
├── offline-pkg-arm64
│   ├── conntrack_1%3a1.4.4+snapshot20161117-6ubuntu2_arm64.deb
│   ├── cri-tools_1.13.0-01_arm64.deb
│   ├── docker-ce_18.06.3~ce~3-0~ubuntu_arm64.deb
│   ├── haveged_1.9.1-6_arm64.deb
│   ├── keyutils_1.5.9-9.2ubuntu2_arm64.deb
│   ├── kubeadm_1.17.3-00_arm64.deb
│   ├── kubectl_1.17.3-00_arm64.deb
│   ├── kubelet_1.17.3-00_arm64.deb
│   ├── kubernetes-cni_0.8.6-00_arm64.deb
│   ├── libhavege1_1.9.1-6_arm64.deb
│   ├── libltdl7_2.4.6-2_arm64.deb
│   ├── libnfsidmap2_0.25-5.1_arm64.deb
│   ├── libtirpc1_0.2.5-1.2ubuntu0.1_arm64.deb
│   ├── libtirpc-dev_0.2.5-1.2ubuntu0.1_arm64.deb
│   ├── nfs-common_1%3a1.3.4-2.1ubuntu5.5_arm64.deb
│   ├── nfs-kernel-server_1%3a1.3.4-2.1ubuntu5.5_arm64.deb
│   ├── rpcbind_0.2.3-0.6ubuntu0.18.04.4_arm64.deb
│   ├── socat_1.7.3.2-2ubuntu2_arm64.deb
│   └── sshpass_1.06-1_arm64.deb
├── offline-pkg-arm64.zip
└── yamls
    ├── ascendplugin-310-v20.2.0.yaml
    ├── ascendplugin-volcano-v20.2.0.yaml
    ├── cadvisor-v0.34.0-r40.yaml
    ├── calico.yaml
    ├── hccl-controller-v20.2.0.yaml
    ├── npu-exporter-v20.2.0.yaml
    └── volcano-v1.0.1-r40.yaml

3 directories, 46 files
root@ubuntu:~/123#
```

  

注\* 其中yamls文件在下方链接中下载  

https://gitee.com/ascend/mindxdl-deploy/tree/20201230-V20.2.0/

  

  

    6.配置免密登陆  

  

```
root@ubuntu:~# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:07dTbsAycQqT2w7HdCwjIyJig5T20FQ/eHZGxWg7pbY root@ubuntu
The key's randomart image is:
+---[RSA 2048]----+
| .+...   .+.     |
|o+ .  o .+ +     |
|+o+ ...=BoO +    |
|...o .o.+/ O     |
|        S @ + .  |
|         E + =   |
|          . o o  |
|             o   |
|                 |
+----[SHA256]-----+
root@ubuntu:~# 
root@ubuntu:~# ssh-copy-id -i 127.0.0.1
```

  

    7.配置安装ansible  

  

```
root@ubuntu:~# 
root@ubuntu:~# apt install ansible
root@ubuntu:~# vim /etc/ansible/hosts

#配置内容如下
[all:vars]
# default shared directory, you can change it as yours
nfs_shared_dir=/data/atlas_dls

# NFS service IP
nfs_service_ip=192.168.1.110

# Master IP
master_ip=192.168.1.110

# dls install package dir
dls_root_dir=/root/123

# set proxy
proxy=""

# Command for logging in to the Asend hub
ascendhub_login_command="login_command"

# Generally, you do not need to change the value or delete it.
ascendhub_prefix="ascendhub.huawei.com/public-ascendhub"

# versions
deviceplugin_version="v20.2.0"
cadvisor_version="v0.34.0-r40"
volcano_version="v1.0.1-r40"
hccl_version="v20.2.0"

[nfs_server]
ubuntu ansible_host=192.168.1.110 ansible_ssh_user="root" ansible_ssh_pass="123123"

[localnode]
ubuntu ansible_host=192.168.1.110 ansible_ssh_user="root" ansible_ssh_pass="123123"

[training_node]
ubuntu ansible_host=192.168.1.110 ansible_ssh_user="root" ansible_ssh_pass="123123"

[inference_node]

[A300T_node]

[arm]
ubuntu ansible_host=192.168.1.110 ansible_ssh_user="root" ansible_ssh_pass="123123"

[x86]

[workers:children]
training_node
inference_node
A300T_node


root@ubuntu:~/mindxdl/deploy/offline/steps# vim /etc/ansible/ansible.cfg
log_path = /var/log/ansible.log
host_key_checking = False
deprecation_warnings = False

```

  

注\* 参数说明，请根据实际写入：

  

```
nfs-host-ip：NFS节点服务器IP地址，即服务器IP地址，如果不安装NFS可设置为空字符串，如：""。
master-host-ip：管理节点服务器IP地址，即服务器IP地址。
install_dir：基础软件包、镜像包和yamls文件夹的上传目录。
proxy_address：代理地址，请根据实际情况配置，如果不需要代理，设置为空字符串，如：""。
login_command：从Ascend Hub中心获取镜像需要使用的登录命令，仅在线安装需要配置，如："docker login -u xxxxxx@xxxxxx -p xxxxxxxx ascendhub.huawei.com"，注意不要遗漏命令前后的引号，获取方式请参见获取MindX DL镜像中1~2。离线安装可设置为空字符串，如：""。
single-node-host-name：请使用单节点主机名，可通过hostname命令查看。
IP：服务器IP地址。
username：登录服务器的用户名。建议使用root用户，避免权限不足。
passwd：登录服务器的用户密码。

```

  

二、一键安装  

  

```
root@ubuntu:~/sshpass# apt install sshpass
root@ubuntu:~/mindxdl/deploy/offline/steps# dos2unix *
root@ubuntu:~/mindxdl/deploy/offline/steps# chmod 500 entry.sh
root@ubuntu:~/mindxdl/deploy/offline/steps# bash -x entry.sh
```

  

三、安装后进行验证  

  

    1.docker信息查看  

  

```
root@ubuntu:~# docker info
Containers: 35
 Running: 30
 Paused: 0
 Stopped: 5
Images: 18
Server Version: 18.06.3-ce
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: systemd
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
Swarm: inactive
Runtimes: ascend runc
Default Runtime: ascend
Init Binary: docker-init
containerd version: 468a545b9edcd5932818eb9de8e72413e616e86e
runc version: a592beb5bc4c4092b1b1bac971afed27687340c5
init version: fec3683
Security Options:
 apparmor
 seccomp
  Profile: default
Kernel Version: 4.15.0-112-generic
Operating System: Ubuntu 18.04.5 LTS
OSType: linux
Architecture: aarch64
CPUs: 192
Total Memory: 503.6GiB
Name: ubuntu
ID: MUTU:QOYU:2P6F:P2QB:4JKZ:QNKE:PPMQ:PQLL:3PDG:QEYU:LMDK:KNMF
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 docker.mirrors.ustc.edu.cn
 127.0.0.0/8
Registry Mirrors:
 https://dockerhub.azk8s.cn/
 https://docker.mirrors.ustc.edu.cn/
 http://hub-mirror.c.163.com/
Live Restore Enabled: false

WARNING: No swap limit support
```

  

    2. kubectl的pod信息查看  

  

```
root@ubuntu:~# kubectl get pod --all-namespaces
NAMESPACE        NAME                                       READY   STATUS      RESTARTS   AGE
cadvisor         cadvisor-nsn4r                             1/1     Running     0          5m23s
default          hccl-controller-645bb466f-5fqq6            1/1     Running     0          5m34s
kube-system      ascend-device-plugin-daemonset-vxj8s       1/1     Running     0          5m23s
kube-system      calico-kube-controllers-8464785d6b-bnjdn   1/1     Running     0          5m50s
kube-system      calico-node-blshl                          1/1     Running     0          5m51s
kube-system      coredns-6955765f44-5jr59                   1/1     Running     0          5m50s
kube-system      coredns-6955765f44-wbzvz                   1/1     Running     0          5m50s
kube-system      etcd-ubuntu                                1/1     Running     0          5m43s
kube-system      kube-apiserver-ubuntu                      1/1     Running     0          5m43s
kube-system      kube-controller-manager-ubuntu             1/1     Running     0          5m43s
kube-system      kube-proxy-b78fm                           1/1     Running     0          5m51s
kube-system      kube-scheduler-ubuntu                      1/1     Running     0          5m43s
volcano-system   volcano-admission-74776688c8-g9p9q         1/1     Running     0          5m31s
volcano-system   volcano-admission-init-sbktn               0/1     Completed   0          5m31s
volcano-system   volcano-controllers-6786db54f-vn797        1/1     Running     0          5m31s
volcano-system   volcano-scheduler-844f9b547b-xxjm7         1/1     Running     0          5m31s
root@ubuntu:~# 
root@ubuntu:~# kubectl describe node ubuntu
Name:               ubuntu
Roles:              master,worker
Labels:             accelerator=huawei-Ascend910
                    beta.kubernetes.io/arch=arm64
                    beta.kubernetes.io/os=linux
                    host-arch=huawei-arm
                    kubernetes.io/arch=arm64
                    kubernetes.io/hostname=ubuntu
                    kubernetes.io/os=linux
                    masterselector=dls-master-node
                    node-role.kubernetes.io/master=
                    node-role.kubernetes.io/worker=worker
                    workerselector=dls-worker-node
Annotations:        huawei.com/Ascend910: Ascend910-1,Ascend910-2,Ascend910-3,Ascend910-0
                    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 192.168.1.110/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 10.30.243.192
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Thu, 05 Aug 2021 16:34:33 +0800
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  ubuntu
  AcquireTime:     <unset>
  RenewTime:       Thu, 05 Aug 2021 16:41:29 +0800
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Thu, 05 Aug 2021 16:35:06 +0800   Thu, 05 Aug 2021 16:35:06 +0800   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Thu, 05 Aug 2021 16:40:30 +0800   Thu, 05 Aug 2021 16:34:27 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Thu, 05 Aug 2021 16:40:30 +0800   Thu, 05 Aug 2021 16:34:27 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Thu, 05 Aug 2021 16:40:30 +0800   Thu, 05 Aug 2021 16:34:27 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Thu, 05 Aug 2021 16:40:30 +0800   Thu, 05 Aug 2021 16:35:19 +0800   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  192.168.1.110
  Hostname:    ubuntu
Capacity:
  cpu:                   192
  ephemeral-storage:     920422204Ki
  huawei.com/Ascend910:  4
  hugepages-2Mi:         0
  memory:                528101392Ki
  pods:                  110
Allocatable:
  cpu:                   192
  ephemeral-storage:     848261101802
  huawei.com/Ascend910:  4
  hugepages-2Mi:         0
  memory:                527998992Ki
  pods:                  110
System Info:
  Machine ID:                 3996e745414f461b9e0e990f6d0b597e
  System UUID:                CD56756C-607E-BD02-EB11-5292EAFB068C
  Boot ID:                    adb96127-7fdc-4d84-8867-a13005f9b535
  Kernel Version:             4.15.0-112-generic
  OS Image:                   Ubuntu 18.04.5 LTS
  Operating System:           linux
  Architecture:               arm64
  Container Runtime Version:  docker://18.6.3
  Kubelet Version:            v1.17.3
  Kube-Proxy Version:         v1.17.3
PodCIDR:                      10.30.0.0/24
PodCIDRs:                     10.30.0.0/24
Non-terminated Pods:          (15 in total)
  Namespace                   Name                                        CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                   ----                                        ------------  ----------  ---------------  -------------  ---
  cadvisor                    cadvisor-nsn4r                              500m (0%)     1 (0%)      300Mi (0%)       2000Mi (0%)    6m17s
  default                     hccl-controller-645bb466f-5fqq6             500m (0%)     500m (0%)   300Mi (0%)       300Mi (0%)     6m28s
  kube-system                 ascend-device-plugin-daemonset-vxj8s        500m (0%)     500m (0%)   500Mi (0%)       500Mi (0%)     6m17s
  kube-system                 calico-kube-controllers-8464785d6b-bnjdn    0 (0%)        0 (0%)      0 (0%)           0 (0%)         6m44s
  kube-system                 calico-node-blshl                           250m (0%)     0 (0%)      0 (0%)           0 (0%)         6m45s
  kube-system                 coredns-6955765f44-5jr59                    100m (0%)     0 (0%)      70Mi (0%)        170Mi (0%)     6m44s
  kube-system                 coredns-6955765f44-wbzvz                    100m (0%)     0 (0%)      70Mi (0%)        170Mi (0%)     6m44s
  kube-system                 etcd-ubuntu                                 0 (0%)        0 (0%)      0 (0%)           0 (0%)         6m37s
  kube-system                 kube-apiserver-ubuntu                       250m (0%)     0 (0%)      0 (0%)           0 (0%)         6m37s
  kube-system                 kube-controller-manager-ubuntu              200m (0%)     0 (0%)      0 (0%)           0 (0%)         6m37s
  kube-system                 kube-proxy-b78fm                            0 (0%)        0 (0%)      0 (0%)           0 (0%)         6m45s
  kube-system                 kube-scheduler-ubuntu                       100m (0%)     0 (0%)      0 (0%)           0 (0%)         6m37s
  volcano-system              volcano-admission-74776688c8-g9p9q          500m (0%)     500m (0%)   300Mi (0%)       300Mi (0%)     6m25s
  volcano-system              volcano-controllers-6786db54f-vn797         500m (0%)     500m (0%)   300Mi (0%)       300Mi (0%)     6m25s
  volcano-system              volcano-scheduler-844f9b547b-xxjm7          500m (0%)     500m (0%)   300Mi (0%)       300Mi (0%)     6m25s
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource              Requests     Limits
  --------              --------     ------
  cpu                   4 (2%)       3500m (1%)
  memory                2140Mi (0%)  4040Mi (0%)
  ephemeral-storage     0 (0%)       0 (0%)
  huawei.com/Ascend910  0            0
Events:
  Type    Reason                   Age                    From                Message
  ----    ------                   ----                   ----                -------
  Normal  NodeHasSufficientMemory  7m10s (x8 over 7m11s)  kubelet, ubuntu     Node ubuntu status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    7m10s (x7 over 7m11s)  kubelet, ubuntu     Node ubuntu status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     7m10s (x6 over 7m11s)  kubelet, ubuntu     Node ubuntu status is now: NodeHasSufficientPID
  Normal  Starting                 6m37s                  kubelet, ubuntu     Starting kubelet.
  Normal  NodeHasSufficientMemory  6m37s                  kubelet, ubuntu     Node ubuntu status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    6m37s                  kubelet, ubuntu     Node ubuntu status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     6m37s                  kubelet, ubuntu     Node ubuntu status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  6m37s                  kubelet, ubuntu     Updated Node Allocatable limit across pods
  Normal  Starting                 6m33s                  kube-proxy, ubuntu  Starting kube-proxy.
  Normal  NodeReady                6m17s                  kubelet, ubuntu     Node ubuntu status is now: NodeReady
root@ubuntu:~#
```

  

注\* 再此信息中可以看到CPU和加速卡的信息

  

```
Capacity:
  cpu:                   192
  ephemeral-storage:     920422204Ki
  huawei.com/Ascend910:  4
  hugepages-2Mi:         0
  memory:                528101392Ki
  pods:                  110
Allocatable:
  cpu:                   192
  ephemeral-storage:     848261101802
  huawei.com/Ascend910:  4
  hugepages-2Mi:         0
  memory:                527998992Ki
  pods:                  110
```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5192ab3e43e640fd8d935795ea96b768~tplv-k3u1fbpfcp-zoom-1.image)

  

  

**详情可以查看华为官方文档：  
**

**https://support.huaweicloud.com/mindxdl201/**

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce668ce68dbc4666a4b7ad3a8f24d97e~tplv-k3u1fbpfcp-zoom-1.image)

  

  

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