---
layout: post
cid: 47
title: 最新版 Harbor 在ubuntu系统上安装
slug: 47
date: 2021/12/30 17:10:00
updated: 2022/03/25 15:43:13
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


**最新版 Harbor 在ubuntu系统上安装**

The latest version of Harbor is installed on the ubuntu system

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc0278692b6d4489aee626bf3543869c~tplv-k3u1fbpfcp-zoom-1.image)

  

**安装docker**  

Install docker

  

```
root@hello:~# curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
root@hello:~#
```

  

**配置Docker Compose**

Configure Docker Compose

  

```
root@hello:~# sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   633  100   633    0     0   2444      0 --:--:-- --:--:-- --:--:--  2444
100 12.1M  100 12.1M    0     0  10.2M      0  0:00:01  0:00:01 --:--:-- 26.2M
root@hello:~#  sudo chmod +x /usr/local/bin/docker-compose
root@hello:~# sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
root@hello:~# docker-compose --version
docker-compose version 1.29.2, build 5becea4c
root@hello:~#
```

  

**下载Docker Harbor安装包**

Download the Docker Harbor installation package

  

```
root@hello:~# wget https://github.com/goharbor/harbor/releases/download/v2.3.2/harbor-offline-installer-v2.3.2.tgz
root@hello:~#
```

  

**解压安装包**

Unzip the installation package

  

```
root@hello:~# tar xvf harbor-offline-installer-v2.3.2.tgz  -C /usr/local/ 
harbor/harbor.v2.3.2.tar.gz
harbor/prepare
harbor/LICENSE
harbor/install.sh
harbor/common.sh
harbor/harbor.yml.tmpl
root@hello:~# cd /usr/local/harbor/
```

  

  

**配置证书**

Configure Certificate

  

```
root@hello:/usr/local/harbor# mkdir ca
root@hello:/usr/local/harbor# cd ca/
root@hello:/usr/local/harbor/ca# pwd
/usr/local/harbor/ca
root@hello:/usr/local/harbor/ca# openssl genrsa -des3 -out server.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
......................................+++++
...................................................................................................................................................+++++
e is 65537 (0x010001)
Enter pass phrase for server.key:
Verifying - Enter pass phrase for server.key:
root@hello:/usr/local/harbor/ca# 
root@hello:/usr/local/harbor/ca# 
root@hello:/usr/local/harbor/ca# openssl req -new -key server.key -out server.csr
Enter pass phrase for server.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:


Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
root@hello:/usr/local/harbor/ca# 
root@hello:/usr/local/harbor/ca# cp server.key server.key.org
root@hello:/usr/local/harbor/ca# openssl rsa -in server.key.org -out server.key
Enter pass phrase for server.key.org:
writing RSA key
root@hello:/usr/local/harbor/ca# openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
Signature ok
subject=C = AU, ST = Some-State, O = Internet Widgits Pty Ltd
Getting Private key
root@hello:/usr/local/harbor/ca#
```

  

  

**修改配置文件，修改 hostname 和证书路径 即可** 

Modify the configuration file, modify the hostname and certification path

  

```
root@hello:/usr/local/harbor# cp harbor.yml.tmpl harbor.yml
root@hello:/usr/local/harbor# 
root@hello:/usr/local/harbor# vim harbor.yml


root@hello:/usr/local/harbor# cat harbor.yml
# Configuration file of Harbor

hostname: harbor.chenby.cn


# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80


# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /usr/local/harbor/ca/server.crt
  private_key: /usr/local/harbor/ca/server.key

harbor_admin_password: Harbor12345

----略----

root@hello:/usr/local/harbor#
```

  

**安装**

Install

  

```
root@hello:/usr/local/harbor# ./install.sh

[Step 0]: checking if docker is installed ...

Note: docker version: 20.10.8

[Step 1]: checking docker-compose is installed ...

Note: docker-compose version: 1.29.2

[Step 2]: loading Harbor images ...
Loaded image: goharbor/redis-photon:v2.3.2
Loaded image: goharbor/nginx-photon:v2.3.2
Loaded image: goharbor/harbor-portal:v2.3.2
Loaded image: goharbor/trivy-adapter-photon:v2.3.2
Loaded image: goharbor/chartmuseum-photon:v2.3.2
Loaded image: goharbor/notary-signer-photon:v2.3.2
Loaded image: goharbor/harbor-core:v2.3.2
Loaded image: goharbor/harbor-log:v2.3.2
Loaded image: goharbor/harbor-registryctl:v2.3.2
Loaded image: goharbor/harbor-exporter:v2.3.2
Loaded image: goharbor/notary-server-photon:v2.3.2
Loaded image: goharbor/prepare:v2.3.2
Loaded image: goharbor/harbor-db:v2.3.2
Loaded image: goharbor/harbor-jobservice:v2.3.2
Loaded image: goharbor/registry-photon:v2.3.2

[Step 3]: preparing environment ...

[Step 4]: preparing harbor configs ...
prepare base dir is set to /usr/local/harbor
Clearing the configuration file: /config/portal/nginx.conf
Clearing the configuration file: /config/log/rsyslog_docker.conf
Clearing the configuration file: /config/log/logrotate.conf
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /data/secret/keys/secretkey
Successfully called func: create_root_cert
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir

[Step 5]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating harbor-portal ... done
Creating harbor-db     ... done
Creating registryctl   ... done
Creating redis         ... done
Creating registry      ... done
Creating harbor-core   ... done
Creating harbor-jobservice ... done
Creating nginx             ... done
? ----Harbor has been installed and started successfully.----
root@hello:/usr/local/harbor#
```

  

**配置dns解析，或者在本地host中配置，具体配置略**

Configure dns resolution, or configure in the local host, the specific configuration is omitted

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b75f4e007c147fa835abe195dfdf269~tplv-k3u1fbpfcp-zoom-1.image)

  

**登陆**

Sign in

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f375b72dae584633b3235b55bfec303d~tplv-k3u1fbpfcp-zoom-1.image)

  

**默认账号：admin**

**默认密码：Harbor12345**

Default account: admin

Default password: Harbor12345

  

**客户端使用**

Client use

  

```
root@hello:~# vim /etc/docker/daemon.json
root@hello:~# 
root@hello:~# cat /etc/docker/daemon.json
{
  "insecure-registries": ["https://harbor.chenby.cn"]
}
root@hello:~# 
root@hello:~# systemctl daemon-reload
root@hello:~# 
root@hello:~# 
root@hello:~# sudo systemctl restart docker
root@hello:~# docker login https://harbor.chenby.cn/
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store


Login Succeeded
root@hello:~#
```

  

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