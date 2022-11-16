---
layout: post
cid: 60
title: Docker容器中使用GPU
slug: 60
date: 2021/12/30 17:13:00
updated: 2022/03/25 15:42:19
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


**背景**

  

容器封装了应用程序的依赖项，以提供可重复和可靠的应用程序和服务执行，而无需整个虚拟机的开销。如果您曾经花了一天的时间为一个科学或 深度学习 应用程序提供一个包含大量软件包的服务器，或者已经花费数周的时间来确保您的应用程序可以在多个 linux 环境中构建和部署，那么 Docker 容器非常值得您花费时间。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/392fd0434e4c4e4298215235ff8d5ef9~tplv-k3u1fbpfcp-zoom-1.image)

**安装添加docker源**

  

```
[root@localhost ~]# sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror, langpacks
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
[root@localhost ~]#
[root@localhost ~]# cat /etc/yum.repos.d/docker-ce.repo
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg


[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg


[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://download.docker.com/linux/centos/$releasever/source/stable
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg


[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg


[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg


[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://download.docker.com/linux/centos/$releasever/source/test
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg


[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg


[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg


[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://download.docker.com/linux/centos/$releasever/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg
[root@localhost ~]#
```

**下载安装包**

  

```
[root@localhost ~]# cd docker
[root@localhost docker]#
[root@localhost docker]# repotrack docker-ce
```

  

**安装docker 并设置开机自启**

  

```
[root@localhost docker]# yum install ./*
[root@localhost docker]# systemctl  start docker
[root@localhost docker]#
[root@localhost docker]# systemctl  enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
[root@localhost docker]#
```

  

**配置nvidia-docker的源**

  

```
[root@localhost docker]# distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
>    && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo
[root@localhost docker]# cat /etc/yum.repos.d/nvidia-docker.repo
[libnvidia-container]
name=libnvidia-container
baseurl=https://nvidia.github.io/libnvidia-container/stable/centos7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://nvidia.github.io/libnvidia-container/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt


[libnvidia-container-experimental]
name=libnvidia-container-experimental
baseurl=https://nvidia.github.io/libnvidia-container/experimental/centos7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=0
gpgkey=https://nvidia.github.io/libnvidia-container/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt


[nvidia-container-runtime]
name=nvidia-container-runtime
baseurl=https://nvidia.github.io/nvidia-container-runtime/stable/centos7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://nvidia.github.io/nvidia-container-runtime/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt


[nvidia-container-runtime-experimental]
name=nvidia-container-runtime-experimental
baseurl=https://nvidia.github.io/nvidia-container-runtime/experimental/centos7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=0
gpgkey=https://nvidia.github.io/nvidia-container-runtime/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt


[nvidia-docker]
name=nvidia-docker
baseurl=https://nvidia.github.io/nvidia-docker/centos7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://nvidia.github.io/nvidia-docker/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
[root@localhost docker]#
```

  

**安装下载nvidia-docker**

  

```
[root@localhost ~]# mkdir nvidia-docker2
[root@localhost ~]# cd nvidia-docker2
[root@localhost nvidia-docker2]# yum update -y
[root@localhost nvidia-docker2]# repotrack nvidia-docker2
[root@localhost nvidia-docker2]# yum install ./*


[root@localhost ~]# mkdir nvidia-container-toolkit
[root@localhost ~]# cd nvidia-container-toolkit
[root@localhost nvidia-container-toolkit]# repotrack nvidia-container-toolkit
[root@ai-rd nvidia-container-toolkit]# yum install ./*
```

  

**下载镜像，并保存**

  

