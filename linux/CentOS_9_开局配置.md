---
layout: post
cid: 288
title: CentOS 9 开局配置
slug: 288
date: 2022/09/25 19:11:36
updated: 2022/09/25 19:14:32
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: CentOS 9 开局配置
keywords: 小陈运维,小陈,小陈博客,文档,
mode: default
thumb: 
video: 
---


#  CentOS 9 开局配置 

CentOS 9 发布有几年了，一直没有尝试使用，CentOS 9 有一些变动。

### 查看系统基础信息
```
# 查看系统基础信息
[root@chenby ~]# neofetch
                 ..                    cby@chenby
               .PLTJ.                  ----------
              <><><><>                 OS: CentOS Stream 9 x86_64
     KKSSV' 4KKK LJ KKKL.'VSSKK        Host: VMware Virtual Platform None
     KKV' 4KKKKK LJ KKKKAL 'VKK        Kernel: 5.14.0-165.el9.x86_64
     V' ' 'VKKKK LJ KKKKV' ' 'V        Uptime: 1 min
     .4MA.' 'VKK LJ KKV' '.4Mb.        Packages: 651 (rpm)
   . KKKKKA.' 'V LJ V' '.4KKKKK .      Shell: bash 5.1.8
 .4D KKKKKKKA.'' LJ ''.4KKKKKKK FA.    Resolution: 800x600
<QDD ++++++++++++  ++++++++++++ GFD>   Terminal: /dev/pts/0
 'VD KKKKKKKK'.. LJ ..'KKKKKKKK FV     CPU: AMD Ryzen 9 3950X (32) @ 3.500GHz
   ' VKKKKK'. .4 LJ K. .'KKKKKV '      GPU: 00:0f.0 VMware SVGA II Adapter
      'VK'. .4KK LJ KKA. .'KV'         Memory: 375MiB / 7909MiB
     A. . .4KKKK LJ KKKKA. . .4
     KKA. 'KKKKK LJ KKKKK' .4KK
     KKSSA. VKKK LJ KKKV .4SSKK
              <><><><>
               'MKKM'
                 ''

[root@chenby ~]#

```

