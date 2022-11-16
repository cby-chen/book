---
layout: post
cid: 16
title: MINIO搭建单机以及集群
slug: 16
date: 2021/12/30 17:01:58
updated: 2021/12/30 17:01:58
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/639caaa0fe8b47a18330ca0090e4a052~tplv-k3u1fbpfcp-zoom-1.image)

**MINIO简介**

    Minio是Apache License v2.0下发布的对象存储服务器。它与Amazon S3云存储服务兼容。它最适合存储非结构化数据，如照片，视频，日志文件，备份和容器/VM映像。对象的大小可以从几KB到最大5TB。Minio服务器足够轻，可以与应用程序堆栈捆绑在一起，类似于NodeJS，Redis和MySQL。

      https://docs.minio.io/

  

**一、单机版搭建**  

<table><tbody><tr><td width="268" valign="top" style="word-break: break-all;"><section style="text-indent: 0em;">操作系统<br></section></td><td width="268" valign="top" style="word-break: break-all;"><section style="text-indent: 0em;">搭建方式<br></section></td></tr><tr><td width="268" valign="top" style="word-break: break-all;"><section style="text-indent: 0em;">Linux<br></section></td><td width="268" valign="top" style="word-break: break-all;"><section style="text-indent: 0em;">docker<br></section></td></tr><tr><td width="268" valign="top" style="word-break: break-all;"><section style="text-indent: 0em;">Linux<br></section></td><td width="268" valign="top" style="word-break: break-all;"><section style="text-indent: 0em;">宿主机<br></section></td></tr></tbody></table>

  

**1\. docker模式搭建**

