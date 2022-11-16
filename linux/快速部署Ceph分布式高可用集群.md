---
layout: post_draft
cid: 254
title: 快速部署Ceph分布式高可用集群
slug: 254
date: 2022/06/15 09:17:41
updated: 2022/06/15 09:17:51
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: 快速部署Ceph分布式高可用集群
keywords: 小陈运维,小陈,小陈博客,文档,
mode: default
thumb: 
video: 
---


# 快速部署Ceph分布式高可用集群

## Ceph简介

Ceph是一个PB，EB级别的分布式存储系统，可以提供文件存储，对象存储、和块存储，它可靠性高，易扩展，管理简便，其中对象存储和块存储可以和其他云平台集成。一个Ceph集群中有Monitor节点、MDS节点（用于文件存储）、OSD守护进程。



## Ceph基础概念

- ceph-deploy

  一个集群自动化部署工具，使用较久，成熟稳定，被很多自动化工具所集成，可用于生产部署；

- cephadm

  从Octopus开始提供的新集群部署工具，支持通过图形界面或者命令行界面添加节点，目前不建议用于生产环境，有兴趣可以尝试；

- manual

  手动部署，一步步部署Ceph集群，支持较多定制化和了解部署细节，安装难度较大，但可以清晰掌握安装部署的细节。

- **admin-node**：

  需要一个安装管理节点，安装节点负责集群整体部署，这里我们用CephNode01为admin-node和Ceph-Mon节点；

- **mon**：

  monitor节点，即是Ceph的监视管理节点，承担Ceph集群重要的管理任务，一般需要3或5个节点，此处部署简单的一个Monitor节点；

- **osd**：

  OSD即Object Storage Daemon，实际负责数据存储的节点，3个节点上分别有2块100G的磁盘充当OSD角色。



## Ceph系统初始化

### 配置主机信息

```
# 设置主机名

#node1
hostnamectl set-hostname node1

#node2
hostnamectl set-hostname node2

#node3
hostnamectl set-hostname node3

# 写入hosts
cat >> /etc/hosts <<EOF
192.168.1.156  node1
192.168.1.157  node2
192.168.1.159  node3
EOF

cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.156  node1
192.168.1.157  node2
192.168.1.159  node3

```



### 配置免密

```
# 配置免密 （二选一）
ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:nK3CqSGRBGZfrE5rncPEQ2eU/Gq6dttYMLIiesXHyO8 root@ceph-01
The key's randomart image is:
+---[RSA 3072]----+
|.o  ..o..        |
|o.. .o =         |
|  ..+ o .        |
| . + + . +       |
|  =o=+ooS .      |
|   ==*=+o.       |
| .oo.+B ..       |
|. o..=.o+        |
|..  ooEo..       |
+----[SHA256]-----+

# 将免密传输到各个主机上
ssh-copy-id root@node1
ssh-copy-id root@node2
ssh-copy-id root@node3


# 使用懒人方式配置免密 （二选一）
yum install -y sshpass
ssh-keygen -f /root/.ssh/id_rsa -P ''
export IP="node1 node2 node3"
export SSHPASS=123123
for HOST in $IP;do
     sshpass -e ssh-copy-id -o StrictHostKeyChecking=no $HOST
done
```



### 配置基础环境

```
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
Removed /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.

# 关闭swap
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# 关闭selinux
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
```



### 配置YUM源

```
# 配置yum源

sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/$contentdir|baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-*.repo

# 配置ceph源
cat > /etc/yum.repos.d/ceph.repo <<EOF
[noarch] 
name=Ceph noarch 
baseurl=https://mirrors.ustc.edu.cn/ceph/rpm-17.2.0/el8/noarch/ 
enabled=1 
gpgcheck=0 

[x86_64] 
name=Ceph x86_64 
baseurl=https://mirrors.ustc.edu.cn/ceph/rpm-17.2.0/el8/x86_64/ 
enabled=1 
gpgcheck=0
EOF
```



### 安装基础环境

```
# 更新yum源
yum update -y
# 安装工具包、python-setuptools一定要安装、不然会报错的
yum install -y chrony conntrack ipset jq iptables curl sysstat libseccomp wget socat git vim epel-release epel-next-release
```



### 调整时区\间

