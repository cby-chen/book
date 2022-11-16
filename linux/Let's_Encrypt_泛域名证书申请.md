---
layout: post
cid: 139
title: Let's Encrypt 泛域名证书申请
slug: 139
date: 2022/04/15 14:58:00
updated: 2022/07/27 10:41:31
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


泛域名
===

泛域名证书又名通配符证书是SSL证书中的其中一种形式，一般会以通配符的形式(如：\*.domain.com)来指定证书所要保护的域名。

OV证书和DV证书都会有通配符的域名形式提供，而EV证书一般没有通配符的证书形式。

1.配置灵活方便

由于采用了通配符的形式对域名进行配置，那么对于拥有多个二级域名的网站是一件非常便利的事情。只要申请一张通配符证书，就能用于所有的二级域名网站中。而且如果以后需要继续增加二级域名，也不需要再去申请购买证书，只需继续使用原有的证书就可以，对于网站管理者来说确实是非常的方便。

2.性价比高

一般而言，通配符证书是会比单域名证书会贵上不少，但是假如按每个二级域名的证书价格摊分下来，那其实证书单价是及其的低。当然这要看你的二级域名数量总数有多少而定。但如今互联网时代，很多公司企业他们都会用户多个二级域名。对于这些企业而言，通配符证书无疑是一种高性价比的SSL证书。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54bf914a089f4666ad17c100745bc89c~tplv-k3u1fbpfcp-zoom-1.image)

安装所需环境
======

```
root@cby:~# apt-get install socat -y


root@cby:~# curl  https://get.acme.sh | sh -s email=cby@chenby.cn
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   937    0   937    0     0    788      0 --:--:--  0:00:01 --:--:--   789
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  210k  100  210k    0     0   131k      0  0:00:01  0:00:01 --:--:--  131k
[Fri 15 Apr 2022 11:54:09 AM CST] Installing from online archive.
[Fri 15 Apr 2022 11:54:09 AM CST] Downloading https://github.com/acmesh-official/acme.sh/archive/master.tar.gz
[Fri 15 Apr 2022 11:54:11 AM CST] Extracting master.tar.gz
[Fri 15 Apr 2022 11:54:11 AM CST] Installing to /root/.acme.sh
[Fri 15 Apr 2022 11:54:11 AM CST] Installed to /root/.acme.sh/acme.sh
[Fri 15 Apr 2022 11:54:11 AM CST] Installing alias to '/root/.bashrc'
[Fri 15 Apr 2022 11:54:11 AM CST] OK, Close and reopen your terminal to start using acme.sh
[Fri 15 Apr 2022 11:54:11 AM CST] Installing cron job
49 0 * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
[Fri 15 Apr 2022 11:54:11 AM CST] Good, bash is found, so change the shebang to use bash as preferred.
[Fri 15 Apr 2022 11:54:12 AM CST] OK
[Fri 15 Apr 2022 11:54:12 AM CST] Install success!
root@cby:~# 

```

进入导入环境变量并提出申请
=============

