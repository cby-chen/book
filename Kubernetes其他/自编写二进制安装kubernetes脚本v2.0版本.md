---
layout: post
cid: 134
title: 自编写二进制安装kubernetes脚本v2.0版本
slug: 134
date: 2022/04/04 23:15:00
updated: 2022/04/04 23:23:59
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


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63bf8ed4a3a744a5b0aae5d9275f2081~tplv-k3u1fbpfcp-zoom-1.image)

  

一键安装 二进制安装Kubernetes（k8s） v2.0

  

手动安装：https://github.com/cby-chen/Kubernetes

  

脚本安装：https://github.com/cby-chen/Binary_installation_of_Kubernetes

  

使用说明：该脚本示例需要十一台服务器，在十一台服务器中有一台是用于执行该脚本的，

  

另外有八台k8s服务器，其他俩台作为lb负载均衡服务器。

  

将其中服务器配置好静态IP，修改如下变量中的IP即可。

  

同时查看服务器中的网卡名，并将其修改。

  

执行脚本可使用bash -x 即可显示执行中详细信息。

  

该脚本已适配centos7和centos8。

  

脚本中hosts有俩处，记得修改。

  

2022-03更新：

  

现已支持centos7 和centos8 自动适配

  

同时支持自定义k8s node节点结构.

  

在变量中需要几台节点就写几台节点即可 注意的是，新增节点，要在脚本中的hosts中也要修改 不建议乱改。

  

2022-04更新：

  

更新kubernetes自主版本选择

  

优化执行结构

  

适配多版本

  

修复BUG

  

```
脚本中是需要在GitHub上下载软件包

手动提前下载好

wget https://github.com/cby-chen/Kubernetes/releases/download/cby/Kubernetes.tar
wget https://github.com/cby-chen/Kubernetes/releases/download/v1.23.4/kubernetes-v1.23.4.tar
wget https://github.com/cby-chen/Kubernetes/releases/download/v1.23.5/kubernetes-v1.23.5.tar

下载脚本

wget https://www.oiox.cn/Binary_installation_of_Kubernetes.sh

修改参数

vim Binary_installation_of_Kubernetes.sh

如下：

#每个节点的IP，以及vip
export k8s_master01="192.168.1.61"
export k8s_master02="192.168.1.61"
export k8s_master03="192.168.1.63"
export k8s_node01="192.168.1.64"
export k8s_node02="192.168.1.65"
export k8s_node03="192.168.1.66"
export k8s_node04="192.168.1.67"
export k8s_node05="192.168.1.68"
export lb_01="192.168.1.57"
export lb_02="192.168.1.58"
export lb_vip="192.168.1.59"

#物理网络ip地址段，注意反斜杠转译
export ip_segment="192.168.1.0\/24"

#k8s自定义域名
export domain="x.oiox.cn"

#服务器网卡名
export eth="ens18"


修改hosts（有俩处）

cat > /etc/hosts <<EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
$k8s_master01 k8s-master01
$k8s_master02 k8s-master02
$k8s_master03 k8s-master03
$k8s_node01 k8s-node01
$k8s_node02 k8s-node02
$k8s_node03 k8s-node03
$k8s_node04 k8s-node04
$k8s_node05 k8s-node05
$lb_01 lb01
$lb_02 lb02
$lb_vip lb-vip
EOF

执行脚本

bash -x Binary_installation_of_Kubernetes.sh

dashboard

查看端口号
kubectl get svc kubernetes-dashboard -n kubernetes-dashboard

查看token
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk 

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
