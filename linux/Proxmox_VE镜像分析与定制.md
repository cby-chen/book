---
layout: post
cid: 15
title: Proxmox VE镜像分析与定制
slug: 15
date: 2021/12/30 17:01:00
updated: 2022/03/25 15:52:08
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


    Proxmox VE（Proxmox Virtual Environment，简称PVE）是一个开源的服务器虚拟化环境Linux发行版，基于Debian，使用给予Ubuntu的定制内核。相比于其他虚拟化平台，PVE具有的一个显著的特点就是无需master节点，安装完成后，无需特殊配置即可将多个节点组成集群。

  

由于工程要求，PVE需要大规模部署在物理服务器上，所以定制镜像就显得很有必要。

  

定制目标包括

  

（1）修改initrd中init脚本的提示信息

  

（2）删除GRUB界面多余选项，直接进入安装界面

  

（3）添加预装软件

  

（4）在安装过程中对软件进行个性化配置

  

（5）修改PVE安装界面，在PVE安装界面中的所有输入框设置默认文本

  

**Proxmox VE镜像分析**

  

下载Proxmox VE 6.4版镜像后挂载，观察文件结构

  

```
$ tree -L 2
.
├── boot
│   ├── boot.cat
│   ├── grub
│   ├── initrd.img
│   ├── linux26
│   └── memtest86+.bin
├── COPYING
├── COPYRIGHT
├── debian -> .
├── dists
│   └── stretch
├── efi.img
├── EULA
├── mach_kernel
├── proxmox
│   ├── country.dat
│   ├── packages
│   └── pve-base.cnt
├── pve-base.squashfs
├── pve-installer.squashfs
├── Release.txt
└── System
    └── Library

9 directories, 14 files
```

  

其中：

  

```
grub文件夹：包含引导程序GRUB所用到的文件。
initrd.img：系统初始化所使用的镜像，里面包含一个最小化的系统，包含了/dev、/etc、/bin等很多基本的目录，还有关键的init程序，负责驱动的加载和文件系统的初始化。
linux26：Linux 2.6内核
efi.img：系统引导镜像，内含boot.efi、bootia32.efi、bootx64.efi。
proxmox文件夹：系统预安装包的存放目录
PVE的根系统默认安装包是在proxmox文件夹下的，只要不破坏其依赖关系，可以将需要预安装的包及其依赖放到这个文件夹下。
PVE预安装包时候使用的是循环读取proxmox/packages中的deb，然后使用的安装方法是先解压然后再配置，这样不会产生依赖关系而导致装不上deb的问题。
pve-base.squashfs：安装的根系统，也就是最终的系统
pve-installer.squashfs：安装时需要的系统
```

  

**Proxmox VE安装流程**

PVE安装流程主要分为以下4个步骤：

  

（1）Boot Loader：由 BIOS 加载，用于将后续的 kernel 和 initrd 的装载到内存中。（PVE安装时使用的是UEFI模式的安装，但是又不是传统意义上的UEFI，它先是使用了BIOS加载kernel和initrd到内存，然后又跳到UEFI分区执行efi.img文件，调用proxinstall进入到系统安装界面，然后是挂载pve-base.squashfs进行系统安装）

  

（2）kernel：为 initrd 运行提供基础的运行环境，对应boot目录下的linux26文件

  

（3）initrd：检测并加载各种驱动程序，并执行init，对应boot目录下的initrd.img文件

  

（4）rootfs：根文件系统，用户的各种操作都是基于这个被最后加载的文件系统，这里对应了pve-base.squashfs

  

**Proxmox VE镜像定制**

ISO解压与压缩

  

在原先使用ISO Master作为解压缩ISO的工具中，产生的ISO文件可以直接作为cdrom启动，但刻录进USB设备后缺失MBR等重要部分所以无法启动，因此改用命令行进行解压缩。

  

（1）ISO提取

  

首先挂载镜像文件。

  

```
$ mount -o loop Desktop/proxmox-ve_6.4-1.iso cby/
```

  

挂载点目录中的文件是只读的，所以需要同步到工作目录下。

  

