---
layout: post
cid: 5
title: Docker启动MySQL、MongoDB、Redis、Elasticsearch、Grafana，数据库
slug: 5
date: 2021/12/30 16:57:00
updated: 2022/03/25 15:53:08
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


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d57a34049f8d46f78a989b92b19bbb1d~tplv-k3u1fbpfcp-zoom-1.image)

前言：  

临时使用数据库时可以使用docker运行，这样可以防止在系统上安装破坏环境，同时使用docker启动会比在系统中安装配置要快速，可以说是最快的方式安装部署并启动数据库。

  
  

* * *

**docker配置启动运行MySQL**  

首先创建目录并进入

```
sudo docker run -p 3306:3306 \\
--name mymysql \\
--restart=always \\
-v $PWD/conf:/etc/mysql/conf.d \\
-v $PWD/logs:/logs \\
-v $PWD/data:/var/lib/mysql \\
-e MYSQL_ROOT_PASSWORD=123456 \\
-d mysql:8
```

\--restart=always：在容器退出时总是重启容器

MYSQL_ROOT_PASSWORD=123456：root密码123456

mysql:8  使用MySQL8

\-v $PWD/conf:/etc/mysql/conf.d  配置文件

\-v $PWD/logs:/logs   日志

\-v $PWD/data:/var/lib/mysql    数据

  

* * *

**docker配置启动运行phpMyAdmin**

  

```
docker run -d \\
  -p 8001:80 \\
  -e UPLOAD_LIMIT=128M \\
  -e MAX_EXECUTION_TIME=10000 \\
  --name phpmyadmin \\
  phpmyadmin/phpmyadmin
```

UPLOAD_LIMIT 和 MAX_EXECUTION_TIME 需要设置一下

  

* * *

****docker配置启动运行**MongoDB**

```
docker run -d \\
  -p 27017:27017 \\
  -v mongo-data:/data/db \\
  -v mongo-config:/data/configdb \\
  --name mongo \\
  -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \\
  -e MONGO_INITDB_ROOT_PASSWORD=123123 \\
  -v /data:/mnt/data \\
  mongo
```

  

MONGO_INITDB_ROOT_USERNAME 用户名

MONGO_INITDB_ROOT_PASSWORD 密码

mongo-data 数据目录

mongo-config 配置文件目录

  

* * *

****docker配置启动运行**Mongo Express**

```
  docker run -d \\
  -p 8002:8081 \\
  --name mongo-express \\
  mongo-express
```

* * *

****docker配置启动运行**Redis**

```
docker run -d \\
  -p 6379:6379 \\
  -v redis-data:/data \\
  --name redis \\
  redis
```

* * *

  

****docker配置启动运行**Elasticsearch**

```
docker run -d \\
  -p 9100:9100 -p 9200:9200 \\
  -e discovery.type=single-node \\
  -v es-data:/usr/share/elasticsearch/data \\
  -v es-log:/usr/share/elasticsearch/logs \\
  --name elasticsearch \\
  elasticsearch
```

* * *

****docker配置启动运行**Grafana**

```
docker run -d \\
  -p 8003:3000 \\
  --link mysql:mysql \\
  --link mongo:mongo \\
  --name grafana \\
  grafana/grafana
```

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f8044e051f1458b8a62e9da38458826~tplv-k3u1fbpfcp-zoom-1.image)