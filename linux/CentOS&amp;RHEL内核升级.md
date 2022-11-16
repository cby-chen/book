---
layout: post
cid: 85
title: CentOS&amp;RHEL内核升级
slug: 85
date: 2022/02/08 02:16:00
updated: 2022/03/25 15:38:51
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


  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7310a5b8c8584446892dd5da6467514f~tplv-k3u1fbpfcp-zoom-1.image)

  

**在安装部署一些环境的时候，会要求内核版本的要求，可以通过YUM工具进行安装配置更高版本的内核，当然更新内核有风险，在操作之前慎重，严谨在生产环境操作！**

  

  

安装源

  

```
# 为 RHEL-8或 CentOS-8配置源
yum install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
# 为 RHEL-7 SL-7 或 CentOS-7 安装 ELRepo 
yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
```

  

启用内核源，并安装  

  

```
# 查看可用安装包
yum  --disablerepo="*"  --enablerepo="elrepo-kernel"  list  available

# 安装最新的内核
# 我这里选择的是稳定版kernel-ml   如需更新长期维护版本kernel-lt  
yum  --enablerepo=elrepo-kernel  install  kernel-ml
```

  

查看已有内核  

  

```
# 查看已安装哪些内核
rpm -qa | grep kernel
kernel-core-4.18.0-358.el8.x86_64
kernel-tools-4.18.0-358.el8.x86_64
kernel-ml-core-5.16.7-1.el8.elrepo.x86_64
kernel-ml-5.16.7-1.el8.elrepo.x86_64
kernel-modules-4.18.0-358.el8.x86_64
kernel-4.18.0-358.el8.x86_64
kernel-tools-libs-4.18.0-358.el8.x86_64
kernel-ml-modules-5.16.7-1.el8.elrepo.x86_64
```

  

查看当前使用的并设置  

  

```
# 查看默认内核
grubby --default-kernel
/boot/vmlinuz-5.16.7-1.el8.elrepo.x86_64

# 若不是最新的使用命令设置
grubby --set-default /boot/vmlinuz-「您的内核版本」.x86_64

# 重启生效
reboot
```

  

整合一条命令为：

```
yum install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm -y ; yum  --disablerepo="*"  --enablerepo="elrepo-kernel"  list  available -y ; yum  --enablerepo=elrepo-kernel  install  kernel-ml -y ; grubby --default-kernel ; reboot
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