```
root@cby:~# cd .acme.sh/


root@cby:~/.acme.sh# export DP_Id="abcd"
root@cby:~/.acme.sh# export DP_Key="xxxxxxxxxx"


root@cby:~/.acme.sh# ./acme.sh --issue --dns dns_dp -d *.oiox.cn -d oiox.cn
[Fri 15 Apr 2022 12:05:13 PM CST] Using CA: https://acme.zerossl.com/v2/DV90
[Fri 15 Apr 2022 12:05:13 PM CST] Multi domain='DNS:*.oiox.cn,DNS:oiox.cn'
[Fri 15 Apr 2022 12:05:13 PM CST] Getting domain auth token for each domain
[Fri 15 Apr 2022 12:05:38 PM CST] Getting webroot for domain='*.oiox.cn'
[Fri 15 Apr 2022 12:05:38 PM CST] Getting webroot for domain='oiox.cn'
[Fri 15 Apr 2022 12:05:39 PM CST] Adding txt value: DDuc5hd3b1RIoa5BefBkA53EpEtbAY0Fk8jOVVJcL6E for domain:  _acme-challenge.oiox.cn
[Fri 15 Apr 2022 12:05:39 PM CST] Adding record
[Fri 15 Apr 2022 12:05:39 PM CST] The txt record is added: Success.
[Fri 15 Apr 2022 12:05:40 PM CST] Adding txt value: 43GHnhiHjyxCxsdHSDRDP_A4YqP8dDjc_9YgnkFNk5I for domain:  _acme-challenge.oiox.cn
[Fri 15 Apr 2022 12:05:40 PM CST] Adding record
[Fri 15 Apr 2022 12:05:40 PM CST] The txt record is added: Success.
[Fri 15 Apr 2022 12:05:40 PM CST] Let's check each DNS record now. Sleep 20 seconds first.
[Fri 15 Apr 2022 12:06:01 PM CST] You can use '--dnssleep' to disable public dns checks.
[Fri 15 Apr 2022 12:06:01 PM CST] See: https://github.com/acmesh-official/acme.sh/wiki/dnscheck
[Fri 15 Apr 2022 12:06:02 PM CST] Checking oiox.cn for _acme-challenge.oiox.cn
[Fri 15 Apr 2022 12:06:04 PM CST] Domain oiox.cn '_acme-challenge.oiox.cn' success.
[Fri 15 Apr 2022 12:06:04 PM CST] Checking oiox.cn for _acme-challenge.oiox.cn
[Fri 15 Apr 2022 12:06:05 PM CST] Domain oiox.cn '_acme-challenge.oiox.cn' success.
[Fri 15 Apr 2022 12:06:05 PM CST] All success, let's return
[Fri 15 Apr 2022 12:06:05 PM CST] Verifying: *.oiox.cn
[Fri 15 Apr 2022 12:06:17 PM CST] Processing, The CA is processing your order, please just wait. (1/30)
[Fri 15 Apr 2022 12:06:24 PM CST] Success
[Fri 15 Apr 2022 12:06:24 PM CST] Verifying: oiox.cn
[Fri 15 Apr 2022 12:06:31 PM CST] Processing, The CA is processing your order, please just wait. (1/30)
[Fri 15 Apr 2022 12:06:34 PM CST] Success
[Fri 15 Apr 2022 12:06:34 PM CST] Removing DNS records.
[Fri 15 Apr 2022 12:06:34 PM CST] Removing txt: DDuc5hd3b1RIoa5BefBkA53EpEtbAY0Fk8jOVVJcL6E for domain: _acme-challenge.oiox.cn
[Fri 15 Apr 2022 12:06:35 PM CST] Removed: Success
[Fri 15 Apr 2022 12:06:35 PM CST] Removing txt: 43GHnhiHjyxCxsdHSDRDP_A4YqP8dDjc_9YgnkFNk5I for domain: _acme-challenge.oiox.cn
[Fri 15 Apr 2022 12:06:36 PM CST] Removed: Success
[Fri 15 Apr 2022 12:06:36 PM CST] Verify finished, start to sign.
[Fri 15 Apr 2022 12:06:36 PM CST] Lets finalize the order.
[Fri 15 Apr 2022 12:06:36 PM CST] Le_OrderFinalize='https://acme.zerossl.com/v2/DV90/order/G4Sy37Y-eHjHX1wLMAh5nA/finalize'
[Fri 15 Apr 2022 12:06:44 PM CST] Order status is processing, lets sleep and retry.
[Fri 15 Apr 2022 12:06:44 PM CST] Retry after: 15
[Fri 15 Apr 2022 12:07:00 PM CST] Polling order status: https://acme.zerossl.com/v2/DV90/order/G4Sy37Y-eHjHX1wLMAh5nA
[Fri 15 Apr 2022 12:07:03 PM CST] Downloading cert.
[Fri 15 Apr 2022 12:07:03 PM CST] Le_LinkCert='https://acme.zerossl.com/v2/DV90/cert/r4l-4WevkiEwiZA3U340ig'
[Fri 15 Apr 2022 12:07:10 PM CST] Cert success.
-----BEGIN CERTIFICATE-----
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
E9Q19NWLnTA2YU52
-----END CERTIFICATE-----
[Fri 15 Apr 2022 12:07:10 PM CST] Your cert is in: /root/.acme.sh/*.oiox.cn/*.oiox.cn.cer
[Fri 15 Apr 2022 12:07:10 PM CST] Your cert key is in: /root/.acme.sh/*.oiox.cn/*.oiox.cn.key
[Fri 15 Apr 2022 12:07:10 PM CST] The intermediate CA cert is in: /root/.acme.sh/*.oiox.cn/ca.cer
[Fri 15 Apr 2022 12:07:10 PM CST] And the full chain certs is there: /root/.acme.sh/*.oiox.cn/fullchain.cer

```