```
# 配置系统时区
timedatectl set-timezone Asia/Shanghai

# 配置时钟同步
timedatectl status

# 注：System clock synchronized: yes，表示时钟已同步；NTP service: active，表示开启了时钟同步服务

# 写入硬件时钟

# 将当前的 UTC 时间写入硬件时钟
timedatectl set-local-rtc 0

# 重启依赖于系统时间的服务
systemctl restart rsyslog 
systemctl restart crond
```



### 杂项

```
# 关闭无关服务
systemctl stop postfix && systemctl disable postfix

#  重启
reboot
```



## Ceph系统安装

### 初始化monitor节点

```
yum install ceph -y

# 初始化monitor节点
# 在node1节点生成uuid，并在所有节点导入uuid环境变量

[root@node1 ~]# uuidgen
8d2cfd33-9132-48a7-8c00-3ef10cb5ddeb
#node1
export cephuid=8d2cfd33-9132-48a7-8c00-3ef10cb5ddeb
#node2
export cephuid=8d2cfd33-9132-48a7-8c00-3ef10cb5ddeb
#node3
export cephuid=8d2cfd33-9132-48a7-8c00-3ef10cb5ddeb

# 所有节点创建Ceph配置文件：

cat > /etc/ceph/ceph.conf <<EOF
[global]
fsid = 8d2cfd33-9132-48a7-8c00-3ef10cb5ddeb
mon initial members = node1, node2, node3
mon host = 192.168.1.156, 192.168.1.157, 192.168.1.159
public network = 192.168.1.0/24
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
osd journal size = 1024
osd pool default size = 3
osd pool default min size = 2
osd pool default pg num = 333
osd pool default pgp num = 333
osd crush chooseleaf type = 1
EOF

# 以下操作在node1节点执行
# 为集群创建一个keyring，并生成一个monitor密钥。
#node1
ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'

# 生成administrator keyring，生成client.admin用户并将用户添加到keyring。
#node1
ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *'

# 生成bootstrap-osd keyring，生成client.bootstrap-osd用户并将用户添加到keyring。
#node1
ceph-authtool --create-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring --gen-key -n client.bootstrap-osd --cap mon 'profile bootstrap-osd' --cap mgr 'allow r'

# 将生成的密钥添加到中ceph.mon.keyring。
#node1
ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
ceph-authtool /tmp/ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring

# 将所有者更改为ceph.mon.keyring。
#node1
chown ceph:ceph /tmp/ceph.mon.keyring

# 使用主机名，主机IP地址和FSID生成monitor map。另存为/tmp/monmap：
#node1
monmaptool --create --add node1 192.168.1.156 --add node2 192.168.1.157 --add node3 192.168.1.159 --fsid $cephuid /tmp/monmap

# 复制monitor map到另外2个节点
#node1
scp /tmp/monmap root@node2:/tmp
scp /tmp/monmap root@node3:/tmp

# 复制ceph.client.admin.keyring到另外2个节点
#node1
scp /etc/ceph/ceph.client.admin.keyring root@node2:/etc/ceph/
scp /etc/ceph/ceph.client.admin.keyring root@node3:/etc/ceph/

# 复制ceph.mon.keyring到另外2个节点
#node1
scp /tmp/ceph.mon.keyring root@node2:/tmp/
scp /tmp/ceph.mon.keyring root@node3:/tmp/

#注意修改文件权限
#node2
chown ceph:ceph /tmp/ceph.mon.keyring
#node3
chown ceph:ceph /tmp/ceph.mon.keyring

# 创建monitor数据目录
#node1
sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node1
#node2
sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node2
#node3
sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node3

# 用monitor map和keyring填充monitor守护程序。
#node1
sudo -u ceph ceph-mon --mkfs -i node1 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
#node2
sudo -u ceph ceph-mon --mkfs -i node2 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
#node3
sudo -u ceph ceph-mon --mkfs -i node3 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring

# 查看生成的文件
#node1
ls /var/lib/ceph/mon/ceph-node1/
keyring  kv_backend  store.db

# 启动monitor服务
#node1
systemctl restart ceph-mon@node1
systemctl enable ceph-mon@node1
#node2
systemctl restart ceph-mon@node2
systemctl enable ceph-mon@node2
#node3
systemctl restart ceph-mon@node3
systemctl enable ceph-mon@node3

# 查看当前集群状态

ceph -s
  cluster:
    id:     8d2cfd33-9132-48a7-8c00-3ef10cb5ddeb
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum node1,node2,node3 (age 0.35737s)
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:      
 

# 若异常则启用msgr2
# ceph mon enable-msgr2
```



