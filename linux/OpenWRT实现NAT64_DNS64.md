---
layout: post
cid: 297
title: OpenWRT实现NAT64/DNS64
slug: 297
date: 2022/10/28 18:54:45
updated: 2022/10/28 18:59:01
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


# OpenWRT实现NAT64/DNS64



### 连接到核心路由器

```
# 连接到核心路由器
[C:\~]$ ssh root@10.0.0.1
Connecting to 10.0.0.1:22...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

WARNING! The remote SSH server rejected X11 forwarding request.


BusyBox v1.35.0 (2022-10-23 20:45:02 UTC) built-in shell (ash)

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 -----------------------------------------------------
 OpenWrt 22.03.0, r19685-512e76967f
 -----------------------------------------------------
root@OpenWrt:~# 
root@OpenWrt:~# 
```



### 测试访问IPv6是否正常

```
# 测试访问IPv6是否正常
root@OpenWrt:~# ping www.oiox.cn -6
PING www.oiox.cn (2409:8c44:2:160:50::): 56 data bytes
64 bytes from 2409:8c44:2:160:50::: seq=0 ttl=56 time=23.455 ms
64 bytes from 2409:8c44:2:160:50::: seq=1 ttl=56 time=22.949 ms
64 bytes from 2409:8c44:2:160:50::: seq=2 ttl=56 time=23.338 ms
64 bytes from 2409:8c44:2:160:50::: seq=3 ttl=56 time=23.695 ms
^C
--- www.oiox.cn ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 22.949/23.359/23.695 ms
```



### 安装tayga实现NAT64

```
# 安装tayga实现NAT64
root@OpenWrt:~# opkg update
root@OpenWrt:~# opkg install tayga
```



### 配置/etc/config/network文件

```
# 配置/etc/config/network文件
# 重点配置 globals 和  interface 'nat64'

config globals 'globals'
	option ula_prefix 'ddbe:48ec:56c6::/48'


config interface 'nat64'
        option proto 'tayga'
        option ifname 'tayga-nat64'
        option ipv4_addr '192.168.1.1'
        option prefix 'ddbe:48ec:56c6:1111::/96'	
        option dynamic_pool '192.168.1.0/24'
        option accept_ra '0'
        option send_rs '0'
        
        
# 完整配置如下
root@OpenWrt:~# vim /etc/config/network
root@OpenWrt:~# cat /etc/config/network 

config interface 'loopback'
	option device 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config globals 'globals'
	option ula_prefix 'ddbe:48ec:56c6::/48'

config device
	option name 'br-lan'
	option type 'bridge'
	list ports 'eth0'
	list ports 'eth1'
	list ports 'eth2'

config interface 'lan'
	option device 'br-lan'
	option proto 'static'
	option ipaddr '10.0.0.1'
	option netmask '255.0.0.0'
	option ip6assign '64'

config interface 'wan'
	option proto 'dhcp'
	option device 'eth3'

config interface 'wan6'
	option proto 'dhcpv6'
	option device 'eth3'
	option reqaddress 'try'
	option reqprefix 'auto'

config interface 'nat64'
        option proto 'tayga'
        option ifname 'tayga-nat64'
        option ipv4_addr '192.168.1.1'
        option prefix 'ddbe:48ec:56c6:1111::/96'	
        option dynamic_pool '192.168.1.0/24'
        option accept_ra '0'
        option send_rs '0'
root@OpenWrt:~# 
```



### 配置/etc/config/firewall