```
$ cd cby
$ sudo rsync -av /home/cby/cby/ /home/cby/
```

  

同步之后就即可修改ISO内的文件。

  

```
$ sudo umount /home/cby/cby 
$ ll
total 386672
dr-xr-xr-x 10 root root      4096 Apr 27 04:26 ./
dr-xr-xr-x 23 root root      4096 May 19 18:56 ../
dr-xr-xr-x  3 root root      4096 Apr 27 04:26 boot/
-r--r--r--  1 root root        89 Apr 27 04:26 .cd-info
-r--r--r--  1 root root     32386 Apr 27 04:26 COPYING
-r--r--r--  1 root root       955 Apr 27 04:26 COPYRIGHT
lrwxrwxrwx  1 root root         1 Apr 27 04:26 debian -> ./
dr-xr-xr-x  3 root root      4096 Apr 27 04:26 dists/
-r--r--r--  1 root root   2949120 Apr 27 04:26 efi.img
-r--r--r--  1 root root      4470 Apr 27 04:26 EULA
-r--r--r--  1 root root         0 Apr 27 04:26 mach_kernel
dr-xr-xr-x  3 root root      4096 Apr 27 04:26 proxmox/
dr-xr-xr-x  2 root root      4096 Apr 27 04:26 .pve-base/
-r--r--r--  1 root root 101306368 Apr 27 04:26 pve-base.squashfs
-r--r--r--  1 root root        37 Apr 27 04:26 .pve-cd-id.txt
dr-xr-xr-x  2 root root      4096 Apr 27 04:26 .pve-installer/
dr-xr-xr-x  2 root root      4096 Apr 27 04:26 .pve-installer-mp/
-r--r--r--  1 root root 291586048 Apr 27 04:26 pve-installer.squashfs
-r--r--r--  1 root root     15792 Apr 27 04:26 Release.txt
dr-xr-xr-x  3 root root      4096 Apr 27 04:26 System/
dr-xr-xr-x  2 root root      4096 Apr 27 04:26 .workdir/
```

  

（2）ISO压缩

  

使用原镜像的MBR（前512字节）作为定制镜像的MBR

  

```
$ sudo dd if=/home/cby/proxmox-ve_6.4-1.iso bs=512 count=1 of=proxmox.mbr
1+0 records in
1+0 records out
512 bytes copied, 0.000134541 s, 3.8 MB/s
```

  

打包ISO镜像  

  

```
$ sudo xorriso -as mkisofs -o proxmox-ve_6.4-1.iso -r -V 'inspur' --grub2-mbr proxmox.mbr --protective-msdos-label -efi-boot-part --efi-boot-image  -c '/boot/boot.cat' -b '/boot/grub/i386-pc/eltorito.img' -no-emul-boot -boot-load-size 4 -boot-info-table --grub2-boot-info -eltorito-alt-boot -e '/efi.img' -no-emul-boot .
xorriso 1.5.2 : RockRidge filesystem manipulator, libburnia project.

Drive current: -outdev 'stdio:proxmox-ve_6.4-1.iso'
Media current: stdio file, overwriteable
Media status : is blank
Media summary: 0 sessions, 0 data blocks, 0 data, 80.6g free
xorriso : WARNING : -volid text does not comply to ISO 9660 / ECMA 119 rules
Added to ISO image: directory '/'='/home/cby/chenby'
xorriso : UPDATE :    1421 files added in 1 seconds
xorriso : UPDATE :    1421 files added in 1 seconds
xorriso : NOTE : Copying to System Area: 512 bytes from file '/home/cby/chenby/proxmox.mbr'
xorriso : UPDATE :  1.00% done
xorriso : UPDATE :  42.39% done
xorriso : UPDATE :  86.68% done
ISO image produced: 453265 sectors
Written to medium : 453265 sectors at LBA 0
Writing to 'stdio:proxmox-ve_6.4-1.iso' completed successfully.
```

  

