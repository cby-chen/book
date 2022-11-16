---
layout: post
cid: 14
title: 人工智能NVIDIA显卡计算（CUDA+CUDNN）平台搭建
slug: 14
date: 2021/12/30 17:01:00
updated: 2022/03/25 15:52:35
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


    NVIDIA是GPU（图形处理器）的发明者，也是人工智能计算的引领者。我们创建了世界上最大的游戏平台和世界上最快的超级计算机。

      

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1df8d980dd4e4e8280c92cfcace864a3~tplv-k3u1fbpfcp-zoom-1.image)

  

    第一步，首先安装N卡驱动。  

```
cby@cby-Inspiron-7577:~$ sudo add-apt-repository ppa:graphics-drivers/ppa
[sudo] cby 的密码：
PPA publishes dbgsym, you may need to include 'main/debug' component
Repository: 'deb http://ppa.launchpad.net/graphics-drivers/ppa/ubuntu/ hirsute main'
Description:
Fresh drivers from upstream, currently shipping Nvidia.

## Current Status

Current long-lived branch release: `nvidia-430` (430.40)
Dropped support for Fermi series (https://nvidia.custhelp.com/app/answers/detail/a_id/4656)

Old long-lived branch release: `nvidia-390` (390.129)

For GF1xx GPUs use `nvidia-390` (390.129)
For G8x, G9x and GT2xx GPUs use `nvidia-340` (340.107)
For NV4x and G7x GPUs use `nvidia-304` (304.137) End-Of-Life!

Support timeframes for Unix legacy GPU releases:
https://nvidia.custhelp.com/app/answers/detail/a_id/3142

## What we're working on right now:

- Normal driver updates
- Help Wanted: Mesa Updates for Intel/AMD users, ping us if you want to help do this work, we're shorthanded.

## WARNINGS:

This PPA is currently in testing, you should be experienced with packaging before you dive in here:

Volunteers welcome!

### How you can help:

## Install PTS and benchmark your gear:

    sudo apt-get install phoronix-test-suite

Run the benchmark:

    phoronix-test-suite default-benchmark openarena xonotic tesseract gputest unigine-valley

and then say yes when it asks you to submit your results to openbechmarking.org. Then grab a cup of coffee, it takes a bit for the benchmarks to run. Depending on the version of Ubuntu you're using it might preferable for you to grabs PTS from upstream directly: http://www.phoronix-test-suite.com/?k=downloads

## Share your results with the community:

Post a link to your results (or any other feedback to): https://launchpad.net/~graphics-drivers-testers

Remember to rerun and resubmit the benchmarks after driver upgrades, this will allow us to gather a bunch of data on performance that we can share with everybody.

If you run into old documentation referring to other PPAs, you can help us by consolidating references to this PPA.

If someone wants to go ahead and start prototyping on `software-properties-gtk` on what the GUI should look like, please start hacking!

## Help us Help You!

We use the donation funds to get the developers hardware to test and upload these drivers, please consider donating to the "community" slider on the donation page if you're loving this PPA:

http://www.ubuntu.com/download/desktop/contribute
More info: https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa
Adding repository.
Press [ENTER] to continue or Ctrl-c to cancel.
Adding deb entry to /etc/apt/sources.list.d/graphics-drivers-ubuntu-ppa-hirsute.list
Adding disabled deb-src entry to /etc/apt/sources.list.d/graphics-drivers-ubuntu-ppa-hirsute.list
Adding key to /etc/apt/trusted.gpg.d/graphics-drivers-ubuntu-ppa.gpg with fingerprint 2388FF3BE10A76F638F80723FCAE110B1118213C
命中:1 http://security.ubuntu.com/ubuntu hirsute-security InRelease
命中:3 http://dl.google.com/linux/chrome/deb stable InRelease                 
获取:4 http://ppa.launchpad.net/graphics-drivers/ppa/ubuntu hirsute InRelease [24.4 kB]
命中:5 http://cn.archive.ubuntu.com/ubuntu hirsute InRelease                  
获取:6 http://cn.archive.ubuntu.com/ubuntu hirsute-updates InRelease [109 kB]
获取:7 http://ppa.launchpad.net/graphics-drivers/ppa/ubuntu hirsute/main amd64 Packages [23.4 kB]
命中:8 http://cn.archive.ubuntu.com/ubuntu hirsute-backports InRelease        
获取:9 http://ppa.launchpad.net/graphics-drivers/ppa/ubuntu hirsute/main i386 Packages [10.6 kB]
获取:10 http://ppa.launchpad.net/graphics-drivers/ppa/ubuntu hirsute/main Translation-en [5,880 B]
忽略:2 https://developer.download.nvidia.cn/compute/machine-learning/repos/ubuntu1604/x86_64  InRelease
命中:11 https://developer.download.nvidia.cn/compute/machine-learning/repos/ubuntu1604/x86_64  Release
已下载 173 kB，耗时 21秒 (8,156 B/s)
正在读取软件包列表... 完成
cby@cby-Inspiron-7577:~$

```

  

第二步，更新源同时查看可安装的驱动

  

```
cby@cby-Inspiron-7577:~$ sudo apt update
命中:1 http://dl.google.com/linux/chrome/deb stable InRelease
命中:2 http://cn.archive.ubuntu.com/ubuntu hirsute InRelease                  
获取:4 http://cn.archive.ubuntu.com/ubuntu hirsute-updates InRelease [109 kB] 
命中:5 http://ppa.launchpad.net/graphics-drivers/ppa/ubuntu hirsute InRelease 
命中:6 http://security.ubuntu.com/ubuntu hirsute-security InRelease           
忽略:3 https://developer.download.nvidia.cn/compute/machine-learning/repos/ubuntu1604/x86_64  InRelease
命中:7 https://developer.download.nvidia.cn/compute/machine-learning/repos/ubuntu1604/x86_64  Release
命中:8 http://cn.archive.ubuntu.com/ubuntu hirsute-backports InRelease
已下载 109 kB，耗时 2秒 (64.2 kB/s)
正在读取软件包列表... 完成
正在分析软件包的依赖关系树... 完成
正在读取状态信息... 完成                 
有 4 个软件包可以升级。请执行 ‘apt list --upgradable’ 来查看它们。

cby@cby-Inspiron-7577:~$ ubuntu-drivers devices
WARNING:root:_pkg_get_support nvidia-driver-390: package has invalid Support Legacyheader, cannot determine support level
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00001C8Csv00001028sd000007FAbc03sc02i00
vendor   : NVIDIA Corporation
model    : GP107M [GeForce GTX 1050 Ti Mobile]
manual_install: True
driver   : nvidia-driver-460 - distro non-free recommended
driver   : nvidia-driver-450-server - distro non-free
driver   : nvidia-driver-390 - distro non-free
driver   : nvidia-driver-465 - distro non-free
driver   : nvidia-driver-460-server - distro non-free
driver   : nvidia-driver-418-server - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin

cby@cby-Inspiron-7577:~$ sudo apt install nvidia-driver-465
...略...
```

   

