---
layout: post
cid: 42
title: KVM WEB管理工具 WebVirtMgr
slug: 42
date: 2021/12/30 17:09:18
updated: 2021/12/30 17:09:18
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


**一、webvirtmgr介绍及环境说明**

温馨提示：安装KVM是需要2台都操作的，因为我们是打算将2台都设置为宿主机所有都需要安装KVM相关组件

  

github地址https://github.com/retspen/webvirtmgr

  

WebVirtMgr是一个基于libvirt的Web界面，用于管理虚拟机。它允许您创建和配置新域，并调整域的资源分配。VNC查看器为来宾域提供完整的图形控制台。KVM是目前唯一支持的虚拟机管理程序。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c4c4b18d6b6495c8d291a1befc1d826~tplv-k3u1fbpfcp-zoom-1.image)

  

**查看服务器版本号**

```
[root@webc ~]# cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
```

  

**内核版本**

```
[root@webc ~]# uname -r
3.10.0-1160.42.2.el7.x86_64
```

  

**关闭Selinux & 防火墙**

```
[root@webc ~]# systemctl stop firewalld
[root@webc ~]# systemctl disable firewalld
[root@webc ~]# setenforce 0
setenforce: SELinux is disabled
[root@webc ~]# sed -i '/SELINUX/s/enforcing/disabled/' /etc/selinux/config
```

  

**更新软件包并安装epel扩展源**

```
[root@webc ~]# yum update
[root@webc ~]# yum install epel*
```

  

**查看python版本**

```
[root@webc ~]# python -V
Python 2.7.5
[root@webc ~]#
```

  

**查看KVM 驱动是否加载**

```
[root@webc ~]# lsmod | grep kvm
kvm_intel             188740  0
kvm                   637515  1 kvm_intel
irqbypass              13503  1 kvm
[root@webc ~]#
[root@webc ~]#
[root@webc ~]# modprobe -a kvm
[root@webc ~]# modprobe -a kvm_intel
[root@webc ~]#
```

  

**免密配置**

```
[root@webc ~]# ssh-keygen
[root@webc ~]# ssh-copy-id -i .ssh/id_rsa.pub root@192.168.1.104
```

  

**二、安装KVM**

  

**安装KVM依赖包及管理工具**

**kvm属于内核态，不需要安装。但是需要一些管理工具包**

```
[root@webc ~]# yum install qemu-img qemu-kvm qemu-kvm-tools virt-manager virt-viewer virt-v2v virt-top libvirt libvirt-Python libvirt-client python-virtinst bridge-utils tunctl
[root@webc ~]# yum install -y virt-install
[root@webc ~]#
[root@webc ~]# systemctl start libvirtd.service
[root@webc ~]# systemctl enable libvirtd.service
[root@webc ~]#
[root@webc ~]# cd cby/kvm/
[root@webc kvm]#
[root@webc kvm]#
[root@webc kvm]#  git clone https://github.com/palli/python-virtinst.git
[root@webc kvm]# cd python-virtinst/
[root@webc python-virtinst]#  python setup.py install
[root@webc python-virtinst]# virt-install
[root@webc python-virtinst]# yum install bridge-utils
[root@webc python-virtinst]#
[root@webc python-virtinst]# vim /etc/sysconfig/network-scripts/ifcfg-br0
[root@webc python-virtinst]#
[root@webc python-virtinst]#
[root@webc python-virtinst]#
[root@webc python-virtinst]#
[root@webc python-virtinst]#
[root@webc python-virtinst]# cat /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE=br0
TYPE=Bridge
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.1.49
NETMASK=255.225.255.0
GATEWAY=192.168.1.1
DNS1=192.168.1.1
[root@webc python-virtinst]# brctl show
bridge name bridge id       STP enabled interfaces
br-0d093958d245     8000.0242d5824d14   no      
br-2e2d3c481379     8000.0242884030e2   no      
br-36a6ad3375a8     8000.0242d7d7f1ef   no      
br-66a9675a6dd5     8000.024248a61c72   no      
br-b7daf4844ff7     8000.024263dd4715   no      
br-deba197eb09e     8000.0242b290e104   no      
br0     8000.000000000000   no      
docker0     8000.0242858c017c   no      vethe14f7ac
docker_gwbridge     8000.0242588c6db0   no      
virbr0      8000.5254009ba65a   yes     virbr0-nic
[root@webc python-virtinst]# ln -s /usr/libexec/qemu-kvm /usr/sbin/

```

**三、WebVirtMgr 安装**

  

**安装pip、git及supervisor && Nginx**

  

**WebVirtMgr只在管理端安装**

```
[root@webc ~]# yum -y install git python-pip libvirt-python libxml2-python python-websockify supervisor gcc python-devel

```

  

**使用pip安装Python扩展程序库**

```
[root@webc ~]# pip install numpy

```

  

**git克隆配置并运行WebVirMgr**

