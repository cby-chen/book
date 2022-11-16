---
layout: post
cid: 29
title: 华为 A800-9000 服务器 离线安装MindX DL 可视化环境+监控
slug: 29
date: 2021/12/30 17:04:00
updated: 2022/03/25 15:50:11
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


    MindX DL Sample主要应用于企业的数据中心或超算中心机房中，针对不同的应用场景为客户提供AI深度学习端到端解决方案。

  

    传统行业：用户无自建深度学习平台，希望能够提供简单易用、软硬件一体化的深度学习平台。

    互联网和安防行业：用户有自建深度学习平台，希望提供适配客户深度学习平台的开源插件，快速上线昇腾系列AI处理器的深度学习。

    超算中心和公有云行业：用户无AI深度学习集群，希望提供大规模AI深度学习集群、支持超高密部署、整柜交付，缩短项目交付周期，加速业务上线，节省安装部署及调测成本。

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b9e7276bdb94153a970476da4185a75~tplv-k3u1fbpfcp-zoom-1.image)

  

  

    说明：此文档需要先将基础kubernetes环境下的DL搭建完成，参考《华为 A800-9000 服务器 离线安装MindX DL》  

  

一、 修改ansible配置文件  

  

```
root@ubuntu:/etc/ansible# vim hosts
root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# cat /etc/ansible/hosts
[all:vars]
# NFS service IP
nfs_service_ip=192.168.1.99

# Master IP
master_ip=192.168.1.99

[workers]
localnode ansible_host=192.168.1.99 ansible_ssh_user=root ansible_ssh_pass=123123

root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# vi /etc/ansible/ansible.cfg

# 取消以下两行内容的注释并更改deprecation_warnings为“False”。

host_key_checking = False
deprecation_warnings = False
```

  

二、下载基础镜像  

  

```
root@ubuntu:/etc/ansible# docker pull redis:5.0.8
5.0.8: Pulling from library/redis
3d48095d71a3: Pull complete 
773882920678: Pull complete 
b04905edf724: Pull complete 
90e236b4682b: Pull complete 
fb7d8181d1c6: Pull complete 
532c81fe8c61: Pull complete 
Digest: sha256:96bdb5e2984b15e3cf4de74077f650c911cb26ec0981e0772df35a1a5cb19798
Status: Downloaded newer image for redis:5.0.8
root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# docker pull prom/prometheus:v2.10.0
v2.10.0: Pulling from prom/prometheus
596fa44d463e: Pull complete 
ae7a0e9c5457: Pull complete 
3e3e880277a4: Pull complete 
d884b32e16d7: Pull complete 
6f45dfbc8251: Pull complete 
e7275b596775: Pull complete 
d3c1f1d7d1d1: Pull complete 
f040a278aa08: Pull complete 
403fefd2b7ea: Pull complete 
Digest: sha256:b89e9c7ffbfbc8efebd6d8ff89b33175625bb2c7ae2751fbcd89f0884cfbdcab
Status: Downloaded newer image for prom/prometheus:v2.10.0
root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# docker pull mysql/mysql-server:8.0.13
8.0.13: Pulling from mysql/mysql-server
5530262403b2: Pull complete 
01c05f6b9ab3: Pull complete 
f521094e248f: Pull complete 
495eb6103d23: Pull complete 
Digest: sha256:59a5854dca16488305aee60c8dea4d88b68d816aee627de022b19d9bead48d04
Status: Downloaded newer image for mysql/mysql-server:8.0.13
root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# docker pull grafana/grafana:7.0.2
7.0.2: Pulling from grafana/grafana
29e5d40040c1: Pull complete 
c33923c8c811: Pull complete 
3fd85f7a4ab6: Pull complete 
987cf1afe976: Pull complete 
a27d86f46de8: Pull complete 
285316502f38: Pull complete 
Digest: sha256:5b9c9e18a8279a818144d90431ac0631bc17f520aa5c2fd6dd70bf767b48e632
Status: Downloaded newer image for grafana/grafana:7.0.2
root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# docker pull python:3.7.5
3.7.5: Pulling from library/python
af4800279257: Pull complete 
8fae2ec46cd5: Pull complete 
8a8718b9412e: Pull complete 
4908f8b44725: Pull complete 
54e0fac9e6c6: Pull complete 
2b1da11f97bb: Pull complete 
d93a637093d0: Pull complete 
c79746565cc4: Pull complete 
3dfacccebd97: Pull complete 
Digest: sha256:88d11783cbbfa06f1c12ca50c73c340b0bff34bf599c6e1dd27fb836a8de506d
Status: Downloaded newer image for python:3.7.5
root@ubuntu:/etc/ansible# 
root@ubuntu:/etc/ansible# docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
fda1cca7a3cc: Pull complete 
Digest: sha256:7bd7a9ca99f868bf69c4b6212f64f2af8e243f97ba13abb3e641e03a7ceb59e8
Status: Downloaded newer image for ubuntu:18.04
root@ubuntu:/etc/ansible#
```

  

三、配置NGINX镜像配置  

  

```
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/nginx# vim Dockerfile
FROM ubuntu:18.04


#(可选Part1)如用户环境需要配置代理连接网络，此处需要按如下示例配置代理，可直接取消注释并填写相应字段（大括号不保留）。
# ENV http_proxy http://{username}:{password}@{IP}:{port}
# ENV https_proxy http://{username}:{password}@{IP}:{port}


#(可选Part2)如用户环境需要配置代理连接网络，此处需要按如下示例配置代理，可直接取消注释并填写相应字段（大括号不保留）。
# RUN echo 'Acquire::http::Proxy "http://{username}:{password}@{IP}:{port}"; Acquire::https::Proxy "http://{username}:{password}@{IP}:{port}";' > /etc/apt/apt.conf.d/80proxy

##(可选Part3)配置apt源，ARM环境示例，请根据环境架构在Part3和Part4中二选一
#RUN echo 'deb http://mirrors.aliyun.com/ubuntu-ports/ bionic main restricted universe multiverse \n\
#deb http://mirrors.aliyun.com/ubuntu-ports/ bionic-security main restricted universe multiverse \n\
#deb http://mirrors.aliyun.com/ubuntu-ports/ bionic-updates main restricted universe multiverse \n\
#deb http://mirrors.aliyun.com/ubuntu-ports/ bionic-proposed main restricted universe multiverse \n\
#deb http://mirrors.aliyun.com/ubuntu-ports/ bionic-backports main restricted universe multiverse \n\
#deb-src http://mirrors.aliyun.com/ubuntu-ports/ bionic main restricted universe multiverse \n\
#deb-src http://mirrors.aliyun.com/ubuntu-ports/ bionic-security main restricted universe multiverse \n\
#deb-src http://mirrors.aliyun.com/ubuntu-ports/ bionic-updates main restricted universe multiverse \n\
#deb-src http://mirrors.aliyun.com/ubuntu-ports/ bionic-proposed main restricted universe multiverse \n\
#deb-src http://mirrors.aliyun.com/ubuntu-ports/ bionic-backports main restricted universe multiverse'  > /etc/apt/sources.list

##(可选Part4)配置apt源，x86环境示例，请根据环境架构在Part3和Part4中二选一
#RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse \n\
#deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse \n\
#deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse \n\
#deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse \n\
#deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse \n\
#deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse \n\
#deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse \n\
#deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse \n\
#deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse \n\
#deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse'  > /etc/apt/sources.list

RUN echo 'deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic main restricted \n\
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted \n\
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic universe \n\
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-updates universe \n\
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic multiverse \n\
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-updates multiverse \n\
deb http://cn.ports.ubuntu.com/ubuntu-ports/ bionic-backports main restricted universe multiverse \n\
deb http://ports.ubuntu.com/ubuntu-ports bionic-security main restricted \n\ 
deb http://ports.ubuntu.com/ubuntu-ports bionic-security universe \n\
deb http://ports.ubuntu.com/ubuntu-ports bionic-security multiverse' > /etc/apt/sources.list



RUN apt-get update && \
    apt-get install -y build-essential && \
    apt-get install -y libtool && \
    apt-get install -y libpcre3 libpcre3-dev && \
    apt-get install -y zlib1g-dev && \
    apt-get install -y openssl && \
    apt-get install libssl-dev && \
    apt-get install -y wget && \
    apt-get install -y git

RUN export GIT_SSL_NO_VERIFY=1 && \
    git clone https://github.com/masterzen/nginx-upload-progress-module.git && \
    git clone https://github.com/fdintino/nginx-upload-module.git && \
    wget --no-check-certificate http://nginx.org/download/nginx-1.17.3.tar.gz && \
    tar -zxvf nginx-1.17.3.tar.gz && \
    cd nginx-1.17.3 && \
    ./configure --add-module=../nginx-upload-module/ --add-module=../nginx-upload-progress-module/ --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-g -O2 -fdebug-prefix-map=/tmp/tmp.UI0oSlj34i/nginx-1.17.10=. -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie' && \
    make && make install && ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx && \
    mkdir -pv /var/cache/nginx/client_temp  && \
    cd ..

#RUN rm /etc/apt/apt.conf.d/80proxy /etc/apt/sources.list && \
#    rm nginx-1.17.3.tar.gz && \
#    rm -rf nginx-1.17.3 nginx-upload-module nginx-upload-progress-module

#(可选Part5)如在Part1中配置了代理，需要取消下面两行注释取消对应代理配置。
#ENV http_proxy ""
#ENV https_proxy ""

#(可选Part6)如在Part2中配置了代理，需要取消下面一行注释取消对应代理配置。
#RUN echo "" > /etc/apt/apt.conf.d/80proxy

root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/nginx# docker build -t nginx-upload:v1 .

root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/nginx# docker images | grep ng
nginx-upload                          v1                  15bbdc3677d7        4 minutes ago       372MB

```

  

四、安装前端所需工具，并编译前端代码  

  

```
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/webgui# curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/webgui# nvm install node
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/webgui# apt install -y npm
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/webgui# apt install -y node
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/webgui# wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.6/install.sh | bash
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/webgui# nvm install node
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/webgui# npm config set registry http://r.cnpmjs.org/;
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/webgui# npm install --global vue-cli;
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/src/webgui# npm install webpack -g;
```

  

五、修改TJM配置文件

  

```
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/yamls# vim tjm.yaml
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/yamls# 
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/yamls# 
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/yamls# 
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/yamls# cat tjm.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tjm
  namespace: default
  labels:
    app: tjm
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tjm
  template:
    metadata:
      labels:
        app: tjm
    spec:
      nodeSelector:
        masterselector: dls-master-node
      containers:
        - image: tjm:v0.0.1
          name: tjm-core
          imagePullPolicy: IfNotPresent
          command:
            - "/bin/bash"
            - "-c"
            - "/bin/bash /tjm/start.sh"
          env:
            # The following env variables set up basic auth twith the default admin user and admin password.
            - name: NFS_IP
              value: 192.168.1.99
            - name: NFS_ROOT_DIR
              value: "/data/atlas_dls"
            - name: INCLUSTER_FLAG
              value: "TRUE"
            - name: SERVICE_DOMAIN_NAME
              value: 192.168.1.99
            - name: CONTAINER_ROOT_DIR
              value: "/datanfs"
            - name: TENSORBOARD_IMAGE
              value: "tensorboard:latest"
            - name: TENSORBOARD_CPU
              value: "190"
            - name: TENSORBOARD_MEMORY
              value: "200Mi"
            - name: MINDINSIGHT_IMAGE
              value: "mindinsight:latest"
            - name: MINDINSIGHT_CPU
              value: "190"
            - name: MINDINSIGHT_MEMORY
              value: "2048Mi"
            - name: TIMEZONE
              value: "8"
          volumeMounts:
            - name: dls-data
              mountPath: /datanfs
            - name: localtime
              mountPath: /etc/localtime
            - name: dls-log
              mountPath: /var/log/atlas_dls/tjm
          resources:
            requests:
              cpu: "500m"
              memory: "100Mi"
            limits:
              cpu: "5000m"
              memory: "50000Mi"
      volumes:
        - name: dls-data
          nfs:
            server: 192.168.1.99
            path: /data/atlas_dls
        - name: localtime
          hostPath:
            path: /etc/localtime
        - name: dls-log
          hostPath:
            path: /var/log/atlas_dls/tjm


---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: tjm
  name: tjm
  namespace: default
spec:
  ports:
    - port: 5003
      targetPort: 5003
  selector:
    app: tjm
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/yamls# 



tjm.yaml

修改env字段中以下参数配置：

- name: TENSORBOARD_IMAGE
  value: "tensorboard:latest"  #Tensorboard镜像名和版本。
- name: TENSORBOARD_CPU
  value: "1"  #Tensorboard任务运行所需CPU，建议≥1。
- name: TENSORBOARD_MEMORY
  value: "200Mi"  #Tensorboard任务运行所需内存，建议≥100Mi。
- name: MINDINSIGHT_IMAGE
  value: "mindinsight:latest"  #Mindinsight镜像名和版本。
- name: MINDINSIGHT_CPU
  value: "4"  #Mindinsight任务运行所需CPU，建议≥4。
- name: MINDINSIGHT_MEMORY
  value: "2048Mi"  #Mindinsight任务运行所需内存，建议≥2048Mi。
- name: TIMEZONE
  value: "8"   #默认为8（东八区），根据实际配置时区。

```

  

