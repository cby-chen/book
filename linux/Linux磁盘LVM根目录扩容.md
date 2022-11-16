---
layout: post
cid: 130
title: Linux磁盘LVM根目录扩容
slug: 130
date: 2022/03/31 14:12:11
updated: 2022/03/31 14:12:28
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


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b66badf2955413c829a4b6eee45c833~tplv-k3u1fbpfcp-zoom-1.image)

  

LVM 的基本概念

  

物理卷 Physical Volume (PV)：可以在上面建立卷组的媒介，可以是硬盘分区，也可以是硬盘本身或者回环文件（loopback file）。物理卷包括一个特殊的 header，其余部分被切割为一块块物理区域（physical extents）

  

卷组 Volume group (VG)：将一组物理卷收集为一个管理单元

  

逻辑卷 Logical volume (LV)：虚拟分区，由物理区域（physical extents）组成

  

物理区域 Physical extent (PE)：硬盘可供指派给逻辑卷的最小单位（通常为 4MB）

  

  

  

新建分区

  

```
root@hello:~# fdisk /dev/sda


Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


GPT PMBR size mismatch (209715199 != 629145599) will be corrected by write.
The backup GPT table is not on the end of the device. This problem will be corrected by write.


Command (m for help): n
Partition number (4-128, default 4): 
First sector (209713152-629145566, default 209713152): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (209713152-629145566, default 629145566): 


Created a new partition 4 of type 'Linux filesystem' and of size 200 GiB.


Command (m for help): p
Disk /dev/sda: 300 GiB, 322122547200 bytes, 629145600 sectors
Disk model: QEMU HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9FD87B41-A400-4DD9-AE87-C852ACB2854D


Device         Start       End   Sectors  Size Type
/dev/sda1       2048      4095      2048    1M BIOS boot
/dev/sda2       4096   2101247   2097152    1G Linux filesystem
/dev/sda3    2101248 209713151 207611904   99G Linux filesystem
/dev/sda4  209713152 629145566 419432415  200G Linux filesystem


Command (m for help): w
The partition table has been altered.
Syncing disks.


root@hello:~#
```

  

  

扩展pv并查看

  

  

```
root@hello:~# pvcreate  /dev/sda4
  Physical volume "/dev/sda4" successfully created.

root@hello:~# pvdisplay 
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               <99.00 GiB / not usable 0   
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              25343
  Free PE               0
  Allocated PE          25343
  PV UUID               DfUNvp-F4D5-J6Rj-sMcS-PqE1-bKf6-2XLfb2


  --- Physical volume ---
  PV Name               /dev/sda4
  VG Name               ubuntu-vg
  PV Size               200.00 GiB / not usable 4.98 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              51199
  Free PE               0
  Allocated PE          51199
  PV UUID               yeXSs6-mUWY-Q31q-oD0J-yWHA-zPfp-92AKrO
```

  

扩展vg并查看

  

```
root@hello:~# vgextend ubuntu-vg /dev/sda4
  Volume group "ubuntu-vg" successfully extended

root@hello:~# vgdisplay 
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               298.99 GiB
  PE Size               4.00 MiB
  Total PE              76542
  Alloc PE / Size       76542 / 298.99 GiB
  Free  PE / Size       0 / 0   
  VG UUID               QpG0NU-5tPY-q5Bw-mLIu-Axl6-mdtP-19WWq6
```

  

扩展lv并查看

  

```
root@hello:~# lvextend /dev/ubuntu-vg/ubuntu-lv /dev/sda4
  Size of logical volume ubuntu-vg/ubuntu-lv changed from <99.00 GiB (25343 extents) to 298.99 GiB (76542 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.

root@hello:~# lvdisplay 
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                ODU5p6-8E8i-30tn-3hzs-AifF-iF4Z-HzMuFL
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2022-03-29 09:07:30 +0000
  LV Status              available
  # open                 1
  LV Size                298.99 GiB
  Current LE             76542
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

  

扩展根文件系统  

  

```
root@hello:~# resize2fs /dev/ubuntu-vg/ubuntu-lv
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
old_desc_blocks = 13, new_desc_blocks = 38
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 78379008 (4k) blocks long.
```

  

若是xfs文件系统，可以使用xfs_growfs命令

  

```
root@hello:~# df -hT | grep ubuntu--vg-ubuntu--lv
/dev/mapper/ubuntu--vg-ubuntu--lv ext4      294G   14G  268G   5% /
root@hello:~#
```

  

查看块设备的依赖关系

  

```
root@hello:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 31.1M  1 loop /snap/snapd/10707
loop1                       7:1    0 55.4M  1 loop /snap/core18/1944
loop2                       7:2    0 69.9M  1 loop /snap/lxd/19188
sda                         8:0    0  300G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    1G  0 part /boot
├─sda3                      8:3    0   99G  0 part 
│ └─ubuntu--vg-ubuntu--lv 253:0    0  299G  0 lvm  /
└─sda4                      8:4    0  200G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0  299G  0 lvm  /
sr0                        11:0    1  1.1G  0 rom  
root@hello:~#
```

  



  

  

https://www.oiox.cn/

https://www.chenby.cn/

https://cby-chen.github.io/

https://weibo.com/u/5982474121

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

https://www.jianshu.com/u/0f894314ae2c

https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/

CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客、全网可搜《小陈运维》
