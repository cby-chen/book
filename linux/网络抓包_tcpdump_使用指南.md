---
layout: post
cid: 84
title: 网络抓包 tcpdump 使用指南
slug: 84
date: 2022/01/14 13:59:52
updated: 2022/01/14 13:59:52
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


 在网络问题的调试中，tcpdump应该说是一个必不可少的工具，和大部分linux下优秀工具一样，它的特点就是简单而强大。它是基于Unix系统的命令行式的数据包嗅探工具，可以抓取流动在网卡上的数据包。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/15a327784e3f45e5b7a80681c9d2c310~tplv-k3u1fbpfcp-zoom-1.image)

  

监听所有网卡所有包

```
tcpdump
```

  

监听指定网卡的包

```
tcpdump -i ens18
```

  

监听指定IP的包

```
tcpdump host 192.168.1.11
```

  

监听指定来源IP

```
tcpdump src host 192.168.1.11
```

  

监听目标地址IP

```
tcpdump dst host 192.168.1.11
```

  

监听指定端口

```
tcpdump port 80
```

  

监听TCP

```
tcpdump tcp
```

  

监听UDP

```
tcpdump udp
```

  

监听192.168.1.11的tcp协议的80端口的数据包

```
tcpdump tcp port 80 and src host 192.168.1.11

11:59:07.836563 IP 192.168.1.11.39680 > hello.http: Flags [.], ack 867022485, win 502, length 0
11:59:07.836711 IP 192.168.1.11.39680 > hello.http: Flags [P.], seq 0:77, ack 1, win 502, length 77: HTTP: HEAD / HTTP/1.1
11:59:07.838462 IP 192.168.1.11.39680 > hello.http: Flags [.], ack 248, win 501, length 0
11:59:07.838848 IP 192.168.1.11.39680 > hello.http: Flags [F.], seq 77, ack 248, win 501, length 0
11:59:07.839192 IP 192.168.1.11.39680 > hello.http: Flags [.], ack 249, win 501, length 0
```

  

  

监听IP之间的包

```
tcpdump ip host 192.168.1.11 and 192.168.1.60

11:57:52.742468 IP 192.168.1.11.38978 > hello.http: Flags [S], seq 3437424457, win 64240, options [mss 1460,sackOK,TS val 2166810854 ecr 0,nop,wscale 7], length 0
11:57:52.742606 IP hello.http > 192.168.1.11.38978: Flags [S.], seq 3541873211, ack 3437424458, win 64240, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0
11:57:52.742841 IP 192.168.1.11.38978 > hello.http: Flags [.], ack 1, win 502, length 0
11:57:52.742927 IP 192.168.1.11.38978 > hello.http: Flags [P.], seq 1:78, ack 1, win 502, length 77: HTTP: HEAD / HTTP/1.1
11:57:52.742943 IP hello.http > 192.168.1.11.38978: Flags [.], ack 78, win 502, length 0
11:57:52.744407 IP hello.http > 192.168.1.11.38978: Flags [P.], seq 1:248, ack 78, win 502, length 247: HTTP: HTTP/1.1 200 OK
11:57:52.744613 IP 192.168.1.11.38978 > hello.http: Flags [.], ack 248, win 501, length 0
11:57:52.744845 IP 192.168.1.11.38978 > hello.http: Flags [F.], seq 78, ack 248, win 501, length 0
11:57:52.745614 IP hello.http > 192.168.1.11.38978: Flags [F.], seq 248, ack 79, win 502, length 0
11:57:52.745772 IP 192.168.1.11.38978 > hello.http: Flags [.], ack 249, win 501, length 0
```

  

  

监听除了与192.168.1.4之外的数据包

```
tcpdump ip host 192.168.1.60 and ! 192.168.1.4

11:57:20.862575 IP 192.168.1.9.47190 > hello.9200: Flags [P.], seq 3233461117:3233461356, ack 1301434191, win 9399, length 239
11:57:20.878165 IP hello.9200 > 192.168.1.9.47190: Flags [P.], seq 1:4097, ack 239, win 3081, length 4096
11:57:20.878340 IP hello.9200 > 192.168.1.9.47190: Flags [P.], seq 4097:8193, ack 239, win 3081, length 4096
11:57:20.878417 IP 192.168.1.9.47190 > hello.9200: Flags [.], ack 4097, win 9384, length 0
```

  

  

