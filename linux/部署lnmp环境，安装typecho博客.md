---
layout: post
cid: 70
title: 部署lnmp环境，安装typecho博客
slug: 70
date: 2022/01/03 16:43:00
updated: 2022/01/03 16:47:04
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


安装nginx和PHP环境
=============

  

```
root@cby:~# apt install nginx php7.4 php7.4-mysql php7.4-fpm
```

  

修改nginx配置文件
===========

```
root@cby:~# vim /etc/nginx/sites-available/default
root@cby:~# cat /etc/nginx/sites-available/default
server {
        listen 80;
        listen [::]:80;
        
        #填写域名或者IP
        server_name www.oiox.cn; 

        # SSL configuration
        #
        
        #开启ssl证书监听端口
        listen 443 ssl; 
        listen [::]:443;
        
        #配置证书
        ssl_certificate /var/www/ssl/www.oiox.cn_nginx/www.oiox.cn_bundle.pem; 
        ssl_certificate_key /var/www/ssl/www.oiox.cn_nginx/www.oiox.cn.key;
        ssl_session_timeout  5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        root /var/www/html;

        # 配置默认访问页面
        index index.php index.html index.htm index.nginx-debian.html;

        #配置访问路径
        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }
        
        # 配置跳转路由
        if (-f $request_filename/index.html) {
        rewrite (.*) $1/index.html break;
        }
        if (-f $request_filename/index.php) {
        rewrite (.*) $1/index.php;
        }
        if (!-f $request_filename) {
        rewrite (.*) /index.php;
        }

        #配置PHP访问路由
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;

                # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
                # With php-cgi (or other tcp sockets):
                #fastcgi_pass 127.0.0.1:9000;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                include        fastcgi_params;
                fastcgi_intercept_errors  on;
        }
}


# 配置其他的域名访问
server {
        listen 80;
        listen [::]:80;

        server_name aliyun.chenby.cn;

        root /var/www/cby;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }
}
root@cby:~#
```

  

启动服务并设置开机自启
===========

```
root@cby:~# nginx -t 
root@cby:~# systemctl restart nginx 


root@cby:~# systemctl  enable php7.4-fpm
Synchronizing state of php7.4-fpm.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable php7.4-fpm
root@cby:~# 
root@cby:~# 
root@cby:~# systemctl  enable nginx
Synchronizing state of nginx.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable nginx
root@cby:~#
```

  

安装docker，并使用docker启动MySQL服务
===========================

```
root@cby:~# curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

root@cby:~# mkdir /mysql 
root@cby:~# cd /mysql
root@cby:/mysql# docker run -p 3306:3306 --name mymysql --restart=always -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Cby123.. -d mysql:5.7


#登录MySQL数据库执行创建数据库
create database typecho;
```

  

  

部署typecho
=========

```
root@cby:~# cd /var/www/html/
root@cby:/var/www/html# wget https://typecho.org/downloads/1.1-17.10.30-release.tar.gz
root@cby:/var/www/html# tar xvf 1.1-17.10.30-release.tar.gz 
root@cby:/var/www/html# mv build/* .
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d61153e4ca254d199ab8e02500dc72f0~tplv-k3u1fbpfcp-zoom-1.image)

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
