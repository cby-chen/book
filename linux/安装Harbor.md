---
layout: post
cid: 304
title: 安装Harbor
slug: 304
date: 2022/11/16 15:18:03
updated: 2022/11/16 15:18:03
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


# 安装Harbor

### 安装docker
```
# 安装 apt 依赖包
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

# 添加 Docker 的官方 GPG 密钥
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

# 使用以下指令设置稳定版仓库
add-apt-repository \
   "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/ \
  $(lsb_release -cs) \
  stable"

# 安装最新版本的 Docker Engine-Community 和 containerd 
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
```

### 安装docker compose
```
# 配置Docker Compose
root@cby:~# wget https://ghproxy.com/https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-linux-x86_64
root@cby:~# mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
root@cby:~# chmod +x /usr/local/bin/docker-compose
root@cby:~# docker-compose --version
Docker Compose version v2.12.2
root@cby:~# 
```

### 下载harbor安装包
```
# 下载Docker Harbor安装包
wget https://ghproxy.com/https://github.com/goharbor/harbor/releases/download/v2.6.2/harbor-offline-installer-v2.6.2.tgz

# 解压安装包
root@cby:~# tar xvf harbor-offline-installer-v2.6.2.tgz  -C /usr/local/
harbor/harbor.v2.6.2.tar.gz
harbor/prepare
harbor/LICENSE
harbor/install.sh
harbor/common.sh
harbor/harbor.yml.tmpl
root@cby:~# cd /usr/local/harbor/
```

### 创建证书
```
# 创建ca证书目录
root@cby:/usr/local/harbor# mkdir ca
root@cby:/usr/local/harbor# cd ca/
root@cby:/usr/local/harbor/ca# 

# 生成CA证书私钥
root@cby:/usr/local/harbor/ca# openssl genrsa -out ca.key 4096

# 生成CA证书

root@cby:/usr/local/harbor/ca# openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=hb.oiox.cn" \
 -key ca.key \
 -out ca.crt


# 生成服务器证书 生成私钥
root@cby:/usr/local/harbor/ca# openssl genrsa -out hb.oiox.cn.key 4096

# 生成证书签名请求（CSR）
root@cby:/usr/local/harbor/ca# openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=hb.oiox.cn" \
    -key hb.oiox.cn.key \
    -out hb.oiox.cn.csr

# 生成一个x509 v3扩展文件
root@cby:/usr/local/harbor/ca# cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=oiox.cn
DNS.2=hb.oiox.cn
DNS.3=www.oiox.cn
EOF

# 使用该v3.ext文件为您的Harbor主机生成证书
root@cby:/usr/local/harbor/ca# openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in hb.oiox.cn.csr \
    -out hb.oiox.cn.crt

```

### 配置docker证书
```
# 转换crt为cert，供Docker使用，Docker守护程序将.crt文件解释为CA证书，并将.cert文件解释为客户端证书
root@cby:/usr/local/harbor/ca# openssl x509 -inform PEM -in hb.oiox.cn.crt -out hb.oiox.cn.cert


# 将服务器证书，密钥和CA文件复制到Harbor主机上的Docker证书文件夹中。您必须首先创建适当的文件夹
root@cby:/usr/local/harbor/ca# mkdir -p /etc/docker/certs.d/hb.oiox.cn/
root@cby:/usr/local/harbor/ca# cp hb.oiox.cn.cert /etc/docker/certs.d/hb.oiox.cn/
root@cby:/usr/local/harbor/ca# cp hb.oiox.cn.key /etc/docker/certs.d/hb.oiox.cn/
root@cby:/usr/local/harbor/ca# cp ca.crt /etc/docker/certs.d/hb.oiox.cn/

# 如果将默认nginx端口443 映射到其他端口，请创建文件夹
# /etc/docker/certs.d/yourdomain.com:port


# 重新启动Docker Engine
root@cby:/usr/local/harbor/ca# systemctl restart docker
```

### 查看文件
```
# 查看目录下证书文件
root@cby:/usr/local/harbor/ca# ll
total 36
drwxr-xr-x 2 root root 4096 Nov 16 06:23 ./
drwxr-xr-x 5 root root 4096 Nov 16 06:16 ../
-rw-r--r-- 1 root root 2041 Nov 16 06:20 ca.crt
-rw------- 1 root root 3272 Nov 16 06:16 ca.key
-rw-r--r-- 1 root root 2143 Nov 16 06:23 hb.oiox.cn.cert
-rw-r--r-- 1 root root 2143 Nov 16 06:22 hb.oiox.cn.crt
-rw-r--r-- 1 root root 1704 Nov 16 06:22 hb.oiox.cn.csr
-rw------- 1 root root 3268 Nov 16 06:22 hb.oiox.cn.key
-rw-r--r-- 1 root root  261 Nov 16 06:22 v3.ext
root@cby:/usr/local/harbor/ca# 
```