组合示例

```
tcpdump tcp -i ens18 -v -nn -t -A -s 0 -c 50 and dst port ! 22 and src net 192.168.1.0/24 -w ./cby.cap

(1)tcp: ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型
(2)-i eth1 : 只抓经过接口eth1的包
(3)-t : 不显示时间戳
(4)-s 0 : 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包
(5)-c 50 : 只抓取50个数据包
(6)dst port ! 22 : 不抓取目标端口是22的数据包
(7)src net 192.168.1.0/24 : 数据包的源网络地址为192.168.1.0/24
(8)-w ./cby.cap : 保存成cap文件，方便用ethereal(即wireshark)分析
(9)-v 使用 -v，-vv 和 -vvv 来显示更多的详细信息，通常会显示更多与特定协议相关的信息。
(10)-nn 单个 n 表示不解析域名，直接显示 IP；两个 n 表示不解析域名和端口。
(11)-A 表示使用 ASCII 字符串打印报文的全部数据
```

  

```
组合过滤器  《与/AND/&&》 《或/OR/||》 《非/not/!》
and or &&
or or ||
not or !
```

  

  

  

在HTTP中提取用户头

  

```
tcpdump -nn -A -s0 -l | grep "User-Agent:"

User-Agent: Prometheus/2.30.0
User-Agent: Microsoft-Delivery-Optimization/10.0
```

  

  

在HTTP中同时提取用户头和主机信息

```
tcpdump -nn -A -s0 -l | egrep -i 'User-Agent:|Host:'

Host: 192.168.1.42:9200
User-Agent: Prometheus/2.30.0
HOST: 239.255.255.250:1900
USER-AGENT: Microsoft Edge/97.0.1072.55 Windows
```

  

  

抓取 HTTP GET 流量

```
tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'

11:55:13.704801 IP (tos 0x0, ttl 64, id 14605, offset 0, flags [DF], proto TCP (6), length 291)
    localhost.35498 > localhost.9200: Flags [P.], cksum 0x849a (incorrect -> 0xd0b0), seq 3090925559:3090925798, ack 809492640, win 630, options [nop,nop,TS val 2076158003 ecr 842090965], length 239
E..#9.@.@.}C... ...+..#..;..0?.....v.......
{..321I.GET /metrics HTTP/1.1
Host: 192.168.1.43:9200
User-Agent: Prometheus/2.30.0
Accept: application/openmetrics-text; version=0.0.1,text/plain;version=0.0.4;q=0.5,*/*;q=0.1
Accept-Encoding: gzip
X-Prometheus-Scrape-Timeout-Seconds: 10
```

  

抓取 HTTP POST 请求流量

  

