---
layout: post
cid: 293
title: 在线编写Markdown
slug: 293
date: 2022/10/27 20:56:00
updated: 2022/10/27 23:44:59
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


# 在线编写Markdown


### 安装Nginx服务
```
apt install nginx
yum install nginx
```

### 修改Nginx配置

```
root@cby:~# vim /etc/nginx/sites-available/default
root@cby:~# cat /etc/nginx/sites-available/default
server {
        listen 80;
        listen [::]:80;

        server_name md.oiox.cn;

        listen 443 ssl;
        listen [::]:443;
        ssl_certificate /ssl/cert.pem;
        ssl_certificate_key /ssl/cert.key;
        ssl_session_timeout  5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        root /var/www/md;
        index index.html;

        location / {
        }

}
```

### 创建网址目录
```
root@cby:~# mkdir -pv /var/www/md/
root@cby:~# cd /var/www/md/
root@cby:/var/www/md#
```

### 克隆项目

```
root@cby:/var/www/md# git clone https://github.com/pandao/editor.md.git
Cloning into 'editor.md'...
remote: Enumerating objects: 2578, done.
remote: Total 2578 (delta 0), reused 0 (delta 0), pack-reused 2578
Receiving objects: 100% (2578/2578), 15.16 MiB | 7.84 MiB/s, done.
Resolving deltas: 100% (1313/1313), done.
root@cby:/var/www/md#
root@cby:/var/www/md# mv editor.md/ editormd/
```


### 编写首页HTML
```
root@cby:/var/www/md# vim index.html
root@cby:/var/www/md# cat index.html 
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<link rel="stylesheet" href="editormd/css/editormd.css" />
<div id="test-editor">
    <textarea style="display:none;">### 关于 Editor.md

**Editor.md** 是一款开源的、可嵌入的 Markdown 在线编辑器（组件），基于 CodeMirror、jQuery 和 Marked 构建。
    </textarea>
</div>
<script src="https://cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
<script src="editormd/editormd.js"></script>
<script type="text/javascript">
    $(function() {
        var editor = editormd("test-editor", {
            // width  : "100%",
            // height : "100%",
            path   : "editormd/lib/"
        });
    });
</script>
root@cby:/var/www/md#
```



### 访问地址
https://md.oiox.cn/
https://md.oiox.cn/editormd/examples/




> **关于**
>
> https://www.oiox.cn/
>
> https://www.oiox.cn/index.php/start-page.html
>
> **CSDN、GitHub、知乎、开源中国、思否、掘金、简书、华为云、阿里云、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客**
>
> **全网可搜《小陈运维》**
>
> **文章主要发布于微信**