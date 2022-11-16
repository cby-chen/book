---
layout: post
cid: 88
title: 使用二进制方式安装Docker
slug: 88
date: 2022/02/11 09:59:00
updated: 2022/03/25 15:37:51
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


  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c00fc713caab4f9eb63bf906cd86338f~tplv-k3u1fbpfcp-zoom-1.image)

  

长期使用安装工具进行安装docker，今天用二进制方式手动安装一下docker环境。

  

```
二进制包下载地址：https://download.docker.com/linux/static/stable/x86_64/

#解压
tar xf docker-20.10.9.tgz 
#拷贝二进制文件
cp docker/* /usr/bin/
#创建containerd的service文件,并且启动
cat >/etc/systemd/system/containerd.service <<EOF
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target


[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/bin/containerd
Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=1048576
TasksMax=infinity
OOMScoreAdjust=-999


[Install]
WantedBy=multi-user.target
EOF


systemctl enable --now containerd.service
```

  

准备docker的service文件

  

```
#准备docker的service文件
cat > /etc/systemd/system/docker.service <<EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket containerd.service


[Service]
Type=notify
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
OOMScoreAdjust=-500


[Install]
WantedBy=multi-user.target
EOF
#准备docker的socket文件
cat > /etc/systemd/system/docker.socket <<EOF
[Unit]
Description=Docker Socket for the API


[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker


[Install]
WantedBy=sockets.target
EOF
```

  

启动验证  

  

```
#创建docker组
groupadd docker
#启动docker
systemctl enable --now docker.socket  && systemctl enable --now docker.service
#验证
docker info
```

  

创建docker配置文件  

  

```
cat >/etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com"
  ],
  "max-concurrent-downloads": 10,
  "log-driver": "json-file",
  "log-level": "warn",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
    },
  "data-root": "/var/lib/docker"
}
EOF
systemctl restart docker
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