修改initrd

    initrd.img位于原始镜像的boot目录下，修改initrd的目的是修改安装过程中的输出文本，是一个比较特殊的部分，要从initrd引入的目的讲起。

  

    initrd 的英文含义是 boot loader initialized RAM disk，就是由 boot loader 初始化的内存盘。initrd的最初的目的是为了把kernel的启动分成两个阶段：在kernel中保留最少最基本的启动代码，然后把对各种各样硬件设备的支持以模块的方式放在initrd中，这样就在启动过程中可以从initrd所mount的根文件系统中装载需要的模块。这样的一个好处就是在保持kernel不变的情况下，通过修改initrd中的内容就可以灵活的支持不同的硬件。在启动完成的最后阶段，根文件系统可以重新mount到其他设备上。也就是说由于initrd会在内存虚拟一个文件系统，然后可以根据不同的硬件加载不同的驱动，而不需要重新编译整个核心。所以，大部分的发行版都会通过这种方式对驱动进行加载。

  

initrd引入之后Linux的引导会变成如下流程。

  

（1）boot loader 把内核以及 initrd 文件加载到内存的特定位置。

（2）内核判断initrd的文件格式，如果是cpio格式。

（3）将initrd的内容释放到rootfs中。

（4）执行initrd中的/init文件，执行到这一点，内核的工作全部结束，完全交给/init文件处理。

  

    根据核心版本的不同，initrd文件有两种格式：image和cpio。\*\*kernel 2.4只使用image格式，而kernel 2.6可同时支持两种格式。\*\*它们不单格式不一样，而且运作的机制和流程也完全不同，甚至制作方法也不一样。pve的kernel版本是2.6，所以在此只讲cpio格式的initrd制作。

  

initrd解压、修改与压缩流程：

  

（1）解压proxmox-ve_6.4-1.iso，boot目录下的initrd.img就是gz格式的压缩文件

  

（2）将initrd.img备份后重命名为initrd.org.img，并解压缩

  

```
$ sudo gzip -d -S ".img" ./initrd.org.img
```

  

执行file后查看格式

  

```
$ sudo file initrd.org
initrd.org: ASCII cpio archive (SVR4 with no CRC)
```

  

（3）创建initrd.tmp目录以存放后续还原出来的文件，然后执行cpio命令将文件还原

  

```
$ sudo mkdir initrd.tmp
$ cd initrd.tmp
$ sudo cpio -id < ../initrd.org
241820 blocks
$ ls
bin  dev  devfs  etc  init  lib  lib64  mnt  proc  sbin  sys  tmp  usr
```

  

去除GRUB界面

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76f87ea449a14ac5960020d6a0fa4a6f~tplv-k3u1fbpfcp-zoom-1.image)

  

    pve在安装时使用了GRUB2，所以想要去除掉GRUB界面需要找到原始镜像中boot/grub/grub.cfg文件，添加set timeout=0，就可以直接进入默认选项Install Proxmox VE模式。如果有需要我们也可以修改默认选项来实现直接进入其他模式的功能。

  

```
$ vim grub.cfg 
$ cat grub.cfg
insmod gzio
insmod iso9660
insmod png

loadfont /boot/grub/unicode.pf2

set gfxmode=640x400
# set kernel parameter vga=791
# do not specify color depth here (else efifb can fall back to 800x600)
set gfxpayload=1024x768
#set gfxmode=auto
#set gfxpayload=keep

set timeout=0

insmod all_video
insmod gfxterm

set theme=/boot/grub/pvetheme/theme.txt

...
```

  

**定制预装软件**

    Proxmox VE所有的预装软件都以deb包的形式存放在镜像的proxmox/packages下，并将在安装pve的过程中统一安装这些软件包，全部安装完成之后再进行配置，这样可以避免依赖关系出现问题。

  

    所以定制预装软件只需要在proxmox/packages目录下放入需要的deb包，pve将会自动安装并进行默认配置。

  

**配置预装程序**

    pve在配置软件是只会按照默认的配置，如果希望将软件配置成我们想要的形式，则只需要修改pve-installer.squashfs里的usr/bin/proxinstall文件。pve-installer.squashfs是pve安装时由initrd加载的系统，安装过程中proxinstall负责所有业务逻辑，其中配置软件部分的代码如下：

