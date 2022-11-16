---
layout: post
cid: 27
title: 使用frp进行内网穿透
slug: 27
date: 2021/12/30 17:04:00
updated: 2022/03/25 15:50:45
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


    frp 是一个专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。

    **frp is a high-performance reverse proxy application focusing on intranet penetration, supporting multiple protocols such as TCP, UDP, HTTP, and HTTPS. Intranet services can be exposed to the public network through a relay with public network IP nodes in a safe and convenient way.  
**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c43d130b17f0466cb879fde903b9b75f~tplv-k3u1fbpfcp-zoom-1.image)

**为什么使用 frp ？**

**Why use frp?**

  

    通过在具有公网 IP 的节点上部署 frp 服务端，可以轻松地将内网服务穿透到公网，同时提供诸多专业的功能特性，这包括：

    **By deploying the frp server on a node with a public network IP, you can easily penetrate the internal network service to the public network, while providing many professional features, including:**  

  

    客户端服务端通信支持 TCP、KCP 以及 Websocket 等多种协议。

    **The client-server communication supports multiple protocols such as TCP, KCP, and Websocket.**  

  

    采用 TCP 连接流式复用，在单个连接间承载更多请求，节省连接建立时间。

    **Use TCP connection streaming multiplexing to carry more requests between a single connection, saving connection establishment time.**  

  

    代理组间的负载均衡。

    **Load balancing between proxy groups.**  

  

    端口复用，多个服务通过同一个服务端端口暴露。

    **Port reuse, multiple services are exposed through the same server port.**  

  

    多个原生支持的客户端插件（静态文件查看，HTTP、SOCK5 代理等）便于独立使用 frp 客户端完成某些工作。

    **Multiple natively supported client plug-ins (static file viewing, HTTP, SOCK5 proxy, etc.) facilitate independent use of frp client to complete certain tasks.**  

  

    高度扩展性的服务端插件系统，方便结合自身需求进行功能扩展。

    **The highly extensible server-side plug-in system facilitates functional expansion according to your own needs.**  

  

    服务端和客户端 UI 页面。

     **Server and client UI pages.**  

  

    简单来说，frp是一个反向代理软件，他的体积小巧功能强大，讲内网IP进行frp反向代理后，即可使用代理IP进行访问内网机器的服务，例如远程桌面，虽然远程桌面有第三方软件来代替，例如向日葵，teamviewer，等一些软件进行远程，这些软件都有一些诟病，向日葵没有会员会限速，而tv登录远程连接会比较慢。所以可以考虑到使用内网穿透或者反向代理。

  **To put it simply, frp is a reverse proxy software, its size is small and powerful, after talking about the intranet IP for frp reverse proxy, you can use the proxy IP to access the services of the intranet machine, such as remote desktop, although remote desktop There are third-party software to replace, such as Sunflower, teamviewer, and other software for remote. These softwares have some criticisms. Sunflower does not have a membership rate limit, and the tv login remote connection will be slow. So you can consider using intranet penetration or reverse proxy.**

  

    内网穿透可参考：[有一个公网IP地址](http://mp.weixin.qq.com/s?__biz=MzI0MzA4NTM2NQ==&mid=2247483964&idx=1&sn=2b857cb34f7b678c1d1a5492cfee7efd&chksm=e9733b66de04b27024387be235f680f85165e69b5152960a08fc45f653d397a10381e71a5ebb&scene=21#wechat_redirect) 

 **Intranet penetration can refer to: there is a public IP address**

  

使用端口进行访问时，原理如下  

**When using the port for access, the principle is as follows**  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/addffe28d90e4e76a54d1ef079971a16~tplv-k3u1fbpfcp-zoom-1.image)

  

  

**准备工作：**

**Ready to work:**  

  

    1、首先得有一台云服务器进行提供网络带宽，frp代理带宽一般受限于该服务器带宽  

    2、一台目标机器，也就是需要反向代理的机器

  

 **  1. First, there must be a cloud server to provide network bandwidth, and the frp proxy bandwidth is generally limited by the server bandwidth**

 **2. A target machine, that is, a machine that needs a reverse proxy**

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/755f0bc472fb46608a89945de95ad5b6~tplv-k3u1fbpfcp-zoom-1.image)

**云服务器端配置：**  

**Cloud server configuration:  
**

  

    使用命令查看云服务器的架构，一般云服务器架构为x86

 **Use commands to view the architecture of the cloud server, the general cloud server architecture is x86**