```
tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354'

11:53:10.831855 IP (tos 0x0, ttl 63, id 0, offset 0, flags [none], proto TCP (6), length 643)
    localhost.47702 > dns50.online.tj.cn.http-alt: Flags [P.], cksum 0x1a41 (correct), seq 3331055769:3331056372, ack 799860501, win 4096, length 603: HTTP, length: 603
        POST /?tk=391f8956e632962ee9c1dc661a9b46779d86ca43fe252bddbfc09d2cc66bf875323f6e7f03b881db21133b1bf2ae5bc5 HTTP/1.1
        Host: 220.194.116.50:8080
        Accept: */*
        Accept-Language: zh-CN,zh-Hans;q=0.9
        Q-Guid: e54764008893a559b852b6e9f1c8ae268958471308f41a96fd42e477e26323b8
        Q-UA: 
        Accept-Encoding: gzip,deflate
        Q-UA2: QV=3&PL=IOS&RF=SDK&PR=IBS&PP=com.tencent.mqq&PPVN=3.8.0.1824&TBSVC=18500&DE=PHONE&VE=GA&CO=IMTT&RL=1170*2532&MO=iPhone14,2&CHID=50001&LCID=9751&OS=15.1.1
        Content-Length: 144
        User-Agent: 
        QQ-S-ZIP: gzip
        Connection: keep-alive
        Content-Type: application/multipart-formdata
        Q-Auth: 

E.......?.f.......t2.V....../...P....A..POST /?tk=391f8956e632962ee9c1dc661a9b46779d86ca43fe252bddbfc09d2cc66bf875323f6e7f03b881db21133b1bf2ae5bc5 HTTP/1.1
Host: 220.194.116.50:8080
Accept: */*
Accept-Language: zh-CN,zh-Hans;q=0.9
Q-Guid: e54764008893a559b852b6e9f1c8ae268958471308f41a96fd42e477e26323b8
Q-UA: 
Accept-Encoding: gzip,deflate
Q-UA2: QV=3&PL=IOS&RF=SDK&PR=IBS&PP=com.tencent.mqq&PPVN=3.8.0.1824&TBSVC=18500&DE=PHONE&VE=GA&CO=IMTT&RL=1170*2532&MO=iPhone14,2&CHID=50001&LCID=9751&OS=15.1.1
Content-Length: 144
User-Agent: 
QQ-S-ZIP: gzip
Connection: keep-alive
Content-Type: application/multipart-formdata
Q-Auth:
```

  

注意：一个 POST 请求会被分割为多个 TCP 数据包

  

  

提取 HTTP 请求的主机名和路径

```
root@pve:~# tcpdump -s 0 -v -n -l | egrep -i "POST /|GET /|Host:"

tcpdump: listening on eno1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
        GET /gchatpic_new/2779153238/851197814-3116860870-F4902AF1432FE48B812982F082A31097/0?term=255&pictype=0 HTTP/1.1
        Host: 112.80.128.33
        GET /gchatpic_new/2779153238/851197814-3116860870-F4902AF1432FE48B812982F082A31097/0?term=255&pictype=0 HTTP/1.1
        Host: 112.80.128.33
        POST /mmtls/74ce36ed HTTP/1.1
        Host: extshort.weixin.qq.com
        POST /mmtls/74ce36ed HTTP/1.1
        Host: extshort.weixin.qq.com
```

从 HTTP 请求中提取密码和主机名

```
tcpdump -s 0 -v -n -l | egrep -i "POST /|GET /|pwd=|passwd=|password=|Host:"

        POST /index.php/action/login?_=b395d487431320461e9a6741e3828918 HTTP/1.1
        Host: x.oiox.cn
        name=cby&password=Cby****&referer=http%3A%2F%2Fx.oiox.cn%2Fadmin%2Fwelcome.php [|http]
        POST /index.php/action/login?_=b395d487431320461e9a6741e3828918 HTTP/1.1
        Host: x.oiox.cn
        name=cby&password=Cby****&referer=http%3A%2F%2Fx.oiox.cn%2Fadmin%2Fwelcome.php [|http]
        GET /admin/welcome.php HTTP/1.1
        Host: x.oiox.cn
```

从 HTTP 请求中提取Cookie信息

  

```
tcpdump -nn -A -s0 -l -v | egrep -i 'Set-Cookie|Host:|Cookie:'

Host: x.oiox.cn
Cookie: 8bf110c223e1a04b7b63ca5aa97c9f61__typecho_uid=1; 8bf110cxxxxxxxb7b63ca5aa97c9f61__typecho_authCode=%24T%24W3hV7B9vRfefa6593049ba02c33b3c4796a7cfa35; PHPSESSID=bq67s1n0cb9ml6dq254qpdvfec
```

  

  

通过排除 echo 和 reply 类型的数据包使抓取到的数据包不包括标准的 ping 包

  