```
# needed for postfix postinst in case no other NIC is active
syscmd("chroot $targetdir ifup lo");

my $cmd = "chroot $targetdir dpkg $dpkg_opts --force-confold --configure -a";
$count = 0; 
run_command ($cmd, sub {
    my $line = shift;
    if ($line =~ m/Setting up\s+(\S+)/) {
    update_progress ((++$count)/$pkg_count, 0.75, 0.95,
             "configuring $1");
    }    
});
```

...  

```
# set apt mirror
if (my $mirror = $cmap->{country}->{$country}->{mirror}) {
    my $fn = "$targetdir/etc/apt/sources.list";
    syscmd ("sed -i 's/ftp\\.debian\\.org/$mirror/' '$fn'");
}

# create extended_states for apt (avoid cron job warning if that
# file does not exist)
write_config ('', "$targetdir/var/lib/apt/extended_states");

# allow ssh root login
syscmd(['sed', '-i', 's/^#\?PermitRootLogin.*/PermitRootLogin yes/', "$targetdir/etc/ssh/sshd_config"]);
```

  

    可以看出pve也是对部分程序进行了个性化的配置，所以对配置文件的编辑的代码只需要仿照后者，使用syscmd函数，将修改的命令作为参数，写在前者之后即可。

  

**定制安装界面**

    在pve-installer.squashfs里的usr/bin/proxinstall文件中，有create_main_window函数，这个函数的功能是创建图形界面窗口里的各种组件，通过分析这个函数我们可以得到安装UI的结构。

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90857943437e4edf91a77c7e9f77705e~tplv-k3u1fbpfcp-zoom-1.image)

  

    顶部的image、中心的htmlview窗口以及下方的cmdbox构成了我们所看到的外观。在此只修改image和htmlview。

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49987692502f454999d981cd118a1417~tplv-k3u1fbpfcp-zoom-1.image)

  

**定制安装界面**

    在pve-installer.squashfs里的usr/bin/proxinstall文件中，有create_main_window函数，这个函数的功能是创建图形界面窗口里的各种组件，通过分析这个函数我们可以得到安装UI的结构。

  

    顶部的image、中心的htmlview窗口以及下方的cmdbox构成了我们所看到的外观。在此只修改image和htmlview。

  

    顶部的image是在1785行加载pve-installer下var/lib/pve-installer/pve-banner.png来完成的，所以只需要用一个尺寸同样为1024X164的图像替代。

  

    中心的htmlview是通过在每个create_\*函数中调用display_html函数来加载，加载的html文件都位于var/lib/pve-installer/html文件夹下，对应的只需要修改每个html文件就可以实现外观上的替换。

  

    另外由于窗口运行环境openbox的语言设置默认不是中文，所以使用中文字符展示会出现乱码，因此可以由html加载含中文的图片，以此来展示中文。

  

    默认输入信息的修改就只需要在proxinstall中找到对应的输入框，修改预设文本。

  

使用命令unsquashfs将unsquashfs格式的镜像将其解压  

  

```
$ sudo unsquashfs pve-installer.squashfs
Parallel unsquashfs: Using 16 processors
20078 inodes (25826 blocks) to write

[===========================================================\] 25826/25826 100%

created 19247 files
created 2620 directories
created 819 symlinks
created 0 devices
created 0 fifos

$ ll
total 3256748
dr-xr-xr-x 12 root root       4096 May 19 19:55 ./
dr-xr-xr-x 24 root root       4096 May 19 19:29 ../
-rw-r--r--  1 root root  348389376 May 19 19:42 pve-installer.squashfs
drwxr-xr-x 17 root root       4096 Apr 27 04:23 squashfs-root/

...
```

  

    解压完成后会出现pve-installer.squashfs镜像盘的squashfs-root/ 文件夹，进入该文件夹即可看到安装时的引导系统  

  

