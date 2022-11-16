---
layout: post
cid: 36
title: Python 人工智能 5秒钟偷走你的声音
slug: 36
date: 2021/12/30 17:08:12
updated: 2021/12/30 17:08:12
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


介绍

Python 深度学习AI - 声音克隆、声音模仿，是一个三阶段的深度学习框架，允许从几秒钟的音频中创建语音的数字表示，并用它来调节文本到语音模型，该模型经过培训，可以概括到新的声音。

环境准备与安装

原始英文版地址：

https://github.com/CorentinJ/Real-Time-Voice-Cloning

中文二次开发版（本文使用该版本）：

https://github.com/babysor/MockingBird

pycharm环境下载：

https://www.jetbrains.com/pycharm/download/#section=windows

conda虚拟环境：

https://www.anaconda.com/products/individual

FFmpeg ：

https://github.com/BtbN/FFmpeg-Builds/releases

模型文件：

https://pan.baidu.com/s/1PI-hM3sn5wbeChRryX-RCQ 提取码 2021

在电脑系统上安装 FFmpeg 工具

下载zip压缩包连接为：https://github.com/BtbN/FFmpeg-Builds/releases/download/autobuild-2021-11-09-12-23/ffmpeg-N-104488-ga13646639f-win64-gpl.zip

下载完成后将其解压到一个目录后在系统的环境变量中添加该目录

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc5860656f3b46b987cefc067b5e60e0~tplv-k3u1fbpfcp-zoom-1.image)

  

打开新的cmd中查看是否安装成功

ffmpeg -version

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1068725e289043d2b8fd0230ade44f17~tplv-k3u1fbpfcp-zoom-1.image)

  

使用打开项目目录后，创建时使用conda的Python 3.9虚拟环境

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8eee53b28c5948568b10998b8d55cc34~tplv-k3u1fbpfcp-zoom-1.image)

  

创建完成后，在cmd中查看现有的虚拟环境，并进入刚刚创建的虚拟环境

conda env list

activate pythonProject1

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c1e01a94936403fa8ef3e90c97b8d59~tplv-k3u1fbpfcp-zoom-1.image)

  

进入环境后在进行安装pip所需依赖，并使用国内源进行安装实现下载加速

pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b83937d957b743de9af41e331846e0fb~tplv-k3u1fbpfcp-zoom-1.image)

  

在虚拟环境下安装pytorch

pip install torch  -i https://pypi.tuna.tsinghua.edu.cn/simple

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33c3bde9f5d8410aac64af1f9bcc2abf~tplv-k3u1fbpfcp-zoom-1.image)

  

回到pycharm中，将模型导入到项目目录下，把目录复制黏贴到项目中

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31a4ff2f688b4205835a6ebd3a26b7e4~tplv-k3u1fbpfcp-zoom-1.image)

  

修改一行代码，在 synthesizer/utils/symbols.py 文件中

  

```
修改为：
_characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz12340!'(),-.:;? '
```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/410cfa4dedb84fe59b0a676b2292716d~tplv-k3u1fbpfcp-zoom-1.image)

  

之后在terminal中启动工具箱

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4808629d6b5d4bf9ba24d23a397e85b3~tplv-k3u1fbpfcp-zoom-1.image)

  

使用音频合成工具箱

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36858a67eb56462599197da8a65f91a2~tplv-k3u1fbpfcp-zoom-1.image)

  

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
> **文章主要发布于微信**