```
[root@cby ~]# arch
x86_64

```

  

    使用命令下载frp软件包  

 **Use command to download frp package**  

```
[root@cby ~]# wget https://github.com/fatedier/frp/releases/download/v0.35.1/frp_0.35.1_linux_amd64.tar.gz

```

  

    下载完成后进行解压

 **Unzip after downloading**  

```
[root@cby ~]# tar -xvf frp_0.35.1_linux_amd64.tar.gz 
frp_0.35.1_linux_amd64/
frp_0.35.1_linux_amd64/frps.ini
frp_0.35.1_linux_amd64/frps_full.ini
frp_0.35.1_linux_amd64/systemd/
frp_0.35.1_linux_amd64/systemd/frpc@.service
frp_0.35.1_linux_amd64/systemd/frpc.service
frp_0.35.1_linux_amd64/systemd/frps.service
frp_0.35.1_linux_amd64/systemd/frps@.service
frp_0.35.1_linux_amd64/frpc
frp_0.35.1_linux_amd64/frpc_full.ini
frp_0.35.1_linux_amd64/frps
frp_0.35.1_linux_amd64/frpc.ini
frp_0.35.1_linux_amd64/LICENSE

```

  

    修改文件夹名称  

    **Modify folder name**  

```
[root@cby ~]# cp -r frp_0.35.1_linux_amd64 frp

[root@cby ~]# 
[root@cby ~]# ll
total 8508
drwxr-xr-x 3 root  root    4096 Feb 19 22:13 frp
drwxr-xr-x 3 mysql  116    4096 Jan 25 16:25 frp_0.35.1_linux_amd64
-rw-r--r-- 1 root  root 8695632 Jan 25 16:25 frp_0.35.1_linux_amd64.tar.gz
```

‍

  

    只需要关注如下几个文件

    **Only need to pay attention to the following files**  

```
frps
frps.ini
frpc
frpc.ini
```

  

    frps 、frps.ini 这俩个文件是服务端的配置文件和启动程序

    frpc、frpc.ini 这俩个文件是客户端的配置文件和启动程序

 **The two files frps and frps.ini are the configuration files and startup programs of the server**

 **The two files frpc and frpc.ini are the configuration files and startup programs of the client**

  

    编辑并添加以下内容  

    **Edit and add the following**  

```
[root@cby frp]# vim frps.ini 
[root@cby frp]# cat frps.ini
[common]
bind_port = 7000
dashboard_port = 7500
token = 12345678
dashboard_user = admin
dashboard_pwd = admin
vhost_http_port = 10080
vhost_https_port = 10443

```

  

    解释如下  

    **Explain as follows**  

```
“bind_port”表示用于客户端和服务端连接的端口，这个端口号我们之后在配置客户端的时候要用到。
“dashboard_port”是服务端仪表板的端口，若使用7500端口，在配置完成服务启动后可以通过浏览器访问 x.x.x.x:7500 （其中x.x.x.x为VPS的IP）查看frp服务运行信息。
“token”是用于客户端和服务端连接的口令，请自行设置并记录，稍后会用到。
“dashboard_user”和“dashboard_pwd”表示打开仪表板页面登录的用户名和密码，自行设置即可。
“vhost_http_port”和“vhost_https_port”用于反向代理HTTP主机时使用，本文不涉及HTTP协议，因而照抄或者删除这两条均可。
```

  

    文件修改完成后即可使用该命令进行启动  

    **After the file is modified, you can use this command to start**  

```
[root@cby frp]# ./frps -c frps.ini
2021/02/19 22:18:45 [I] [root.go:108] frps uses config file: frps.ini
2021/02/19 22:18:45 [I] [service.go:190] frps tcp listen on 0.0.0.0:7000
2021/02/19 22:18:45 [I] [service.go:232] http service listen on 0.0.0.0:10080
2021/02/19 22:18:45 [I] [service.go:253] https service listen on 0.0.0.0:10443
2021/02/19 22:18:45 [I] [service.go:289] Dashboard listen on 0.0.0.0:7500
2021/02/19 22:18:45 [I] [root.go:217] frps started successfully

```

  

    若使用云服务器记得需要放行所需端口  

    **If you use cloud server, remember to release the required port**  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae9842ca26144c09874396b07dd3f9ca~tplv-k3u1fbpfcp-zoom-1.image)

  

    此时访问 x.x.x.x:7500 并使用自己设置的用户名密码登录，即可看到仪表板界面

    **At this time, visit x.x.x.x:7500 and log in with the username and password you set, you can see the dashboard interface**  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/838bf4d05d234ce08f08a50dd7ef67c7~tplv-k3u1fbpfcp-zoom-1.image)

  

    把服务在后台运行即可  

    **Just run the service in the background**  