查看已申请出来证书
=========

```
root@cby:~/.acme.sh# cd \*.oiox.cn
root@cby:~/.acme.sh/*.oiox.cn# ll
total 44
drwxr-xr-x 2 root root 4096 Jul 27 10:31  ./
drwx------ 7 root root 4096 Jul 27 10:28  ../
-rw-r--r-- 1 root root 4399 Jul 27 10:31  ca.cer
-rw-r--r-- 1 root root 6684 Jul 27 10:31  fullchain.cer
-rw-r--r-- 1 root root 2285 Jul 27 10:31 '*.oiox.cn.cer'
-rw-r--r-- 1 root root  556 Jul 27 10:31 '*.oiox.cn.conf'
-rw-r--r-- 1 root root  956 Jul 27 10:28 '*.oiox.cn.csr'
-rw-r--r-- 1 root root  156 Jul 27 10:28 '*.oiox.cn.csr.conf'
-rw------- 1 root root 1679 Jul 27 10:28 '*.oiox.cn.key'
root@cby:~/.acme.sh/*.oiox.cn#
```

Nginx部署证书
=========

```
示例：
server {
        listen 80;
        listen [::]:80;

        listen 443 ssl;
        listen [::]:443;
        ssl_certificate /ssl/cert.pem;
        ssl_certificate_key /ssl/cert.key;
        ssl_session_timeout  5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        server_name dns.oiox.cn;

        root /var/www/dns;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }
}

# 转化证书
acme.sh --install-cert -d *.oiox.cn --key-file  /ssl/cert.key  --fullchain-file /ssl/cert.pem --reloadcmd  "service nginx force-reload"
[Wed 27 Jul 2022 10:34:41 AM CST] Installing key to: /ssl/cert.key
[Wed 27 Jul 2022 10:34:41 AM CST] Installing full chain to: /ssl/cert.pem
[Wed 27 Jul 2022 10:34:41 AM CST] Run reload cmd: service nginx force-reload
[Wed 27 Jul 2022 10:34:41 AM CST] Reload success

```
证书更新
=====

目前通过 acme.sh 生成的证书会在60天过期
安装 acme.sh 时会自动创建一个 cronjob，每天定期检查所有证书，如果证书需要更新会自动更新证书。

```
# 1.手动更新
acme.sh --renew -d example.com --force

# 2.自动更新
crontab -l
12 0 * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null

# 3.主程序自动更新
acme.sh  --upgrade  --auto-upgrade
```

附录
=====

```
上面使用DNSPOD进行动态解析申请证书

阿里云DNS申请
export Ali_Key="abcd"
export Ali_Secret="xxxxxxxxxx"
# RSA 证书
acme.sh --issue --dns dns_ali -d blog.exsvc.cn -d *.exsvc.cn
# ECC 证书
acme.sh --issue --dns dns_ali -d blog.exsvc.cn -d *.exsvc.cn --keylength ec-256

腾讯云DNS申请
root@cby:~/.acme.sh# export DP_Id="abcd"
root@cby:~/.acme.sh# export DP_Key="xxxxxxxxxx"
root@cby:~/.acme.sh# ./acme.sh --issue --dns dns_dp -d *.oiox.cn -d oiox.cn

更多申请方式见：https://github.com/acmesh-official/acme.sh/wiki/dnsapi

```

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