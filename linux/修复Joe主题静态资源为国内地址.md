---
layout: post
cid: 269
title: 修复Joe主题静态资源为国内地址
slug: 269
date: 2022/07/02 11:40:57
updated: 2022/07/02 11:41:10
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 修复Joe主题静态资源为国内地址
description: 修复Joe主题静态资源为国内地址
keywords: 小陈运维,小陈,小陈博客,文档,
mode: default
thumb: 
video: 
---


# 修复Joe主题静态资源为国内地址

之前一直是在良好的网络环境中使用我的博客系统，一直没有发现资源加载异常问题，如今我回到内蒙古之后发现这边运营商的DNS污染问题挺严重，就连GitHub都无法正常访问，包括我的博客系统中很多静态资源加载并不正常。所以今天我将我的博客静态资源进行了修复。

我将修改后的主题代码放在了GitHub上面，可以访问进行查看 https://github.com/cby-chen/blog

其中`static`目录是我修改后的需要用到的静态文件。`Joe`目录是把主题修改后可以直接使用的，将Joe目录放在 `typecho` 博客系统的  `themes`主题文件夹下即可直接使用。

注：主题是引用了我自己的博客网站静态资源 www.oiox.cn



> **关于**
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