六、修改MMS配置文件  

  

```
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/yamls# vim mms.yaml
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/yamls# cat mms.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dls-mms-deploy
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dls-mms
  template:
    metadata:
      labels:
        app: dls-mms
    spec:
      nodeSelector:
        masterselector: dls-master-node
      containers:
        - name: dls-mms
          image: mms:v0.0.1
          imagePullPolicy: IfNotPresent
          command: [ "/bin/bash", "-c", "--" ]
          args: [ "chmod 550 /data/mms/start.sh;/data/mms/start.sh" ]
          ports:
            - containerPort: 5000
          env:
            - name: NFS_IP
              value: 192.168.1.99
            - name: NFS_ROOT_DIR
              value: "/data/atlas_dls"
            - name: INCLUSTER_FLAG
              value: "TRUE"
            - name: CONTAINER_ROOT_DIR
              value: "/datanfs"
            - name: MODEL_CONVERT_IMAGE
              value: "tf-c73:b033-with-atc"
          volumeMounts:
            - name: dls-data
              mountPath: /datanfs
            - name: dls-log
              mountPath: /var/log/atlas_dls/mms
            - name: localtime
              mountPath: /etc/localtime
          resources:
            requests:
              cpu: "500m"
              memory: "100Mi"
            limits:
              cpu: "5000m"
              memory: "50000Mi"

      volumes:
        - name: dls-data
          nfs:
            server: 192.168.1.99
            path: "/data/atlas_dls"
        - name: dls-log
          hostPath:
            path: /var/log/atlas_dls/mms
        - name: localtime
          hostPath:
            path: /etc/localtime

---
apiVersion: v1
kind: Service
metadata:
  name: dls-mms
  namespace: default
  labels:
    app: atlas-dls
spec:
  ports:
    - name: dls-mms
      port: 5002
      protocol: TCP
      targetPort: 5002
  selector:
    app:  dls-mms
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/yamls# 




mms.yaml

修改env字段中以下参数配置：

- name: MODEL_CONVERT_IMAGE
  value: "tf-c73:b033-with-atc"  #模型转换镜像名
```

  

  

七、自动化安装，SHELL回显略

  

```
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/playbooks# ansible-playbook deploy.yaml
```

  

八、拉去训练镜像  

  

https://ascendhub.huawei.com/#/index

  

```
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/playbooks# docker login -u 15648907522 -p tDYqC45NFQ2VVb1Wk48C0AK4PC02SBjBpoUDOjo7oqIpdjBSaPzZyR112lzbzBWve ascendhub.huawei.com
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/playbooks# docker pull ascendhub.huawei.com/public-ascendhub/ascend-mindspore-arm:21.0.1.spc001
21.0.1.spc001: Pulling from public-ascendhub/ascend-mindspore-arm
fda1cca7a3cc: Already exists 
11c90fde7ae4: Pull complete 
7253b5e27781: Pull complete 
bbd60ae31b0e: Pull complete 
5ff0775b62ee: Pull complete 
4bd7bd3ffee1: Pull complete 
8f7e244558aa: Pull complete 
2782e4575e5b: Pull complete 
6082e54e59ee: Pull complete 
bd1b2e5f115e: Pull complete 
53142cd42310: Pull complete 
d746749ab006: Pull complete 
691a8a68558b: Pull complete 
ecbb81572dc3: Pull complete 
65d85230814c: Pull complete 
Digest: sha256:29069b29542554d5ac8f79c2be3ba78ace77751546bfad24b480acf782e39fa7
Status: Downloaded newer image for ascendhub.huawei.com/public-ascendhub/ascend-mindspore-arm:21.0.1.spc001
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/playbooks# docker pull ascendhub.huawei.com/public-ascendhub/ascend-tensorflow-arm:21.0.1
21.0.1: Pulling from public-ascendhub/ascend-tensorflow-arm
04da93b342eb: Pull complete 
b235194751de: Pull complete 
606a67bb8db9: Pull complete 
4af9b0a6671e: Pull complete 
aa86c517c858: Pull complete 
7d232de65c7f: Pull complete 
396a8ab7f029: Pull complete 
bc0d0369a1f9: Pull complete 
86f265d63edf: Pull complete 
0cef5083e917: Pull complete 
360419151a16: Pull complete 
06436a96fc81: Pull complete 
13611bc87943: Pull complete 
2127513261ef: Pull complete 
Digest: sha256:0e6fac7ec1cb09bb57bbd076d6b6054f44f91afe6dfadfca2270a51ba2fc53e0
Status: Downloaded newer image for ascendhub.huawei.com/public-ascendhub/ascend-tensorflow-arm:21.0.1
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/playbooks# docker pull ascendhub.huawei.com/public-ascendhub/ascend-pytorch-arm:21.0.1
21.0.1: Pulling from public-ascendhub/ascend-pytorch-arm
04da93b342eb: Already exists 
b235194751de: Already exists 
606a67bb8db9: Already exists 
e93c757bff45: Pull complete 
3cf04b2262e0: Pull complete 
b747b6b075df: Pull complete 
9d1ca426ec5d: Pull complete 
6709e46dffeb: Pull complete 
c9a5fa495f7c: Pull complete 
88232903df71: Pull complete 
ef57b6fb083b: Pull complete 
ea40e63aedc6: Pull complete 
fb00b3f33bf9: Pull complete 
7fa0e069227a: Pull complete 
c875eb36de84: Pull complete 
Digest: sha256:57f724d4b938753102d09d0ffbab5b0c0697cf3e37b1e8ac948bb8a8df356958
Status: Downloaded newer image for ascendhub.huawei.com/public-ascendhub/ascend-pytorch-arm:21.0.1
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/playbooks# 
root@ubuntu:~/123/mindxdl-sample-20210715-V2.0.2/deploy/playbooks#
```

  

  

九、构建jupyter-notebook镜像  

  