```
# 配置/etc/config/firewall
config zone
	option name 'lan'
	list network 'lan'
	option input 'ACCEPT'
	option output 'ACCEPT'
	option forward 'ACCEPT'

# 完整配置如下
root@OpenWrt:~# vim /etc/config/firewall
root@OpenWrt:~# cat /etc/config/firewall

config defaults
	option input 'ACCEPT'
	option output 'ACCEPT'
	option synflood_protect '1'
	option forward 'ACCEPT'

config zone
	option name 'lan'
	list network 'lan'
	option input 'ACCEPT'
	option output 'ACCEPT'
	option forward 'ACCEPT'

config zone
	option name 'wan'
	list network 'wan'
	list network 'wan6'
	list network 'nat64'
	option input 'ACCEPT'
	option output 'ACCEPT'
	option forward 'ACCEPT'
	option masq '1'
	option mtu_fix '1'

config forwarding
	option src 'lan'
	option dest 'wan'

config rule
	option target 'ACCEPT'
	option name 'IPv'
	option src '*'
	option dest '*'

config rule
	option name 'Allow-DHCP-Renew'
	option src 'wan'
	option proto 'udp'
	option dest_port '68'
	option target 'ACCEPT'
	option family 'ipv4'

config rule
	option name 'Allow-Ping'
	option src 'wan'
	option proto 'icmp'
	option icmp_type 'echo-request'
	option family 'ipv4'
	option target 'ACCEPT'

config rule
	option name 'Allow-IGMP'
	option src 'wan'
	option proto 'igmp'
	option family 'ipv4'
	option target 'ACCEPT'

config rule
	option name 'Allow-DHCPv6'
	option src 'wan'
	option proto 'udp'
	option dest_port '546'
	option family 'ipv6'
	option target 'ACCEPT'

config rule
	option name 'Allow-MLD'
	option src 'wan'
	option proto 'icmp'
	option src_ip 'fe80::/10'
	list icmp_type '130/0'
	list icmp_type '131/0'
	list icmp_type '132/0'
	list icmp_type '143/0'
	option family 'ipv6'
	option target 'ACCEPT'

config rule
	option name 'Allow-ICMPv6-Input'
	option src 'wan'
	option proto 'icmp'
	list icmp_type 'echo-request'
	list icmp_type 'echo-reply'
	list icmp_type 'destination-unreachable'
	list icmp_type 'packet-too-big'
	list icmp_type 'time-exceeded'
	list icmp_type 'bad-header'
	list icmp_type 'unknown-header-type'
	list icmp_type 'router-solicitation'
	list icmp_type 'neighbour-solicitation'
	list icmp_type 'router-advertisement'
	list icmp_type 'neighbour-advertisement'
	option limit '1000/sec'
	option family 'ipv6'
	option target 'ACCEPT'

config rule
	option name 'Allow-ICMPv6-Forward'
	option src 'wan'
	option dest '*'
	option proto 'icmp'
	list icmp_type 'echo-request'
	list icmp_type 'echo-reply'
	list icmp_type 'destination-unreachable'
	list icmp_type 'packet-too-big'
	list icmp_type 'time-exceeded'
	list icmp_type 'bad-header'
	list icmp_type 'unknown-header-type'
	option limit '1000/sec'
	option family 'ipv6'
	option target 'ACCEPT'

config rule
	option name 'Allow-IPSec-ESP'
	option src 'wan'
	option dest 'lan'
	option proto 'esp'
	option target 'ACCEPT'

config rule
	option name 'Allow-ISAKMP'
	option src 'wan'
	option dest 'lan'
	option dest_port '500'
	option proto 'udp'
	option target 'ACCEPT'

root@OpenWrt:~# 
```



### 重启network与firewall

```
# 重启network与firewall
root@OpenWrt:~# /etc/init.d/network restart
root@OpenWrt:~# /etc/init.d/firewall restart
```



### 测试tayga功能

```
# 测试tayga功能
root@OpenWrt:~# ping -6 ddbe:48ec:56c6:1111::8.8.8.8
PING ddbe:48ec:56c6:1111::8.8.8.8 (ddbe:48ec:56c6:1111::808:808): 56 data bytes
64 bytes from ddbe:48ec:56c6:1111::808:808: seq=0 ttl=51 time=57.846 ms
64 bytes from ddbe:48ec:56c6:1111::808:808: seq=1 ttl=51 time=58.418 ms
64 bytes from ddbe:48ec:56c6:1111::808:808: seq=2 ttl=51 time=57.077 ms
64 bytes from ddbe:48ec:56c6:1111::808:808: seq=3 ttl=51 time=57.571 ms
^C
--- ddbe:48ec:56c6:1111::8.8.8.8 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 57.077/57.728/58.418 ms
root@OpenWrt:~# 
root@OpenWrt:~# 
root@OpenWrt:~# ping -6 ddbe:48ec:56c6:1111::1.1.1.1
PING ddbe:48ec:56c6:1111::1.1.1.1 (ddbe:48ec:56c6:1111::101:101): 56 data bytes
64 bytes from ddbe:48ec:56c6:1111::101:101: seq=0 ttl=50 time=212.821 ms
64 bytes from ddbe:48ec:56c6:1111::101:101: seq=1 ttl=50 time=212.753 ms
64 bytes from ddbe:48ec:56c6:1111::101:101: seq=2 ttl=50 time=212.087 ms
64 bytes from ddbe:48ec:56c6:1111::101:101: seq=3 ttl=50 time=212.161 ms
^C
--- ddbe:48ec:56c6:1111::1.1.1.1 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 212.087/212.455/212.821 ms
root@OpenWrt:~# 
```



