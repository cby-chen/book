---
layout: post
cid: 73
title: GitLab 安装部署使用
slug: 73
date: 2022/01/04 10:43:19
updated: 2022/01/04 10:43:19
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


  

GitLab介绍

GitLab：是一个基于Git实现的在线代码仓库托管软件，你可以用gitlab自己搭建一个类似于Github一样的系统，一般用于在企业、学校等内部网络搭建git私服。

功能：Gitlab 是一个提供代码托管、提交审核和问题跟踪的代码管理平台。对于软件工程质量管理非常重要。

版本：GitLab 分为社区版（CE） 和企业版（EE）。

  

Gitlab的服务构成

Nginx：静态web服务器。

gitlab-shell：用于处理Git命令和修改authorized keys列表。（Ruby）

gitlab-workhorse: 轻量级的反向代理服务器。（go）

logrotate：日志文件管理工具。

postgresql：数据库。

redis：缓存数据库。

sidekiq：用于在后台执行队列任务（异步执行）。（Ruby）

unicorn：An HTTP server for Rack applications，GitLab Rails应用是托管在这个服务器上面的。（Ruby Web Server,主要使用Ruby编写）

  

\* GitLab Workhorse是一个敏捷的反向代理。它会处理一些大的HTTP请求，比如文件上传、文件下载、Git push/pull和Git包下载。其它请求会反向代理到GitLab Rails应用，即反向代理给后端的unicorn。  
  

01

—

  

安装Gitlab主程序
===========

```
root@hello:~# apt update && apt upgrade
root@hello:~# apt install -y curl openssh-server ca-certificates tzdata perl

root@hello:~# apt install -y postfix
root@hello:~# curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash

root@hello:~# apt install gitlab-ee
```

  

02

—

修改配置文件
======

```
root@hello:~# vim /etc/gitlab/gitlab.rb

external_url 'http://192.168.1.88'

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qiye.aliyun.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "cby"
gitlab_rails['smtp_password'] = "Cby123.."
gitlab_rails['smtp_domain'] = "chenby.cn"
gitlab_rails['smtp_authentication'] = "plain"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['smtp_pool'] = false


root@hello:~# gitlab-ctl reconfigure

root@hello:~#  gitlab-ctl restart
ok: run: alertmanager: (pid 63590) 1s
ok: run: gitaly: (pid 63610) 1s
ok: run: gitlab-exporter: (pid 63641) 0s
ok: run: gitlab-workhorse: (pid 63643) 1s
ok: run: grafana: (pid 63659) 0s
ok: run: logrotate: (pid 63676) 1s
ok: run: nginx: (pid 63682) 0s
ok: run: node-exporter: (pid 63718) 1s
ok: run: postgres-exporter: (pid 63728) 0s
ok: run: postgresql: (pid 63737) 0s
ok: run: prometheus: (pid 63746) 1s
ok: run: puma: (pid 63777) 1s
ok: run: redis: (pid 63782) 0s
ok: run: redis-exporter: (pid 63788) 1s
ok: run: sidekiq: (pid 63887) 1s
root@hello:~#
```

  

03

—

查看root密码
========

```
root@hello:~# cat /etc/gitlab/initial_root_password
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: HUd9b632LHN89WXYEVYPssWGpyJrgK7BJLbVLC4VCas=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
root@hello:~# 

```

  

04

—

常用命令
====

```
    gitlab-ctl start    # 启动所有 gitlab 组件；
    gitlab-ctl stop        # 停止所有 gitlab 组件；
    gitlab-ctl restart        # 重启所有 gitlab 组件；
    gitlab-ctl status        # 查看服务状态；
    vim /etc/gitlab/gitlab.rb        # 修改gitlab配置文件；
    gitlab-ctl reconfigure        # 重新编译gitlab的配置；
    gitlab-rake gitlab:check SANITIZE=true --trace    # 检查gitlab；
    gitlab-ctl tail        # 查看日志；
    gitlab-ctl tail nginx/gitlab_access.log
    
    日志地址：/var/log/gitlab/   # 对应各服务的打印日志 
    服务地址：/var/opt/gitlab/   # 对应各服务的主目录
```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22cd5b80fca44d1691c263ce61aeb410~tplv-k3u1fbpfcp-zoom-1.image)

  

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