```
root@ubuntu:~/123# mkdir jupyter-notebook
root@ubuntu:~/123# 
root@ubuntu:~/123# cd jupyter-notebook/
root@ubuntu:~/123/jupyter-notebook# ls
root@ubuntu:~/123/jupyter-notebook# 
root@ubuntu:~/123/jupyter-notebook# 
root@ubuntu:~/123/jupyter-notebook# vi Dockerfile
root@ubuntu:~/123/jupyter-notebook# 
root@ubuntu:~/123/jupyter-notebook# 
root@ubuntu:~/123/jupyter-notebook# docker build -t notebook .^C
root@ubuntu:~/123/jupyter-notebook# 
root@ubuntu:~/123/jupyter-notebook# 
root@ubuntu:~/123/jupyter-notebook# 
root@ubuntu:~/123/jupyter-notebook# cat Dockerfile 
From python:3.7.5

# -------------------------(Optional) Configure the proxy and source.-----------------------
#ENV http_proxy http://<user>:<password>@ip:port
#ENV https_proxy http://<user>:<password>@ip:port
#ENV ftp_proxy ftp://<user>:<password>@ip:port

RUN mkdir -p ~/.pip \
&& echo '[global] \n\ 
index-url=https://pypi.doubanio.com/simple/\n\ 
trusted-host=pypi.doubanio.com' >> ~/.pip/pip.conf 

RUN pip install jupyter \
&& jupyter notebook --generate-config \ 
&& echo "c.NotebookApp.ip='0.0.0.0' \n\
c.NotebookApp.open_browser = False \n\
c.NotebookApp.token = '' \n\
c.NotebookApp.port =8888" >> ~/.jupyter/jupyter_notebook_config.py

RUN apt-get update \
&& apt-get install -y openssh-server

RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config \
&& sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config 

ENV http_proxy ''
ENV https_proxy ''
ENV ftp_proxy ''


RUN useradd -d /home/hwMindX -u 9000 -m -s /bin/bash hwMindX && \
    useradd -d /home/HwHiAiUser -u 1000 -m -s /bin/bash HwHiAiUser && \
    usermod -a -G HwHiAiUser hwMindX

RUN echo root:123123 | chpasswd 

USER hwMindX


ENTRYPOINT jupyter notebook --allow-root

root@ubuntu:~/123/jupyter-notebook# docker build -t notebook .
Sending build context to Docker daemon  3.072kB
Step 1/12 : From python:3.7.5
 ---> a4356c370cda
Step 2/12 : RUN mkdir -p ~/.pip && echo '[global] \nindex-url=https://pypi.doubanio.com/simple/\ntrusted-host=pypi.doubanio.com' >> ~/.pip/pip.conf
 ---> Running in 841fef0bfec2
Removing intermediate container 841fef0bfec2
 ---> 2e9619b95f3c
Step 3/12 : RUN pip install jupyter && jupyter notebook --generate-config && echo "c.NotebookApp.ip='0.0.0.0' \nc.NotebookApp.open_browser = False \nc.NotebookApp.token = '' \nc.NotebookApp.port =8888" >> ~/.jupyter/jupyter_notebook_config.py
 ---> Running in e8a384542bf2
Looking in indexes: https://pypi.doubanio.com/simple/
Collecting jupyter
  Downloading https://pypi.doubanio.com/packages/83/df/0f5dd132200728a86190397e1ea87cd76244e42d39ec5e88efd25b2abd7e/jupyter-1.0.0-py2.py3-none-any.whl
Collecting nbconvert
  Downloading https://pypi.doubanio.com/packages/fd/12/7b225ea00a5fe32df30b2c303dcc8c21c8db533ea7c0e38b4ac5a41bd8f0/nbconvert-6.1.0-py3-none-any.whl (551kB)
Collecting qtconsole
  Downloading https://pypi.doubanio.com/packages/3a/57/c8fc1fc6fb6bc03caca20ace9cd0ac0e16cc052b51cbe3acbeeb53abcb18/qtconsole-5.1.1-py3-none-any.whl (119kB)
Collecting ipykernel
  Downloading https://pypi.doubanio.com/packages/d4/9a/59010716573b2aae10ccf88ea275c9a50943a7f8d4a123ad3c6f385a6c94/ipykernel-6.2.0-py3-none-any.whl (122kB)
Collecting ipywidgets
  Downloading https://pypi.doubanio.com/packages/11/53/084940a83a8158364e630a664a30b03068c25ab75243224d6b488800d43a/ipywidgets-7.6.3-py2.py3-none-any.whl (121kB)
Collecting notebook
  Downloading https://pypi.doubanio.com/packages/3c/0e/9883ebfa204c7328fab473e04bda8066b00960fadc6698972afa62ddf0ce/notebook-6.4.3-py3-none-any.whl (9.9MB)
Collecting jupyter-console
  Downloading https://pypi.doubanio.com/packages/59/cd/aa2670ffc99eb3e5bbe2294c71e4bf46a9804af4f378d09d7a8950996c9b/jupyter_console-6.4.0-py3-none-any.whl
Collecting nbformat>=4.4
  Downloading https://pypi.doubanio.com/packages/e7/c7/dd50978c637a7af8234909277c4e7ec1b71310c13fb3135f3c8f5b6e045f/nbformat-5.1.3-py3-none-any.whl (178kB)
Collecting testpath
  Downloading https://pypi.doubanio.com/packages/ac/87/5422f6d056bfbded920ccf380a65de3713a3b95a95ba2255be2a3fb4f464/testpath-0.5.0-py3-none-any.whl (84kB)
Collecting traitlets>=5.0
  Downloading https://pypi.doubanio.com/packages/f6/7d/3ecb0ebd0ce8dcdfa7bd47ab85c1d4a521e6770ef283d0824f5804994dfe/traitlets-5.0.5-py3-none-any.whl (100kB)
Collecting mistune<2,>=0.8.1
  Downloading https://pypi.doubanio.com/packages/09/ec/4b43dae793655b7d8a25f76119624350b4d65eb663459eb9603d7f1f0345/mistune-0.8.4-py2.py3-none-any.whl
Collecting defusedxml
  Downloading https://pypi.doubanio.com/packages/07/6c/aa3f2f849e01cb6a001cd8554a88d4c77c5c1a31c95bdf1cf9301e6d9ef4/defusedxml-0.7.1-py2.py3-none-any.whl
Collecting bleach
  Downloading https://pypi.doubanio.com/packages/b6/23/d06c0bddcef0df58dd2c9ac02f8639533a6671bed0ef3e236888bb3b0a3c/bleach-4.0.0-py2.py3-none-any.whl (146kB)
Collecting pandocfilters>=1.4.1
  Downloading https://pypi.doubanio.com/packages/28/78/bd59a9adb72fa139b1c9c186e6f65aebee52375a747e4b6a6dcf0880956f/pandocfilters-1.4.3.tar.gz
Collecting entrypoints>=0.2.2
  Downloading https://pypi.doubanio.com/packages/ac/c6/44694103f8c221443ee6b0041f69e2740d89a25641e62fb4f2ee568f2f9c/entrypoints-0.3-py2.py3-none-any.whl
Collecting jupyter-core
  Downloading https://pypi.doubanio.com/packages/53/40/5af36bffa0af3ac71d3a6fc6709de10e4f6ff7c01745b8bc4715372189c9/jupyter_core-4.7.1-py3-none-any.whl (82kB)
Collecting pygments>=2.4.1
  Downloading https://pypi.doubanio.com/packages/78/c8/8d9be2f72d8f465461f22b5f199c04f7ada933add4dae6e2468133c17471/Pygments-2.10.0-py3-none-any.whl (1.0MB)
Collecting jupyterlab-pygments
  Downloading https://pypi.doubanio.com/packages/a8/6f/c34288766797193b512c6508f5994b830fb06134fdc4ca8214daba0aa443/jupyterlab_pygments-0.1.2-py2.py3-none-any.whl
Collecting nbclient<0.6.0,>=0.5.0
  Downloading https://pypi.doubanio.com/packages/a7/ed/b764fa931614cb7ed9bebbc42532daecef405d6bef660eeda882f6c23b98/nbclient-0.5.4-py3-none-any.whl (66kB)
Collecting jinja2>=2.4
  Downloading https://pypi.doubanio.com/packages/80/21/ae597efc7ed8caaa43fb35062288baaf99a7d43ff0cf66452ddf47604ee6/Jinja2-3.0.1-py3-none-any.whl (133kB)
Collecting ipython-genutils
  Downloading https://pypi.doubanio.com/packages/fa/bc/9bd3b5c2b4774d5f33b2d544f1460be9df7df2fe42f352135381c347c69a/ipython_genutils-0.2.0-py2.py3-none-any.whl
Collecting pyzmq>=17.1
  Downloading https://pypi.doubanio.com/packages/61/48/308d03af40bf44c86ef826d942b12bdab5fbae6282e858bdff8bd55b0818/pyzmq-22.2.1-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (1.8MB)
Collecting jupyter-client>=4.1
  Downloading https://pypi.doubanio.com/packages/b0/21/2104133f07e34f58712c87f0feaacdd38b5baff1ddb6ab72bb4baf16fc4a/jupyter_client-7.0.1-py3-none-any.whl (122kB)
Collecting qtpy
  Downloading https://pypi.doubanio.com/packages/21/0d/1cc56aa1df049d9f989520ee8214a6ccfd236095d56060967afdb0b8f0d8/QtPy-1.10.0-py2.py3-none-any.whl (54kB)
Collecting tornado<7.0,>=4.2
  Downloading https://pypi.doubanio.com/packages/27/27/95912ec1ecbd5f3cc1ce76a8d62cb63d62ebee575acf02116814d42ea5eb/tornado-6.1-cp37-cp37m-manylinux2014_aarch64.whl (428kB)
Collecting importlib-metadata<5; python_version < "3.8.0"
  Downloading https://pypi.doubanio.com/packages/c0/72/4512a88e402d4dc3bab49a845130d95ac48936ef3a9469b55cc79a60d84d/importlib_metadata-4.6.4-py3-none-any.whl
Collecting ipython<8.0,>=7.23.1
  Downloading https://pypi.doubanio.com/packages/25/a0/e0b850415984ac29f14775b075efc54d73b38f0d50c6ebdea7820ffb1c12/ipython-7.26.0-py3-none-any.whl (786kB)
Collecting argcomplete>=1.12.3; python_version < "3.8.0"
  Downloading https://pypi.doubanio.com/packages/b7/9e/9dc74d330c07866d72f62d553fe8bdbe32786ff247a14e68b5659963e6bd/argcomplete-1.12.3-py2.py3-none-any.whl
Collecting debugpy<2.0,>=1.0.0
  Downloading https://pypi.doubanio.com/packages/c5/2d/876b1140b1544fe2187235ae9f52cdcd1e77d2bad641ea2aef413e882751/debugpy-1.4.1-py2.py3-none-any.whl (4.2MB)
Collecting matplotlib-inline<0.2.0,>=0.1.0
  Downloading https://pypi.doubanio.com/packages/7f/de/6c111d687335729cf8c156394c8d119b0dc3c34b6966ff2a2f7fe4aa79cf/matplotlib_inline-0.1.2-py3-none-any.whl
Collecting jupyterlab-widgets>=1.0.0; python_version >= "3.6"
  Downloading https://pypi.doubanio.com/packages/18/b5/3473d275e3b2359efdf5768e9df95537308b93a31ad94fa92814ac565826/jupyterlab_widgets-1.0.0-py3-none-any.whl (243kB)
Collecting widgetsnbextension~=3.5.0
  Downloading https://pypi.doubanio.com/packages/6c/7b/7ac231c20d2d33c445eaacf8a433f4e22c60677eb9776c7c5262d7ddee2d/widgetsnbextension-3.5.1-py2.py3-none-any.whl (2.2MB)
Collecting argon2-cffi
  Downloading https://pypi.doubanio.com/packages/74/fd/d78e003a79c453e8454197092fce9d1c6099445b7e7da0b04eb4fe1dbab7/argon2-cffi-20.1.0.tar.gz (1.8MB)
  Installing build dependencies: started
  Installing build dependencies: finished with status 'done'
  Getting requirements to build wheel: started
  Getting requirements to build wheel: finished with status 'done'
    Preparing wheel metadata: started
    Preparing wheel metadata: finished with status 'done'
Collecting terminado>=0.8.3
  Downloading https://pypi.doubanio.com/packages/5b/a8/0c428a9a2432b611566b0309d1a50a051f4ec965ce274528c4ba6b6c0205/terminado-0.11.1-py3-none-any.whl
Collecting Send2Trash>=1.5.0
  Downloading https://pypi.doubanio.com/packages/47/26/3435896d757335ea53dce5abf8d658ca80757a7a06258451b358f10232be/Send2Trash-1.8.0-py3-none-any.whl
Collecting prometheus-client
  Downloading https://pypi.doubanio.com/packages/09/da/4e8471ff825769581593b5b84769d32f58e5373b59fccaf355d3529ad530/prometheus_client-0.11.0-py2.py3-none-any.whl (56kB)
Collecting prompt-toolkit!=3.0.0,!=3.0.1,<3.1.0,>=2.0.0
  Downloading https://pypi.doubanio.com/packages/c6/37/ec72228971dbaf191243b8ee383c6a3834b5cde23daab066dfbfbbd5438b/prompt_toolkit-3.0.20-py3-none-any.whl (370kB)
Collecting jsonschema!=2.5.0,>=2.4
  Downloading https://pypi.doubanio.com/packages/c5/8f/51e89ce52a085483359217bc72cdbf6e75ee595d5b1d4b5ade40c7e018b8/jsonschema-3.2.0-py2.py3-none-any.whl (56kB)
Collecting webencodings
  Downloading https://pypi.doubanio.com/packages/f4/24/2a3e3df732393fed8b3ebf2ec078f05546de641fe1b667ee316ec1dcf3b7/webencodings-0.5.1-py2.py3-none-any.whl
Collecting packaging
  Downloading https://pypi.doubanio.com/packages/3c/77/e2362b676dc5008d81be423070dd9577fa03be5da2ba1105811900fda546/packaging-21.0-py3-none-any.whl (40kB)
Collecting six>=1.9.0
  Downloading https://pypi.doubanio.com/packages/d9/5a/e7c31adbe875f2abbb91bd84cf2dc52d792b5a01506781dbcf25c91daf11/six-1.16.0-py2.py3-none-any.whl
Collecting nest-asyncio
  Downloading https://pypi.doubanio.com/packages/52/e2/9b37da54e6e9094d2f558ae643d1954a0fa8215dfee4fa261f31c5439796/nest_asyncio-1.5.1-py3-none-any.whl
Collecting MarkupSafe>=2.0
  Downloading https://pypi.doubanio.com/packages/a3/01/8d5fd91ccc1a61b7a9e2803819b8b60c3bac37290bbbd3df33d8d548f9c1/MarkupSafe-2.0.1-cp37-cp37m-manylinux2014_aarch64.whl
Collecting python-dateutil>=2.1
  Downloading https://pypi.doubanio.com/packages/36/7a/87837f39d0296e723bb9b62bbb257d0355c7f6128853c78955f57342a56d/python_dateutil-2.8.2-py2.py3-none-any.whl (247kB)
Collecting zipp>=0.5
  Downloading https://pypi.doubanio.com/packages/92/d9/89f433969fb8dc5b9cbdd4b4deb587720ec1aeb59a020cf15002b9593eef/zipp-3.5.0-py3-none-any.whl
Collecting typing-extensions>=3.6.4; python_version < "3.8"
  Downloading https://pypi.doubanio.com/packages/2e/35/6c4fff5ab443b57116cb1aad46421fb719bed2825664e8fe77d66d99bcbc/typing_extensions-3.10.0.0-py3-none-any.whl
Collecting backcall
  Downloading https://pypi.doubanio.com/packages/4c/1c/ff6546b6c12603d8dd1070aa3c3d273ad4c07f5771689a7b69a550e8c951/backcall-0.2.0-py2.py3-none-any.whl
Collecting pexpect>4.3; sys_platform != "win32"
  Downloading https://pypi.doubanio.com/packages/39/7b/88dbb785881c28a102619d46423cb853b46dbccc70d3ac362d99773a78ce/pexpect-4.8.0-py2.py3-none-any.whl (59kB)
Collecting jedi>=0.16
  Downloading https://pypi.doubanio.com/packages/f9/36/7aa67ae2663025b49e8426ead0bad983fee1b73f472536e9790655da0277/jedi-0.18.0-py2.py3-none-any.whl (1.4MB)
Requirement already satisfied: setuptools>=18.5 in /usr/local/lib/python3.7/site-packages (from ipython<8.0,>=7.23.1->ipykernel->jupyter) (41.6.0)
Collecting pickleshare
  Downloading https://pypi.doubanio.com/packages/9a/41/220f49aaea88bc6fa6cba8d05ecf24676326156c23b991e80b3f2fc24c77/pickleshare-0.7.5-py2.py3-none-any.whl
Collecting decorator
  Downloading https://pypi.doubanio.com/packages/6a/36/b1b9bfdf28690ae01d9ca0aa5b0d07cb4448ac65fb91dc7e2d094e3d992f/decorator-5.0.9-py3-none-any.whl
Collecting cffi>=1.0.0
  Downloading https://pypi.doubanio.com/packages/7c/d7/027b40eab051119083fa64be7f86c40fc96643c627fd5068462b88f72111/cffi-1.14.6-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (206kB)
Collecting ptyprocess; os_name != "nt"
  Downloading https://pypi.doubanio.com/packages/22/a6/858897256d0deac81a172289110f31629fc4cee19b6f01283303e18c8db3/ptyprocess-0.7.0-py2.py3-none-any.whl
Collecting wcwidth
  Downloading https://pypi.doubanio.com/packages/59/7c/e39aca596badaf1b78e8f547c807b04dae603a433d3e7a7e04d67f2ef3e5/wcwidth-0.2.5-py2.py3-none-any.whl
Collecting pyrsistent>=0.14.0
  Downloading https://pypi.doubanio.com/packages/f4/d7/0fa558c4fb00f15aabc6d42d365fcca7a15fcc1091cd0f5784a14f390b7f/pyrsistent-0.18.0.tar.gz (104kB)
  Installing build dependencies: started
  Installing build dependencies: finished with status 'done'
  Getting requirements to build wheel: started
  Getting requirements to build wheel: finished with status 'done'
    Preparing wheel metadata: started
    Preparing wheel metadata: finished with status 'done'
Collecting attrs>=17.4.0
  Downloading https://pypi.doubanio.com/packages/20/a9/ba6f1cd1a1517ff022b35acd6a7e4246371dfab08b8e42b829b6d07913cc/attrs-21.2.0-py2.py3-none-any.whl (53kB)
Collecting pyparsing>=2.0.2
  Downloading https://pypi.doubanio.com/packages/8a/bb/488841f56197b13700afd5658fc279a2025a39e22449b7cf29864669b15d/pyparsing-2.4.7-py2.py3-none-any.whl (67kB)
Collecting parso<0.9.0,>=0.8.0
  Downloading https://pypi.doubanio.com/packages/a9/c4/d5476373088c120ffed82f34c74b266ccae31a68d665b837354d4d8dc8be/parso-0.8.2-py2.py3-none-any.whl (94kB)
Collecting pycparser
  Downloading https://pypi.doubanio.com/packages/ae/e7/d9c3a176ca4b02024debf82342dab36efadfc5776f9c8db077e8f6e71821/pycparser-2.20-py2.py3-none-any.whl (112kB)
Building wheels for collected packages: argon2-cffi, pyrsistent
  Building wheel for argon2-cffi (PEP 517): started
  Building wheel for argon2-cffi (PEP 517): finished with status 'done'
  Created wheel for argon2-cffi: filename=argon2_cffi-20.1.0-cp37-abi3-linux_aarch64.whl size=94330 sha256=a7e904c263450da07cbc3a73891e049bab3035c37841778963599e544d3df91c
  Stored in directory: /root/.cache/pip/wheels/bd/00/bf/afb81816e42f4cd82890c552262451d2683fce154d72a2a7f2
  Building wheel for pyrsistent (PEP 517): started
  Building wheel for pyrsistent (PEP 517): finished with status 'done'
  Created wheel for pyrsistent: filename=pyrsistent-0.18.0-cp37-cp37m-linux_aarch64.whl size=124391 sha256=4aa68debe7bd18cef92085f61bcff6ca4a308e7a0c3baa6d5e9b79d011a0860b
  Stored in directory: /root/.cache/pip/wheels/41/82/79/02feba19913ccf53c135a504fd28677aeecf1980d76c1bfad5
Successfully built argon2-cffi pyrsistent
Building wheels for collected packages: pandocfilters
  Building wheel for pandocfilters (setup.py): started
  Building wheel for pandocfilters (setup.py): finished with status 'done'
  Created wheel for pandocfilters: filename=pandocfilters-1.4.3-cp37-none-any.whl size=7991 sha256=74eac0d992a7841f6066709dcd1dcabfa4fa58eb171649fbaef06b141bcd033c
  Stored in directory: /root/.cache/pip/wheels/f3/b7/38/9a1fac073c6eb0cc2524036dfcef319b829ce237c80c31c044
Successfully built pandocfilters
Installing collected packages: ipython-genutils, traitlets, jupyter-core, pyrsistent, six, attrs, zipp, typing-extensions, importlib-metadata, jsonschema, nbformat, testpath, mistune, defusedxml, webencodings, pyparsing, packaging, bleach, pandocfilters, entrypoints, pygments, jupyterlab-pygments, nest-asyncio, pyzmq, python-dateutil, tornado, jupyter-client, nbclient, MarkupSafe, jinja2, nbconvert, backcall, ptyprocess, pexpect, parso, jedi, matplotlib-inline, pickleshare, decorator, wcwidth, prompt-toolkit, ipython, argcomplete, debugpy, ipykernel, qtpy, qtconsole, jupyterlab-widgets, pycparser, cffi, argon2-cffi, terminado, Send2Trash, prometheus-client, notebook, widgetsnbextension, ipywidgets, jupyter-console, jupyter
Successfully installed MarkupSafe-2.0.1 Send2Trash-1.8.0 argcomplete-1.12.3 argon2-cffi-20.1.0 attrs-21.2.0 backcall-0.2.0 bleach-4.0.0 cffi-1.14.6 debugpy-1.4.1 decorator-5.0.9 defusedxml-0.7.1 entrypoints-0.3 importlib-metadata-4.6.4 ipykernel-6.2.0 ipython-7.26.0 ipython-genutils-0.2.0 ipywidgets-7.6.3 jedi-0.18.0 jinja2-3.0.1 jsonschema-3.2.0 jupyter-1.0.0 jupyter-client-7.0.1 jupyter-console-6.4.0 jupyter-core-4.7.1 jupyterlab-pygments-0.1.2 jupyterlab-widgets-1.0.0 matplotlib-inline-0.1.2 mistune-0.8.4 nbclient-0.5.4 nbconvert-6.1.0 nbformat-5.1.3 nest-asyncio-1.5.1 notebook-6.4.3 packaging-21.0 pandocfilters-1.4.3 parso-0.8.2 pexpect-4.8.0 pickleshare-0.7.5 prometheus-client-0.11.0 prompt-toolkit-3.0.20 ptyprocess-0.7.0 pycparser-2.20 pygments-2.10.0 pyparsing-2.4.7 pyrsistent-0.18.0 python-dateutil-2.8.2 pyzmq-22.2.1 qtconsole-5.1.1 qtpy-1.10.0 six-1.16.0 terminado-0.11.1 testpath-0.5.0 tornado-6.1 traitlets-5.0.5 typing-extensions-3.10.0.0 wcwidth-0.2.5 webencodings-0.5.1 widgetsnbextension-3.5.1 zipp-3.5.0
WARNING: You are using pip version 19.3.1; however, version 21.2.4 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Writing default config to: /root/.jupyter/jupyter_notebook_config.py
Removing intermediate container e8a384542bf2
 ---> cc59b531125b
Step 4/12 : RUN apt-get update && apt-get install -y openssh-server
 ---> Running in 2e945ea6c6c8
Get:1 http://deb.debian.org/debian buster InRelease [122 kB]
Get:2 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]
Get:3 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]
Get:4 http://security.debian.org/debian-security buster/updates/main arm64 Packages [296 kB]
Get:5 http://deb.debian.org/debian buster/main arm64 Packages [7735 kB]
Get:6 http://deb.debian.org/debian buster-updates/main arm64 Packages [14.5 kB]
Fetched 8285 kB in 3s (2598 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  dbus dmsetup libapparmor1 libargon2-1 libcryptsetup12 libdbus-1-3
  libdevmapper1.02.1 libidn11 libip4tc0 libjson-c3 libkmod2 libnss-systemd
  libpam-systemd libsystemd0 libwrap0 libxmuu1 ncurses-term openssh-client
  openssh-sftp-server systemd systemd-sysv xauth
Suggested packages:
  default-dbus-session-bus | dbus-session-bus keychain libpam-ssh monkeysphere
  ssh-askpass molly-guard rssh ufw systemd-container policykit-1
The following NEW packages will be installed:
  dbus dmsetup libapparmor1 libargon2-1 libcryptsetup12 libdbus-1-3
  libdevmapper1.02.1 libidn11 libip4tc0 libjson-c3 libkmod2 libnss-systemd
  libpam-systemd libwrap0 libxmuu1 ncurses-term openssh-server
  openssh-sftp-server systemd systemd-sysv xauth
The following packages will be upgraded:
  libsystemd0 openssh-client
2 upgraded, 21 newly installed, 0 to remove and 130 not upgraded.
Need to get 7006 kB of archives.
After this operation, 23.5 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian buster/main arm64 libapparmor1 arm64 2.13.2-10 [93.8 kB]
Get:2 http://security.debian.org/debian-security buster/updates/main arm64 libsystemd0 arm64 241-7~deb10u8 [314 kB]
Get:3 http://deb.debian.org/debian buster/main arm64 libargon2-1 arm64 0~20171227-0.2 [18.9 kB]
Get:4 http://deb.debian.org/debian buster/main arm64 dmsetup arm64 2:1.02.155-3 [83.9 kB]
Get:5 http://deb.debian.org/debian buster/main arm64 libdevmapper1.02.1 arm64 2:1.02.155-3 [124 kB]
Get:6 http://deb.debian.org/debian buster/main arm64 libjson-c3 arm64 0.12.1+ds-2+deb10u1 [26.8 kB]
Get:7 http://deb.debian.org/debian buster/main arm64 libcryptsetup12 arm64 2:2.1.0-5+deb10u2 [181 kB]
Get:8 http://security.debian.org/debian-security buster/updates/main arm64 systemd arm64 241-7~deb10u8 [3256 kB]
Get:9 http://deb.debian.org/debian buster/main arm64 libidn11 arm64 1.33-2.2 [113 kB]
Get:10 http://deb.debian.org/debian buster/main arm64 libip4tc0 arm64 1.8.2-4 [69.6 kB]
Get:11 http://deb.debian.org/debian buster/main arm64 libkmod2 arm64 26-1 [49.4 kB]
Get:12 http://deb.debian.org/debian buster/main arm64 libdbus-1-3 arm64 1.12.20-0+deb10u1 [206 kB]
Get:13 http://deb.debian.org/debian buster/main arm64 dbus arm64 1.12.20-0+deb10u1 [227 kB]
Get:14 http://deb.debian.org/debian buster/main arm64 ncurses-term all 6.1+20181013-2+deb10u2 [490 kB]
Get:15 http://deb.debian.org/debian buster/main arm64 openssh-client arm64 1:7.9p1-10+deb10u2 [757 kB]
Get:16 http://deb.debian.org/debian buster/main arm64 libwrap0 arm64 7.6.q-28 [58.4 kB]
Get:17 http://security.debian.org/debian-security buster/updates/main arm64 systemd-sysv arm64 241-7~deb10u8 [100 kB]
Get:18 http://security.debian.org/debian-security buster/updates/main arm64 libpam-systemd arm64 241-7~deb10u8 [201 kB]
Get:19 http://security.debian.org/debian-security buster/updates/main arm64 libnss-systemd arm64 241-7~deb10u8 [197 kB]
Get:20 http://deb.debian.org/debian buster/main arm64 libxmuu1 arm64 2:1.1.2-2+b3 [24.1 kB]
Get:21 http://deb.debian.org/debian buster/main arm64 openssh-sftp-server arm64 1:7.9p1-10+deb10u2 [42.8 kB]
Get:22 http://deb.debian.org/debian buster/main arm64 openssh-server arm64 1:7.9p1-10+deb10u2 [334 kB]
Get:23 http://deb.debian.org/debian buster/main arm64 xauth arm64 1:1.0.10-1 [38.2 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 7006 kB in 1s (6820 kB/s)
(Reading database ... 24499 files and directories currently installed.)
Preparing to unpack .../libsystemd0_241-7~deb10u8_arm64.deb ...
Unpacking libsystemd0:arm64 (241-7~deb10u8) over (241-7~deb10u2) ...
Setting up libsystemd0:arm64 (241-7~deb10u8) ...
Selecting previously unselected package libapparmor1:arm64.
(Reading database ... 24499 files and directories currently installed.)
Preparing to unpack .../0-libapparmor1_2.13.2-10_arm64.deb ...
Unpacking libapparmor1:arm64 (2.13.2-10) ...
Selecting previously unselected package libargon2-1:arm64.
Preparing to unpack .../1-libargon2-1_0~20171227-0.2_arm64.deb ...
Unpacking libargon2-1:arm64 (0~20171227-0.2) ...
Selecting previously unselected package dmsetup.
Preparing to unpack .../2-dmsetup_2%3a1.02.155-3_arm64.deb ...
Unpacking dmsetup (2:1.02.155-3) ...
Selecting previously unselected package libdevmapper1.02.1:arm64.
Preparing to unpack .../3-libdevmapper1.02.1_2%3a1.02.155-3_arm64.deb ...
Unpacking libdevmapper1.02.1:arm64 (2:1.02.155-3) ...
Selecting previously unselected package libjson-c3:arm64.
Preparing to unpack .../4-libjson-c3_0.12.1+ds-2+deb10u1_arm64.deb ...
Unpacking libjson-c3:arm64 (0.12.1+ds-2+deb10u1) ...
Selecting previously unselected package libcryptsetup12:arm64.
Preparing to unpack .../5-libcryptsetup12_2%3a2.1.0-5+deb10u2_arm64.deb ...
Unpacking libcryptsetup12:arm64 (2:2.1.0-5+deb10u2) ...
Selecting previously unselected package libidn11:arm64.
Preparing to unpack .../6-libidn11_1.33-2.2_arm64.deb ...
Unpacking libidn11:arm64 (1.33-2.2) ...
Selecting previously unselected package libip4tc0:arm64.
Preparing to unpack .../7-libip4tc0_1.8.2-4_arm64.deb ...
Unpacking libip4tc0:arm64 (1.8.2-4) ...
Selecting previously unselected package libkmod2:arm64.
Preparing to unpack .../8-libkmod2_26-1_arm64.deb ...
Unpacking libkmod2:arm64 (26-1) ...
Selecting previously unselected package systemd.
Preparing to unpack .../9-systemd_241-7~deb10u8_arm64.deb ...
Unpacking systemd (241-7~deb10u8) ...
Setting up libapparmor1:arm64 (2.13.2-10) ...
Setting up libargon2-1:arm64 (0~20171227-0.2) ...
Setting up libjson-c3:arm64 (0.12.1+ds-2+deb10u1) ...
Setting up libidn11:arm64 (1.33-2.2) ...
Setting up libip4tc0:arm64 (1.8.2-4) ...
Setting up libkmod2:arm64 (26-1) ...
Setting up libdevmapper1.02.1:arm64 (2:1.02.155-3) ...
Setting up libcryptsetup12:arm64 (2:2.1.0-5+deb10u2) ...
Setting up systemd (241-7~deb10u8) ...
Created symlink /etc/systemd/system/getty.target.wants/getty@tty1.service → /lib/systemd/system/getty@.service.
Created symlink /etc/systemd/system/multi-user.target.wants/remote-fs.target → /lib/systemd/system/remote-fs.target.
Created symlink /etc/systemd/system/dbus-org.freedesktop.timesync1.service → /lib/systemd/system/systemd-timesyncd.service.
Created symlink /etc/systemd/system/sysinit.target.wants/systemd-timesyncd.service → /lib/systemd/system/systemd-timesyncd.service.
Setting up dmsetup (2:1.02.155-3) ...
Selecting previously unselected package systemd-sysv.
(Reading database ... 25328 files and directories currently installed.)
Preparing to unpack .../00-systemd-sysv_241-7~deb10u8_arm64.deb ...
Unpacking systemd-sysv (241-7~deb10u8) ...
Selecting previously unselected package libdbus-1-3:arm64.
Preparing to unpack .../01-libdbus-1-3_1.12.20-0+deb10u1_arm64.deb ...
Unpacking libdbus-1-3:arm64 (1.12.20-0+deb10u1) ...
Selecting previously unselected package dbus.
Preparing to unpack .../02-dbus_1.12.20-0+deb10u1_arm64.deb ...
Unpacking dbus (1.12.20-0+deb10u1) ...
Selecting previously unselected package libpam-systemd:arm64.
Preparing to unpack .../03-libpam-systemd_241-7~deb10u8_arm64.deb ...
Unpacking libpam-systemd:arm64 (241-7~deb10u8) ...
Selecting previously unselected package ncurses-term.
Preparing to unpack .../04-ncurses-term_6.1+20181013-2+deb10u2_all.deb ...
Unpacking ncurses-term (6.1+20181013-2+deb10u2) ...
Preparing to unpack .../05-openssh-client_1%3a7.9p1-10+deb10u2_arm64.deb ...
Unpacking openssh-client (1:7.9p1-10+deb10u2) over (1:7.9p1-10+deb10u1) ...
Selecting previously unselected package libwrap0:arm64.
Preparing to unpack .../06-libwrap0_7.6.q-28_arm64.deb ...
Unpacking libwrap0:arm64 (7.6.q-28) ...
Selecting previously unselected package libxmuu1:arm64.
Preparing to unpack .../07-libxmuu1_2%3a1.1.2-2+b3_arm64.deb ...
Unpacking libxmuu1:arm64 (2:1.1.2-2+b3) ...
Selecting previously unselected package openssh-sftp-server.
Preparing to unpack .../08-openssh-sftp-server_1%3a7.9p1-10+deb10u2_arm64.deb ...
Unpacking openssh-sftp-server (1:7.9p1-10+deb10u2) ...
Selecting previously unselected package openssh-server.
Preparing to unpack .../09-openssh-server_1%3a7.9p1-10+deb10u2_arm64.deb ...
Unpacking openssh-server (1:7.9p1-10+deb10u2) ...
Selecting previously unselected package xauth.
Preparing to unpack .../10-xauth_1%3a1.0.10-1_arm64.deb ...
Unpacking xauth (1:1.0.10-1) ...
Selecting previously unselected package libnss-systemd:arm64.
Preparing to unpack .../11-libnss-systemd_241-7~deb10u8_arm64.deb ...
Unpacking libnss-systemd:arm64 (241-7~deb10u8) ...
Setting up systemd-sysv (241-7~deb10u8) ...
Setting up openssh-client (1:7.9p1-10+deb10u2) ...
Setting up libnss-systemd:arm64 (241-7~deb10u8) ...
First installation detected...
Checking NSS setup...
Setting up libwrap0:arm64 (7.6.q-28) ...
Setting up libdbus-1-3:arm64 (1.12.20-0+deb10u1) ...
Setting up dbus (1.12.20-0+deb10u1) ...
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
Setting up libpam-systemd:arm64 (241-7~deb10u8) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
Setting up libxmuu1:arm64 (2:1.1.2-2+b3) ...
Setting up ncurses-term (6.1+20181013-2+deb10u2) ...
Setting up openssh-sftp-server (1:7.9p1-10+deb10u2) ...
Setting up openssh-server (1:7.9p1-10+deb10u2) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline

Creating config file /etc/ssh/sshd_config with new version
Creating SSH2 RSA key; this may take some time ...
2048 SHA256:cEaUdyT6Hx04XRMm9rM/3XwpPfYW1pk8aLuPj+H/H74 root@2e945ea6c6c8 (RSA)
Creating SSH2 ECDSA key; this may take some time ...
256 SHA256:SdNsg649xnvS1iDxsRnLZyYbAoBuVC0AAgBn2BrpBIA root@2e945ea6c6c8 (ECDSA)
Creating SSH2 ED25519 key; this may take some time ...
256 SHA256:gp7YaazhqkMCaueTtGPhN4eqXQM7EY/qo6CIMJKJUAM root@2e945ea6c6c8 (ED25519)
Created symlink /etc/systemd/system/sshd.service → /lib/systemd/system/ssh.service.
Created symlink /etc/systemd/system/multi-user.target.wants/ssh.service → /lib/systemd/system/ssh.service.
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
Setting up xauth (1:1.0.10-1) ...
Processing triggers for systemd (241-7~deb10u8) ...
Processing triggers for libc-bin (2.28-10) ...
Removing intermediate container 2e945ea6c6c8
 ---> 2e0c25b239f8
Step 5/12 : RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config && sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
 ---> Running in feb082ef3d54
Removing intermediate container feb082ef3d54
 ---> 04e7d111667f
Step 6/12 : ENV http_proxy ''
 ---> Running in d85cabc29931
Removing intermediate container d85cabc29931
 ---> 7be3a0816c55
Step 7/12 : ENV https_proxy ''
 ---> Running in fde04a902add
Removing intermediate container fde04a902add
 ---> ab4b80bbf41d
Step 8/12 : ENV ftp_proxy ''
 ---> Running in e69f82c2dbdb
Removing intermediate container e69f82c2dbdb
 ---> b5e9f01546bd
Step 9/12 : RUN useradd -d /home/hwMindX -u 9000 -m -s /bin/bash hwMindX &&     useradd -d /home/HwHiAiUser -u 1000 -m -s /bin/bash HwHiAiUser &&     usermod -a -G HwHiAiUser hwMindX
 ---> Running in 4d1821f15a61
Removing intermediate container 4d1821f15a61
 ---> 62891e7cf121
Step 10/12 : RUN echo root:123123 | chpasswd
 ---> Running in 6f31c83a1b24
Removing intermediate container 6f31c83a1b24
 ---> 90fb27e9b246
Step 11/12 : USER hwMindX
 ---> Running in 3dca31695032
Removing intermediate container 3dca31695032
 ---> 3c8e3d5fb2df
Step 12/12 : ENTRYPOINT jupyter notebook --allow-root
 ---> Running in 0af4a0c4ebf0
Removing intermediate container 0af4a0c4ebf0
 ---> 36ec8fff3e06
Successfully built 36ec8fff3e06
Successfully tagged notebook:latest
root@ubuntu:~/123/jupyter-notebook# 
root@ubuntu:~/123/jupyter-notebook# docker images | grep notebook 
notebook                                                      latest              36ec8fff3e06        2 minutes ago       1.04GB
root@ubuntu:~/123/jupyter-notebook#
```

  