**1.1安装docker**

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
[root@localhost ~]# systemctl start docker #启动docker
[root@localhost ~]# docker ps -a #查看一下命令是否可以执行
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@localhost ~]#
```

  

**1.2使用docker安装MINIO**  

```
[root@localhost ~]# docker search minio
NAME                           DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
minio/minio                    Kubernetes Native, High Performance Object S…   445                  [OK]
minio/mc                       Minio Client (mc) provides a modern alternat…   22                   [OK]
bitnami/minio                  Bitnami MinIO Docker Image                      6                    
pixelchrome/minio-arm          This Dockerfile installs Minio on your ARM-P…   5                    
jessestuart/minio              Minio server — supports arm (arm32v6, arm32v…   5                    
minio/console                  A graphical user interface for MinIO server     4                    
webhippie/minio                Docker images for Minio                         3                    [OK]
opennms/minion                 Application container runs Minion by OpenNMS…   3                    [OK]
bitnami/minio-client           Bitnami MinIO Client Docker Image               3                    
rook/minio                     Minio is a high performance distributed obje…   2                    
rancher/minio-minio                                                            1                    
zenithar/minio-server          Minio.io Server in Alpine Linux docker          1                    [OK]
teamwork/minio                 Minio for Teamwork                              1                    
azinchen/minio                 Minio server Docker image. Always up-to-date…   1                    
minio/mint                     Collection of tests to detect overall correc…   0                    [OK]
tobilg/minio-dcos              minio on DC/OS                                  0                    [OK]
topdockercat/minio-unraid      Minio is an Amazon S3 compatible object stor…   0                    [OK]
keikoproj/minion-manager       https://github.com/orkaproj/minion-manager      0                    
joepll/minio-exporter          Prometheus exporter for Minio server            0                    
opsmx11/minio                  Minio for Openshift                             0                    [OK]
leviy/minio                    Minio image for development and testing of (…   0                    [OK]
minio/k8s-operator             Minio Operator for k8s https://kubernetes.io/   0                    
rwsdockercf/minio-resource                                                     0                    
nerc/minio                     Minio container for use in the datalab proje…   0                    [OK]
kazesberger/miniomc-postgres   this image is used to create postgres dumps …   0                    
[root@localhost ~]# 
[root@localhost ~]# docker pull minio/minio
Using default tag: latest
latest: Pulling from minio/minio
8f403cb21126: Pull complete 
65c0f2178ac8: Pull complete 
6e32ce08526e: Pull complete 
932fb72de569: Pull complete 
71bfd33c61af: Pull complete 
588b2addab38: Pull complete 
093f7de724c9: Pull complete 
Digest: sha256:fe69dcaed404faa1a36953513bf2fe2d5427071fa612487295eddb2b18cfe918
Status: Downloaded newer image for minio/minio:latest
docker.io/minio/minio:latest
[root@localhost ~]# 
[root@localhost ~]# docker run -p 9000:9000 --name minio1 \
> -e "MINIO_ACCESS_KEY=admin" \
> -e "MINIO_SECRET_KEY=12345678" \
> -v /Users/xiyou/my_minio/data:/data \
> -v /Users/xiyou/my_minio/config:/root/.minio \
> minio/minio server /data
Endpoint: http://172.17.0.2:9000  http://127.0.0.1:9000 

Browser Access:
   http://172.17.0.2:9000  http://127.0.0.1:9000

Object API (Amazon S3 compatible):
   Go:         https://docs.min.io/docs/golang-client-quickstart-guide
   Java:       https://docs.min.io/docs/java-client-quickstart-guide
   Python:     https://docs.min.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide
IAM initialization complete

```

  

**1.3使用该命令将其放置在后台启动**

```
[root@localhost ~]# docker run -d -p 9000:9000 --name minio1 \
> -e "MINIO_ACCESS_KEY=admin" \
> -e "MINIO_SECRET_KEY=12345678" \
> -v /my_minio/data:/data \
> -v /my_minio/config:/root/.minio \
> minio/minio server /data
b6fca0d91b4bc5ef5f5b4b2a77a6f4761fc18e1d4b08f88519304813b52586d7
[root@localhost ~]#

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95417991985346ed98981199dbd13400~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce7ec3609a034f689bb1f295dbad545b~tplv-k3u1fbpfcp-zoom-1.image)

  

  

**1.4宿主机安装**

  

```
[root@localhost ~]# wget https://dl.min.io/server/minio/release/linux-amd64/minio
--2021-05-12 23:14:06--  https://dl.min.io/server/minio/release/linux-amd64/minio
Resolving dl.min.io (dl.min.io)... 178.128.69.202
Connecting to dl.min.io (dl.min.io)|178.128.69.202|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 58748928 (56M) [application/octet-stream]
Saving to: ‘minio’

minio                                   100%[============================================================================>]  56.03M  7.24MB/s    in 9.5s    

2021-05-12 23:14:17 (5.89 MB/s) - ‘minio’ saved [58748928/58748928]

[root@localhost ~]# chmod +x minio
[root@localhost ~]#  MINIO_ACCESS_KEY=minio MINIO_SECRET_KEY=minio123 ./minio server /cby
Endpoint: http://192.168.100.139:9000  http://127.0.0.1:9000         
RootUser: minio 
RootPass: minio123 

Browser Access:
   http://192.168.100.139:9000  http://127.0.0.1:9000        

Command-line Access: https://docs.min.io/docs/minio-client-quickstart-guide
   $ mc alias set myminio http://192.168.100.139:9000 minio minio123

Object API (Amazon S3 compatible):
   Go:         https://docs.min.io/docs/golang-client-quickstart-guide
   Java:       https://docs.min.io/docs/java-client-quickstart-guide
   Python:     https://docs.min.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide
IAM initialization complete

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eefb6210052e4f3ebba536185bbffe7d~tplv-k3u1fbpfcp-zoom-1.image)

  

  

**二、集群版搭建**

  

**2\. 集群搭建**

**2.1 集群服务器配置及启动**

  

  

启动一个分布式Minio实例，你只需要把硬盘位置做为参数传给minio server命令即可，然后，你需要在所有其它节点运行同样的命令。

  

**\*注意**

  

    分布式Minio里所有的节点需要有同样的access秘钥和secret秘钥，这样这些节点才能建立联接。为了实现这个，你需要在执行minio server命令之前，先将access秘钥和secret秘钥export成环境变量。同时分布式Minio使用的磁盘里必须是干净的，里面没有数据。

  

    分布式Minio里的节点时间差不能超过3秒，你可以使用NTP 来保证时间一致。在Windows下运行分布式Minio处于实验阶段，请悠着点使用。

参考：https://docs.min.io/cn/distributed-minio-quickstart-guide.html

  

<table><tbody><tr><td width="268" valign="top" style="word-break: break-all;">名称<br></td><td width="268" valign="top" style="word-break: break-all;">IP<br></td></tr><tr><td width="268" valign="top" style="word-break: break-all;">node</td><td width="268" valign="top" style="word-break: break-all;">192.168.100.138<br></td></tr><tr><td width="268" valign="top" style="word-break: break-all;">node<br></td><td width="268" valign="top" style="word-break: break-all;">192.168.100.139<br></td></tr><tr><td valign="top" colspan="1" rowspan="1" style="word-break: break-all;">node<br></td><td valign="top" colspan="1" rowspan="1" style="word-break: break-all;">192.168.100.140</td></tr><tr><td valign="top" colspan="1" rowspan="1" style="word-break: break-all;">node<br></td><td valign="top" colspan="1" rowspan="1" style="word-break: break-all;">192.168.100.141</td></tr></tbody></table>

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5de3f81de174774bbdb7fa403292d3b~tplv-k3u1fbpfcp-zoom-1.image)

**2.2每台上面搭建**  

  

```
[root@localhost software]# mkdir -p /chenby/software
[root@localhost software]# cd /chenby/software
[root@localhost software]# wget https://dl.min.io/server/minio/release/linux-amd64/minio
--2021-05-13 00:06:25--  https://dl.min.io/server/minio/release/linux-amd64/minio
Resolving dl.min.io (dl.min.io)... 178.128.69.202
Connecting to dl.min.io (dl.min.io)|178.128.69.202|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 58748928 (56M) [application/octet-stream]
Saving to: ‘minio’

minio                                   100%[============================================================================>]  56.03M  1.95MB/s    in 21s     

2021-05-13 00:06:48 (2.63 MB/s) - ‘minio’ saved [58748928/58748928]

[root@localhost software]# chmod +x minio
[root@localhost software]# MINIO_ACCESS_KEY=minio MINIO_SECRET_KEY=minio123 ./minio server http://192.168.100.138/chenby/software/cby http://192.168.100.139/chenby/software/cby http://192.168.100.140/chenby/software/cby http://192.168.100.141/chenby/software/cby
Waiting for all MinIO sub-systems to be initialized.. lock acquired
All MinIO sub-systems initialized successfully
Status:         4 Online, 0 Offline. 
Endpoint: http://192.168.100.138:9000  http://127.0.0.1:9000       
RootUser: minio 
RootPass: minio123 

Browser Access:
   http://192.168.100.138:9000  http://127.0.0.1:9000      

Command-line Access: https://docs.min.io/docs/minio-client-quickstart-guide
   $ mc alias set myminio http://192.168.100.138:9000 minio minio123

Object API (Amazon S3 compatible):
   Go:         https://docs.min.io/docs/golang-client-quickstart-guide
   Java:       https://docs.min.io/docs/java-client-quickstart-guide
   Python:     https://docs.min.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide
Waiting for all MinIO IAM sub-system to be initialized.. lock acquired
IAM initialization complete

```

  

**2.3 访问测试**  

```
[root@localhost ~]# curl -I http://192.168.100.138:9000/minio/login
HTTP/1.1 403 Forbidden
Accept-Ranges: bytes
Content-Length: 256
Content-Type: application/xml
Server: MinIO
Vary: Origin
Date: Wed, 12 May 2021 16:09:19 GMT

[root@localhost ~]# curl -I http://192.168.100.139:9000/minio/login
HTTP/1.1 403 Forbidden
Accept-Ranges: bytes
Content-Length: 256
Content-Type: application/xml
Server: MinIO
Vary: Origin
Date: Wed, 12 May 2021 16:09:25 GMT

[root@localhost ~]# curl -I http://192.168.100.140:9000/minio/login
HTTP/1.1 403 Forbidden
Accept-Ranges: bytes
Content-Length: 256
Content-Type: application/xml
Server: MinIO
Vary: Origin
Date: Wed, 12 May 2021 16:09:32 GMT

[root@localhost ~]# curl -I http://192.168.100.141:9000/minio/login
HTTP/1.1 403 Forbidden
Accept-Ranges: bytes
Content-Length: 256
Content-Type: application/xml
Server: MinIO
Vary: Origin
Date: Wed, 12 May 2021 16:09:36 GMT

```

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0338969963c241f4af29984109badbee~tplv-k3u1fbpfcp-zoom-1.image)

  

  

  

  

  

  

  

```