第三步，查看显卡信息  

  

```
cby@cby-Inspiron-7577:~$ nvidia-smi
Sun Jun  6 21:58:45 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 465.27       Driver Version: 465.27       CUDA Version: 11.3     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:01:00.0 Off |                  N/A |
| N/A   59C    P3    N/A /  N/A |    919MiB /  4042MiB |      6%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1976      G   /usr/lib/xorg/Xorg                351MiB |
|    0   N/A  N/A      2620      G   /usr/bin/gnome-shell              213MiB |
|    0   N/A  N/A      4020      G   gnome-control-center                1MiB |
|    0   N/A  N/A      4258      G   ...AAAAAAAAA= --shared-files      349MiB |
|    0   N/A  N/A      6333      G   /usr/bin/nvidia-settings            1MiB |
+-----------------------------------------------------------------------------+
cby@cby-Inspiron-7577:~$

```

  

第四步，安装 CUDA Toolkit Archive

  

```
https://developer.nvidia.com/cuda-toolkit-archive
```

  

在如上官网中下载安装CUDA  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ed331476af6408489e1da55e83924ad~tplv-k3u1fbpfcp-zoom-1.image)

  

第五步，根据官方教程进行安装  

  

```
cby@cby-Inspiron-7577:~$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
--2021-06-06 21:59:36--  https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
正在解析主机 developer.download.nvidia.com (developer.download.nvidia.com)... 192.254.94.202, 45.43.32.210, 45.43.32.211, ...
正在连接 developer.download.nvidia.com (developer.download.nvidia.com)|192.254.94.202|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 301 Moved Permanently
位置：https://developer.download.nvidia.cn/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin [跟随至新的 URL]
--2021-06-06 21:59:37--  https://developer.download.nvidia.cn/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
正在解析主机 developer.download.nvidia.cn (developer.download.nvidia.cn)... 124.132.138.66, 124.132.138.73, 124.132.138.69
正在连接 developer.download.nvidia.cn (developer.download.nvidia.cn)|124.132.138.66|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：190 [application/octet-stream]
正在保存至: ‘cuda-ubuntu2004.pin’

cuda-ubuntu2004.pin 100%[==================>]     190  --.-KB/s    用时 0s    

2021-06-06 21:59:37 (26.1 MB/s) - 已保存 ‘cuda-ubuntu2004.pin’ [190/190])

cby@cby-Inspiron-7577:~$ sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
cby@cby-Inspiron-7577:~$ wget https://developer.download.nvidia.com/compute/cuda/11.3.1/local_installers/cuda-repo-ubuntu2004-11-3-local_11.3.1-465.19.01-1_amd64.deb
--2021-06-06 21:59:48--  https://developer.download.nvidia.com/compute/cuda/11.3.1/local_installers/cuda-repo-ubuntu2004-11-3-local_11.3.1-465.19.01-1_amd64.deb
正在解析主机 developer.download.nvidia.com (developer.download.nvidia.com)... 45.43.32.210, 192.254.94.203, 45.43.32.211, ...
正在连接 developer.download.nvidia.com (developer.download.nvidia.com)|45.43.32.210|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 301 Moved Permanently
位置：https://developer.download.nvidia.cn/compute/cuda/11.3.1/local_installers/cuda-repo-ubuntu2004-11-3-local_11.3.1-465.19.01-1_amd64.deb [跟随至新的 URL]
--2021-06-06 21:59:48--  https://developer.download.nvidia.cn/compute/cuda/11.3.1/local_installers/cuda-repo-ubuntu2004-11-3-local_11.3.1-465.19.01-1_amd64.deb
正在解析主机 developer.download.nvidia.cn (developer.download.nvidia.cn)... 124.132.138.66, 124.132.138.73, 124.132.138.69
正在连接 developer.download.nvidia.cn (developer.download.nvidia.cn)|124.132.138.66|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：2374574280 (2.2G) [application/x-deb]
正在保存至: ‘cuda-repo-ubuntu2004-11-3-local_11.3.1-465.19.01-1_amd64.deb’

cuda-repo-ubuntu200 100%[==================>]   2.21G  44.0MB/s    用时 51s   

2021-06-06 22:00:40 (44.3 MB/s) - 已保存 ‘cuda-repo-ubuntu2004-11-3-local_11.3.1-465.19.01-1_amd64.deb’ [2374574280/2374574280])

cby@cby-Inspiron-7577:~$ sudo dpkg -i cuda-repo-ubuntu2004-11-3-local_11.3.1-465.19.01-1_amd64.deb
正在选中未选择的软件包 cuda-repo-ubuntu2004-11-3-local。
(正在读取数据库 ... 系统当前共安装有 209119 个文件和目录。)
准备解压 cuda-repo-ubuntu2004-11-3-local_11.3.1-465.19.01-1_amd64.deb  ...
正在解压 cuda-repo-ubuntu2004-11-3-local (11.3.1-465.19.01-1) ...
正在设置 cuda-repo-ubuntu2004-11-3-local (11.3.1-465.19.01-1) ...
cby@cby-Inspiron-7577:~$ sudo apt-key add /var/cuda-repo-ubuntu2004-11-3-local/7fa2af80.pub
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
OK
cby@cby-Inspiron-7577:~$ sudo apt-get update
获取:1 file:/var/cuda-repo-ubuntu2004-11-3-local  InRelease
忽略:1 file:/var/cuda-repo-ubuntu2004-11-3-local  InRelease
获取:2 file:/var/cuda-repo-ubuntu2004-11-3-local  Release [564 B]
获取:2 file:/var/cuda-repo-ubuntu2004-11-3-local  Release [564 B]
获取:3 file:/var/cuda-repo-ubuntu2004-11-3-local  Release.gpg [836 B]         
获取:3 file:/var/cuda-repo-ubuntu2004-11-3-local  Release.gpg [836 B]         
获取:4 file:/var/cuda-repo-ubuntu2004-11-3-local  Packages [30.4 kB]          
命中:5 http://cn.archive.ubuntu.com/ubuntu hirsute InRelease                  
获取:7 http://cn.archive.ubuntu.com/ubuntu hirsute-updates InRelease [109 kB] 
命中:8 http://dl.google.com/linux/chrome/deb stable InRelease                 
命中:9 http://ppa.launchpad.net/graphics-drivers/ppa/ubuntu hirsute InRelease 
命中:10 http://security.ubuntu.com/ubuntu hirsute-security InRelease          
忽略:6 https://developer.download.nvidia.cn/compute/machine-learning/repos/ubuntu1604/x86_64  InRelease
命中:11 https://developer.download.nvidia.cn/compute/machine-learning/repos/ubuntu1604/x86_64  Release
命中:13 http://cn.archive.ubuntu.com/ubuntu hirsute-backports InRelease
已下载 109 kB，耗时 2秒 (52.6 kB/s)
正在读取软件包列表... 完成
cby@cby-Inspiron-7577:~$ 
cby@cby-Inspiron-7577:~$ 
cby@cby-Inspiron-7577:~$ sudo apt-get -y install cuda
正在读取软件包列表... 完成
正在分析软件包的依赖关系树... 完成
正在读取状态信息... 完成                 
下列软件包是自动安装的并且现在不需要了：
  libaccinj64-11.2 libcub-dev libcublas11 libcublaslt11 libcudart11.0
  libcufft10 libcufftw10 libcuinj64-11.2 libcupti-dev libcupti-doc
  libcupti11.2 libcurand10 libcusolver11 libcusolvermg11 libcusparse11
  libegl-dev libgl-dev libgl1-mesa-dev libgles-dev libgles1 libglvnd-dev
  libglx-dev libnppc11 libnppial11 libnppicc11 libnppidei11 libnppif11
  libnppig11 libnppim11 libnppist11 libnppisu11 libnppitc11 libnpps11
  libnvblas11 libnvjpeg11 libnvrtc11.2 libnvtoolsext1 libnvvm4 libopengl-dev
  libopengl0 libpthread-stubs0-dev libthrust-dev libvdpau-dev libx11-dev
  libxau-dev libxcb1-dev libxdmcp-dev node-html5shiv nsight-compute
  nsight-compute-target nsight-systems nsight-systems-target nvidia-cuda-gdb
  nvidia-cuda-toolkit-doc nvidia-opencl-dev nvidia-profiler
  nvidia-visual-profiler ocl-icd-opencl-dev opencl-c-headers
  opencl-clhpp-headers x11proto-dev xorg-sgml-doctools xtrans-dev
使用'sudo apt autoremove'来卸载它(它们)。
将会同时安装下列软件：
  cuda-11-3 cuda-command-line-tools-11-3 cuda-compiler-11-3 cuda-cudart-11-3
  cuda-cudart-dev-11-3 cuda-cuobjdump-11-3 cuda-cupti-11-3
  cuda-cupti-dev-11-3 cuda-cuxxfilt-11-3 cuda-demo-suite-11-3
  cuda-documentation-11-3 cuda-driver-dev-11-3 cuda-drivers cuda-drivers-465
  cuda-gdb-11-3 cuda-libraries-11-3 cuda-libraries-dev-11-3
  cuda-memcheck-11-3 cuda-nsight-11-3 cuda-nsight-compute-11-3
  cuda-nsight-systems-11-3 cuda-nvcc-11-3 cuda-nvdisasm-11-3
  cuda-nvml-dev-11-3 cuda-nvprof-11-3 cuda-nvprune-11-3 cuda-nvrtc-11-3
  cuda-nvrtc-dev-11-3 cuda-nvtx-11-3 cuda-nvvp-11-3 cuda-runtime-11-3
  cuda-samples-11-3 cuda-sanitizer-11-3 cuda-thrust-11-3 cuda-toolkit-11-3
  cuda-toolkit-11-3-config-common cuda-toolkit-11-config-common
  cuda-toolkit-config-common cuda-tools-11-3 cuda-visual-tools-11-3
  default-jre default-jre-headless libcublas-11-3 libcublas-dev-11-3
  libcufft-11-3 libcufft-dev-11-3 libcurand-11-3 libcurand-dev-11-3
  libcusolver-11-3 libcusolver-dev-11-3 libcusparse-11-3 libcusparse-dev-11-3
  libnpp-11-3 libnpp-dev-11-3 libnvjpeg-11-3 libnvjpeg-dev-11-3
  nsight-compute-2021.1.1 nsight-systems-2021.1.3 nvidia-modprobe
  nvidia-settings openjdk-11-jre openjdk-11-jre-headless
建议安装：
  fonts-ipafont-gothic fonts-ipafont-mincho fonts-wqy-microhei
  | fonts-wqy-zenhei
下列【新】软件包将被安装：
  cuda cuda-11-3 cuda-command-line-tools-11-3 cuda-compiler-11-3
  cuda-cudart-11-3 cuda-cudart-dev-11-3 cuda-cuobjdump-11-3 cuda-cupti-11-3
  cuda-cupti-dev-11-3 cuda-cuxxfilt-11-3 cuda-demo-suite-11-3
  cuda-documentation-11-3 cuda-driver-dev-11-3 cuda-drivers cuda-drivers-465
  cuda-gdb-11-3 cuda-libraries-11-3 cuda-libraries-dev-11-3
  cuda-memcheck-11-3 cuda-nsight-11-3 cuda-nsight-compute-11-3
  cuda-nsight-systems-11-3 cuda-nvcc-11-3 cuda-nvdisasm-11-3
  cuda-nvml-dev-11-3 cuda-nvprof-11-3 cuda-nvprune-11-3 cuda-nvrtc-11-3
  cuda-nvrtc-dev-11-3 cuda-nvtx-11-3 cuda-nvvp-11-3 cuda-runtime-11-3
  cuda-samples-11-3 cuda-sanitizer-11-3 cuda-thrust-11-3 cuda-toolkit-11-3
  cuda-toolkit-11-3-config-common cuda-toolkit-11-config-common
  cuda-toolkit-config-common cuda-tools-11-3 cuda-visual-tools-11-3
  default-jre default-jre-headless libcublas-11-3 libcublas-dev-11-3
  libcufft-11-3 libcufft-dev-11-3 libcurand-11-3 libcurand-dev-11-3
  libcusolver-11-3 libcusolver-dev-11-3 libcusparse-11-3 libcusparse-dev-11-3
  libnpp-11-3 libnpp-dev-11-3 libnvjpeg-11-3 libnvjpeg-dev-11-3
  nsight-compute-2021.1.1 nsight-systems-2021.1.3 nvidia-modprobe
  openjdk-11-jre openjdk-11-jre-headless
下列软件包将被升级：
  nvidia-settings
升级了 1 个软件包，新安装了 62 个软件包，要卸载 0 个软件包，有 5 个软件包未被升级。
需要下载 37.4 MB/2,162 MB 的归档。
解压缩后会消耗 4,934 MB 的额外空间。
获取:1 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-toolkit-config-common 11.3.109-1 [16.1 kB]
获取:2 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-toolkit-11-config-common 11.3.109-1 [16.1 kB]
获取:3 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-toolkit-11-3-config-common 11.3.109-1 [16.1 kB]
获取:4 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-cudart-11-3 11.3.109-1 [157 kB]
获取:5 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nvrtc-11-3 11.3.109-1 [26.1 MB]
获取:6 http://cn.archive.ubuntu.com/ubuntu hirsute-updates/main amd64 openjdk-11-jre-headless amd64 11.0.11+9-0ubuntu2 [37.2 MB]
获取:7 file:/var/cuda-repo-ubuntu2004-11-3-local  libcublas-11-3 11.5.1.109-1 [168 MB]
获取:8 file:/var/cuda-repo-ubuntu2004-11-3-local  libcufft-11-3 10.4.2.109-1 [108 MB]
获取:9 file:/var/cuda-repo-ubuntu2004-11-3-local  libcurand-11-3 10.2.4.109-1 [39.4 MB]
获取:10 file:/var/cuda-repo-ubuntu2004-11-3-local  libcusolver-11-3 11.1.2.109-1 [85.5 MB]
获取:11 file:/var/cuda-repo-ubuntu2004-11-3-local  libcusparse-11-3 11.6.0.109-1 [104 MB]
获取:12 file:/var/cuda-repo-ubuntu2004-11-3-local  libnpp-11-3 11.3.3.95-1 [75.1 MB]
获取:13 file:/var/cuda-repo-ubuntu2004-11-3-local  libnvjpeg-11-3 11.5.0.109-1 [1,742 kB]
获取:14 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-libraries-11-3 11.3.1-1 [2,498 B]
获取:15 file:/var/cuda-repo-ubuntu2004-11-3-local  nvidia-modprobe 465.19.01-0ubuntu1 [19.9 kB]
获取:16 file:/var/cuda-repo-ubuntu2004-11-3-local  nvidia-settings 465.19.01-0ubuntu1 [928 kB]
获取:17 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-drivers-465 465.19.01-1 [2,628 B]
获取:18 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-drivers 465.19.01-1 [2,504 B]
获取:19 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-runtime-11-3 11.3.1-1 [2,424 B]
获取:20 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-cuobjdump-11-3 11.3.58-1 [112 kB]
获取:21 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-cuxxfilt-11-3 11.3.58-1 [44.1 kB]
获取:22 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-thrust-11-3 11.3.109-1 [981 kB]
获取:23 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-driver-dev-11-3 11.3.109-1 [26.2 kB]
获取:24 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-cudart-dev-11-3 11.3.109-1 [737 kB]
获取:25 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nvcc-11-3 11.3.109-1 [46.5 MB]
获取:26 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nvprune-11-3 11.3.58-1 [54.9 kB]
获取:27 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-compiler-11-3 11.3.1-1 [2,430 B]
获取:28 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nvrtc-dev-11-3 11.3.109-1 [23.4 kB]
获取:29 http://cn.archive.ubuntu.com/ubuntu hirsute/main amd64 default-jre-headless amd64 2:1.11-72 [3,192 B]
获取:30 http://cn.archive.ubuntu.com/ubuntu hirsute-updates/main amd64 openjdk-11-jre amd64 11.0.11+9-0ubuntu2 [177 kB]
获取:31 http://cn.archive.ubuntu.com/ubuntu hirsute/main amd64 default-jre amd64 2:1.11-72 [1,084 B]
获取:32 file:/var/cuda-repo-ubuntu2004-11-3-local  libcublas-dev-11-3 11.5.1.109-1 [171 MB]
获取:33 file:/var/cuda-repo-ubuntu2004-11-3-local  libcufft-dev-11-3 10.4.2.109-1 [181 MB]
获取:34 file:/var/cuda-repo-ubuntu2004-11-3-local  libcurand-dev-11-3 10.2.4.109-1 [39.9 MB]
获取:35 file:/var/cuda-repo-ubuntu2004-11-3-local  libcusolver-dev-11-3 11.1.2.109-1 [22.2 MB]
获取:36 file:/var/cuda-repo-ubuntu2004-11-3-local  libcusparse-dev-11-3 11.6.0.109-1 [104 MB]
获取:37 file:/var/cuda-repo-ubuntu2004-11-3-local  libnpp-dev-11-3 11.3.3.95-1 [71.7 MB]
获取:38 file:/var/cuda-repo-ubuntu2004-11-3-local  libnvjpeg-dev-11-3 11.5.0.109-1 [1,435 kB]
获取:39 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-libraries-dev-11-3 11.3.1-1 [2,526 B]
获取:40 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-cupti-11-3 11.3.111-1 [11.7 MB]
获取:41 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-cupti-dev-11-3 11.3.111-1 [2,404 kB]
获取:42 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nvdisasm-11-3 11.3.58-1 [32.9 MB]
获取:43 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-gdb-11-3 11.3.109-1 [3,622 kB]
获取:44 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-memcheck-11-3 11.3.109-1 [145 kB]
获取:45 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nvprof-11-3 11.3.111-1 [1,925 kB]
获取:46 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nvtx-11-3 11.3.109-1 [51.1 kB]
获取:47 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-sanitizer-11-3 11.3.111-1 [7,543 kB]
获取:48 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-command-line-tools-11-3 11.3.1-1 [2,478 B]
获取:49 file:/var/cuda-repo-ubuntu2004-11-3-local  nsight-compute-2021.1.1 2021.1.1.5-1 [273 MB]
获取:50 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nsight-compute-11-3 11.3.1-1 [3,704 B]
获取:51 file:/var/cuda-repo-ubuntu2004-11-3-local  nsight-systems-2021.1.3 2021.1.3.14-b695ea9 [248 MB]
获取:52 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nsight-systems-11-3 11.3.1-1 [3,296 B]
获取:53 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nsight-11-3 11.3.109-1 [119 MB]
获取:54 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nvml-dev-11-3 11.3.58-1 [73.3 kB]
获取:55 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-nvvp-11-3 11.3.111-1 [114 MB]
获取:56 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-visual-tools-11-3 11.3.1-1 [2,866 B]
获取:57 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-tools-11-3 11.3.1-1 [2,378 B]
获取:58 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-samples-11-3 11.3.58-1 [59.2 MB]
获取:59 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-documentation-11-3 11.3.111-1 [48.1 kB]
获取:60 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-toolkit-11-3 11.3.1-1 [3,300 B]
获取:61 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-demo-suite-11-3 11.3.58-1 [3,978 kB]
获取:62 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda-11-3 11.3.1-1 [2,450 B]
获取:63 file:/var/cuda-repo-ubuntu2004-11-3-local  cuda 11.3.1-1 [2,396 B]    
已下载 37.4 MB，耗时 15秒 (2,527 kB/s)                                        
Requesting to save current system state
Successfully saved as "autozsys_el84cc"
正在从软件包中解出模板：100%
正在选中未选择的软件包 cuda-toolkit-config-common。
(正在读取数据库 ... 系统当前共安装有 209232 个文件和目录。)
准备解压 .../00-cuda-toolkit-config-common_11.3.109-1_all.deb  ...
正在解压 cuda-toolkit-config-common (11.3.109-1) ...
正在选中未选择的软件包 cuda-toolkit-11-config-common。
准备解压 .../01-cuda-toolkit-11-config-common_11.3.109-1_all.deb  ...
正在解压 cuda-toolkit-11-config-common (11.3.109-1) ...
正在选中未选择的软件包 cuda-toolkit-11-3-config-common。
准备解压 .../02-cuda-toolkit-11-3-config-common_11.3.109-1_all.deb  ...
正在解压 cuda-toolkit-11-3-config-common (11.3.109-1) ...
正在选中未选择的软件包 cuda-cudart-11-3。
准备解压 .../03-cuda-cudart-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-cudart-11-3 (11.3.109-1) ...
正在选中未选择的软件包 cuda-nvrtc-11-3。
准备解压 .../04-cuda-nvrtc-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-nvrtc-11-3 (11.3.109-1) ...
正在选中未选择的软件包 libcublas-11-3。
准备解压 .../05-libcublas-11-3_11.5.1.109-1_amd64.deb  ...
正在解压 libcublas-11-3 (11.5.1.109-1) ...
正在选中未选择的软件包 libcufft-11-3。
准备解压 .../06-libcufft-11-3_10.4.2.109-1_amd64.deb  ...
正在解压 libcufft-11-3 (10.4.2.109-1) ...
正在选中未选择的软件包 libcurand-11-3。
准备解压 .../07-libcurand-11-3_10.2.4.109-1_amd64.deb  ...
正在解压 libcurand-11-3 (10.2.4.109-1) ...
正在选中未选择的软件包 libcusolver-11-3。
准备解压 .../08-libcusolver-11-3_11.1.2.109-1_amd64.deb  ...
正在解压 libcusolver-11-3 (11.1.2.109-1) ...
正在选中未选择的软件包 libcusparse-11-3。
准备解压 .../09-libcusparse-11-3_11.6.0.109-1_amd64.deb  ...
正在解压 libcusparse-11-3 (11.6.0.109-1) ...
正在选中未选择的软件包 libnpp-11-3。
准备解压 .../10-libnpp-11-3_11.3.3.95-1_amd64.deb  ...
正在解压 libnpp-11-3 (11.3.3.95-1) ...
正在选中未选择的软件包 libnvjpeg-11-3。
准备解压 .../11-libnvjpeg-11-3_11.5.0.109-1_amd64.deb  ...
正在解压 libnvjpeg-11-3 (11.5.0.109-1) ...
正在选中未选择的软件包 cuda-libraries-11-3。
准备解压 .../12-cuda-libraries-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-libraries-11-3 (11.3.1-1) ...
正在选中未选择的软件包 nvidia-modprobe。
准备解压 .../13-nvidia-modprobe_465.19.01-0ubuntu1_amd64.deb  ...
正在解压 nvidia-modprobe (465.19.01-0ubuntu1) ...
准备解压 .../14-nvidia-settings_465.19.01-0ubuntu1_amd64.deb  ...
正在解压 nvidia-settings (465.19.01-0ubuntu1) 并覆盖 (460.56-0ubuntu2) ...
正在选中未选择的软件包 cuda-drivers-465。
准备解压 .../15-cuda-drivers-465_465.19.01-1_amd64.deb  ...
正在解压 cuda-drivers-465 (465.19.01-1) ...
正在选中未选择的软件包 cuda-drivers。
准备解压 .../16-cuda-drivers_465.19.01-1_amd64.deb  ...
正在解压 cuda-drivers (465.19.01-1) ...
正在选中未选择的软件包 cuda-runtime-11-3。
准备解压 .../17-cuda-runtime-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-runtime-11-3 (11.3.1-1) ...
正在选中未选择的软件包 cuda-cuobjdump-11-3。
准备解压 .../18-cuda-cuobjdump-11-3_11.3.58-1_amd64.deb  ...
正在解压 cuda-cuobjdump-11-3 (11.3.58-1) ...
正在选中未选择的软件包 cuda-cuxxfilt-11-3。
准备解压 .../19-cuda-cuxxfilt-11-3_11.3.58-1_amd64.deb  ...
正在解压 cuda-cuxxfilt-11-3 (11.3.58-1) ...
正在选中未选择的软件包 cuda-thrust-11-3。
准备解压 .../20-cuda-thrust-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-thrust-11-3 (11.3.109-1) ...
正在选中未选择的软件包 cuda-driver-dev-11-3。
准备解压 .../21-cuda-driver-dev-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-driver-dev-11-3 (11.3.109-1) ...
正在选中未选择的软件包 cuda-cudart-dev-11-3。
准备解压 .../22-cuda-cudart-dev-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-cudart-dev-11-3 (11.3.109-1) ...
正在选中未选择的软件包 cuda-nvcc-11-3。
准备解压 .../23-cuda-nvcc-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-nvcc-11-3 (11.3.109-1) ...
正在选中未选择的软件包 cuda-nvprune-11-3。
准备解压 .../24-cuda-nvprune-11-3_11.3.58-1_amd64.deb  ...
正在解压 cuda-nvprune-11-3 (11.3.58-1) ...
正在选中未选择的软件包 cuda-compiler-11-3。
准备解压 .../25-cuda-compiler-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-compiler-11-3 (11.3.1-1) ...
正在选中未选择的软件包 cuda-nvrtc-dev-11-3。
准备解压 .../26-cuda-nvrtc-dev-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-nvrtc-dev-11-3 (11.3.109-1) ...
正在选中未选择的软件包 libcublas-dev-11-3。
准备解压 .../27-libcublas-dev-11-3_11.5.1.109-1_amd64.deb  ...
正在解压 libcublas-dev-11-3 (11.5.1.109-1) ...
正在选中未选择的软件包 libcufft-dev-11-3。
准备解压 .../28-libcufft-dev-11-3_10.4.2.109-1_amd64.deb  ...
正在解压 libcufft-dev-11-3 (10.4.2.109-1) ...
正在选中未选择的软件包 libcurand-dev-11-3。
准备解压 .../29-libcurand-dev-11-3_10.2.4.109-1_amd64.deb  ...
正在解压 libcurand-dev-11-3 (10.2.4.109-1) ...
正在选中未选择的软件包 libcusolver-dev-11-3。
准备解压 .../30-libcusolver-dev-11-3_11.1.2.109-1_amd64.deb  ...
正在解压 libcusolver-dev-11-3 (11.1.2.109-1) ...
正在选中未选择的软件包 libcusparse-dev-11-3。
准备解压 .../31-libcusparse-dev-11-3_11.6.0.109-1_amd64.deb  ...
正在解压 libcusparse-dev-11-3 (11.6.0.109-1) ...
正在选中未选择的软件包 libnpp-dev-11-3。
准备解压 .../32-libnpp-dev-11-3_11.3.3.95-1_amd64.deb  ...
正在解压 libnpp-dev-11-3 (11.3.3.95-1) ...
正在选中未选择的软件包 libnvjpeg-dev-11-3。
准备解压 .../33-libnvjpeg-dev-11-3_11.5.0.109-1_amd64.deb  ...
正在解压 libnvjpeg-dev-11-3 (11.5.0.109-1) ...
正在选中未选择的软件包 cuda-libraries-dev-11-3。
准备解压 .../34-cuda-libraries-dev-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-libraries-dev-11-3 (11.3.1-1) ...
正在选中未选择的软件包 cuda-cupti-11-3。
准备解压 .../35-cuda-cupti-11-3_11.3.111-1_amd64.deb  ...
正在解压 cuda-cupti-11-3 (11.3.111-1) ...
正在选中未选择的软件包 cuda-cupti-dev-11-3。
准备解压 .../36-cuda-cupti-dev-11-3_11.3.111-1_amd64.deb  ...
正在解压 cuda-cupti-dev-11-3 (11.3.111-1) ...
正在选中未选择的软件包 cuda-nvdisasm-11-3。
准备解压 .../37-cuda-nvdisasm-11-3_11.3.58-1_amd64.deb  ...
正在解压 cuda-nvdisasm-11-3 (11.3.58-1) ...
正在选中未选择的软件包 cuda-gdb-11-3。
准备解压 .../38-cuda-gdb-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-gdb-11-3 (11.3.109-1) ...
正在选中未选择的软件包 cuda-memcheck-11-3。
准备解压 .../39-cuda-memcheck-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-memcheck-11-3 (11.3.109-1) ...
正在选中未选择的软件包 cuda-nvprof-11-3。
准备解压 .../40-cuda-nvprof-11-3_11.3.111-1_amd64.deb  ...
正在解压 cuda-nvprof-11-3 (11.3.111-1) ...
正在选中未选择的软件包 cuda-nvtx-11-3。
准备解压 .../41-cuda-nvtx-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-nvtx-11-3 (11.3.109-1) ...
正在选中未选择的软件包 cuda-sanitizer-11-3。
准备解压 .../42-cuda-sanitizer-11-3_11.3.111-1_amd64.deb  ...
正在解压 cuda-sanitizer-11-3 (11.3.111-1) ...
正在选中未选择的软件包 cuda-command-line-tools-11-3。
准备解压 .../43-cuda-command-line-tools-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-command-line-tools-11-3 (11.3.1-1) ...
正在选中未选择的软件包 nsight-compute-2021.1.1。
准备解压 .../44-nsight-compute-2021.1.1_2021.1.1.5-1_amd64.deb  ...
正在解压 nsight-compute-2021.1.1 (2021.1.1.5-1) ...
正在选中未选择的软件包 cuda-nsight-compute-11-3。
准备解压 .../45-cuda-nsight-compute-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-nsight-compute-11-3 (11.3.1-1) ...
正在选中未选择的软件包 nsight-systems-2021.1.3。
准备解压 .../46-nsight-systems-2021.1.3_2021.1.3.14-1_amd64.deb  ...
正在解压 nsight-systems-2021.1.3 (2021.1.3.14-b695ea9) ...
正在选中未选择的软件包 cuda-nsight-systems-11-3。
准备解压 .../47-cuda-nsight-systems-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-nsight-systems-11-3 (11.3.1-1) ...
正在选中未选择的软件包 openjdk-11-jre-headless:amd64。
准备解压 .../48-openjdk-11-jre-headless_11.0.11+9-0ubuntu2_amd64.deb  ...
正在解压 openjdk-11-jre-headless:amd64 (11.0.11+9-0ubuntu2) ...
正在选中未选择的软件包 default-jre-headless。
准备解压 .../49-default-jre-headless_2%3a1.11-72_amd64.deb  ...
正在解压 default-jre-headless (2:1.11-72) ...
正在选中未选择的软件包 openjdk-11-jre:amd64。
准备解压 .../50-openjdk-11-jre_11.0.11+9-0ubuntu2_amd64.deb  ...
正在解压 openjdk-11-jre:amd64 (11.0.11+9-0ubuntu2) ...
正在选中未选择的软件包 default-jre。
准备解压 .../51-default-jre_2%3a1.11-72_amd64.deb  ...
正在解压 default-jre (2:1.11-72) ...
正在选中未选择的软件包 cuda-nsight-11-3。
准备解压 .../52-cuda-nsight-11-3_11.3.109-1_amd64.deb  ...
正在解压 cuda-nsight-11-3 (11.3.109-1) ...
正在选中未选择的软件包 cuda-nvml-dev-11-3。
准备解压 .../53-cuda-nvml-dev-11-3_11.3.58-1_amd64.deb  ...
正在解压 cuda-nvml-dev-11-3 (11.3.58-1) ...
正在选中未选择的软件包 cuda-nvvp-11-3。
准备解压 .../54-cuda-nvvp-11-3_11.3.111-1_amd64.deb  ...
正在解压 cuda-nvvp-11-3 (11.3.111-1) ...
正在选中未选择的软件包 cuda-visual-tools-11-3。
准备解压 .../55-cuda-visual-tools-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-visual-tools-11-3 (11.3.1-1) ...
正在选中未选择的软件包 cuda-tools-11-3。
准备解压 .../56-cuda-tools-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-tools-11-3 (11.3.1-1) ...
正在选中未选择的软件包 cuda-samples-11-3。
准备解压 .../57-cuda-samples-11-3_11.3.58-1_amd64.deb  ...
正在解压 cuda-samples-11-3 (11.3.58-1) ...
正在选中未选择的软件包 cuda-documentation-11-3。
准备解压 .../58-cuda-documentation-11-3_11.3.111-1_amd64.deb  ...
正在解压 cuda-documentation-11-3 (11.3.111-1) ...
正在选中未选择的软件包 cuda-toolkit-11-3。
准备解压 .../59-cuda-toolkit-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-toolkit-11-3 (11.3.1-1) ...
正在选中未选择的软件包 cuda-demo-suite-11-3。
准备解压 .../60-cuda-demo-suite-11-3_11.3.58-1_amd64.deb  ...
正在解压 cuda-demo-suite-11-3 (11.3.58-1) ...
正在选中未选择的软件包 cuda-11-3。
准备解压 .../61-cuda-11-3_11.3.1-1_amd64.deb  ...
正在解压 cuda-11-3 (11.3.1-1) ...
正在选中未选择的软件包 cuda。
准备解压 .../62-cuda_11.3.1-1_amd64.deb  ...
正在解压 cuda (11.3.1-1) ...
正在设置 cuda-toolkit-config-common (11.3.109-1) ...
正在设置 cuda-cuxxfilt-11-3 (11.3.58-1) ...
正在设置 cuda-toolkit-11-3-config-common (11.3.109-1) ...
Setting alternatives
update-alternatives: 使用 /usr/local/cuda-11.3 来在自动模式中提供 /usr/local/cuda (cuda)
update-alternatives: 使用 /usr/local/cuda-11.3 来在自动模式中提供 /usr/local/cuda-11 (cuda-11)
正在设置 cuda-toolkit-11-config-common (11.3.109-1) ...
正在设置 cuda-nvtx-11-3 (11.3.109-1) ...
正在设置 cuda-memcheck-11-3 (11.3.109-1) ...
正在设置 cuda-nvprune-11-3 (11.3.58-1) ...
正在设置 cuda-driver-dev-11-3 (11.3.109-1) ...
正在设置 openjdk-11-jre-headless:amd64 (11.0.11+9-0ubuntu2) ...
update-alternatives: 使用 /usr/lib/jvm/java-11-openjdk-amd64/bin/java 来在自动模式中提供 /usr/bin/java (java)
update-alternatives: 使用 /usr/lib/jvm/java-11-openjdk-amd64/bin/jjs 来在自动模式中提供 /usr/bin/jjs (jjs)
update-alternatives: 使用 /usr/lib/jvm/java-11-openjdk-amd64/bin/keytool 来在自动模式中提供 /usr/bin/keytool (keytool)
update-alternatives: 使用 /usr/lib/jvm/java-11-openjdk-amd64/bin/rmid 来在自动模式中提供 /usr/bin/rmid (rmid)
update-alternatives: 使用 /usr/lib/jvm/java-11-openjdk-amd64/bin/rmiregistry 来在自动模式中提供 /usr/bin/rmiregistry (rmiregistry)
update-alternatives: 使用 /usr/lib/jvm/java-11-openjdk-amd64/bin/pack200 来在自动模式中提供 /usr/bin/pack200 (pack200)
update-alternatives: 使用 /usr/lib/jvm/java-11-openjdk-amd64/bin/unpack200 来在自动模式中提供 /usr/bin/unpack200 (unpack200)
update-alternatives: 使用 /usr/lib/jvm/java-11-openjdk-amd64/lib/jexec 来在自动模式中提供 /usr/bin/jexec (jexec)
正在设置 openjdk-11-jre:amd64 (11.0.11+9-0ubuntu2) ...
正在设置 libnvjpeg-11-3 (11.5.0.109-1) ...
正在设置 cuda-nvprof-11-3 (11.3.111-1) ...
正在设置 nvidia-modprobe (465.19.01-0ubuntu1) ...
正在设置 cuda-thrust-11-3 (11.3.109-1) ...
正在设置 cuda-nvml-dev-11-3 (11.3.58-1) ...
正在设置 libnvjpeg-dev-11-3 (11.5.0.109-1) ...
正在设置 cuda-cudart-11-3 (11.3.109-1) ...
正在设置 cuda-cudart-dev-11-3 (11.3.109-1) ...
正在设置 nsight-compute-2021.1.1 (2021.1.1.5-1) ...
正在设置 libnpp-11-3 (11.3.3.95-1) ...
正在设置 libcusparse-11-3 (11.6.0.109-1) ...
正在设置 nsight-systems-2021.1.3 (2021.1.3.14-b695ea9) ...
update-alternatives: 使用 /opt/nvidia/nsight-systems/2021.1.3/target-linux-x64/nsys 来在自动模式中提供 /usr/local/bin/nsys (nsys)
update-alternatives: 错误: alternative path /opt/nvidia/nsight-systems/2021.1.3/host-linux-x64/nsight-sys doesn't exist
update-alternatives: 错误: 无 nsight-sys 的候选项
update-alternatives: 使用 /opt/nvidia/nsight-systems/2021.1.3/host-linux-x64/nsys-ui 来在自动模式中提供 /usr/local/bin/nsys-ui (nsys-ui)
正在设置 cuda-nvdisasm-11-3 (11.3.58-1) ...
正在设置 nvidia-settings (465.19.01-0ubuntu1) ...
正在设置 libcurand-11-3 (10.2.4.109-1) ...
正在设置 cuda-cuobjdump-11-3 (11.3.58-1) ...
正在设置 libcufft-11-3 (10.4.2.109-1) ...
正在设置 libcusparse-dev-11-3 (11.6.0.109-1) ...
正在设置 cuda-nvrtc-11-3 (11.3.109-1) ...
正在设置 cuda-sanitizer-11-3 (11.3.111-1) ...
正在设置 libcusolver-11-3 (11.1.2.109-1) ...
正在设置 libcublas-11-3 (11.5.1.109-1) ...
正在设置 cuda-nsight-systems-11-3 (11.3.1-1) ...
正在设置 cuda-nvrtc-dev-11-3 (11.3.109-1) ...
正在设置 libcurand-dev-11-3 (10.2.4.109-1) ...
正在设置 default-jre-headless (2:1.11-72) ...
正在设置 libcublas-dev-11-3 (11.5.1.109-1) ...
正在设置 cuda-nsight-compute-11-3 (11.3.1-1) ...
正在设置 cuda-nvcc-11-3 (11.3.109-1) ...
正在设置 cuda-drivers-465 (465.19.01-1) ...
正在设置 libnpp-dev-11-3 (11.3.3.95-1) ...
正在设置 cuda-libraries-11-3 (11.3.1-1) ...
正在设置 cuda-gdb-11-3 (11.3.109-1) ...
正在设置 default-jre (2:1.11-72) ...
正在设置 cuda-compiler-11-3 (11.3.1-1) ...
正在设置 cuda-drivers (465.19.01-1) ...
正在设置 libcufft-dev-11-3 (10.4.2.109-1) ...
正在设置 cuda-nvvp-11-3 (11.3.111-1) ...
正在设置 libcusolver-dev-11-3 (11.1.2.109-1) ...
正在设置 cuda-runtime-11-3 (11.3.1-1) ...
正在设置 cuda-cupti-11-3 (11.3.111-1) ...
正在设置 cuda-nsight-11-3 (11.3.109-1) ...
正在设置 cuda-demo-suite-11-3 (11.3.58-1) ...
正在设置 cuda-cupti-dev-11-3 (11.3.111-1) ...
正在设置 cuda-libraries-dev-11-3 (11.3.1-1) ...
正在设置 cuda-visual-tools-11-3 (11.3.1-1) ...
正在设置 cuda-samples-11-3 (11.3.58-1) ...
正在设置 cuda-command-line-tools-11-3 (11.3.1-1) ...
正在设置 cuda-documentation-11-3 (11.3.111-1) ...
正在设置 cuda-tools-11-3 (11.3.1-1) ...
正在设置 cuda-toolkit-11-3 (11.3.1-1) ...
Setting alternatives
正在设置 cuda-11-3 (11.3.1-1) ...
正在设置 cuda (11.3.1-1) ...
正在处理用于 libc-bin (2.33-0ubuntu5) 的触发器 ...
正在处理用于 man-db (2.9.4-2) 的触发器 ...
正在处理用于 mailcap (3.68ubuntu1) 的触发器 ...
正在处理用于 desktop-file-utils (0.26-1ubuntu1) 的触发器 ...
正在处理用于 hicolor-icon-theme (0.17-2) 的触发器 ...
正在处理用于 gnome-menus (3.36.0-1ubuntu1) 的触发器 ...
ZSys is adding automatic system snapshot to GRUB menu

```

  

