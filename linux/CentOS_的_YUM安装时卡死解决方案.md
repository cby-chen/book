---
layout: post
cid: 21
title: CentOS 的 YUM安装时卡死解决方案
slug: 21
date: 2021/12/30 17:03:00
updated: 2021/12/30 17:03:00
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


  

YUM是基于RPM的软件包管理器

YUM is an RPM-based package manager

  

**补充说明**

**Supplementary note**

yum命令 是在Fedora和RedHat以及SUSE中基于rpm的软件包管理器，它可以使系统管理人员交互和自动化地更新与管理RPM软件包，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。yum提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

    The yum command is a rpm-based package manager in Fedora, RedHat and SUSE. It enables system administrators to interactively and automatically update and manage RPM packages. It can automatically download and install RPM packages from a specified server, and can be processed automatically Dependency relationship, and install all dependent software packages at one time, no need to download and install tediously again and again. Yum provides commands to find, install, delete a certain, a group or even all packages, and the commands are concise and easy to remember.

  

**问题：**

    在使用yum安装时，卡死并且无法Ctrl+c终止，需要将其杀死才能停止。

如下图：

    When using yum to install, it is stuck and cannot be terminated by Ctrl+c. You need to kill it to stop.

As shown below:

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/118881abd6294aad84628b1a8577beed~tplv-k3u1fbpfcp-zoom-1.image)

  

解决方案一：

Solution 1:

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d6b1ae48f6b4261b44e1bbf1929ac85~tplv-k3u1fbpfcp-zoom-1.image)

    删除rpm数据文件后再重建rpm数据文件：

 Rebuild the rpm data file after deleting the rpm data file:

  

    删除rpm数据文件

 Delete rpm data file  

```
rm -f /var/lib/rpm/__db.00*
```

  

重建rpm数据文件

Rebuild rpm data file

```
rpm -vv --rebuilddb
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43b8beadf721433c95cafb5dc3025d3f~tplv-k3u1fbpfcp-zoom-1.image)

  

清空缓存后再重新缓存

Re-cache after clearing the cache

```
yum clean all 
yum makecache
```

  

执行完一般情况就可以正常使用了，若依旧无法使用请参考以下方式二  

After executing the general situation, it can be used normally, if it still cannot be used, please refer to the following method two

  

解决方案二：

Solution two:

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2c9ab9426a441c39761f6d3992d1e4b~tplv-k3u1fbpfcp-zoom-1.image)

  

将这俩个文件删除后在进行测试

Test after deleting these two files

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67e2d09420d4403bb3713243f3fded27~tplv-k3u1fbpfcp-zoom-1.image)

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e53c0af8883a488aacbb62ba5286bd7d~tplv-k3u1fbpfcp-zoom-1.image)