十、构建tensorboard镜像

  

```
root@ubuntu:~/123# mkdir tensorboard
root@ubuntu:~/123# cd tensorboard
root@ubuntu:~/123/tensorboard# vi Dockerfile
root@ubuntu:~/123/tensorboard# cat Dockerfile 
FROM python:3.7.5

# -------------------------(Optional) Configure the pip source.--[l(1] ---------------------
#ENV http_proxy http://<user>:<password>@ip:port
#ENV https_proxy http://<user>:<password>@ip:port
#ENV ftp_proxy ftp://<user>:<password>@ip:port

RUN mkdir -p ~/.pip \
&& echo '[global] \n\ 
index-url=https://pypi.doubanio.com/simple/\n\ 
trusted-host=pypi.doubanio.com' >> ~/.pip/pip.conf

RUN pip install --upgrade pip && pip install tensorboard==1.15.0

ENV http_proxy ''
ENV https_proxy ''
ENV ftp_proxy ''


RUN useradd -d /home/hwMindX -u 9000 -m -s /bin/bash hwMindX && \
    useradd -d /home/HwHiAiUser -u 1000 -m -s /bin/bash HwHiAiUser && \
    usermod -a -G HwHiAiUser hwMindX

USER hwMindX

root@ubuntu:~/123/tensorboard# docker build -t tensorboard:latest .
Sending build context to Docker daemon   2.56kB
Step 1/8 : FROM python:3.7.5
 ---> a4356c370cda
Step 2/8 : RUN mkdir -p ~/.pip && echo '[global] \nindex-url=https://pypi.doubanio.com/simple/\ntrusted-host=pypi.doubanio.com' >> ~/.pip/pip.conf
 ---> Using cache
 ---> 2e9619b95f3c
Step 3/8 : RUN pip install --upgrade pip && pip install tensorboard==1.15.0
 ---> Running in 7d03c4546a34
Looking in indexes: https://pypi.doubanio.com/simple/
Collecting pip
  Downloading https://pypi.doubanio.com/packages/ca/31/b88ef447d595963c01060998cb329251648acf4a067721b0452c45527eb8/pip-21.2.4-py3-none-any.whl (1.6MB)
Installing collected packages: pip
  Found existing installation: pip 19.3.1
    Uninstalling pip-19.3.1:
      Successfully uninstalled pip-19.3.1
Successfully installed pip-21.2.4
Looking in indexes: https://pypi.doubanio.com/simple/
Collecting tensorboard==1.15.0
  Downloading https://pypi.doubanio.com/packages/1e/e9/d3d747a97f7188f48aa5eda486907f3b345cd409f0a0850468ba867db246/tensorboard-1.15.0-py3-none-any.whl (3.8 MB)
Requirement already satisfied: setuptools>=41.0.0 in /usr/local/lib/python3.7/site-packages (from tensorboard==1.15.0) (41.6.0)
Collecting markdown>=2.6.8
  Downloading https://pypi.doubanio.com/packages/6e/33/1ae0f71395e618d6140fbbc9587cc3156591f748226075e0f7d6f9176522/Markdown-3.3.4-py3-none-any.whl (97 kB)
Collecting six>=1.10.0
  Downloading https://pypi.doubanio.com/packages/d9/5a/e7c31adbe875f2abbb91bd84cf2dc52d792b5a01506781dbcf25c91daf11/six-1.16.0-py2.py3-none-any.whl (11 kB)
Collecting numpy>=1.12.0
  Downloading https://pypi.doubanio.com/packages/5c/61/b2f14fb5aa1198fa63c6c90205dc2557df5cacdeb0b16d66abc6af8724b8/numpy-1.21.2-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (13.0 MB)
Collecting werkzeug>=0.11.15
  Downloading https://pypi.doubanio.com/packages/bd/24/11c3ea5a7e866bf2d97f0501d0b4b1c9bbeade102bb4b588f0d2919a5212/Werkzeug-2.0.1-py3-none-any.whl (288 kB)
Requirement already satisfied: wheel>=0.26 in /usr/local/lib/python3.7/site-packages (from tensorboard==1.15.0) (0.33.6)
Collecting grpcio>=1.6.3
  Downloading https://pypi.doubanio.com/packages/8d/87/f66686884e21e4350746a18e664202ab9b39a3cd527df3ff54a022935ee5/grpcio-1.39.0-cp37-cp37m-manylinux_2_24_aarch64.whl (38.5 MB)
Collecting protobuf>=3.6.0
  Downloading https://pypi.doubanio.com/packages/fd/5f/6d6f7a5859caf79894685ec543354edc05538a0a34d63a411a2a7cb4ecfd/protobuf-3.17.3-cp37-cp37m-manylinux2014_aarch64.whl (922 kB)
Collecting absl-py>=0.4
  Downloading https://pypi.doubanio.com/packages/23/47/835652c7e19530973c73c65e652fc53bd05725d5a7cf9bb8706777869c1e/absl_py-0.13.0-py3-none-any.whl (132 kB)
Collecting importlib-metadata
  Downloading https://pypi.doubanio.com/packages/c0/72/4512a88e402d4dc3bab49a845130d95ac48936ef3a9469b55cc79a60d84d/importlib_metadata-4.6.4-py3-none-any.whl (17 kB)
Collecting zipp>=0.5
  Downloading https://pypi.doubanio.com/packages/92/d9/89f433969fb8dc5b9cbdd4b4deb587720ec1aeb59a020cf15002b9593eef/zipp-3.5.0-py3-none-any.whl (5.7 kB)
Collecting typing-extensions>=3.6.4
  Downloading https://pypi.doubanio.com/packages/2e/35/6c4fff5ab443b57116cb1aad46421fb719bed2825664e8fe77d66d99bcbc/typing_extensions-3.10.0.0-py3-none-any.whl (26 kB)
Installing collected packages: zipp, typing-extensions, six, importlib-metadata, werkzeug, protobuf, numpy, markdown, grpcio, absl-py, tensorboard
Successfully installed absl-py-0.13.0 grpcio-1.39.0 importlib-metadata-4.6.4 markdown-3.3.4 numpy-1.21.2 protobuf-3.17.3 six-1.16.0 tensorboard-1.15.0 typing-extensions-3.10.0.0 werkzeug-2.0.1 zipp-3.5.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
Removing intermediate container 7d03c4546a34
 ---> 4dd40311445c
Step 4/8 : ENV http_proxy ''
 ---> Running in 51e014124cef
Removing intermediate container 51e014124cef
 ---> 90694cc782ca
Step 5/8 : ENV https_proxy ''
 ---> Running in ddaaf7ceeded
Removing intermediate container ddaaf7ceeded
 ---> 66a6f22d6688
Step 6/8 : ENV ftp_proxy ''
 ---> Running in d18d506c95bb
Removing intermediate container d18d506c95bb
 ---> a6f8d3009ce3
Step 7/8 : RUN useradd -d /home/hwMindX -u 9000 -m -s /bin/bash hwMindX &&     useradd -d /home/HwHiAiUser -u 1000 -m -s /bin/bash HwHiAiUser &&     usermod -a -G HwHiAiUser hwMindX
 ---> Running in c29af506d5ed
Removing intermediate container c29af506d5ed
 ---> 5faf1754daba
Step 8/8 : USER hwMindX
 ---> Running in d032b1b13d00
Removing intermediate container d032b1b13d00
 ---> d00db512a587
Successfully built d00db512a587
Successfully tagged tensorboard:latest
root@ubuntu:~/123/tensorboard# docker images | grep tensorboard
tensorboard                                                   latest              d00db512a587        3 minutes ago       1.12GB
root@ubuntu:~/123/tensorboard#
```

  