第六步，配置环境变量  

  

```
cby@cby-Inspiron-7577:~$ vim ~/.bashrc 
cby@cby-Inspiron-7577:~$ cat ~/.bashrc
# ...略...

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-11.3/lib64

export PATH=$PATH:/usr/local/cuda-11.3/bin

export CUDA_HOME=$CUDA_HOME:/usr/local/cuda-11.3

cby@cby-Inspiron-7577:~$ source ~/.bashrc
```

  

第七步，下载安装CUDNN  

  

```
https://developer.nvidia.com/cudnn
```

  

访问如向连接，进行注册英伟达开发者，注册完成后下载安装包

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/428a416f9f334cbe95e8c787f00e7d1a~tplv-k3u1fbpfcp-zoom-1.image)

  

第八步，下载完成后进入目录后进行安装刚刚下载的安装包  

  

```
cby@cby-Inspiron-7577:~/Downloads$ ls
libcudnn8_8.2.0.53-1+cuda11.3_amd64.deb
libcudnn8-dev_8.2.0.53-1+cuda11.3_amd64.deb
libcudnn8-samples_8.2.0.53-1+cuda11.3_amd64.deb
pycharm-community-2021.1.2.tar.gz
cby@cby-Inspiron-7577:~/Downloads$ sudo apt install ./libcudnn8*
正在读取软件包列表... 完成
正在分析软件包的依赖关系树... 完成
正在读取状态信息... 完成                 
注意，选中 'libcudnn8' 而非 './libcudnn8_8.2.0.53-1+cuda11.3_amd64.deb'
注意，选中 'libcudnn8-dev' 而非 './libcudnn8-dev_8.2.0.53-1+cuda11.3_amd64.deb'
注意，选中 'libcudnn8-samples' 而非 './libcudnn8-samples_8.2.0.53-1+cuda11.3_amd64.deb'
下列软件包是自动安装的并且现在不需要了：
  libaccinj64-11.2 libcub-dev libcublas11 libcublaslt11 libcudart11.0
  libcufft10 libcufftw10 libcuinj64-11.2 libcupti-dev libcupti-doc
  libcupti11.2 libcurand10 libcusolver11 libcusolvermg11 libcusparse11
  libegl-dev libgl-dev libgl1-mesa-dev libgles-dev libgles1 libglvnd-dev
  libglx-dev libnppc11 libnppial11 libnppicc11 libnppidei11 libnppif11
  libnppig11 libnppim11 libnppist11 libnppisu11 libnppitc11 libnpps11
  libnvblas11 libnvjpeg11 libnvrtc11.2 libnvtoolsext1 libnvvm4 libopengl-dev
  libopengl0 libpthread-stubs0-dev libthrust-dev libvdpau-dev libx11-dev
  libxau-dev libxcb1-dev libxdmcp-dev node-html5shiv nsight-compute
  nsight-compute-target nsight-systems nsight-systems-target nvidia-cuda-gdb
  nvidia-cuda-toolkit-doc nvidia-opencl-dev nvidia-profiler
  nvidia-visual-profiler ocl-icd-opencl-dev opencl-c-headers
  opencl-clhpp-headers x11proto-dev xorg-sgml-doctools xtrans-dev
使用'sudo apt autoremove'来卸载它(它们)。
下列【新】软件包将被安装：
  libcudnn8 libcudnn8-dev libcudnn8-samples
升级了 0 个软件包，新安装了 3 个软件包，要卸载 0 个软件包，有 5 个软件包未被升级。
需要下载 0 B/823 MB 的归档。
解压缩后会消耗 2,899 MB 的额外空间。
获取:1 /home/cby/Downloads/libcudnn8_8.2.0.53-1+cuda11.3_amd64.deb libcudnn8 amd64 8.2.0.53-1+cuda11.3 [454 MB]
获取:2 /home/cby/Downloads/libcudnn8-dev_8.2.0.53-1+cuda11.3_amd64.deb libcudnn8-dev amd64 8.2.0.53-1+cuda11.3 [366 MB]
获取:3 /home/cby/Downloads/libcudnn8-samples_8.2.0.53-1+cuda11.3_amd64.deb libcudnn8-samples amd64 8.2.0.53-1+cuda11.3 [1,672 kB]
Requesting to save current system state
Successfully saved as "autozsys_mptxg3"
正在选中未选择的软件包 libcudnn8。
(正在读取数据库 ... 系统当前共安装有 216273 个文件和目录。)
准备解压 .../libcudnn8_8.2.0.53-1+cuda11.3_amd64.deb  ...
正在解压 libcudnn8 (8.2.0.53-1+cuda11.3) ...
正在选中未选择的软件包 libcudnn8-dev。
准备解压 .../libcudnn8-dev_8.2.0.53-1+cuda11.3_amd64.deb  ...
正在解压 libcudnn8-dev (8.2.0.53-1+cuda11.3) ...
正在选中未选择的软件包 libcudnn8-samples。
准备解压 .../libcudnn8-samples_8.2.0.53-1+cuda11.3_amd64.deb  ...
正在解压 libcudnn8-samples (8.2.0.53-1+cuda11.3) ...
正在设置 libcudnn8 (8.2.0.53-1+cuda11.3) ...
正在设置 libcudnn8-dev (8.2.0.53-1+cuda11.3) ...
update-alternatives: 使用 /usr/include/x86_64-linux-gnu/cudnn_v8.h 来在自动模
式中提供 /usr/include/cudnn.h (libcudnn)
正在设置 libcudnn8-samples (8.2.0.53-1+cuda11.3) ...
正在处理用于 libc-bin (2.33-0ubuntu5) 的触发器 ...
ZSys is adding automatic system snapshot to GRUB menu
N: 由于文件'/home/cby/Downloads/libcudnn8_8.2.0.53-1+cuda11.3_amd64.deb'无法被用户'_apt'访问，已脱离沙盒并提权为根用户来进行下载。- pkgAcquire::Run (13: 权限不够)
cby@cby-Inspiron-7577:~/Downloads$

```

  

  

第九步，进行验证  

  

```
cby@cby-Inspiron-7577:~/Downloads$ python 
Python 3.9.5 (default, May 11 2021, 08:20:37) 
[GCC 10.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import paddle
>>> paddle.utils.run_check()
Running verify PaddlePaddle program ... 
W0606 22:39:35.543371 63149 device_context.cc:404] Please NOTE: device: 0, GPU Compute Capability: 6.1, Driver API Version: 11.3, Runtime API Version: 10.2
W0606 22:39:35.572693 63149 device_context.cc:422] device: 0, cuDNN Version: 8.2.
PaddlePaddle works well on 1 GPU.
PaddlePaddle works well on 1 GPUs.
PaddlePaddle is installed successfully! Let's start deep learning with PaddlePaddle now.
>>>

```

  

第十步，运行情感训练模型，查看结果  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba97d1a1f81c4459b6278b8ce8523ec8~tplv-k3u1fbpfcp-zoom-1.image)

  

至此，已完成安装，需要注意的是：  

1.  在windows下安装及其困难，部分工具无法安装，导致无法正常运行  
    
2.  AMD的显卡是无法使用GPU进行人工智能计算的
    
3.  特别注意IDE开发环境中的PYTHON和系统中的环境
    

  
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