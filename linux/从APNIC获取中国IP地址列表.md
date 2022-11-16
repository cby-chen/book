---
layout: post
cid: 18
title: 从APNIC获取中国IP地址列表
slug: 18
date: 2021/12/30 17:02:22
updated: 2021/12/30 17:02:22
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/687a7e51eed640e386cc22bd3091e60a~tplv-k3u1fbpfcp-zoom-1.image)

  

**关于APNIC**  

    全球IP地址块被IANA(Internet Assigned Numbers Authority)分配给全球三大地区性IP地址分配机构，它们分别是：

  

**ARIN (American Registry for Internet Numbers)**

    负责北美、南美、加勒比以及非洲撒哈啦部分的IP地址分配。同时还要给全球NSP(Network Service Providers)分配地址。

  

**RIPE (Reseaux IP Europeens)**

    负责欧洲、中东、北非、西亚部分地区(前苏联)

  

**APNIC (Asia Pacific Network Information Center)**

    负责亚洲、太平洋地区

  

  

  

**APNIC IP地址分配信息总表的获取：**

  

```
APNIC提供了每日更新的亚太地区IPv4，IPv6，AS号分配的信息表：http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest
```

  

```
该文件的格式与具体内容参见：ftp://ftp.apnic.net/pub/apnic/stats/apnic/README.TXT
```

  

  

通过该文件我们能够得到APNIC辖下IPv4地址空间的分配情况。

  

**脚本获取IP地址**

  

```
#!/bin/bash
wget -c http://ftp.apnic.net/stats/apnic/delegated-apnic-latest

cat delegated-apnic-latest | awk -F '|' '/CN/&&/ipv4/ {print $4 "/" 32-log($5)/log(2)}' | cat > ipv4.txt

cat delegated-apnic-latest | awk -F '|' '/CN/&&/ipv6/ {print $4 "/" 32-log($5)/log(2)}' | cat > ipv6.txt

cat delegated-apnic-latest | awk -F '|' '/HK/&&/ipv4/ {print $4 "/" 32-log($5)/log(2)}' | cat > ipv4-hk.txt

cat delegated-apnic-latest | awk -F '|' '/HK/&&/ipv6/ {print $4 "/" 32-log($5)/log(2)}' | cat > ipv6-hk.txt
```

  

**执行脚本：**  

```
[root@cby cby]# ./ip.sh 
--2021-04-29 12:17:13--  http://ftp.apnic.net/stats/apnic/delegated-apnic-latest
Resolving ftp.apnic.net (ftp.apnic.net)... 203.119.102.40, 2001:dd8:8:701::40
Connecting to ftp.apnic.net (ftp.apnic.net)|203.119.102.40|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3352151 (3.2M) [text/plain]
Saving to: ‘delegated-apnic-latest’

delegated-apnic-latest            100%[=============================================================>]   3.20M  61.3KB/s    in 44s     

2021-04-29 12:17:58 (74.0 KB/s) - ‘delegated-apnic-latest’ saved [3352151/3352151]
[root@cby cby]# ls
delegated-apnic-latest  index.html  ip.sh  ipv4-hk.txt  ipv4.txt  ipv6-hk.txt  ipv6.txt

```

  

  

每日凌晨十二点十分会进行同步，若需要IP地址，可以访问如下地址：

**http://aliyun.chenby.cn/**

  

**定时任务：**  

```
[root@cby cby]# crontab -l
10 0 * * *  /www/server/cron/3ab48c27ec99cb9787749c362afae517 >> /www/server/cron/3ab48c27ec99cb9787749c362afae517.log 2>&1
10 0 * * *  rm -rf /www/wwwroot/www.chenby.cn/cby/ipv4.txt /www/wwwroot/www.chenby.cn/cby/ipv4-hk.txt /www/wwwroot/www.chenby.cn/cby/ipv6.txt /www/wwwroot/www.chenby.cn/cby/ipv6-hk.txt /www/wwwroot/www.chenby.cn/cby/delegated-apnic-latest 
11 0 * * *  /www/wwwroot/www.chenby.cn/cby/ip.sh >> /home/ip.txt
```

  

```