十一、构建mindinsight镜像

  

```
root@ubuntu:~/123# mkdir mindinsight
root@ubuntu:~/123# cd mindinsight
root@ubuntu:~/123/mindinsight# wget https://ms-release.obs.cn-north-4.myhuaweicloud.com/1.3.0/MindInsight/any/mindinsight-1.3.0-py3-none-any.whl
--2021-08-21 11:29:24--  https://ms-release.obs.cn-north-4.myhuaweicloud.com/1.3.0/MindInsight/any/mindinsight-1.3.0-py3-none-any.whl
Resolving ms-release.obs.cn-north-4.myhuaweicloud.com (ms-release.obs.cn-north-4.myhuaweicloud.com)... 49.4.112.92, 49.4.112.3, 49.4.112.91
Connecting to ms-release.obs.cn-north-4.myhuaweicloud.com (ms-release.obs.cn-north-4.myhuaweicloud.com)|49.4.112.92|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5422450 (5.2M) [binary/octet-stream]
Saving to: ‘mindinsight-1.3.0-py3-none-any.whl’

mindinsight-1.3.0-py3-none-any.whl                   100%[====================================================================================================================>]   5.17M  24.3MB/s    in 0.2s    

2021-08-21 11:29:25 (24.3 MB/s) - ‘mindinsight-1.3.0-py3-none-any.whl’ saved [5422450/5422450]

root@ubuntu:~/123/mindinsight# vi Dockerfile
root@ubuntu:~/123/mindinsight# 
root@ubuntu:~/123/mindinsight# 
root@ubuntu:~/123/mindinsight# 
root@ubuntu:~/123/mindinsight# cat Dockerfile 
FROM python:3.7.5
 
# -------------------------(Optional) Configure the pip source.--[l(1] ---------------------
#ENV http_proxy http://<user>:<password>@ip:port
#ENV https_proxy http://<user>:<password>@ip:port
#ENV ftp_proxy ftp://<user>:<password>@ip:port
 
WORKDIR /tmp
COPY . ./
 
RUN mkdir -p ~/.pip \
&& echo '[global] \n\ 
index-url=https://pypi.doubanio.com/simple/\n\ 
trusted-host=pypi.doubanio.com' >> ~/.pip/pip.conf
 
RUN pip install --upgrade pip && pip3 install mindinsight-1.3.0-py3-none-any.whl && \
    rm mindinsight-1.3.0-py3-none-any.whl && rm Dockerfile

RUN sed -i "/^HOST/cHOST = '0.0.0.0'" /usr/local/lib/python3.7/site-packages/mindinsight/conf/constants.py
 
ENV http_proxy ''
ENV https_proxy ''
ENV ftp_proxy ''
RUN useradd -d /home/hwMindX -u 9000 -m -s /bin/bash hwMindX && \
    useradd -d /home/HwHiAiUser -u 1000 -m -s /bin/bash HwHiAiUser && \
    usermod -a -G HwHiAiUser hwMindX
root@ubuntu:~/123/mindinsight# vi start.sh
root@ubuntu:~/123/mindinsight# 
root@ubuntu:~/123/mindinsight# 
root@ubuntu:~/123/mindinsight# vi start.sh
root@ubuntu:~/123/mindinsight# 
root@ubuntu:~/123/mindinsight# 
root@ubuntu:~/123/mindinsight# 
root@ubuntu:~/123/mindinsight# cat start.sh 
#!/bin/bash
 
param=$*
 
mindinsight start $param
while true
do
    sleep 30
    processnum=`ps -ef | grep -w gunicorn | grep -v grep | wc -l`
    if [ $processnum -le 0 ];then
      exit 1  
    fi
done

root@ubuntu:~/123/mindinsight# docker build -t mindinsight:latest .
Sending build context to Docker daemon  5.427MB
Step 1/10 : FROM python:3.7.5
 ---> a4356c370cda
Step 2/10 : WORKDIR /tmp
 ---> Using cache
 ---> 6dca759650eb
Step 3/10 : COPY . ./
 ---> 1c047c1f27a0
Step 4/10 : RUN mkdir -p ~/.pip && echo '[global] \nindex-url=https://pypi.doubanio.com/simple/\ntrusted-host=pypi.doubanio.com' >> ~/.pip/pip.conf
 ---> Running in 9ec07e2141f8
Removing intermediate container 9ec07e2141f8
 ---> d2770d9bceb0
Step 5/10 : RUN pip install --upgrade pip && pip3 install mindinsight-1.3.0-py3-none-any.whl &&     rm mindinsight-1.3.0-py3-none-any.whl && rm Dockerfile
 ---> Running in 4f6b33bf9638
Looking in indexes: https://pypi.doubanio.com/simple/
Collecting pip
  Downloading https://pypi.doubanio.com/packages/ca/31/b88ef447d595963c01060998cb329251648acf4a067721b0452c45527eb8/pip-21.2.4-py3-none-any.whl (1.6MB)
Installing collected packages: pip
  Found existing installation: pip 19.3.1
    Uninstalling pip-19.3.1:
      Successfully uninstalled pip-19.3.1
Successfully installed pip-21.2.4
Looking in indexes: https://pypi.doubanio.com/simple/
Processing ./mindinsight-1.3.0-py3-none-any.whl
Collecting treelib>=1.6.1
  Downloading https://pypi.doubanio.com/packages/04/b0/2269c328abffbb63979f7143351a24a066776b87526d79956aea5018b80a/treelib-1.6.1.tar.gz (24 kB)
Collecting XlsxWriter>=1.3.2
  Downloading https://pypi.doubanio.com/packages/68/51/f6f6aa86106712dad0db9663c7d24b6b32d2103b626d900fa68d48a9b262/XlsxWriter-3.0.1-py3-none-any.whl (148 kB)
Collecting pandas>=1.0.4
  Downloading https://pypi.doubanio.com/packages/08/dc/d3513ec40c7df37a0e55b749a9b3a715f0d8b992c34c6ec6050bfd4a1703/pandas-1.3.2-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (10.7 MB)
Collecting MarkupSafe>=1.1.1
  Downloading https://pypi.doubanio.com/packages/70/fc/5a7253a9c1c4e2a3feadb80a5def4563500daa4b2d4a39cae39483afa1b0/MarkupSafe-2.0.1-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (26 kB)
Collecting scikit-learn>=0.23.1
  Downloading https://pypi.doubanio.com/packages/f3/78/b4aa1db778c2eb4a2e7ee0df84461ecb1bb2019031fd08723bd4c923452f/scikit_learn-0.24.2-cp37-cp37m-manylinux2014_aarch64.whl (24.0 MB)
Collecting psutil>=5.7.0
  Downloading https://pypi.doubanio.com/packages/e1/b0/7276de53321c12981717490516b7e612364f2cb372ee8901bd4a66a000d7/psutil-5.8.0.tar.gz (470 kB)
Collecting marshmallow>=3.10.0
  Downloading https://pypi.doubanio.com/packages/2b/fb/d42cbb318e07c4d709a3cb8d85a2ca75fbb373fc536cb0afd36039233f32/marshmallow-3.13.0-py2.py3-none-any.whl (47 kB)
Collecting yapf>=0.30.0
  Downloading https://pypi.doubanio.com/packages/5f/0d/8814e79eb865eab42d95023b58b650d01dec6f8ea87fc9260978b1bf2167/yapf-0.31.0-py2.py3-none-any.whl (185 kB)
Collecting Jinja2>=2.10.1
  Downloading https://pypi.doubanio.com/packages/80/21/ae597efc7ed8caaa43fb35062288baaf99a7d43ff0cf66452ddf47604ee6/Jinja2-3.0.1-py3-none-any.whl (133 kB)
Collecting Flask>=1.1.1
  Downloading https://pypi.doubanio.com/packages/54/4f/1b294c1a4ab7b2ad5ca5fc4a9a65a22ef1ac48be126289d97668852d4ab3/Flask-2.0.1-py3-none-any.whl (94 kB)
Collecting protobuf>=3.8.0
  Downloading https://pypi.doubanio.com/packages/fd/5f/6d6f7a5859caf79894685ec543354edc05538a0a34d63a411a2a7cb4ecfd/protobuf-3.17.3-cp37-cp37m-manylinux2014_aarch64.whl (922 kB)
Collecting google-pasta>=0.1.8
  Downloading https://pypi.doubanio.com/packages/a3/de/c648ef6835192e6e2cc03f40b19eeda4382c49b5bafb43d88b931c4c74ac/google_pasta-0.2.0-py3-none-any.whl (57 kB)
Collecting pyyaml>=5.3.1
  Downloading https://pypi.doubanio.com/packages/32/ac/a9383af90be713b0cb2ee7c7eb4317ab76957ed0a7e4aa9b9c170a992565/PyYAML-5.4.1-cp37-cp37m-manylinux2014_aarch64.whl (716 kB)
Collecting gunicorn>=20.0.4
  Downloading https://pypi.doubanio.com/packages/e4/dd/5b190393e6066286773a67dfcc2f9492058e9b57c4867a95f1ba5caf0a83/gunicorn-20.1.0-py3-none-any.whl (79 kB)
Collecting pillow>=6.2.0
  Downloading https://pypi.doubanio.com/packages/df/97/e6e1aae9d75a7ac638cd7e5c5ddd1cf0ed3813275c07a43b68d081e1d479/Pillow-8.3.1-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (2.9 MB)
Collecting Click>=7.0
  Downloading https://pypi.doubanio.com/packages/76/0a/b6c5f311e32aeb3b406e03c079ade51e905ea630fc19d1262a46249c1c86/click-8.0.1-py3-none-any.whl (97 kB)
Collecting scipy>=1.5.2
  Downloading https://pypi.doubanio.com/packages/01/0b/279f3a059ee7e59aa087ccfe0c81e69fc286b869a9052f8c119ba1138a7b/scipy-1.7.1-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (27.3 MB)
Collecting grpcio<=1.36.0,>=1.35.0
  Downloading https://pypi.doubanio.com/packages/9d/9e/18e92a4042fdee8613f5613a37cf7162d32b5674f1b12d0f7b042e7e710b/grpcio-1.36.0.tar.gz (21.5 MB)
Collecting Flask-Cors>=3.0.8
  Downloading https://pypi.doubanio.com/packages/db/84/901e700de86604b1c4ef4b57110d4e947c218b9997adf5d38fa7da493bce/Flask_Cors-3.0.10-py2.py3-none-any.whl (14 kB)
Collecting numpy>=1.17.0
  Downloading https://pypi.doubanio.com/packages/5c/61/b2f14fb5aa1198fa63c6c90205dc2557df5cacdeb0b16d66abc6af8724b8/numpy-1.21.2-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (13.0 MB)
Collecting six>=1.12.0
  Downloading https://pypi.doubanio.com/packages/d9/5a/e7c31adbe875f2abbb91bd84cf2dc52d792b5a01506781dbcf25c91daf11/six-1.16.0-py2.py3-none-any.whl (11 kB)
Collecting itsdangerous>=1.1.0
  Downloading https://pypi.doubanio.com/packages/9c/96/26f935afba9cd6140216da5add223a0c465b99d0f112b68a4ca426441019/itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting Werkzeug>=1.0.0
  Downloading https://pypi.doubanio.com/packages/bd/24/11c3ea5a7e866bf2d97f0501d0b4b1c9bbeade102bb4b588f0d2919a5212/Werkzeug-2.0.1-py3-none-any.whl (288 kB)
Collecting importlib-metadata
  Downloading https://pypi.doubanio.com/packages/c0/72/4512a88e402d4dc3bab49a845130d95ac48936ef3a9469b55cc79a60d84d/importlib_metadata-4.6.4-py3-none-any.whl (17 kB)
Requirement already satisfied: setuptools>=3.0 in /usr/local/lib/python3.7/site-packages (from gunicorn>=20.0.4->mindinsight==1.3.0) (41.6.0)
Collecting pytz>=2017.3
  Downloading https://pypi.doubanio.com/packages/70/94/784178ca5dd892a98f113cdd923372024dc04b8d40abe77ca76b5fb90ca6/pytz-2021.1-py2.py3-none-any.whl (510 kB)
Collecting python-dateutil>=2.7.3
  Downloading https://pypi.doubanio.com/packages/36/7a/87837f39d0296e723bb9b62bbb257d0355c7f6128853c78955f57342a56d/python_dateutil-2.8.2-py2.py3-none-any.whl (247 kB)
Collecting threadpoolctl>=2.0.0
  Downloading https://pypi.doubanio.com/packages/c6/e8/c216b9b60cbba4642d3ca1bae7a53daa0c24426f662e0e3ce3dc7f6caeaa/threadpoolctl-2.2.0-py3-none-any.whl (12 kB)
Collecting joblib>=0.11
  Downloading https://pypi.doubanio.com/packages/55/85/70c6602b078bd9e6f3da4f467047e906525c355a4dacd4f71b97a35d9897/joblib-1.0.1-py3-none-any.whl (303 kB)
Collecting future
  Downloading https://pypi.doubanio.com/packages/45/0b/38b06fd9b92dc2b68d58b75f900e97884c45bedd2ff83203d933cf5851c9/future-0.18.2.tar.gz (829 kB)
Collecting typing-extensions>=3.6.4
  Downloading https://pypi.doubanio.com/packages/2e/35/6c4fff5ab443b57116cb1aad46421fb719bed2825664e8fe77d66d99bcbc/typing_extensions-3.10.0.0-py3-none-any.whl (26 kB)
Collecting zipp>=0.5
  Downloading https://pypi.doubanio.com/packages/92/d9/89f433969fb8dc5b9cbdd4b4deb587720ec1aeb59a020cf15002b9593eef/zipp-3.5.0-py3-none-any.whl (5.7 kB)
Building wheels for collected packages: grpcio, psutil, treelib, future
  Building wheel for grpcio (setup.py): started
  Building wheel for grpcio (setup.py): still running...
  Building wheel for grpcio (setup.py): finished with status 'done'
  Created wheel for grpcio: filename=grpcio-1.36.0-cp37-cp37m-linux_aarch64.whl size=43277028 sha256=0f9c41905a2331f6405bee3d2e7dcf083ea1aa6e39f85ae1190ff3a2ce98069a
  Stored in directory: /root/.cache/pip/wheels/dd/72/74/30b696f7d2a6abedf42d201eccd5f7a03f84931dfaa4b147db
  Building wheel for psutil (setup.py): started
  Building wheel for psutil (setup.py): finished with status 'done'
  Created wheel for psutil: filename=psutil-5.8.0-cp37-cp37m-linux_aarch64.whl size=295178 sha256=34f88db3e2056672357f438d0d338027bb65594437eafb4ae7c391ad0380d8bb
  Stored in directory: /root/.cache/pip/wheels/2f/c7/77/86efc5d98b9a79575ab1aaa9c24651f8841e57d46862979efd
  Building wheel for treelib (setup.py): started
  Building wheel for treelib (setup.py): finished with status 'done'
  Created wheel for treelib: filename=treelib-1.6.1-py3-none-any.whl size=18370 sha256=437a681028a4c479a0cc23c87a01a2cc51be82dc9fd355f3baa9e1363fe7fa5b
  Stored in directory: /root/.cache/pip/wheels/44/dd/b6/a9967a60d3575162a2e29846347f826dcb20466b0a1b67198e
  Building wheel for future (setup.py): started
  Building wheel for future (setup.py): finished with status 'done'
  Created wheel for future: filename=future-0.18.2-py3-none-any.whl size=491056 sha256=5b3be1990014b279b2ec75bfb2ba81737b5f2eee949d6573c7924391bdd19ed7
  Stored in directory: /root/.cache/pip/wheels/97/83/dc/8e47b8e3874918101250790fbce5d89fef8d0c33eb8097ad07
Successfully built grpcio psutil treelib future
Installing collected packages: zipp, typing-extensions, MarkupSafe, importlib-metadata, Werkzeug, six, numpy, Jinja2, itsdangerous, Click, threadpoolctl, scipy, pytz, python-dateutil, joblib, future, Flask, yapf, XlsxWriter, treelib, scikit-learn, pyyaml, psutil, protobuf, pillow, pandas, marshmallow, gunicorn, grpcio, google-pasta, Flask-Cors, mindinsight
Successfully installed Click-8.0.1 Flask-2.0.1 Flask-Cors-3.0.10 Jinja2-3.0.1 MarkupSafe-2.0.1 Werkzeug-2.0.1 XlsxWriter-3.0.1 future-0.18.2 google-pasta-0.2.0 grpcio-1.36.0 gunicorn-20.1.0 importlib-metadata-4.6.4 itsdangerous-2.0.1 joblib-1.0.1 marshmallow-3.13.0 mindinsight-1.3.0 numpy-1.21.2 pandas-1.3.2 pillow-8.3.1 protobuf-3.17.3 psutil-5.8.0 python-dateutil-2.8.2 pytz-2021.1 pyyaml-5.4.1 scikit-learn-0.24.2 scipy-1.7.1 six-1.16.0 threadpoolctl-2.2.0 treelib-1.6.1 typing-extensions-3.10.0.0 yapf-0.31.0 zipp-3.5.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
Removing intermediate container 4f6b33bf9638
 ---> 2ef8fa6e4a74
Step 6/10 : RUN sed -i "/^HOST/cHOST = '0.0.0.0'" /usr/local/lib/python3.7/site-packages/mindinsight/conf/constants.py
 ---> Running in 1da0a898001b
Removing intermediate container 1da0a898001b
 ---> 08940216b1cc
Step 7/10 : ENV http_proxy ''
 ---> Running in 34dee7b0cd32
Removing intermediate container 34dee7b0cd32
 ---> 7b7d62b31d09
Step 8/10 : ENV https_proxy ''
 ---> Running in ccc705d304f8
Removing intermediate container ccc705d304f8
 ---> a533d73340db
Step 9/10 : ENV ftp_proxy ''
 ---> Running in 583ab62da6b4
Removing intermediate container 583ab62da6b4
 ---> e9377900aedc
Step 10/10 : RUN useradd -d /home/hwMindX -u 9000 -m -s /bin/bash hwMindX &&     useradd -d /home/HwHiAiUser -u 1000 -m -s /bin/bash HwHiAiUser &&     usermod -a -G HwHiAiUser hwMindX
 ---> Running in c111b84f6220
Removing intermediate container c111b84f6220
 ---> 7c11b719966b
Successfully built 7c11b719966b
Successfully tagged mindinsight:latest
root@ubuntu:~/123/mindinsight# docker images | grep mindinsight
mindinsight                                                   latest              7c11b719966b        17 seconds ago      1.52GB
root@ubuntu:~/123/mindinsight#
```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1a58c86830a4135999669e6d94a091a~tplv-k3u1fbpfcp-zoom-1.image)

  