### 配置 bind-server 实现DNS64

```
# 配置 bind-server 实现DNS64
root@OpenWrt:~# opkg install bind-server
root@OpenWrt:~# 

root@OpenWrt:~# opkg install bind-rndc
root@OpenWrt:~# 

```



Bind是Tayga官方最推荐的DNS软件，因此接下就使用Bind来配置DNS64功能。Bind的配置项有很多，好在官方给出了详细的

https://downloads.isc.org/isc/bind9/9.16.7/doc/arm/html/reference.html#options-statement-grammar

Bind的配置需要修改 /etc/bind/named.conf 文件。对于DNS64来说，主要关注 forwarders 、dns64 、 dnssec-validation 这几个字段。

forwarders 用来表明要把Bind作为转发器来用，在 forwarders 里面指定要将收到的DNS请求转发给那些外部的DNS服务器。

dns64 这个字段需要指定在tayga中配置的NAT64前缀（这里的前缀可以有多个），并且其下面还有许多配置项。 clients 用来指定客户端ACL，来决定哪些客户端会受到DNS64的影响，默认为 any ；mapped 用来指定哪些IPv4地址要进行DNS64转换，默认为 any ； exclude 用来指定哪些出现在AAAA记录中的IPv6地址要被忽略，默认是 ::ffff:0.0.0.0/96 。

dnssec-validation 用来指定是否启用DNSSEC验证。 dnssec-enable 已被废除，在这里不起作用。



### 完整配置如下

```
# 完整配置如下

root@OpenWrt:~# vim /etc/bind/named.conf 
root@OpenWrt:~# cat /etc/bind/named.conf 
// This is the primary configuration file for the BIND DNS server named.

options {
	directory "/tmp";

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.
	listen-on port 53 { any; };
	listen-on-v6 port 53 { any; };
	allow-query { any; };
	allow-query-cache { any; };
	recursion yes;
	allow-recursion { any; };
	forwarders {
	    // 0.0.0.0;
	    202.106.46.151;
	    202.106.0.20;
	    //114.114.114.114;
	    //8.8.8.8;
	};
	dns64 ddbe:48ec:56c6:1111::/96 {
	clients { any; };
	mapped { any; };
	exclude { ddbe:48ec:56c6:1111::/96; ::ffff:0000:0000/96; };
	suffix ::;
	};
	dnssec-validation no;
	auth-nxdomain no; # conform to RFC1035

};

include "/etc/bind/named-rndc.conf";

include "/tmp/bind/named.conf.local";

// prime the server with knowledge of the root servers
zone "." {
	type hint;
	file "/etc/bind/db.root";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
	type master;
	file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
	type master;
	file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
	type master;
	file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
	type master;
	file "/etc/bind/db.255";
};
root@OpenWrt:~# 
```



```
# 重新DNS服务

# 关闭默认dnsmasq 
# 启用新安装named

root@OpenWrt:~# service dnsmasq stop
root@OpenWrt:~# service named start
root@OpenWrt:~# 
```

### 测试NAT64使用
![image-635bad167b22d](https://tc-1256707790.cos.ap-beijing.myqcloud.com/2022/10/28/635bad167b22d.jpg)

### 测试DNS64使用
![image-635bad160aad5](https://tc-1256707790.cos.ap-beijing.myqcloud.com/2022/10/28/635bad160aad5.jpg)





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
> **文章主要发布于微信**