### 使用国内镜像源
```
# 使用国内镜像源

cat >  /etc/yum.repos.d/centos.repo << EOF 
[AppStream]
name=CentOS-\$releasever - AppStream - mirrors.ustc.edu.cn
#failovermethod=priority
baseurl=https://mirrors.ustc.edu.cn/centos-stream/\$stream/AppStream/\$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos-stream/RPM-GPG-KEY-CentOS-Official

[BaseOS]
name=CentOS-\$releasever - BaseOS - mirrors.ustc.edu.cn
#failovermethod=priority
baseurl=https://mirrors.ustc.edu.cn/centos-stream/\$stream/BaseOS/\$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos-stream/RPM-GPG-KEY-CentOS-Official

[CRB]
name=CentOS-\$releasever - CRB - mirrors.ustc.edu.cn
#failovermethod=priority
baseurl=https://mirrors.ustc.edu.cn/centos-stream/\$stream/CRB/\$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos-stream/RPM-GPG-KEY-CentOS-Official

[HighAvailability]
name=CentOS-\$releasever - HighAvailability - mirrors.ustc.edu.cn
#failovermethod=priority
baseurl=https://mirrors.ustc.edu.cn/centos-stream/\$stream/HighAvailability/\$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos-stream/RPM-GPG-KEY-CentOS-Official

[NFV]
name=CentOS-\$releasever - NFV - mirrors.ustc.edu.cn
#failovermethod=priority
baseurl=https://mirrors.ustc.edu.cn/centos-stream/\$stream/NFV/\$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos-stream/RPM-GPG-KEY-CentOS-Official

[RT]
name=CentOS-\$releasever - RT - mirrors.ustc.edu.cn
#failovermethod=priority
baseurl=https://mirrors.ustc.edu.cn/centos-stream/\$stream/RT/\$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos-stream/RPM-GPG-KEY-CentOS-Official

[ResilientStorage]
name=CentOS-\$releasever - ResilientStorage - mirrors.ustc.edu.cn
#failovermethod=priority
baseurl=https://mirrors.ustc.edu.cn/centos-stream/\$stream/ResilientStorage/\$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos-stream/RPM-GPG-KEY-CentOS-Official
EOF
```
### 安装epel扩展源
```
# 安装epel扩展源
sudo yum install -y epel-release
```
### 设置为国内源
```
# 设置为国内源
sudo sed -e 's|^metalink=|#metalink=|g' \
         -e 's|^#baseurl=https\?://download.fedoraproject.org/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
         -e 's|^#baseurl=https\?://download.example/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
         -i.bak \
         /etc/yum.repos.d/epel.repo

sudo sed -e 's|^metalink=|#metalink=|g' \
         -e 's|^#baseurl=https\?://download.fedoraproject.org/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
         -e 's|^#baseurl=https\?://download.example/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
         -i.bak \
         /etc/yum.repos.d/epel-next.repo

```
### 更新源信息
```
# 更新源信息
yum makecache && yum update
```
### 配置网卡IP
```
# 配置网卡IP
nmcli con mod ens160 ipv4.addresses 192.168.1.16/24; nmcli con mod ens160 ipv4.gateway 192.168.1.1; nmcli con mod ens160 ipv4.method manual; nmcli con mod ens160 ipv4.dns "8.8.8.8"; nmcli con up ens160

nmcli con mod ens160 ipv6.addresses 2409:8a10:9e1e:7c10::1233; nmcli con mod ens160 ipv6.gateway 2409:8a10:9e1e:7c10::; nmcli con mod ens160 ipv6.method manual; nmcli con mod ens160 ipv6.dns "2001:4860:4860::8888"; nmcli con up ens160

```
### 查看网卡配置
```
# 查看网卡配置
cat /etc/NetworkManager/system-connections/ens160.nmconnection
[connection]
id=ens160
uuid=23c2f83b-e567-3296-bb0a-433ea216449f
type=ethernet
autoconnect-priority=-999
interface-name=ens160
timestamp=1664101928

[ethernet]

[ipv4]
address1=192.168.1.16/24,192.168.1.1
dns=8.8.8.8;
method=manual

[ipv6]
addr-gen-mode=eui64
address1=2409:8a10:9e1e:7c10::1233/128,2409:8a10:9e1e:7c10::
dns=2001:4860:4860::8888;
method=manual

[proxy]

```
### 测试网络
```
# 测试网络
[root@chenby ~]# ping www.oiox.cn -4
PING  (117.161.38.205) 56(84) bytes of data.
64 bytes from 117.161.38.205 (117.161.38.205): icmp_seq=1 ttl=55 time=6.84 ms
64 bytes from 117.161.38.205 (117.161.38.205): icmp_seq=2 ttl=55 time=6.44 ms
64 bytes from 117.161.38.205 (117.161.38.205): icmp_seq=3 ttl=55 time=7.12 ms

---  ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 6.441/6.800/7.117/0.277 ms
[root@bogon ~]# ping www.oiox.cn -6
PING www.oiox.cn(js-ipv6 (2409:8c10:c00:1404:3b::)) 56 data bytes
64 bytes from js-ipv6 (2409:8c10:c00:1404:3b::): icmp_seq=1 ttl=56 time=5.94 ms
64 bytes from js-ipv6 (2409:8c10:c00:1404:3b::): icmp_seq=2 ttl=56 time=6.11 ms
64 bytes from js-ipv6 (2409:8c10:c00:1404:3b::): icmp_seq=3 ttl=56 time=6.01 ms

--- www.oiox.cn ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 5.941/6.020/6.107/0.067 ms
[root@chenby ~]#
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