### 初始化manager节点

```
#node1
ceph auth get-or-create mgr.node1 mon 'allow profile mgr' osd 'allow *' mds 'allow *'
sudo -u ceph mkdir /var/lib/ceph/mgr/ceph-node1
sudo -u ceph vim /var/lib/ceph/mgr/ceph-node1/keyring
[mgr.node1]
	key = AQBk7aZiZD1NDRAAfXyfT2ovmsJwADzkbioHzQ==     

#node2
ceph auth get-or-create mgr.node2 mon 'allow profile mgr' osd 'allow *' mds 'allow *'
sudo -u ceph mkdir /var/lib/ceph/mgr/ceph-node2
sudo -u ceph vim /var/lib/ceph/mgr/ceph-node2/keyring
[mgr.node2]
	key = AQB67aZicvq7DhAAKEUipQSIDZEUZVv740mEuA==

#node3
ceph auth get-or-create mgr.node3 mon 'allow profile mgr' osd 'allow *' mds 'allow *'
sudo -u ceph mkdir /var/lib/ceph/mgr/ceph-node3
sudo -u ceph vim /var/lib/ceph/mgr/ceph-node3/keyring
[mgr.node3]
	key = AQCS7aZiC75UIhAA2aue7yr1XGiBs4cRt8ru3A==

# 启动ceph-mgr守护程序：
#node1
systemctl restart ceph-mgr@node1
systemctl enable ceph-mgr@node1
#node2
systemctl restart ceph-mgr@node2
systemctl enable ceph-mgr@node2
#node3
systemctl restart ceph-mgr@node3
systemctl enable ceph-mgr@node3

# 通过ceph status查看输出来检查mgr是否出现

ceph status
  cluster:
    id:     8d2cfd33-9132-48a7-8c00-3ef10cb5ddeb
    health: HEALTH_WARN
            mons are allowing insecure global_id reclaim
            clock skew detected on mon.node2, mon.node3
            OSD count 0 < osd_pool_default_size 3
 
  services:
    mon: 3 daemons, quorum node1,node2,node3 (age 29s)
    mgr: node3(active, since 19s), standbys: node1, node2
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs: 
```



### 添加OSD

```
# 复制keyring到其他2个节点
#node1
scp /var/lib/ceph/bootstrap-osd/ceph.keyring root@node2:/var/lib/ceph/bootstrap-osd/
scp /var/lib/ceph/bootstrap-osd/ceph.keyring root@node3:/var/lib/ceph/bootstrap-osd/

# 创建OSD
[root@node1 ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0  100G  0 disk 
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   99G  0 part 
  ├─cs-root 253:0    0 61.2G  0 lvm  /
  ├─cs-swap 253:1    0  7.9G  0 lvm  
  └─cs-home 253:2    0 29.9G  0 lvm  /home
sdb           8:16   0   10G  0 disk 


# 3个节点上执行
yum install ceph-volume
ceph-volume lvm create --data /dev/sdb

# 启动各个节点osd进程
#node1
systemctl restart ceph-osd@0
systemctl enable ceph-osd@0
#node2
systemctl restart ceph-osd@1
systemctl enable ceph-osd@1
#node3
systemctl restart ceph-osd@2
systemctl enable ceph-osd@2


# 查看集群状态
ceph -s
  cluster:
    id:     8d2cfd33-9132-48a7-8c00-3ef10cb5ddeb
    health: HEALTH_WARN
            mons are allowing insecure global_id reclaim
 
  services:
    mon: 3 daemons, quorum node1,node2,node3 (age 5m)
    mgr: node3(active, since 4m), standbys: node1, node2
    osd: 3 osds: 3 up (since 7s), 3 in (since 62s)
 
  data:
    pools:   1 pools, 1 pgs
    objects: 2 objects, 577 KiB
    usage:   18 MiB used, 30 GiB / 30 GiB avail
    pgs:     1 active+clean
 
  io:
    client:   1.2 KiB/s rd, 36 KiB/s wr, 1 op/s rd, 1 op/s wr
    recovery: 27 KiB/s, 0 objects/s
```