```
$ ll
total 68
drwxr-xr-x 11 root root 4096 Mar 19 03:08 ./
dr-xr-xr-x 12 root root 4096 May 19 19:55 ../
drwxr-xr-x  2 root root 4096 Mar 19 03:08 boot/
drwxr-xr-x  2 root root 4096 Apr 27 04:25 cdrom/
drwxr-xr-x  2 root root 4096 Apr 27 04:25 devfs/
drwxr-xr-x 40 root root 4096 Apr 27 04:25 etc/
drwxr-xr-x  2 root root 4096 Apr 27 04:25 rpool/
-rwxr-xr-x  1 root root  376 Apr 26 09:53 .spice-vdagent.sh*
drwxr-xr-x  2 root root 4096 Apr 27 04:25 target/
drwxr-xr-x  2 root root 4096 Apr 27 04:25 tmp/
drwxr-xr-x  8 root root 4096 Mar 19 03:08 usr/
drwxr-xr-x  5 root root 4096 Apr 26 09:53 var/
-rw-r--r--  1 root root   87 Apr 26 09:53 .Xdefaults
-rw-r--r--  1 root root  140 Apr 26 09:53 .xinitrc

```

  

把准备好的图片替换

  

```
$ sudo cp /home/cby/Desktop/pve-banner.png .
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5b6be93678b4f5d9461aa6d82d60d97~tplv-k3u1fbpfcp-zoom-1.image)

  

     使用命令解压完成后会出现pve-base.squashfs镜像盘的squashfs-root/ 文件夹

  

```
$ sudo unsquashfs pve-base.squashfs 
Parallel unsquashfs: Using 16 processors
12892 inodes (14248 blocks) to write

[===========================================================-] 14248/14248 100%

created 10856 files
created 1385 directories
created 2024 symlinks
created 9 devices
created 0 fifos

```

  

 进入该文件夹即可看到安装后的系统根目录

  

```
$ ll
total 68
drwxr-xr-x 17 root root 4096 Apr 27 04:23 ./
dr-xr-xr-x 12 root root 4096 May 19 19:55 ../
lrwxrwxrwx  1 root root    7 Apr 27 04:22 bin -> usr/bin/
drwxr-xr-x  3 root root 4096 Apr 27 04:23 boot/
drwxr-xr-x  5 root root 4096 Apr 27 04:23 dev/
drwxr-xr-x 57 root root 4096 Apr 27 04:23 etc/
drwxr-xr-x  2 root root 4096 Mar 19 16:44 home/
lrwxrwxrwx  1 root root    7 Apr 27 04:22 lib -> usr/lib/
lrwxrwxrwx  1 root root    9 Apr 27 04:22 lib32 -> usr/lib32/
lrwxrwxrwx  1 root root    9 Apr 27 04:22 lib64 -> usr/lib64/
lrwxrwxrwx  1 root root   10 Apr 27 04:22 libx32 -> usr/libx32/
drwxr-xr-x  2 root root 4096 Apr 27 04:22 media/
drwxr-xr-x  2 root root 4096 Apr 27 04:22 mnt/
drwxr-xr-x  2 root root 4096 Apr 27 04:22 opt/
drwxr-xr-x  2 root root 4096 Mar 19 16:44 proc/
drwx------  2 root root 4096 Apr 27 04:23 root/
drwxr-xr-x  5 root root 4096 Apr 27 04:23 run/
lrwxrwxrwx  1 root root    8 Apr 27 04:22 sbin -> usr/sbin/
drwxr-xr-x  2 root root 4096 Apr 27 04:22 srv/
drwxr-xr-x  2 root root 4096 Mar 19 16:44 sys/
drwxrwxrwt  2 root root 4096 Apr 27 04:23 tmp/
drwxr-xr-x 13 root root 4096 Apr 27 04:22 usr/
drwxr-xr-x 11 root root 4096 Apr 27 04:22 var/

```

  

修改完需要定制的文件系统后，使用如下命进行打包

  

```
$ sudo mksquashfs  squashfs-root/ pve-installer.squashfs
Parallel mksquashfs: Using 16 processors
Creating 4.0 filesystem on pve-installer.squashfs-, block size 131072.
[===========================================================\] 25008/25008 100%