```
[root@localhost ~]# docker pull nvidia/cuda:11.0-base
11.0-base: Pulling from nvidia/cuda
54ee1f796a1e: Pull complete
f7bfea53ad12: Pull complete
46d371e02073: Pull complete
b66c17bbf772: Pull complete
3642f1a6dfb3: Pull complete
e5ce55b8b4b9: Pull complete
155bc0332b0a: Pull complete
Digest: sha256:774ca3d612de15213102c2dbbba55df44dc5cf9870ca2be6c6e9c627fa63d67a
Status: Downloaded newer image for nvidia/cuda:11.0-base
docker.io/nvidia/cuda:11.0-base
[root@localhost ~]#
[root@localhost ~]# docker images
REPOSITORY    TAG         IMAGE ID       CREATED         SIZE
nvidia/cuda   11.0-base   2ec708416bb8   15 months ago   122MB
[root@localhost ~]#
[root@localhost ~]# docker save -o cuda-11.0.tar nvidia/cuda:11.0-base
[root@localhost ~]#
[root@localhost ~]# ls cuda-11.0.tar
cuda-11.0.tar
[root@localhost ~]#
```

  

**在要测试的服务器上导入镜像**

  

```
[root@ai-rd cby]# docker load -i cuda-11.0.tar
2ce3c188c38d: Loading layer [==================================================>]  75.23MB/75.23MB
ad44aa179b33: Loading layer [==================================================>]  1.011MB/1.011MB
35a91a75d24b: Loading layer [==================================================>]  15.36kB/15.36kB
a4399aeb9a0e: Loading layer [==================================================>]  3.072kB/3.072kB
fa39d0e9f3dc: Loading layer [==================================================>]  18.84MB/18.84MB
232fb43df6ad: Loading layer [==================================================>]  30.08MB/30.08MB
0da51e35db05: Loading layer [==================================================>]  22.53kB/22.53kB
Loaded image: nvidia/cuda:11.0-base
[root@ai-rd cby]#
[root@ai-rd cby]# docker images | grep cuda
nvidia/cuda                          11.0-base   2ec708416bb8   15 months ago   122MB
[root@ai-rd cby]#
```

  

**安装升级内核**

  

```
[root@ai-rd cby]# yum install kernel-headers
[root@ai-rd cby]# yum install kernel-devel
[root@ai-rd cby]# yum update kernel*
```

  

**禁用模块，并升级boot**

  

```
[root@ai-rd cby]# vim /etc/modprobe.d/blacklist-nouveau.conf
[root@ai-rd cby]# cat /etc/modprobe.d/blacklist-nouveau.conf
blacklist nouveau
options nouveau modeset=0
[root@ai-rd cby]#
[root@ai-rd cby]# mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
[root@ai-rd cby]# sudo dracut -v /boot/initramfs-$(uname -r).img $(uname -r)
```

  

**下载驱动并安装**

  

```
[root@localhost ~]# wget https://cn.download.nvidia.cn/tesla/450.156.00/NVIDIA-Linux-x86_64-450.156.00.run
[root@ai-rd cby]# chmod +x NVIDIA-Linux-x86_64-450.156.00.run
[root@ai-rd cby]# ./NVIDIA-Linux-x86_64-450.156.00.run
```

  

**配置docker**

  

```
[root@ai-rd ~]# vim /etc/docker/daemon.json
[root@ai-rd ~]# cat /etc/docker/daemon.json
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}


[root@ai-rd ~]#
[root@ai-rd ~]# systemctl daemon-reload
[root@ai-rd ~]#
[root@ai-rd ~]#
[root@ai-rd ~]#
[root@ai-rd ~]# systemctl  restart docker
[root@ai-rd ~]#
```

  

**测试docker中的调用情况**

  

```
[root@ai-rd ~]#
[root@ai-rd ~]# sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
Tue Nov 23 06:03:04 2021      
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 450.156.00   Driver Version: 450.156.00   CUDA Version: 11.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla T4            Off  | 00000000:86:00.0 Off |                    0 |
| N/A   90C    P0    34W /  70W |      0MiB / 15109MiB |      6%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
[root@ai-rd ~]#
```

  

  
  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/966a8be815a448feae71b0cebcd47020~tplv-k3u1fbpfcp-zoom-1.image)  

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