## 添加MDS

```
# 创建mds数据目录。
#node1
sudo -u ceph mkdir -p /var/lib/ceph/mds/ceph-node1
#node2
sudo -u ceph mkdir -p /var/lib/ceph/mds/ceph-node2
#node3
sudo -u ceph mkdir -p /var/lib/ceph/mds/ceph-node3


# 创建keyring：
#node1
ceph-authtool --create-keyring /var/lib/ceph/mds/ceph-node1/keyring --gen-key -n mds.node1
#node2
ceph-authtool --create-keyring /var/lib/ceph/mds/ceph-node2/keyring --gen-key -n mds.node2
#node3
ceph-authtool --create-keyring /var/lib/ceph/mds/ceph-node3/keyring --gen-key -n mds.node3

# 导入keyring并设置权限：
#node1
ceph auth add mds.node1 osd "allow rwx" mds "allow" mon "allow profile mds" -i /var/lib/ceph/mds/ceph-node1/keyring
chown ceph:ceph /var/lib/ceph/mds/ceph-node1/keyring
#node2
ceph auth add mds.node2 osd "allow rwx" mds "allow" mon "allow profile mds" -i /var/lib/ceph/mds/ceph-node2/keyring
chown ceph:ceph /var/lib/ceph/mds/ceph-node2/keyring
#node3
ceph auth add mds.node3 osd "allow rwx" mds "allow" mon "allow profile mds" -i /var/lib/ceph/mds/ceph-node3/keyring
chown ceph:ceph /var/lib/ceph/mds/ceph-node3/keyring
```



## 收尾

```
所有节点修改ceph.conf配置文件，追加以下内容

cat >> /etc/ceph/ceph.conf <<EOF
[mds.node1]
host = node1

[mds.node2]
host = node2

[mds.node3]
host = node3
EOF


重新启动所有服务

#node1
systemctl restart ceph-mon@node1
systemctl restart ceph-mgr@node1
systemctl restart ceph-mds@node1
systemctl enable ceph-mds@node1
systemctl restart ceph-osd@0

#node2
systemctl restart ceph-mon@node2
systemctl restart ceph-mgr@node2
systemctl restart ceph-mds@node2
systemctl enable ceph-mds@node2
systemctl restart ceph-osd@1

#node3
systemctl restart ceph-mon@node3
systemctl restart ceph-mgr@node3
systemctl restart ceph-mds@node3
systemctl enable ceph-mds@node3
systemctl restart ceph-osd@2


查看集群状态

ceph -s
  cluster:
    id:     8d2cfd33-9132-48a7-8c00-3ef10cb5ddeb
    health: HEALTH_WARN
            mons are allowing insecure global_id reclaim
 
  services:
    mon: 3 daemons, quorum node1,node2,node3 (age 9s)
    mgr: node3(active, since 4s), standbys: node1, node2
    osd: 3 osds: 3 up (since 4s), 3 in (since 2m)
 
  data:
    pools:   1 pools, 1 pgs
    objects: 2 objects, 577 KiB
    usage:   18 MiB used, 30 GiB / 30 GiB avail
    pgs:     1 active+clean


查看osd状态

[root@node1 ~]# ceph osd tree
ID  CLASS  WEIGHT   TYPE NAME       STATUS  REWEIGHT  PRI-AFF
-1         0.02939  root default                             
-3         0.00980      host node1                           
 0    hdd  0.00980          osd.0       up   1.00000  1.00000
-5         0.00980      host node2                           
 1    hdd  0.00980          osd.1       up   1.00000  1.00000
-7         0.00980      host node3                           
 2    hdd  0.00980          osd.2       up   1.00000  1.00000
```



> **关于**
>
> https://www.oiox.cn/
>
> https://www.oiox.cn/index.php/start-page.html
>
> **CSDN、GitHub、知乎、微信、开源中国、思否、掘金、简书、华为云、阿里云、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客、全网可搜《小陈运维》**
>
> **文章主要发布于微信**