Exportable Squashfs 4.0 filesystem, gzip compressed, data block size 131072
  compressed data, compressed metadata, compressed fragments,
  compressed xattrs, compressed ids
  duplicates are removed
Filesystem size 340223.99 Kbytes (332.25 Mbytes)
  33.70% of uncompressed filesystem size (1009698.26 Kbytes)
Inode table size 225637 bytes (220.35 Kbytes)
  29.06% of uncompressed inode table size (776542 bytes)
Directory table size 235667 bytes (230.14 Kbytes)
  38.69% of uncompressed directory table size (609117 bytes)
Xattr table size 673 bytes (0.66 Kbytes)
  7.40% of uncompressed xattr table size (9096 bytes)
Number of duplicate files found 853
Number of inodes 22686
Number of files 19247
Number of fragments 1982
Number of symbolic links  819
Number of device nodes 0
Number of fifo nodes 0
Number of socket nodes 0
Number of directories 2620
Number of ids (unique uids + gids) 9
Number of uids 3
  root (0)
  man (6)
  syslog (104)
Number of gids 7
  root (0)
  shadow (42)
  bluetooth (112)
  utmp (43)
  staff (50)
  man (12)
  tss (111)

```

  

使用该名进行制作ISO镜像盘

  

```
$ sudo xorriso -as mkisofs -o proxmox-ve_6.4-1.iso -r -V 'inspur' --grub2-mbr proxmox.mbr --protective-msdos-label -efi-boot-part --efi-boot-image  -c '/boot/boot.cat' -b '/boot/grub/i386-pc/eltorito.img' -no-emul-boot -boot-load-size 4 -boot-info-table --grub2-boot-info -eltorito-alt-boot -e '/efi.img' -no-emul-boot .
xorriso 1.5.2 : RockRidge filesystem manipulator, libburnia project.

Drive current: -outdev 'stdio:proxmox-ve_6.4-1.iso'
Media current: stdio file, overwriteable
Media status : is blank
Media summary: 0 sessions, 0 data blocks, 0 data, 78.0g free
xorriso : WARNING : -volid text does not comply to ISO 9660 / ECMA 119 rules
Added to ISO image: directory '/'='/home/cby/chenby'
xorriso : UPDATE :   32892 files added in 1 seconds
xorriso : UPDATE :   32892 files added in 1 seconds
xorriso : NOTE : Copying to System Area: 512 bytes from file '/home/cby/chenby/proxmox.mbr'
libisofs: NOTE : Automatically adjusted MBR geometry to 1021/155/32
xorriso : UPDATE :  0.66% done
xorriso : UPDATE :  8.03% done
xorriso : UPDATE :  19.34% done
xorriso : UPDATE :  34.06% done, estimate finish Wed May 19 19:46:25 2021
xorriso : UPDATE :  48.84% done, estimate finish Wed May 19 19:46:24 2021
xorriso : UPDATE :  61.72% done, estimate finish Wed May 19 19:46:24 2021
xorriso : UPDATE :  73.41% done, estimate finish Wed May 19 19:46:25 2021
xorriso : UPDATE :  82.19% done, estimate finish Wed May 19 19:46:25 2021
xorriso : UPDATE :  92.15% done
xorriso : UPDATE :  97.28% done
ISO image produced: 1264917 sectors
Written to medium : 1264917 sectors at LBA 0
Writing to 'stdio:proxmox-ve_6.4-1.iso' completed successfully.
```

  

    使用新创建的ISO镜像盘启动后，已出现修改过后的背景图，以此类推，通过修改根目录文件，可以实现完全定制化的pve系统。  

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2598c6d809e4791b7cac905c02fe787~tplv-k3u1fbpfcp-zoom-1.image)

  

    若修改安装后的管理后台的页面，在proxmox/packages目录下找到pve-manager的deb安装包。

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2830eb0747a04abab976cb403960c347~tplv-k3u1fbpfcp-zoom-1.image)

  

```
$ ls | grep manager
pve-ha-manager_3.1-1_amd64.deb
pve-manager_6.4-4_amd64.deb
```

  

```
$ mkdir extract，在当前目录下新建文件夹，用于存放解压后的内容
$ mkdir extract/DEBIAN，新建DEBIAN目录用于存放包的控制信息
$ sudo dpkg -X ./pve-manager_6.4-4_amd64.deb extract/，将要修改的deb包解压到extract目录下，可以看到：
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d513af1f6a44bfb9788cd5bcc18ca7c~tplv-k3u1fbpfcp-zoom-1.image)

  

  

    在其解压出来的包内修改所需的代码后，导入debian包的控制信息，可以使用命令再次打包成deb包。  

