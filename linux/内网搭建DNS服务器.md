---
layout: post
cid: 67
title: 内网搭建DNS服务器
slug: 67
date: 2021/12/30 17:14:54
updated: 2021/12/30 17:14:54
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


# DNS:Domain Name Service，域名解析服务

  

监听端口：udp/53，tcp/53

应用程序：bind

根域：.

一级域：

组织域：.com, .org, .net, .mil, .edu, .gov, .info, .cc, .me, .tv

国家域：.cn, .us, .uk, .jp, .tw, .hk, .iq, .ir

反向域：.in-addr.arpa

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d0171623b2441a4b070fe95db0a1a4e~tplv-k3u1fbpfcp-zoom-1.image)

  

**DNS 记录类型**：DNS 域名数据库由资源记录和区文件指令组成。

  

**SOA 记录**：起始授权机构记录，SOA 备注说明了众多 NS（name server）记录中谁是主名称服务器，不参与功能，但是不能缺少。

  

**NS 记录**：域授权记录，当请求到达根域的时候，通过 NS 记录找到对应的域。

  

**A 记录**：当通过 NS 记录到达域以后，比如访问 www.baidu.com，通过 NS 我们找到了 baidu.com，此时就需要通过 A 记录找到 www。

  

**MX**：将该域下的所有邮件服务器地址指向邮件服务器。

  

**AAAA 记录**：A 记录处理 IPV4，AAAA 处理 IPV6。

  

**PTR 记录**：反向解析，将 IP 解析成域名。

  

**CNAME**：别名记录，允许多个名字映射到另外一个域名。比如我们 ping 百度的时候可以发现返回其实是 www.a.shifen.com 这个域名返回。所有 www.baidu.com 其实是个别名。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d28428c41d754b6cac837fc3abf9f32a~tplv-k3u1fbpfcp-zoom-1.image)

  

  

# 安装dns服务并配置

  

```
[root@jhr-hub ~]# yum -y install bind-utils bind bind-devel bind-libs


[root@jhr-hub ~]# vim /etc/named.rfc1912.zones
[root@jhr-hub ~]# 
[root@jhr-hub ~]# 
[root@jhr-hub ~]# 
[root@jhr-hub ~]# tail -n 10 /etc/named.rfc1912.zones


zone "chenby.cn" IN {     
        type master;
        file "chenby.cn.zone";  
};
[root@jhr-hub ~]# 


[root@jhr-hub ~]# cd /var/named/
[root@jhr-hub named]# ls
data  dynamic  named.ca  named.empty  named.localhost  named.loopback  pakho.zone  slaves
[root@jhr-hub named]# 
[root@jhr-hub named]# cp named.localhost chenby.cn.zone
[root@jhr-hub named]# 
[root@jhr-hub named]# chown named.named chenby.cn.zone
[root@jhr-hub named]# 
[root@jhr-hub named]# vim chenby.cn.zone
[root@jhr-hub named]#
```

  

# 检查配置文件  

  

```
[root@jhr-hub named]# named-checkconf /etc/named.conf
[root@jhr-hub named]# 
[root@jhr-hub named]# 
[root@jhr-hub named]# named-checkzone chenby.cn /var/named/chenby.cn.zone 
zone chenby.cn/IN: loaded serial 0
OK
[root@jhr-hub named]#
```

  

  

# 启动服务，并设置开机自启  

  

