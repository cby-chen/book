---
layout: post
cid: 20
title: Python安装-在Linux系统中使用编译进行安装
slug: 20
date: 2021/12/30 17:02:47
updated: 2021/12/30 17:02:47
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2be64bec43d454abe1b2c003d3322cf~tplv-k3u1fbpfcp-zoom-1.image)

**Python安装-在Linux系统中使用编译进行安装**  

  

你可以使用Ubuntu自带的Python3，不过你不能自由的控制版本，还要单独安装pip3，如果你想升级pip3，还会出现一些让人不愉快的使用问题。而在CentOS系统中，默认只有Python2，通过yum安装Python3，也同样面临版本落后以及pip3的问题。如果不自己编译安装，还有什么别的方法来一直保持使用最新的版本呢？！除非你用Win系统。

You can use the Python3 that comes with Ubuntu, but you can't control the version freely. You have to install pip3 separately. If you want to upgrade pip3, there will be some unpleasant usage problems. In the CentOS system, there is only Python2 by default. Installing Python3 through yum also faces the problems of backward version and pip3. If you don’t compile and install it yourself, what other methods are there to keep using the latest version? ! Unless you use Win system.

在CentOS中安装Python3需要的依赖库

 Install the dependency libraries required by Python3 in CentOS

  

```
sudo yum install zlib-devel bzip2-developenssl-devel ncurses-devel sqlite-devel readline-devel tk-devel libffi-develexpat-devel gdbm-devel xz-devel db4-devel libpcap-devel make
```

在Ubuntu中安装Python3需要的依赖库

Install the dependency libraries required by Python3 in Ubuntu

```
$ sudo apt install libreadline-gplv2-devlibncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libbz2-devzlib1g-dev libffi-dev liblzma-dev
```

**安装GCC**

**Install GCC**

  

CentOS的minimal版本，以及Ubuntu，都没有预装gcc，如果你用的是这两个版本，需要确保系统有gcc编译器可以使用。安装和查看gcc的方法：

The minimal version of CentOS and Ubuntu do not have gcc pre-installed. If you are using these two versions, you need to make sure that the system has a gcc compiler that can be used. How to install and view gcc:

   

  

```
$ sudo yum install gcc  # install gcc in centos
$ sudo apt install gcc  # install gcc in ubuntu
$ which gcc # check if gcc is there
$ gcc --version  # check gcc version
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce578f7240e5458ca63771461d0a120c~tplv-k3u1fbpfcp-zoom-1.image)

下载Python3源码并解压

Download the Python3 source code and unzip it

  

Python3的官方源码下载页面是：https://www.python.org/downloads/

  

The official source code download page of Python3 is:

https://www.python.org/downloads/

   

  

使用curl或wget下载，然后解压：

Use curl or wget to download, and then unzip:

```
Wget https://www.python.org/ftp/python/3.9.2/Python-3.9.2.tgz
tar xvf Python-3.9.2.tgz
```

执行configure

Execute configure

  

进入上一步的解压目录，然后执行configure：

Enter the unzipped directory of the previous step, and then execute configure:

```
$ cd Python-3.7.3
$ ./configure --prefix=/usr/local/python-3.9.2
```

make和install

make and install

  

最后，我们执行make和install的指令。

Finally, we execute the make and install instructions.

```
$ make && sudo make install
```

  

make install 前要有sudo，因为我们在configure的时候，指定的安装路径为系统路径，不是用户的/home/user路径。

There must be sudo before make install, because when we configure, the specified installation path is the system path, not the user's /home/user path.

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94f2204a9d594c589db6dec5ae3ac221~tplv-k3u1fbpfcp-zoom-1.image)

  

```
ln -s /usr/local/python-3.9.2/bin/python3.9/usr/bin/python3
ln -s /usr/local/python-3.9.2/bin/pip3.9/usr/bin/pip3
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a62f462b9cc444f836d108ace3c3ba2~tplv-k3u1fbpfcp-zoom-1.image)