```
[root@webc ~]# cd cby/
[root@webc cby]# mkdir kvm
[root@webc cby]# cd kvm
[root@webc kvm]# pwd
/root/cby/kvm
[root@webc kvm]#
[root@webc kvm]# git clone git://github.com/retspen/webvirtmgr.git
正克隆到 'webvirtmgr'...
remote: Enumerating objects: 5614, done.
remote: Total 5614 (delta 0), reused 0 (delta 0), pack-reused 5614
接收对象中: 100% (5614/5614), 2.97 MiB | 748.00 KiB/s, done.
处理 delta 中: 100% (3606/3606), done.
[root@webc kvm]#
[root@webc kvm]#
[root@webc kvm]# cd webvirtmgr
[root@webc webvirtmgr]# pip install -r requirements.txt
```

```
#初始化环境
[root@webc webvirtmgr]# ./manage.py syncdb

#配置Django 静态页面
[root@webc webvirtmgr]# ./manage.py collectstatic
```

  

**启动WebVirMgr**

**前台启动WebVirMgr，默认是Debug模式同时日志打印在前台**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/789f53319bb245c69de4230c340b1585~tplv-k3u1fbpfcp-zoom-1.image)  

**用户名和密码是我们刚刚创建的**

  

**下载Nginx**

```
[root@webc webvirtmgr]# cd ..
[root@webc kvm]# ls
webvirtmgr
[root@webc kvm]#
[root@webc kvm]# mkdir nginx
[root@webc kvm]# cd nginx
[root@webc nginx]# wget https://nginx.org/download/nginx-1.20.1.tar.gz
[root@webc nginx]# tar xf nginx-1.20.1.tar.gz
[root@webc nginx]# cd nginx-1.20.1/
[root@webc nginx-1.20.1]#
```

  

**修改nginx配置文件**

```
[root@webc conf]# vim nginx.conf
[root@webc conf]#
[root@webc conf]# cat nginx.conf
user  root;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       90;
        server_name  192.168.1.104;
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
        location / {
            #root   html;
            #index  index.html index.htm;
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
            proxy_set_header Host $host:$server_port;
            proxy_set_header X-Forwarded-Proto $remote_addr;
            proxy_connect_timeout 600;
            proxy_read_timeout 600;
            proxy_send_timeout 600;
            client_max_body_size 5120M;
        }
        location /static/ {
            root /root/cby/kvm/webvirtmgr;
            expires max;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
[root@webc conf]#
```

  

**安装Nginx**

```
[root@webc nginx-1.20.1]# yum install -y gcc glibc gcc-c++ prce-devel openssl-devel pcre-devel
[root@webc nginx-1.20.1]# useradd -s /sbin/nologin nginx -M
[root@webc nginx-1.20.1]# ./configure --prefix=/root/cby/kvm/nginx/ --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module
[root@webc nginx-1.20.1]# make && make install
```

  

**启动Nginx**

```
[root@webc nginx-1.20.1]# cd /root/cby/kvm/nginx/sbin/
[root@webc sbin]# /root/cby/kvm/nginx/sbin/nginx -t
nginx: the configuration file /root/cby/kvm/nginx//conf/nginx.conf syntax is ok
nginx: configuration file /root/cby/kvm/nginx//conf/nginx.conf test is successful
[root@webc sbin]# /root/cby/kvm/nginx/sbin/nginx
```

  

**使用systemctl启停服务**

```
[root@webc sbin]# cat > /etc/supervisord.d/webvirtmgr.ini << EOF
[program:webvirtmgr]
command=/usr/bin/python /root/cby/kvm/webvirtmgr/manage.py run_gunicorn -c /root/cby/kvm/webvirtmgr/conf/gunicorn.conf.py
directory=/root/cby/kvm/webvirtmgr
autostart=true
autorestart=true
logfile=/var/log/supervisor/webvirtmgr.log
log_stderr=true
user=root
 
[program:webvirtmgr-console]
command=/usr/bin/python /root/cby/kvm/webvirtmgr/console/webvirtmgr-console
directory=/root/cby/kvm/webvirtmgr
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/webvirtmgr-console.log
redirect_stderr=true
user=root
EOF
```

  

**启动supervisor**

```
[root@webc webvirtmgr]# systemctl daemon-reload
[root@webc webvirtmgr]# systemctl stop supervisord
[root@webc webvirtmgr]# systemctl start supervisord
```

  

**查看是否启动成功**

```
[root@webc webvirtmgr]# supervisorctl status
webvirtmgr                       RUNNING   pid 23783, uptime 0:00:11
webvirtmgr-console               RUNNING   pid 23782, uptime 0:00:11
[root@webc webvirtmgr]#
```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8eb2b3bafc90410ca3460ccfae88d5cd~tplv-k3u1fbpfcp-zoom-1.image)

  

**四、Web界面配置webvirtmgr**

4.1 添加主机设置存储

1.Add Connection 添加宿主机(即KVM主机)

2.点击SSH连接

3.Label 为主机名，必须为主机名做免密

4.IP 为宿主机IP

5.用户名为服务器用户名

6.点击添加

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8da70c67a54f4f79a3b1bd14bb07873f~tplv-k3u1fbpfcp-zoom-1.image)

  

  
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