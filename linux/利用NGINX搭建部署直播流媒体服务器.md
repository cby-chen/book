---
layout: post
cid: 106
title: 利用NGINX搭建部署直播流媒体服务器
slug: 106
date: 2022/03/23 17:45:32
updated: 2022/03/23 17:46:21
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


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5dfb3a7d70874dc5acedba95eb63499d~tplv-k3u1fbpfcp-zoom-1.image)

直播如今是一个老生常谈的问题，怎么用于直播，大多数人只晓得，大佬某平台直播软件，点击开始即可直播。那么如何来搭建一个简易的直播平台呢？仅仅是有直播功能，没有涉及转码以及播放软件。

  

```
安装nginx以及rtmp模块

root@cby:~# apt install nginx
root@cby:~# apt install libnginx-mod-rtmp

修改配置以支持rtmp
root@cby:~# vim /etc/nginx/nginx.conf
rtmp {
    server {
        listen 1935;
        chunk_size 4096;
        application live {
            live on;
        }
    }
}

检查是否有报错
root@cby:~# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
root@cby:~# 

重启nginx
root@cby:~# systemctl restart nginx


```

  

使用obs直播工具进行推流操作  

  

rtmp://<你的域名或者IP>:1935/live

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b45d78b6803a4461a72ac6f84b4a0430~tplv-k3u1fbpfcp-zoom-1.image)

  

  

使用vlc拉流播放

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/213ab9de78394063be0d21d55a4e4aa2~tplv-k3u1fbpfcp-zoom-1.image)

  

查看效果

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2cf4ac8ff1f4eacb0139284bfdc6208~tplv-k3u1fbpfcp-zoom-1.image)

  

  


  

https://www.oiox.cn/

https://www.chenby.cn/

https://cby-chen.github.io/

https://weibo.com/u/5982474121

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

https://www.jianshu.com/u/0f894314ae2c

https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/

CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客、全网可搜《小陈运维》