```
tcpdump 'icmp[icmptype] != icmp-echo and icmp[icmptype] != icmp-echoreply'

11:20:32.285428 IP localhost > localhost: ICMP localhost udp port 64594 unreachable, length 36
11:20:32.522061 IP localhost > localhost: ICMP localhost udp port 58617 unreachable, length 36
11:20:37.736249 IP localhost > localhost: ICMP redirect 204.79.197.219 to host localhost, length 48
11:20:44.379646 IP localhost > 111.206.187.34: ICMP localhost udp port 37643 unreachable, length 36
11:20:44.379778 IP localhost > 111.206.187.34: ICMP localhost udp port 37643 unreachable, length 36
11:20:46.351245 IP localhost > localhost: ICMP redirect lt-in-f188.1e100.net to host localhost, length 49
```

  

  

可以通过过滤器 ip6 来抓取 IPv6 流量，同时可以指定协议如 TCP

  

```
root@vm371841:~# tcpdump -nn ip6 proto 6 -v

tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
06:40:26.060313 IP6 (flowlabel 0xfe65e, hlim 64, next-header TCP (6) payload length: 40) 2a00:b700::e831:2aff:fe27:e9d9.44428 > 2001:2030:21:181::26e7.443: Flags [S], cksum 0x451c (incorrect -> 0x24cd), seq 3503520271, win 64800, options [mss 1440,sackOK,TS val 2504544710 ecr 0,nop,wscale 6], length 0
06:40:34.296847 IP6 (flowlabel 0xc9f9c, hlim 64, next-header TCP (6) payload length: 40) 2a00:b700::e831:2aff:fe27:e9d9.55082 > 2a00:1450:4010:c0e::84.443: Flags [S], cksum 0x6754 (incorrect -> 0x0813), seq 3899361154, win 64800, options [mss 1440,sackOK,TS val 2141524802 ecr 0,nop,wscale 6], length 0
```

  

  

发起的出站 DNS 请求和 A 记录响应

  

```
tcpdump -i eth0 -s0 port 53

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
06:44:10.499529 IP vm371841.37357 > dns.yandex.ru.domain: 34151+ [1au] A? czr12g1e.slt-dk.sched.tdnsv8.com. (61)
06:44:10.500992 IP vm371841.56195 > dns.yandex.ru.domain: 45667+ [1au] PTR? 219.3.144.45.in-addr.arpa. (54)
06:44:10.661142 IP dns.yandex.ru.domain > vm371841.56195: 45667 NXDomain 0/1/1 (112)
06:44:10.661438 IP vm371841.56195 > dns.yandex.ru.domain: 45667+ PTR? 219.3.144.45.in-addr.arpa. (43)
06:44:10.687147 IP dns.yandex.ru.domain > vm371841.56195: 45667 NXDomain 0/1/0 (101)
06:44:10.806349 IP dns.yandex.ru.domain > vm371841.37357: 34151 11/0/1 A 139.170.156.155, A 220.200.129.141, A 58.243.200.63, A 113.59.43.25, A 124.152.41.39, A 139.170.156.154, A 59.83.204.154, A 123.157.255.158, A 113.200.17.157, A 43.242.166.42, A 116.177.248.23 (237)
```

  

  

抓取 DHCP 服务的请求和响应报文

```
tcpdump -v -n port 67 or 68

11:50:28.939726 IP (tos 0x0, ttl 64, id 35862, offset 0, flags [DF], proto UDP (17), length 320)
    192.168.1.136.68 > 255.255.255.255.67: BOOTP/DHCP, Request from 70:3a:a6:cb:27:3c, length 292, xid 0x3ccba40c, secs 11529, Flags [none]
          Client-IP 192.168.1.136
          Client-Ethernet-Address 70:3a:a6:cb:27:3c
          Vendor-rfc1048 Extensions
            Magic Cookie 0x63825363
            DHCP-Message (53), length 1: Request
            Client-ID (61), length 7: ether 70:3a:a6:cb:27:3c
            Hostname (12), length 11: "S24G-U_273C"
            Vendor-Class (60), length 13: "CloudSwitch_1"
            MSZ (57), length 2: 800
            Parameter-Request (55), length 5: 
              Subnet-Mask (1), Default-Gateway (3), Hostname (12), Domain-Name-Server (6)
              Vendor-Class (60)
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