```
[root@jhr-hub named]# systemctl restart named
[root@jhr-hub named]# 
[root@jhr-hub named]# systemctl enable named
Created symlink from /etc/systemd/system/multi-user.target.wants/named.service to /usr/lib/systemd/system/named.service.
[root@jhr-hub named]# 


测试是否可行

[root@jhr-hub named]# dig @3.7.191.1 www.chenby.cn


; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.7 <<>> @3.7.191.1 www.chenby.cn
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5275
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3


;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.chenby.cn.                 IN      A


;; ANSWER SECTION:
www.chenby.cn.          86400   IN      A       3.7.191.1


;; AUTHORITY SECTION:
chenby.cn.              86400   IN      NS      chenby.cn.


;; ADDITIONAL SECTION:
chenby.cn.              86400   IN      A       127.0.0.1
chenby.cn.              86400   IN      AAAA    ::1


;; Query time: 0 msec
;; SERVER: 3.7.191.1#53(3.7.191.1)
;; WHEN: Thu Dec 09 14:44:51 CST 2021
;; MSG SIZE  rcvd: 116


[root@jhr-hub named]#
```

  

  

# 附录：  

  

# 1.name.conf文件详解

  

```
options {
listen-on port 53 { 127.0.0.1; };      //设置named服务器监听端口及IP地址
listen-on-v6 port 53 { ::1; };
directory       "/var/named";    //设置区域数据库文件的默认存放地址
dump-file       "/var/named/data/cache_dump.db";
statistics-file "/var/named/data/named_stats.txt";
memstatistics-file "/var/named/data/named_mem_stats.txt";


allow-query     { any; };   //允许DNS查询客户端
allow-query-cache { any; };
};
logging {
channel default_debug {
file "data/named.run";
severity dynamic;
};
};
view localhost_resolver {
match-clients      { any; };
match-destinations { any; };
recursion yes;                  //设置允许递归查询
include "/etc/named.rfc1912.zones";
};
```

  

# 2.区域配置文件/etc/named.rfc1912.zones

  

```
zone "." IN {    //定义了根域
type hint;       //定义服务器类型为hint
file "named.ca";  //定义根域的配置文件名
};


zone "localdomain" IN {   //定义正向DNS区域
type master;              //定义区域类型
file "localdomain.zone";  //设置对应的正向区域地址数据库文件
allow-update { none; };   //设置允许动态更新的客户端地址（none为禁止）
};


zone "localhost" IN {
type master;
file "localhost.zone";
allow-update { none; };
};


zone "0.0.127.in-addr.arpa" IN {   //设置反向DNS区域
type master;
file "named.local";
allow-update { none; };
};
```

  

# 3.根域配置文件named.ca

  

根域配置文件设定根域的域名数据库，包括根域中13台DNS服务器的信息。几乎所有系统的这个文件都是一样的，用户不需要进行修改。

  

# 4.正向域名解析数据库文件

  

```
$TTL 600
@        IN   SOA    dns.cwlinux.com   dnsadmin.cwlinux.com. (//SOA字段
                          2015031288   //版本号    同步一次  +1
                             1H        //更新时间
                             2M        // 更新失败，重试更新时间
                             2D        // 更新失败多长时间后此DNS失效时间
                             1D        //解析不到请求不予回复时间
)
         IN    NS   dns            //有两域名服务器
         IN    NS   ns2
         IN    MX  10 mial        // 定义邮件服务器，10指优先级  0-99 数字越小优先级越高
ns2      IN    A    192.168.1.113  //ns2域名服务器的ip地址
dns      IN    A    192.168.1.10   //dns域名服务器的ip地址
mail     IN    A    192.168.1.111   //邮件服务器的ip地址
www      IN    A    192.168.1.112   //www.cwlinux.com的ip地址
pop      IN   CNAME  mail         //pop的正式名字是mail
ftp      IN   CNAME  www         //ftp的正式名字是www
```

  

# 5.反向域名解析数据库文件

  

```
$TTL 600
@         IN   SOA    dns.cwlinux.com.   dnsadmin.cwlinux.com. (
                             2014031224
                             1H
                             2M
                             2D
                             1D
)
         IN   NS      dns.cwlinux.com.
10       IN   PTR     dns.cwlinux.com.     //反向解析PTR格式
111       IN   PTR     mail.cwlinux.com.
112       IN   PTR     www.cwlinux.com.
// 声明域的时候已经有了，192.168.1 所以我们只需要输入10即代表192.168.1.10jc
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