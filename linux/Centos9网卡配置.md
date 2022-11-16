---
layout: post
cid: 64
title: Centos9网卡配置
slug: 64
date: 2021/12/30 17:14:00
updated: 2022/03/25 15:40:13
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


  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9132a1d6ffec43a08279b9af36f744c2~tplv-k3u1fbpfcp-zoom-1.image)

  

Centos9 网卡配置文件已修改，如下

  

```
[root@bogon ~]# cat /etc/NetworkManager/system-connections/ens18.nmconnection 
[connection]
id=ens18
uuid=8d1ece55-d999-3c97-866b-d2e23832a324
type=ethernet
autoconnect-priority=-999
interface-name=ens18
permissions=
timestamp=1639473429

[ethernet]
mac-address-blacklist=

[ipv4]
address1=192.168.1.92/24,192.168.1.1
dns=8.8.8.8;
dns-search=
method=manual

[ipv6]
addr-gen-mode=eui64
dns-search=
method=auto

[proxy]
[root@bogon ~]#
```

  

  

命令语法：

  

\# nmcli connection modify <interface_name> ipv4.address  <ip/prefix>

  

复制代码注意: 为了简化语句，在 nmcli 命令中，我们通常用 con 关键字替换 connection，并用 mod 关键字替换 modify。

  
  

```
将 IPv4 地址 (192.168.1.91) 分配给 ens18网卡上，
[root@chenby ~]# nmcli con mod ens18 ipv4.addresses 192.168.1.91/24;



复制代码使用下面的 nmcli 命令设置网关，
[root@chenby ~]# nmcli con mod ens18 ipv4.gateway 192.168.1.1;



复制代码设置手动配置（从 dhcp 到 static），
[root@chenby ~]# nmcli con mod ens18 ipv4.method manual;



复制代码设置 DNS 值为 “8.8.8.8”，
[root@chen'b'y ~]# nmcli con mod ens18 ipv4.dns "8.8.8.8";



复制代码要保存上述更改并重新加载，请执行如下 nmcli 命令，
[root@chenby ~]# nmcli con up ens18
```

  

  

合成一句话为：

  

```
nmcli con mod ens18 ipv4.addresses 192.168.1.91/24; nmcli con mod ens18 ipv4.gateway 192.168.1.1; nmcli con mod ens18 ipv4.method manual; nmcli con mod ens18 ipv4.dns "8.8.8.8"; nmcli con up ens18
```

  

  

远程修改IP为：

```
ssh root@192.168.1.197 "nmcli con mod ens18 ipv4.addresses 192.168.1.91/24; nmcli con mod ens18 ipv4.gateway 192.168.1.1; nmcli con mod ens18 ipv4.method manual; nmcli con mod ens18 ipv4.dns "8.8.8.8"; nmcli con up ens18"
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