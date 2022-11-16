---
layout: post
cid: 272
title: kubernetes (k8s) 二进制高可用安装
slug: 272
date: 2022/07/14 18:26:00
updated: 2022/07/14 18:26:36
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: kubernetes (k8s) 二进制高可用安装
keywords: 小陈运维,小陈,小陈博客,文档,
mode: default
thumb: 
video: 
---


# kubernetes (k8s) 二进制高可用安装

[https://github.com/cby-chen/Kubernetes](https://github.com/cby-chen/Kubernetes) 开源不易，帮忙点个star，谢谢了

GitHub访问不通畅可以访问国内GitEE https://gitee.com/cby-inc/Kubernetes

# 常见异常

1. 安装会出现kubelet异常，无法识别 `--node-labels` 字段问题，原因如下。

将 `--node-labels=node.kubernetes.io/node=''` 替换为 `--node-labels=node.kubernetes.io/node=`  将 `''` 删除即可。

2. 注意hosts配置文件中主机名和IP地址对应

3. 在文档7.2，却记别忘记执行`kubectl create -f bootstrap.secret.yaml`命令

# 介绍

kubernetes（k8s）二进制高可用安装部署，支持IPv4+IPv6双栈。

我使用IPV6的目的是在公网进行访问，所以我配置了IPV6静态地址。

若您没有IPV6环境，或者不想使用IPv6，不对主机进行配置IPv6地址即可。

不配置IPV6，不影响后续，不过集群依旧是支持IPv6的。为后期留有扩展可能性。

若不要IPv6 ，不给网卡配置IPv6即可，不要对IPv6相关配置删除或操作，否则会出问题。

强烈建议在Github上查看文档。Github出问题会更新文档，并且后续尽可能第一时间更新新版本文档。

# 当前文档版本

1.21.13 和 1.22.10 和 1.23.3 和 1.23.4 和 1.23.5 和 1.23.6 和 1.23.7 和 1.24.0 和 1.24.1 和 1.24.2 和 1.24.3 ...陆续更新。

# 访问地址

https://github.com/cby-chen/Kubernetes/

手动项目地址：https://github.com/cby-chen/Kubernetes

脚本项目地址：https://github.com/cby-chen/Binary_installation_of_Kubernetes

# 文档

## 二进制安装每个版本文档

### 1.23版本
[v1.23.3-CentOS-binary-install](./v1.23.3-CentOS-binary-install.md)

[v1.23.4-CentOS-binary-install](./v1.23.4-CentOS-binary-install.md)

[v1.23.5-CentOS-binary-install](./v1.23.5-CentOS-binary-install.md)

[v1.23.6-CentOS-binary-install](./v1.23.6-CentOS-binary-install.md)

### 1.24版本
[v1.24.0-CentOS-binary-install-IPv6-IPv4.md](./v1.24.0-CentOS-binary-install-IPv6-IPv4.md)

[v1.24.1-CentOS-binary-install-IPv6-IPv4.md](./v1.24.1-CentOS-binary-install-IPv6-IPv4.md)

[v1.24.2-CentOS-binary-install-IPv6-IPv4.md](./v1.24.2-CentOS-binary-install-IPv6-IPv4.md)

[v1.24.3-CentOS-binary-install-IPv6-IPv4.md](./v1.24.3-CentOS-binary-install-IPv6-IPv4.md)

### 三主俩从版本
[v1.21.13-CentOS-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md](./v1.21.13-CentOS-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md)

[v1.22.10-CentOS-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md](./v1.22.10-CentOS-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md)

[v1.23.7-CentOS-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md](./v1.23.7-CentOS-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md)

[v1.24.0-CentOS-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md](./v1.24.0-CentOS-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md)

[v1.24.1-CentOS-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md](./v1.24.1-CentOS-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md)

[v1.24.1-Ubuntu-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md](./v1.24.1-Ubuntu-binary-install-IPv6-IPv4-Three-Masters-Two-Slaves.md)


## 修复kube-proxy证书权限过大问题

[kube-proxy_permissions.md](./kube-proxy_permissions.md)

## 使用kubeadm初始化IPV4/IPV6集群

[kubeadm-install-IPV6-IPV4.md](./kubeadm-install-IPV6-IPV4.md)

## IPv4集群启用IPv6功能，关闭IPv6则反之

[Enable-implement-IPv4-IPv6.md](./Enable-implement-IPv4-IPv6.md)

# 安装包

（下载更快）我自己的网盘：https://pan.oiox.cn/s/PetV

（下载更快）123网盘：https://www.123pan.com/s/Z8ArVv-PG60d

每个初始版本会打上releases，安装包在releases页面

https://github.com/cby-chen/Kubernetes/releases

注意：1.23.3 版本当时没想到会后续更新，所以当时命名不太规范。

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/cby/Kubernetes.tar

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/v1.23.4/kubernetes-v1.23.4.tar

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/v1.23.5/kubernetes-v1.24.5.tar

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/v1.23.6/kubernetes-v1.23.6.tar

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/v1.23.7/kubernetes-v1.23.7.tar

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/v1.24.0/kubernetes-v1.24.0.tar

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/v1.24.1/kubernetes-v1.24.1.tar

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/v1.24.2/kubernetes-v1.24.2.tar

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/v1.24.3/kubernetes-v1.24.3.tar

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/v1.22.10/kubernetes-v1.22.10.tar

wget https://ghproxy.com/https://github.com/cby-chen/Kubernetes/releases/download/v1.21.13/kubernetes-v1.21.13.tar

# 旧版本地址

建议查看main版本中的文档。https://github.com/cby-chen/Kubernetes/

若找对应版本文档中的安装包，可以在上方下载安装包，可以在在下方地址中查找。

https://github.com/cby-chen/Kubernetes/tree/cby

https://github.com/cby-chen/Kubernetes/tree/v1.23.4

https://github.com/cby-chen/Kubernetes/tree/v1.23.5

https://github.com/cby-chen/Kubernetes/tree/v1.23.6

https://github.com/cby-chen/Kubernetes/tree/v1.23.7

https://github.com/cby-chen/Kubernetes/tree/v1.24.0

https://github.com/cby-chen/Kubernetes/tree/v1.24.1

https://github.com/cby-chen/Kubernetes/tree/v1.24.2

https://github.com/cby-chen/Kubernetes/tree/v1.24.3

https://github.com/cby-chen/Kubernetes/tree/v1.22.10

https://github.com/cby-chen/Kubernetes/tree/v1.21.13

# 其他

- 建议在 [Kubernetes](https://github.com/cby-chen/Kubernetes) 查看文档，后续会陆续更新文档
- 小陈网站：

1. https://blog.oiox.cn/
2. https://www.oiox.cn/
3. https://www.chenby.cn/
4. https://cby-chen.github.io/

- 关于小陈：https://www.oiox.cn/index.php/start-page.html

# 技术交流

作者:

![avatar](https://www.oiox.cn/about/2.png)  

加群:

![avatar](https://www.oiox.cn/about/1.png)  


其他文档请查看如下，欢迎关注微信：
>
> https://www.oiox.cn/
>
> https://www.oiox.cn/index.php/start-page.html
>
> **CSDN、GitHub、知乎、开源中国、思否、掘金、简书、华为云、阿里云、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客**
>
> **全网可搜《小陈运维》**
>
> **文章主要发布于微信**