十二、配置Grafana

  

    打开Prometheus地址 》 选择“Status > Targets” 》 当kubenetes-cadvisor下的“Endpoint”状态为“UP”时，记录“Labels”下的job值，该值为cadvisor所在节点的nodeName，下方文件中的“nodeName”批量替换成此名称。

  

```
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "gnetId": null,
  "graphTooltip": 0,
  "id": 2,
  "links": [],
  "panels": [
    {
      "datasource": null,
      "fieldConfig": {
        "defaults": {
          "custom": {},
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 7,
        "x": 0,
        "y": 0
      },
      "id": 2,
      "options": {
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "mean"
          ],
          "fields": "",
          "values": false
        },
        "showThresholdLabels": false,
        "showThresholdMarkers": true
      },
      "pluginVersion": "7.0.2",
      "targets": [
        {
          "expr": "npu_chip_info_health_status{id=\"0\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片0",
          "refId": "A"
        },
        {
          "expr": "npu_chip_info_health_status{id=\"1\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片1",
          "refId": "B"
        },
        {
          "expr": "npu_chip_info_health_status{id=\"2\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片2",
          "refId": "C"
        },
        {
          "expr": "npu_chip_info_health_status{id=\"3\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片3",
          "refId": "D"
        },
        {
          "expr": "npu_chip_info_health_status{id=\"4\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片4",
          "refId": "E"
        },
        {
          "expr": "npu_chip_info_health_status{id=\"5\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片5",
          "refId": "F"
        },
        {
          "expr": "npu_chip_info_health_status{id=\"6\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片6",
          "refId": "G"
        },
        {
          "expr": "npu_chip_info_health_status{id=\"7\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片7",
          "refId": "H"
        }
      ],
      "timeFrom": null,
      "timeShift": null,
      "title": "npu健康状态(nodeName)",
      "type": "gauge"
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "Prometheus",
      "fieldConfig": {
        "defaults": {
          "custom": {},
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 8,
        "x": 7,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 6,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pluginVersion": "7.0.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "npu_chip_info_temperature{id=\"0\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片0",
          "refId": "A"
        },
        {
          "expr": "npu_chip_info_temperature{id=\"1\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片1",
          "refId": "B"
        },
        {
          "expr": "npu_chip_info_temperature{id=\"2\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片2",
          "refId": "C"
        },
        {
          "expr": "npu_chip_info_temperature{id=\"3\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片3",
          "refId": "D"
        },
        {
          "expr": "npu_chip_info_temperature{id=\"4\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片4",
          "refId": "E"
        },
        {
          "expr": "npu_chip_info_temperature{id=\"5\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片5",
          "refId": "F"
        },
        {
          "expr": "npu_chip_info_temperature{id=\"6\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片6",
          "refId": "G"
        },
        {
          "expr": "npu_chip_info_temperature{id=\"7\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片7",
          "refId": "H"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "npu温度(nodeName)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "datasource": null,
      "fieldConfig": {
        "defaults": {
          "custom": {},
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 8,
        "x": 15,
        "y": 0
      },
      "id": 12,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "mean"
          ],
          "fields": "",
          "values": false
        }
      },
      "pluginVersion": "7.0.2",
      "targets": [
        {
          "expr": "npu_chip_info_voltage{id=\"0\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片0",
          "refId": "A"
        },
        {
          "expr": "npu_chip_info_voltage{id=\"1\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片1",
          "refId": "B"
        },
        {
          "expr": "npu_chip_info_voltage{id=\"2\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片2",
          "refId": "C"
        },
        {
          "expr": "npu_chip_info_voltage{id=\"3\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片3",
          "refId": "D"
        },
        {
          "expr": "npu_chip_info_voltage{id=\"4\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片4",
          "refId": "E"
        },
        {
          "expr": "npu_chip_info_voltage{id=\"5\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片5",
          "refId": "F"
        },
        {
          "expr": "npu_chip_info_voltage{id=\"6\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片6",
          "refId": "G"
        },
        {
          "expr": "npu_chip_info_voltage{id=\"7\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片7",
          "refId": "H"
        }
      ],
      "timeFrom": null,
      "timeShift": null,
      "title": "npu电压(nodeName)",
      "type": "stat"
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": null,
      "fieldConfig": {
        "defaults": {
          "custom": {
            "align": null
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 7,
        "x": 0,
        "y": 8
      },
      "hiddenSeries": false,
      "id": 8,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pluginVersion": "7.0.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "npu_chip_info_used_memory{id=\"0\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片0",
          "refId": "A"
        },
        {
          "expr": "npu_chip_info_used_memory{id=\"1\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片1",
          "refId": "B"
        },
        {
          "expr": "npu_chip_info_used_memory{id=\"2\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片2",
          "refId": "C"
        },
        {
          "expr": "npu_chip_info_used_memory{id=\"3\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片3",
          "refId": "D"
        },
        {
          "expr": "npu_chip_info_used_memory{id=\"4\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片4",
          "refId": "E"
        },
        {
          "expr": "npu_chip_info_used_memory{id=\"5\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片5",
          "refId": "F"
        },
        {
          "expr": "npu_chip_info_used_memory{id=\"6\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片6",
          "refId": "G"
        },
        {
          "expr": "npu_chip_info_used_memory{id=\"7\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片7",
          "refId": "H"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "npu内存使用(nodeName)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": null,
      "description": "",
      "fieldConfig": {
        "defaults": {
          "custom": {
            "align": null
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 8,
        "x": 7,
        "y": 8
      },
      "hiddenSeries": false,
      "id": 10,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pluginVersion": "7.0.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "npu_chip_info_utilization{id=\"0\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片0",
          "refId": "A"
        },
        {
          "expr": "npu_chip_info_utilization{id=\"1\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片1",
          "refId": "B"
        },
        {
          "expr": "npu_chip_info_utilization{id=\"2\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片2",
          "refId": "C"
        },
        {
          "expr": "npu_chip_info_utilization{id=\"3\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片3",
          "refId": "D"
        },
        {
          "expr": "npu_chip_info_utilization{id=\"4\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片4",
          "refId": "E"
        },
        {
          "expr": "npu_chip_info_utilization{id=\"5\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片5",
          "refId": "F"
        },
        {
          "expr": "npu_chip_info_utilization{id=\"6\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片6",
          "refId": "G"
        },
        {
          "expr": "npu_chip_info_utilization{id=\"7\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片7",
          "refId": "H"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "npu_AI_Core使用率(nodeName)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": null,
      "fieldConfig": {
        "defaults": {
          "custom": {
            "align": null
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 8,
        "x": 15,
        "y": 8
      },
      "hiddenSeries": false,
      "id": 4,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pluginVersion": "7.0.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "npu_chip_info_power{id=\"0\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片0",
          "refId": "A"
        },
        {
          "expr": "npu_chip_info_power{id=\"1\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片1",
          "refId": "B"
        },
        {
          "expr": "npu_chip_info_power{id=\"2\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片2",
          "refId": "C"
        },
        {
          "expr": "npu_chip_info_power{id=\"3\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片3",
          "refId": "D"
        },
        {
          "expr": "npu_chip_info_power{id=\"4\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片4",
          "refId": "E"
        },
        {
          "expr": "npu_chip_info_power{id=\"5\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片5",
          "refId": "F"
        },
        {
          "expr": "npu_chip_info_power{id=\"6\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片6",
          "refId": "G"
        },
        {
          "expr": "npu_chip_info_power{id=\"7\",job=\"nodeName\"}",
          "interval": "",
          "legendFormat": "芯片7",
          "refId": "H"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "npu使用功耗(nodeName)",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": null,
      "fieldConfig": {
        "defaults": {
          "custom": {}
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 10,
        "w": 7,
        "x": 0,
        "y": 16
      },
      "hiddenSeries": false,
      "id": 20,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "container_memory_usage_bytes",
          "interval": "",
          "legendFormat": "",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "容器内存使用量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": null,
      "fieldConfig": {
        "defaults": {
          "custom": {}
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 10,
        "w": 7,
        "x": 7,
        "y": 16
      },
      "hiddenSeries": false,
      "id": 18,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "sum(rate(container_network_receive_bytes_total{image!=\"\"}[1m])) without (interface)",
          "interval": "",
          "legendFormat": "",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "容器网络接收量速率",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    }
  ],
  "refresh": "5s",
  "schemaVersion": 25,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-15m",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ]
  },
  "timezone": "",
  "title": "nodeName",
  "uid": "2kWOIniGz",
  "version": 7
}

```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f035f0db96145b89d31fca711d3b0c2~tplv-k3u1fbpfcp-zoom-1.image)

  

  

