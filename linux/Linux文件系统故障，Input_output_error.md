---
layout: post
cid: 13
title: Linux文件系统故障，Input/output error
slug: 13
date: 2021/12/30 17:01:23
updated: 2021/12/30 17:01:23
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/698bb5a1576a4882bb866a23a962fa6c~tplv-k3u1fbpfcp-zoom-1.image)

  

    事情是这样的，在启动某一个应用程序的时候，出现 Input/output error 的报错，磁盘以及目录无法使用的情况下，进行了重启，重启完成后是可以正常使用的，过一段时间后就会再次出现这个问题，一番Google之后怀疑是磁盘出现问题，根据网友的解决方案尝试之后发现，这个方法可行，下文是命令及回显：

      

    使用ls命令查看的时候出现这个报错  

  

```
[root@webc ~]# ls /data/
ls: 无法访问/data/: 输入/输出错误
[root@webc ~]#
```

  

  

    这个是xfs的文件系统，所以使用如下命令进行修复  

  

```
[root@webc ~]# xfs_repair /dev/sdc1
xfs_repair: cannot open /dev/sdc1: 设备或资源忙
```

   

      

    这时这个问题，不要慌，先把磁盘卸载了在进行修复  

  

```
[root@webc ~]# umount /dev/sdc1
[root@webc ~]# xfs_repair /dev/sdc1
Phase 1 - find and verify superblock...
Phase 2 - using internal log
        - zero log...
ERROR: The filesystem has valuable metadata changes in a log which needs to
be replayed.  Mount the filesystem to replay the log, and unmount it before
re-running xfs_repair.  If you are unable to mount the filesystem, then use
the -L option to destroy the log and attempt a repair.
Note that destroying the log may cause corruption -- please attempt a mount
of the filesystem before doing this.
[root@webc ~]# 
[root@webc ~]# 
[root@webc ~]# xfs_repair /dev/sdc1
Phase 1 - find and verify superblock...
Phase 2 - using internal log
        - zero log...
ERROR: The filesystem has valuable metadata changes in a log which needs to
be replayed.  Mount the filesystem to replay the log, and unmount it before
re-running xfs_repair.  If you are unable to mount the filesystem, then use
the -L option to destroy the log and attempt a repair.
Note that destroying the log may cause corruption -- please attempt a mount
of the filesystem before doing this.
[root@webc ~]# xfs_repair /dev/sdc1 -L
Phase 1 - find and verify superblock...
Phase 2 - using internal log
        - zero log...
ALERT: The filesystem has valuable metadata changes in a log which is being
destroyed because the -L option was used.
        - scan filesystem freespace and inode maps...
agi unlinked bucket 31 is 7620063 in ag 5 (inode=10745038303)
sb_icount 533632, counted 533568
sb_ifree 617, counted 614
sb_fdblocks 2852137932, counted 2860186916
        - found root inode chunk
Phase 3 - for each AG...
        - scan and clear agi unlinked lists...
        - process known inodes and perform inode discovery...
        - agno = 0
        - agno = 1
        - agno = 2
        - agno = 3
        - agno = 4
        - agno = 5
correcting bt key (was 91997, now 92001) in inode 10745038303
    data fork, btree block 1343129285
correcting bt key (was 226254, now 226257) in inode 10745038303
    data fork, btree block 1345535075
correcting bt key (was 241554, now 241557) in inode 10745038303
    data fork, btree block 1345535075
correcting bt key (was 795517, now 795515) in inode 10745038303
    data fork, btree block 1343659983
data fork in regular inode 10745038303 claims used block 1353137709
correcting nextents for inode 10745038303
bad data fork in inode 10745038303
cleared inode 10745038303
        - agno = 6
        - agno = 7
        - agno = 8
correcting nextents for inode 17197661037, was 870903 - counted 870911
        - agno = 9
        - agno = 10
correcting bt key (was 1923723, now 1923730) in inode 21481716216
    data fork, btree block 2687659655
correcting bt key (was 1997785, now 1997794) in inode 21481716216
    data fork, btree block 2687659655
correcting nextents for inode 21481716216, was 918874 - counted 918898
        - process newly discovered inodes...
Phase 4 - check for duplicate blocks...
        - setting up duplicate extent list...
        - check for inodes claiming duplicate blocks...
        - agno = 0
        - agno = 3
        - agno = 4
        - agno = 2
        - agno = 5
        - agno = 6
        - agno = 1
        - agno = 7
        - agno = 9
        - agno = 8
        - agno = 10
Phase 5 - rebuild AG headers and trees...
        - reset superblock...
Phase 6 - check inode connectivity...
        - resetting contents of realtime bitmap and summary inodes
        - traversing filesystem ...
        - traversal finished ...
        - moving disconnected inodes to lost+found ...
Phase 7 - verify and correct link counts...
Maximum metadata LSN (15:166217) is ahead of log (1:2).
Format log to cycle 18.
done
[root@webc ~]#
```

  

  

    修复完成后在把磁盘挂上，即可生效  

  

```
[root@webc ~]# mount /dev/sdc1 /data/
```

  

    查看一下这个磁盘是否可以正常使用  

  

```
[root@webc ~]# cd /data/vm/
[root@webc vm]# ls
CentOS7-Clone-1  CentOS7-Clone-3  CentOS7-Clone-4  CentOS7-Clone-5  CentOS8  Ubuntu
```

  

此刻文件系统已修复完毕  

  

注意：  

    修复其他文件系统使用fsck命令进行修复  

    例如ext4文件系统  

  

```
fsck -t ext4 -y /dev/sda1
```

  

不同的文件系统，命令会有些许不同，灵活变通一下

  

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