```
$ sudo dpkg-deb -e ./pve-manager_6.4-4_amd64.deb extract/DEBIAN/
$ ls extract/DEBIAN/
conffiles  control  md5sums  postinst  postrm  preinst  prerm  triggers

$ sudo dpkg-deb -b ./extract 123.deb
dpkg-deb: building package 'pve-manager' in '123.deb'.

$ ll 123.deb 
-rw-r--r-- 1 root root 2042764 May 19 21:54 123.deb

```

  

查看deb包的详细信息。  

  

```
$ dpkg-deb -I 123.deb 
 new Debian package, version 2.0.
 size 2042764 bytes: control archive=16976 bytes.
     320 bytes,    10 lines      conffiles            
    1532 bytes,    15 lines      control              
   56553 bytes,   574 lines      md5sums              
    3246 bytes,   101 lines   *  postinst             #!/bin/sh
    1645 bytes,    44 lines   *  postrm               #!/bin/sh
     192 bytes,     5 lines   *  preinst              #!/bin/sh
     626 bytes,    24 lines   *  prerm                #!/bin/sh
      33 bytes,     1 lines      triggers             
 Package: pve-manager
 Version: 6.4-4
 Architecture: amd64
 Maintainer: Proxmox Support Team <support@proxmox.com>
 Installed-Size: 9876
 Depends: apt-transport-https | apt (>= 1.5~), ca-certificates, cstream, dtach, fonts-font-awesome, gdisk, hdparm, ifenslave (>= 2.6) | ifupdown2 (>= 2.0.1-1+pve8), libapt-pkg-perl, libc6 (>= 2.14), libcrypt-ssleay-perl, libfile-readbackwards-perl, libfilesys-df-perl, libjs-extjs (>= 6.0.1), libjson-perl, liblwp-protocol-https-perl, libnet-dns-perl, libproxmox-acme-perl, libpve-access-control (>= 6.0-6), libpve-cluster-api-perl, libpve-cluster-perl (>= 6.1-6), libpve-common-perl (>= 6.2-2), libpve-guest-common-perl (>= 3.1-5), libpve-http-server-perl (>= 3.2-1), libpve-storage-perl (>= 6.3-6), librados2-perl, libtemplate-perl, libterm-readline-gnu-perl, liburi-perl, libuuid-perl, libwww-perl (>= 6.04-1), logrotate, lsb-base, lzop, zstd, novnc-pve, pciutils, perl (>= 5.10.0-19), postfix | mail-transport-agent, proxmox-mini-journalreader, proxmox-widget-toolkit (>= 2.5-2), pve-cluster (>= 6.0-4), pve-container (>= 2.0-21), pve-docs, pve-firewall, pve-ha-manager, pve-i18n (>= 1.0-3), pve-xtermjs (>= 0.1-1), qemu-server (>= 6.2-17), rsync, spiceterm, systemd, vncterm, wget
 Suggests: libpve-network-perl (>= 0.5-1)
 Conflicts: vlan, vzdump
 Breaks: libpve-network-perl (<< 0.5-1)
 Replaces: vlan, vzdump
 Provides: vlan, vzdump
 Section: admin
 Priority: optional
 Description: Proxmox Virtual Environment Management Tools
  This package contains the Proxmox Virtual Environment management tools.

```

  

    将打好的deb包放回到原目录后，在进行ISO的打包，这样在安装系统后的镜像即可是定制化的页面。  

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b5b253b5505485a9bfb75681a811ef9~tplv-k3u1fbpfcp-zoom-1.image)

  

```