```
[root@cby frp]# nohup ./frps -c frps.ini &
[1] 4852
[root@cby frp]# jobs
[1]+  Running                 nohup ./frps -c frps.ini &

```

  

**客户端配置**  

**Client configuration**  

  

    Windows系统下即可下载这个：  

    You can download this under Windows system:  

```
https://github.com/fatedier/frp/releases/download/v0.35.1/frp_0.35.1_windows_amd64.zip
```

  

    frpc.ini文件内容为  

    **The content of the frpc.ini file is**  

```
[common]
server_addr = 123.56.237.11 
server_port = 7000
token = 12345678
[rdp]
type = tcp
local_ip = 127.0.0.1     
local_port = 3389
remote_port = 7001  
[smb]
type = tcp
local_ip = 127.0.0.1
local_port = 445
remote_port = 7002
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/308ee32d14064e239254b7605de5e1d4~tplv-k3u1fbpfcp-zoom-1.image)

    含义解释  

    **Meaning interpretation**  

```
“server_addr”为服务端IP地址，填入即可。
“server_port”为服务器端口，填入你设置的端口号即可，如果未改变就是7000
“token”是你在服务器上设置的连接口令，原样填入即可。
```

  

    自定义规则如下  

    **The custom rules are as follows**  

```
“[xxx]”表示一个规则名称，自己定义，便于查询即可。
“type”表示转发的协议类型，有TCP和UDP等选项可以选择，如有需要请自行查询frp手册。
“local_port”是本地应用的端口号，按照实际应用工作在本机的端口号填写即可。
“remote_port”是该条规则在服务端开放的端口号，自己填写并记录即可。
```

  

    客户端的启动是需要使用命令行进行启动的， 无法使用双击EXE进行启动。

    **The startup of the client needs to use the command line to start, it cannot be started by double-clicking the EXE.**  
  

```
C:\Users\Administrator>cd c:\
c:\>cd frp
c:\frp>frpc.exe -c frpc.ini
2021/02/19 22:35:49 [I] [service.go:290] [bf2998700defd7c5] login to server success, get run id [bf2998700defd7c5], server udp port [0]
2021/02/19 22:35:49 [I] [proxy_manager.go:144] [bf2998700defd7c5] proxy added: [rdp smb]
2021/02/19 22:35:49 [I] [control.go:180] [bf2998700defd7c5] [rdp] start proxy success
2021/02/19 22:35:49 [I] [control.go:180] [bf2998700defd7c5] [smb] start proxy success

```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1483ad6cb9f4f8aab81a4c718983ad8~tplv-k3u1fbpfcp-zoom-1.image)

  

    配置完成后即可在面板上看到该规则  

    **After the configuration is complete, you can see the rule on the panel**  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/265c7819ce534820be45b97dabe1572c~tplv-k3u1fbpfcp-zoom-1.image)

  

    同时使用远程连接工具使用IP或者域名即可进行连接

  

    但是Windows客户端的cmd是无法关闭的，关闭后就无法使用了，所以需要设置开机自启，使用bat脚本即可做到

  

 **At the same time, use the remote connection tool to connect using IP or domain name**

 **However, the cmd of the Windows client cannot be closed, and it cannot be used after it is closed, so you need to set the boot to start automatically, and you can use the bat script**

  

```
@echo off
if "%1" == "h" goto begin
mshta vbscript:createobject("wscript.shell").run("""%~nx0"" h",0)(window.close)&&exit
:begin
REM
cd C:\frp
frpc.exe -c frpc.ini
exit
```

  

写完之后直接把文件扔到Windows的开机启动文件夹即可

**After writing, throw the file directly into the Windows startup folder.**  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a8bc24841a44df6b5cdd0912485a6d0~tplv-k3u1fbpfcp-zoom-1.image)

  

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/092306b1ce9e41d69ecbbc9a2987570a~tplv-k3u1fbpfcp-zoom-1.image)Linux运维交流社区推荐搜索关键词列表：Linuxfrp

汉威国际

位置：

北京市丰台区丰科西路|九号路

  

  

  

```