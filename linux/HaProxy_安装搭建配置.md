---
layout: post
cid: 45
title: HaProxy 安装搭建配置
slug: 45
date: 2021/12/30 17:09:53
updated: 2021/12/30 17:09:53
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


**HaProxy简介**

**![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6fd39e9d7ad4cd290e7b34eb0022d6c~tplv-k3u1fbpfcp-zoom-1.image)**

  

    HAProxy是一个免费的负载均衡软件，可以运行于大部分主流的Linux操作系统上。

    HAProxy提供了L4(TCP)和L7(HTTP)两种负载均衡能力，具备丰富的功能。HAProxy的社区非常活跃，版本更新快速。最关键的是，HAProxy具备媲美商用负载均衡器的性能和稳定性。

**HaProxy的核心功能**

    负载均衡：L4和L7两种模式，支持RR/静态RR/LC/IP Hash/URI Hash/URL_PARAM Hash/HTTP_HEADER Hash等丰富的负载均衡算法

  

    健康检查：支持TCP和HTTP两种健康检查模式

  

    会话保持：对于未实现会话共享的应用集群，可通过Insert Cookie/Rewrite Cookie/Prefix Cookie，以及上述的多种Hash方式实现会话保持

  

    SSL：HAProxy可以解析HTTPS协议，并能够将请求解密为HTTP后向后端传输

  

    HTTP请求重写与重定向

  

    监控与统计：HAProxy提供了基于Web的统计信息页面，展现健康状态和流量数据。基于此功能，使用者可以开发监控程序来监控HAProxy的状态

  

**HaProxy的关键特性**  

**性能**

  

    1 . 采用单线程、事件驱动、非阻塞模型，减少上下文切换的消耗，能在1ms内处理数百个请求。并且每个会话只占用数KB的内存。

  

    2 . 大量精细的性能优化，如O(1)复杂度的事件检查器、延迟更新技术、Single-buffereing、Zero-copy forwarding等等，这些技术使得HAProxy在中等负载下只占用极低的CPU资源。

  

    3 . HAProxy大量利用操作系统本身的功能特性，使得其在处理请求时能发挥极高的性能，通常情况下，HAProxy自身只占用15%的处理时间，剩余的85%都是在系统内核层完成的。

  

    4 . HAProxy作者在8年前（2009）年使用1.4版本进行了一次测试，单个HAProxy进程的处理能力突破了10万请求/秒，并轻松占满了10Gbps的网络带宽。

  

**稳定性**

  

    在上文中提到过，HAProxy的大部分工作都是在操作系统内核完成的，所以HAProxy的稳定性主要依赖于操作系统，作者建议使用2.6或3.x的Linux内核，对sysctls参数进行精细的优化，并且确保主机有足够的内存。这样HAProxy就能够持续满负载稳定运行数年之久。

  

  

  

  

设置主机名

  
  

```
root@hello:~# hostnamectl  set-hostname haproxy
root@hello:~#
root@hello:~#
root@hello:~# bash
root@haproxy:~#
```

  

  

安装 haproxy

  

```
root@haproxy:~# apt-get install haproxy
root@haproxy:~# cp /etc/haproxy/haproxy.cfg{,.ori}
root@haproxy:~#
root@haproxy:~# vim /etc/haproxy/haproxy.cfg
root@haproxy:~#

```

配置文件如下

  
  

```
root@haproxy:~# cat /etc/haproxy/haproxy.cfg
cat /etc/haproxy/haproxy.cfg
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon


        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private


        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options no-sslv3 no-tlsv10 no-tlsv11 no-tls-tickets


defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http


frontend LOADBALANCER-01
    bind  0.0.0.0:80
    mode http
    default_backend WEBSERVERS-01


backend WEBSERVERS-01
        balance roundrobin
    server      node1 192.168.1.10:9200 check   inter 2000 rise 3 fall 3 weight 1 maxconn   2000
        server      node2 192.168.1.11:9200 check   inter 2000 rise 3 fall 3 weight 1 maxconn   2000
        server      node3 192.168.1.12:9200 check   inter 2000 rise 3 fall 3 weight 1 maxconn   2000
        server      node4 192.168.1.13:9200 check   inter 2000 rise 3 fall 3 weight 1 maxconn   2000
        server      node5 192.168.1.14:9200 check   inter 2000 rise 3 fall 3 weight 1 maxconn   2000
        server      node6 192.168.1.15:9200 check   inter 2000 rise 3 fall 3 weight 1 maxconn   2000
        server      node7 192.168.1.16:9200 check   inter 2000 rise 3 fall 3 weight 1 maxconn   2000 backup
        option httpchk
```

  
  
  
  

启动服务

  

```
root@haproxy:~#
root@haproxy:~# systemctl start haproxy
root@haproxy:~#
```

  

  

设置开机自启

  

```
root@haproxy:~#
root@haproxy:~# systemctl enable haproxy
Synchronizing state of haproxy.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable haproxy
root@haproxy:~#
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