### 配置harbor服务
```
# 配置harbor文件
root@cby:/usr/local/harbor# cp harbor.yml.tmpl harbor.yml
root@cby:/usr/local/harbor# vim harbor.yml 
root@cby:/usr/local/harbor# cat harbor.yml | grep -v '^#' | grep -v '^$' | grep -v '  #'
hostname: hb.oiox.cn
http:
  port: 80
https:
  port: 443
  certificate: /usr/local/harbor/ca/hb.oiox.cn.crt 
  private_key: /usr/local/harbor/ca/hb.oiox.cn.key
harbor_admin_password: Harbor12345
database:
  password: root123
  max_idle_conns: 100
  max_open_conns: 900
data_volume: /data
trivy:
  ignore_unfixed: false
  skip_update: false
  offline_scan: false
  security_check: vuln
  insecure: false
jobservice:
  max_job_workers: 10
notification:
  webhook_job_max_retry: 10
chart:
  absolute_url: disabled
log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor
_version: 2.6.0
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy
upload_purging:
  enabled: true
  age: 168h
  interval: 24h
  dryrun: false
cache:
  enabled: false
  expire_hours: 24
root@cby:/usr/local/harbor# 
```

### 安装harbor
```
# 进行安装
root@cby:/usr/local/harbor# ./install.sh
tput: No value for $TERM and no -T specified
tput: No value for $TERM and no -T specified
tput: No value for $TERM and no -T specified
tput: No value for $TERM and no -T specified
tput: No value for $TERM and no -T specified
tput: No value for $TERM and no -T specified
tput: No value for $TERM and no -T specified
tput: No value for $TERM and no -T specified

[Step 0]: checking if docker is installed ...

Note: docker version: 20.10.21

[Step 1]: checking docker-compose is installed ...

Note: docker-compose version: 2.12.2

[Step 2]: loading Harbor images ...
Loaded image: goharbor/harbor-jobservice:v2.6.2
Loaded image: goharbor/trivy-adapter-photon:v2.6.2
Loaded image: goharbor/chartmuseum-photon:v2.6.2
Loaded image: goharbor/redis-photon:v2.6.2
Loaded image: goharbor/nginx-photon:v2.6.2
Loaded image: goharbor/notary-signer-photon:v2.6.2
Loaded image: goharbor/harbor-core:v2.6.2
Loaded image: goharbor/harbor-db:v2.6.2
Loaded image: goharbor/harbor-registryctl:v2.6.2
Loaded image: goharbor/harbor-exporter:v2.6.2
Loaded image: goharbor/prepare:v2.6.2
Loaded image: goharbor/registry-photon:v2.6.2
Loaded image: goharbor/notary-server-photon:v2.6.2
Loaded image: goharbor/harbor-portal:v2.6.2
Loaded image: goharbor/harbor-log:v2.6.2


[Step 3]: preparing environment ...

[Step 4]: preparing harbor configs ...
prepare base dir is set to /usr/local/harbor
Clearing the configuration file: /config/core/app.conf
Clearing the configuration file: /config/core/env
Clearing the configuration file: /config/jobservice/env
Clearing the configuration file: /config/jobservice/config.yml
Clearing the configuration file: /config/nginx/nginx.conf
Clearing the configuration file: /config/registryctl/env
Clearing the configuration file: /config/registryctl/config.yml
Clearing the configuration file: /config/portal/nginx.conf
Clearing the configuration file: /config/db/env
Clearing the configuration file: /config/registry/passwd
Clearing the configuration file: /config/registry/config.yml
Clearing the configuration file: /config/log/logrotate.conf
Clearing the configuration file: /config/log/rsyslog_docker.conf
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
loaded secret from file: /data/secret/keys/secretkey
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir


Note: stopping existing Harbor instance ...


[Step 5]: starting Harbor ...
[+] Running 10/10
 ⠿ Network harbor_harbor        Created                                                                                                                                                              0.0s
 ⠿ Container harbor-log         Started                                                                                                                                                              0.6s
 ⠿ Container harbor-portal      Started                                                                                                                                                              0.8s
 ⠿ Container registryctl        Started                                                                                                                                                              1.1s
 ⠿ Container redis              Started                                                                                                                                                              0.9s
 ⠿ Container registry           Started                                                                                                                                                              1.1s
 ⠿ Container harbor-db          Started                                                                                                                                                              1.2s
 ⠿ Container harbor-core        Started                                                                                                                                                              1.3s
 ⠿ Container nginx              Started                                                                                                                                                              1.9s
 ⠿ Container harbor-jobservice  Started                                                                                                                                                              2.0s
✔ ----Harbor has been installed and started successfully.----
root@cby:/usr/local/harbor# 
root@cby:/usr/local/harbor# 
root@cby:/usr/local/harbor#
```

### 配置解析和docker
```
# FQDN解析
cat > /etc/hosts <<EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6


192.168.8.61 k8s-master01
192.168.8.62 k8s-master02
192.168.8.63 k8s-master03
192.168.8.64 k8s-node01
192.168.8.65 k8s-node02
192.168.8.66 lb-vip
192.168.8.3 hb.oiox.cn
EOF


# 例如docker的配置
[root@k8s-master-1 ~]# cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "insecure-registries": ["hb.oiox.cn"]
}
EOF

# 重新启动docker
[root@k8s-master-1 ~]# systemctl restart docker && systemctl status docker -l
```

### 测试使用
```
# 登陆 
[root@k8s-master-1 ~]# docker login hb.oiox.cn                                                                                        
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@k8s-master-1 ~]# 

# 测试使用
[root@k8s-master-1 ~]# docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/dashboard:v2.7.0
[root@k8s-master-1 ~]# docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/dashboard:v2.7.0 
[root@k8s-master-1 ~]# docker push hb.oiox.cn/library/dashboard:v2.7.0
[root@k8s-master-1 ~]# docker pull hb.oiox.cn/library/dashboard:v2.7.0
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
> **文章主要发布于微信**