查看pod：  

  

```
root@ubuntu:~# kubectl get pod -A
NAMESPACE        NAME                                       READY   STATUS      RESTARTS   AGE
cadvisor         cadvisor-phj6x                             1/1     Running     1          17h
default          dls-cec-deploy-686bd4d6cd-gtrqb            1/1     Running     2          15h
default          dls-mms-deploy-6fd697754f-8xkkv            1/1     Running     2          15h
default          hccl-controller-645bb466f-lw9b4            1/1     Running     1          17h
default          tjm-77f784dcf-6s2f4                        1/1     Running     2          15h
kube-system      ascend-device-plugin-daemonset-qrfbx       1/1     Running     1          17h
kube-system      calico-kube-controllers-8464785d6b-tl44n   1/1     Running     1          17h
kube-system      calico-node-lm7x5                          1/1     Running     1          17h
kube-system      coredns-6955765f44-czzws                   1/1     Running     1          17h
kube-system      coredns-6955765f44-t2n4z                   1/1     Running     1          17h
kube-system      dls-apigw-deploy-9f58f549-sklgc            1/1     Running     2          15h
kube-system      dls-dms-deploy-76b79854cc-6m54s            1/1     Running     2          15h
kube-system      dls-ims-deploy-5445d6cc9d-96chh            1/1     Running     2          15h
kube-system      dls-nginx-deploy-7c9d889998-84956          1/1     Running     0          15h
kube-system      etcd-ubuntu                                1/1     Running     1          17h
kube-system      grafana-core-f97475d78-b74ww               1/1     Running     0          15h
kube-system      kube-apiserver-ubuntu                      1/1     Running     1          17h
kube-system      kube-controller-manager-ubuntu             1/1     Running     1          17h
kube-system      kube-proxy-rljv7                           1/1     Running     1          17h
kube-system      kube-scheduler-ubuntu                      1/1     Running     1          17h
kube-system      mysql-5cccdd88bd-mqv27                     1/1     Running     0          15h
kube-system      prometheus-58c69548b4-bgw4h                1/1     Running     0          15h
kube-system      redis-deploy-7fbc4fb97d-bkkmb              1/1     Running     0          15h
volcano-system   volcano-admission-74776688c8-45mr8         1/1     Running     1          17h
volcano-system   volcano-admission-init-zgr5t               0/1     Completed   0          17h
volcano-system   volcano-controllers-6786db54f-nwnfw        1/1     Running     1          17h
volcano-system   volcano-scheduler-844f9b547b-qkxgt         1/1     Running     1          17h
root@ubuntu:~#

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