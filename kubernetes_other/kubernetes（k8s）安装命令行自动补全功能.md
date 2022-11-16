---
layout: post
cid: 39
title: kubernetes（k8s）安装命令行自动补全功能
slug: 39
date: 2021/12/30 17:08:00
updated: 2022/03/25 15:44:12
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


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f6c9c76f1d04d2e9939b271bfcfa0ba~tplv-k3u1fbpfcp-zoom-1.image)

  

Ubuntu下安装命令

  

```
root@master1:~# apt install -y bash-completion
Reading package lists... Done
Building dependency tree      
Reading state information... Done
bash-completion is already the newest version (1:2.10-1ubuntu1).
0 upgraded, 0 newly installed, 0 to remove and 29 not upgraded.
```

  

centos下安装命令

  

```
[root@dss ~]# yum install bash-completion -y
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * epel: mirrors.tuna.tsinghua.edu.cn
Package 1:bash-completion-2.1-8.el7.noarch already installed and latest version
Nothing to do
[root@dss ~]#
```

  

```
root@master1:~# locate bash_completion
/etc/bash_completion
/etc/bash_completion.d
/etc/bash_completion.d/apport_completion
/etc/bash_completion.d/git-prompt
/etc/profile.d/bash_completion.sh
/snap/core18/2128/etc/bash_completion
/snap/core18/2128/usr/share/bash-completion/bash_completion
/snap/core18/2128/usr/share/doc/bash/README.md.bash_completion.gz
/snap/core18/2128/usr/share/perl5/Debian/Debhelper/Sequence/bash_completion.pm
/snap/lxd/21029/etc/bash_completion.d
/snap/lxd/21029/etc/bash_completion.d/snap.lxd.lxc
/usr/share/bash-completion/bash_completion
/usr/share/doc/bash/README.md.bash_completion.gz
/usr/share/perl5/Debian/Debhelper/Sequence/bash_completion.pm
/var/lib/docker/overlay2/0f27e9d2ca7fbe8a3b764a525f1c58990345512fa6dfe4162aba3e05ccff5b56/diff/etc/bash_completion.d
/var/lib/docker/overlay2/5eb1b0cb946881e1081bfa7a608b6fa85dbf2cb7e67f84b038f3b8a85bd13196/diff/usr/local/lib/node_modules/npm/node_modules/dashdash/etc/dashdash.bash_completion.in
/var/lib/docker/overlay2/76c41c1d1eb6eaa7b9259bd822a4bffebf180717a24319d2ffec3b4dcae0e66a/merged/etc/bash_completion.d
/var/lib/docker/overlay2/78b8ab76c0e0ad7ee873daab9ab3987a366ec32fda68a4bb56a218c7f8806a58/merged/etc/profile.d/bash_completion.sh
/var/lib/docker/overlay2/78b8ab76c0e0ad7ee873daab9ab3987a366ec32fda68a4bb56a218c7f8806a58/merged/usr/share/bash-completion/bash_completion
/var/lib/docker/overlay2/802133f75f62596a2c173f1b57231efbe210eddd7a43770a62ca94c86ce2ca56/merged/usr/local/lib/node_modules/npm/node_modules/dashdash/etc/dashdash.bash_completion.in
/var/lib/docker/overlay2/ee672bdd0bf0fdf590f9234a8a784ca12c262c47a0ac8ab91acc0942dfafc339/diff/etc/profile.d/bash_completion.sh
/var/lib/docker/overlay2/ee672bdd0bf0fdf590f9234a8a784ca12c262c47a0ac8ab91acc0942dfafc339/diff/usr/share/bash-completion/bash_completion
```

  

临时环境变量

  

```
root@master1:~# source /usr/share/bash-completion/bash_completion
root@master1:~# source <(kubectl completion bash)
root@master1:~#
root@master1:~#
root@master1:~# kubectl
annotate       auth           config         delete         exec           kustomize      plugin         run            uncordon
api-resources  autoscale      cordon         describe       explain        label          port-forward   scale          version
api-versions   certificate    cp             diff           expose         logs           proxy          set            wait
apply          cluster-info   create         drain          get            options        replace        taint          
attach         completion     debug          edit           help           patch          rollout        top            
root@master1:~# kubectl
```

  

永久写入环境变量配置文件

  

```
root@master1:~#
root@master1:~#
root@master1:~# echo "source <(kubectl completion bash)" >> ~/.bashrc
root@master1:~#
root@master1:~# cat ~/.bashrc


----略----


# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'


# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.


if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi


# enable programmable completion features (you don't need to enable
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).
#if [ -f /etc/bash_completion ] && ! shopt -oq posix; then
#    . /etc/bash_completion
#fi


source <(kubectl completion bash)
root@master1:~#
```

  

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f15584c5a770436f943162d88635f215~tplv-k3u1fbpfcp-zoom-1.image)  

  
  

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

知乎、CSDN、开源中国、思否、掘金、哔哩哔哩、腾讯云

```