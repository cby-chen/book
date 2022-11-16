---
layout: post
cid: 25
title: CentOS8删除boot目录恢复
slug: 25
date: 2021/12/30 17:03:48
updated: 2021/12/30 17:03:48
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


 系统安装完之后，boot分区最好做一个备份，因为这个分区 我们基本不会动它，所以备份一次一劳永逸，以防万一。如果我们不小心 误删除了这个目录，也不用慌，正因为这个分区，我们除了开机 其他时候基本用不到，所以恢复起来还是很容易的。而且恢复之后，我们操作系统里的其他服务基本没有影响，我们看一下，如果误删除了/boot，该如何恢复：

  

由于/boot分区一般就是用于存放镜像和相关启动引导文件，所以误删之后，恢复并不影响系统其他服务的正常运行；但是误删之后 系统启动不了了，因为 grub.conf文件在 /boot/grub/中 也被删除了。

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90f852e23d254098ac68d2d484e9fb6a~tplv-k3u1fbpfcp-zoom-1.image)

删除boot目录

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e9b0b0be060455a809ecf08349b3caf~tplv-k3u1fbpfcp-zoom-1.image)

  

已无法启动，进入grub模式

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d1202adf6cd4dd7b19ffa6ae0852a9f~tplv-k3u1fbpfcp-zoom-1.image)

  

   

这时需要进行挂盘修复

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d637fec4500946259a3589609d4b07e2~tplv-k3u1fbpfcp-zoom-1.image)

  

急救模式启动后加载一个shell

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83b6702ed96f47ee9707e1fdd45e8915~tplv-k3u1fbpfcp-zoom-1.image)

  

查看磁盘已自动挂载到/mnt/目录下

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b2d18454a014ebea4237e43a56ec8be~tplv-k3u1fbpfcp-zoom-1.image)

  

使用chroot命令进入到磁盘系统。否则仅在内存系统中。

查看boot目录后是空的。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9a954b4796c40d696e9641d41ec66b5~tplv-k3u1fbpfcp-zoom-1.image)

  

挂载光盘镜像

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2229df13ac59400384bf53fe023f39ba~tplv-k3u1fbpfcp-zoom-1.image)

使用其他的Centos8 系统 查看boot目录下vmlinuz和initramfs生成的包

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3692fc698654492cbfa9b3af234b8e78~tplv-k3u1fbpfcp-zoom-1.image)

  

安装内核

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9572016158f7460399d3b027aeb7115d~tplv-k3u1fbpfcp-zoom-1.image)

  

Boot目录恢复

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7526425e1cd4603b327d1a829b26657~tplv-k3u1fbpfcp-zoom-1.image)

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12d66b1f8e874173bee11118aa09e6f2~tplv-k3u1fbpfcp-zoom-1.image)

   

已可以正常引导

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4866700ba744a868c6ddb24d50ccdfc~tplv-k3u1fbpfcp-zoom-1.image)