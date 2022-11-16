---
layout: post
cid: 92
title: 安装部署keepalived的HA环境
slug: 92
date: 2022/02/23 16:11:00
updated: 2022/03/25 15:36:35
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


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c45170c66b0d47aeb9bb02fe9527e720~tplv-k3u1fbpfcp-zoom-1.image)

  

每一台配置下keepalived

  

```
#master01 配置：
cat >/etc/keepalived/keepalived.conf<<"EOF"
! Configuration File for keepalived
global_defs {
   router_id LVS_DEVEL
   script_user root
   enable_script_security
}
vrrp_script chk_apiserver {
   script "/etc/keepalived/check_apiserver.sh"
   interval 5
   weight -5
   fall 2
   rise 1 #检测一次成功，则认为在线
}
vrrp_instance VI_1 {
   state BACKUP
   nopreempt
   interface ens160
   mcast_src_ip 10.0.0.20
   virtual_router_id 51
   priority 100
   advert_int 2
   authentication {
       auth_type PASS
       auth_pass K8SHA_KA_AUTH
   }
   virtual_ipaddress {
       10.0.0.30
   }
   track_script {
      chk_apiserver
   }
}
EOF


#Master02 配置：
cat >/etc/keepalived/keepalived.conf<<"EOF"
! Configuration File for keepalived
global_defs {
   router_id LVS_DEVEL
   script_user root
   enable_script_security
}
vrrp_script chk_apiserver {
   script "/etc/keepalived/check_apiserver.sh"
  interval 5
   weight -5
   fall 2 
   rise 1
}
vrrp_instance VI_1 {
   state BACKUP
   nopreempt
   interface ens160
   mcast_src_ip 10.0.0.21
   virtual_router_id 51
   priority 99
   advert_int 2
   authentication {
       auth_type PASS
       auth_pass K8SHA_KA_AUTH
   }
   virtual_ipaddress {
       10.0.0.30
   }
   track_script {
      chk_apiserver
   }
}
EOF


#Master03 配置：
cat >/etc/keepalived/keepalived.conf<<"EOF"
! Configuration File for keepalived
global_defs {
   router_id LVS_DEVEL
   script_user root
   enable_script_security
}
vrrp_script chk_apiserver {
   script "/etc/keepalived/check_apiserver.sh"
 interval 5
   weight -5
   fall 2 
   rise 1
}
vrrp_instance VI_1 {
   state BACKUP
   nopreempt
   interface ens160
   mcast_src_ip 10.0.0.22
   virtual_router_id 51
   priority 98
   advert_int 2
   authentication {
       auth_type PASS
       auth_pass K8SHA_KA_AUTH
   }
   virtual_ipaddress {
       10.0.0.30
   }
    track_script {
      chk_apiserver
   }
EOF
```

  

  

健康检查脚本

```
cat > /etc/keepalived/check_apiserver.sh <<"EOF"
#!/bin/bash
err=0
for k in $(seq 1 3)
do
   check_code=$(pgrep haproxy)
   if [[ $check_code == "" ]]; then
       err=$(expr $err + 1)
       sleep 1
       continue
   else
       err=0
       break
   fi
done


if [[ $err != "0" ]]; then
   echo "systemctl stop keepalived"
   /usr/bin/systemctl stop keepalived
   exit 1
else
   exit 0
fi
EOF


chmod u+x /etc/keepalived/check_apiserver.sh
```

  

  
```
启动服务

systemctl daemon-reload
systemctl enable --now keepalived
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