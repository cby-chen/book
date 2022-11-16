---
layout: post
cid: 240
title: Kubernetes 部署 MySQL 高可用读写分离
slug: 240
date: 2022/06/08 10:34:00
updated: 2022/06/14 09:06:09
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: Kubernetes 部署 MySQL 高可用读写分离
keywords: 小陈运维,小陈,小陈博客,文档,
mode: default
thumb: 
video: 
---


# Kubernetes 部署 MySQL 集群



**简介：** 在有状态应用中，MySQL是我们最常见也是最常用的。本文我们将实战部署一个一组多从的MySQL集群。

## 一、配置准备



### configMap

```
cat > mysql-configmap.yaml << EOF 
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql
  labels:
    app: mysql
data:
  master.cnf: |
    # Apply this config only on the master.
    [mysqld]
    log-bin
  slave.cnf: |
    # Apply this config only on slaves.
    [mysqld]
    super-read-only
EOF
```

configMap可以将配置文件和镜像解耦开。
上面的配置意思是，创建一个master.cnf文件配置内容为：log-bin，即开启bin-log日志，供主节点使用。
创建一个slave.cnf文件配置内容为：super-read-only，设为该节点只读，供备用节点使用。



### service

```
cat > mysql-services.yaml << EOF 
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  ports:
  - name: mysql
    port: 3306
  clusterIP: None
  selector:
    app: mysql
---
# Client service for connecting to any MySQL instance for reads.
# For writes, you must instead connect to the master: mysql-0.mysql.
apiVersion: v1
kind: Service
metadata:
  name: mysql-read
  labels:
    app: mysql
spec:
  ports:
  - name: mysql
    port: 3306
  selector:
    app: mysql
EOF
```



### StatefulSet

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  serviceName: mysql
  replicas: 3
  template:
    metadata:
      labels:
        app: mysql
    spec:
      # 设置初始化容器，进行一些准备工作
      initContainers:
      - name: init-mysql
        image: mysql:5.7
        # 为每个MySQL节点配置service-id
        # 如果节点序号是0，则使用master的配置， 其余节点使用slave的配置
        command:
        - bash
        - "-c"
        - |
          set -ex
          # 基于 Pod 序号生成 MySQL 服务器的 ID。
          [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
          ordinal=${BASH_REMATCH[1]}
          echo [mysqld] > /mnt/conf.d/server-id.cnf
          # 添加偏移量以避免使用 server-id=0 这一保留值。
          echo server-id=$((100 + $ordinal)) >> /mnt/conf.d/server-id.cnf
          # Copy appropriate conf.d files from config-map to emptyDir.
          # 将合适的 conf.d 文件从 config-map 复制到 emptyDir。
          if [[ $ordinal -eq 0 ]]; then
            cp /mnt/config-map/master.cnf /mnt/conf.d/
          else
            cp /mnt/config-map/slave.cnf /mnt/conf.d/
          fi
        volumeMounts:
        - name: conf
          mountPath: /mnt/conf.d
        - name: config-map
          mountPath: /mnt/config-map
      - name: clone-mysql
        image: registry.cn-hangzhou.aliyuncs.com/chenby/xtrabackup:1.0
        # 为除了节点序号为0的主节点外的其它节点，备份前一个节点的数据
        command:
        - bash
        - "-c"
        - |
          set -ex
          # 如果已有数据，则跳过克隆。
          [[ -d /var/lib/mysql/mysql ]] && exit 0
          # 跳过主实例（序号索引 0）的克隆。
          [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
          ordinal=${BASH_REMATCH[1]}
          [[ $ordinal -eq 0 ]] && exit 0
          # 从原来的对等节点克隆数据。
          ncat --recv-only mysql-$(($ordinal-1)).mysql 3307 | xbstream -x -C /var/lib/mysql
          # 准备备份。
          xtrabackup --prepare --target-dir=/var/lib/mysql
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
          subPath: mysql
        - name: conf
          mountPath: /etc/mysql/conf.d
      containers:
      - name: mysql
        image: mysql:5.7
        # 设置支持免密登录
        env:
        - name: MYSQL_ALLOW_EMPTY_PASSWORD
          value: "1"
        ports:
        - name: mysql
          containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
          subPath: mysql
        - name: conf
          mountPath: /etc/mysql/conf.d
        resources:
          # 设置启动pod需要的资源，官方文档上需要500m cpu，1Gi memory。
          # 我本地测试的时候，会因为资源不足，报1 Insufficient cpu, 1 Insufficient memory错误，所以我改小了点
          requests:
            # m是千分之一的意思，100m表示需要0.1个cpu
            cpu: 1024m
            # Mi是兆的意思，需要100M 内存
            memory: 1Gi
        livenessProbe:
          # 使用mysqladmin ping命令，对MySQL节点进行探活检测
          # 在节点部署完30秒后开始，每10秒检测一次，超时时间为5秒
          exec:
            command: ["mysqladmin", "ping"]
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
        readinessProbe:
          # 对节点服务可用性进行检测， 启动5秒后开始，每2秒检测一次，超时时间1秒
          exec:
            # 检查我们是否可以通过 TCP 执行查询（skip-networking 是关闭的）。
            command: ["mysql", "-h", "127.0.0.1", "-e", "SELECT 1"]
          initialDelaySeconds: 5
          periodSeconds: 2
          timeoutSeconds: 1
      - name: xtrabackup
        image: registry.cn-hangzhou.aliyuncs.com/chenby/xtrabackup:1.0
        ports:
        - name: xtrabackup
          containerPort: 3307
        # 开始进行备份文件校验、解析和开始同步
        command:
        - bash
        - "-c"
        - |
          set -ex
          cd /var/lib/mysql
          # 确定克隆数据的 binlog 位置（如果有的话）。
          if [[ -f xtrabackup_slave_info && "x$(<xtrabackup_slave_info)" != "x" ]]; then
            # XtraBackup 已经生成了部分的 “CHANGE MASTER TO” 查询
            # 因为我们从一个现有副本进行克隆。(需要删除末尾的分号!)
            cat xtrabackup_slave_info | sed -E 's/;$//g' > change_master_to.sql.in
            # 在这里要忽略 xtrabackup_binlog_info （它是没用的）。
            rm -f xtrabackup_slave_info xtrabackup_binlog_info
          elif [[ -f xtrabackup_binlog_info ]]; then
            # 我们直接从主实例进行克隆。解析 binlog 位置。
            [[ `cat xtrabackup_binlog_info` =~ ^(.*?)[[:space:]]+(.*?)$ ]] || exit 1
            rm -f xtrabackup_binlog_info xtrabackup_slave_info
            echo "CHANGE MASTER TO MASTER_LOG_FILE='${BASH_REMATCH[1]}',\
                  MASTER_LOG_POS=${BASH_REMATCH[2]}" > change_master_to.sql.in
          fi
          # 检查我们是否需要通过启动复制来完成克隆。
          if [[ -f change_master_to.sql.in ]]; then
            echo "Waiting for mysqld to be ready (accepting connections)"
            until mysql -h 127.0.0.1 -e "SELECT 1"; do sleep 1; done
            echo "Initializing replication from clone position"
            mysql -h 127.0.0.1 \
                  -e "$(<change_master_to.sql.in), \
                          MASTER_HOST='mysql-0.mysql', \
                          MASTER_USER='root', \
                          MASTER_PASSWORD='', \
                          MASTER_CONNECT_RETRY=10; \
                        START SLAVE;" || exit 1
            # 如果容器重新启动，最多尝试一次。
            mv change_master_to.sql.in change_master_to.sql.orig
          fi
          # 当对等点请求时，启动服务器发送备份。
          exec ncat --listen --keep-open --send-only --max-conns=1 3307 -c \
            "xtrabackup --backup --slave-info --stream=xbstream --host=127.0.0.1 --user=root"
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
          subPath: mysql
        - name: conf
          mountPath: /etc/mysql/conf.d
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
      volumes:
      - name: conf
        emptyDir: {}
      - name: config-map
        configMap:
          name: mysql
  # 设置PVC
  volumeClaimTemplates:
  - metadata:
      name: data
      annotations:
      	# 配置PVC使用nfs动态供给
        volume.beta.kubernetes.io/storage-class: nfs-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```



## 二、创建所需资源

```
# 创建configMap
kubectl apply -f  mysql-configmap.yaml 
# 创建service
kubectl apply -f  mysql-services.yaml 
# 创建statefulSet
kubectl apply -f  mysql-statefulset.yaml

# 查看创建过程
kubectl get pods --watch

mysql-0                                   0/2     Pending   0             0s
mysql-0                                   0/2     Pending   0             0s
mysql-0                                   0/2     Init:0/2   0             0s
mysql-0                                   0/2     Init:0/2   0             1s
mysql-0                                   0/2     Init:1/2   0             2s
mysql-0                                   0/2     PodInitializing   0             3s
mysql-0                                   1/2     Running           0             4s
mysql-0                                   2/2     Running           0             8s
mysql-1                                   0/2     Pending           0             0s
mysql-1                                   0/2     Pending           0             0s
mysql-1                                   0/2     Init:0/2          0             0s
mysql-1                                   0/2     Init:0/2          0             1s
mysql-1                                   0/2     Init:1/2          0             1s
mysql-1                                   0/2     PodInitializing   0             2s
mysql-1                                   1/2     Running           0             3s
mysql-1                                   2/2     Running           0             8s
mysql-2                                   0/2     Pending           0             0s
mysql-2                                   0/2     Pending           0             0s
mysql-2                                   0/2     Init:0/2          0             0s
mysql-2                                   0/2     Init:0/2          0             1s
mysql-2                                   0/2     Init:1/2          0             2s
mysql-2                                   0/2     PodInitializing   0             3s
mysql-2                                   1/2     Running           0             4s
mysql-2                                   2/2     Running           0             9s
```



## 三、测试主库

### 进入pod进行操作

```
# 进入到pod mysql-0中，进行测试

kubectl exec -it mysql-0 bash

# 用mysql-client链接mysql-0

mysql -h mysql-0
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 276
Server version: 5.7.38-log MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```



### 创建库、表

```
# 创建数据库test
mysql> create database cby;
Query OK, 1 row affected (0.00 sec)

# 使用test库
mysql> use cby;
Database changed

# 创建message表
mysql> create table message (message varchar(50));
Query OK, 0 rows affected (0.01 sec)

# 查看message表结构
mysql> show create table message;
+---------+------------------------------------------------------------------------------------------------------+
| Table   | Create Table                                                                                         |
+---------+------------------------------------------------------------------------------------------------------+
| message | CREATE TABLE `message` (
  `message` varchar(50) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+---------+------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

```



### 插入数据

```
# 插入
mysql> insert into message value("hello chenby");
Query OK, 1 row affected (0.00 sec)

# 查看
mysql> select * from message;
+---------------+
| message       |
+---------------+
| hello chenby |
+---------------+
1 row in set (0.00 sec)

```



## 四、测试备库



### 连接mysql-1

```
 mysql -h mysql-1.mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 362
Server version: 5.7.38 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
mysql> 

```



###  查看库、表结构

```
# 查看数据库列表
mysql> show databases;
+------------------------+
| Database               |
+------------------------+
| information_schema     |
| cby                    |
| mysql                  |
| performance_schema     |
| sys                    |
| test                   |
| xtrabackup_backupfiles |
+------------------------+
7 rows in set (0.01 sec)

# 使用cby库
mysql> use cby;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> 

# 查看表列表
mysql> show tables;
+---------------+
| Tables_in_cby |
+---------------+
| message       |
+---------------+
1 row in set (0.00 sec)

# 查看message表结构
mysql> show create table message;
+---------+------------------------------------------------------------------------------------------------------+
| Table   | Create Table                                                                                         |
+---------+------------------------------------------------------------------------------------------------------+
| message | CREATE TABLE `message` (
  `message` varchar(50) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+---------+------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> 

# 查询数据
mysql> select * from message;
+---------------+
| message       |
+---------------+
| hello chenby |
+---------------+
1 row in set (0.00 sec)

mysql> 

# 写入数据
mysql> insert into message values("hello world");
ERROR 1290 (HY000): The MySQL server is running with the --super-read-only option so it cannot execute this statement
mysql> 
# 这是因为mysql-1是一个只读备库，无法进行写操作。
```



## 五、测试mysql-read服务



循环中运行 `SELECT @@server_id`

```
kubectl run mysql-client-loop --image=mysql:5.7 -i -t --rm --restart=Never --\
>   bash -ic "while sleep 1; do mysql -h mysql-read -e 'SELECT @@server_id,NOW()'; done"
If you don't see a command prompt, try pressing enter.
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         102 | 2022-06-07 09:52:19 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         101 | 2022-06-07 09:52:20 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         100 | 2022-06-07 09:52:21 |
+-------------+---------------------+

```



## 六、扩缩容

```
# 扩容至5副本
kubectl scale statefulset mysql  --replicas=5

# 查看扩容过程
kubectl get pods --watch
mysql-3                                   0/2     Pending             0             0s
mysql-3                                   0/2     Pending             0             1s
mysql-3                                   0/2     Pending             0             2s
mysql-3                                   0/2     Init:0/2            0             2s
mysql-3                                   0/2     Init:0/2            0             2s
mysql-3                                   0/2     Init:0/2            0             3s
mysql-3                                   0/2     Init:1/2            0             4s
mysql-3                                   0/2     Init:1/2            0             5s
mysql-3                                   0/2     PodInitializing     0             12s
mysql-3                                   1/2     Error               0             13s
mysql-3                                   1/2     Running             1 (2s ago)    14s
mysql-3                                   2/2     Running             1 (6s ago)    18s
mysql-4                                   0/2     Pending             0             0s
mysql-4                                   0/2     Pending             0             0s
mysql-4                                   0/2     Pending             0             2s
mysql-4                                   0/2     Init:0/2            0             2s
mysql-4                                   0/2     Init:0/2            0             2s
mysql-4                                   0/2     Init:1/2            0             3s
mysql-4                                   0/2     Init:1/2            0             4s
mysql-4                                   0/2     PodInitializing     0             12s
mysql-4                                   1/2     Error               0             13s
mysql-4                                   1/2     Running             1 (1s ago)    14s
mysql-4                                   2/2     Running             1 (7s ago)    20s


# 缩容只2副本
kubectl scale statefulset mysql  --replicas=2

# 查看缩容过程
kubectl get pods --watch
mysql-4                                   2/2     Terminating         1 (74s ago)   87s
mysql-4                                   2/2     Terminating         1 (104s ago)   117s
mysql-4                                   0/2     Terminating         1              118s
mysql-4                                   0/2     Terminating         1              118s
mysql-4                                   0/2     Terminating         1              118s
mysql-3                                   2/2     Terminating         1 (2m4s ago)   2m16s
mysql-3                                   2/2     Terminating         1 (2m34s ago)   2m46s
mysql-3                                   0/2     Terminating         1               2m47s
mysql-3                                   0/2     Terminating         1               2m47s
mysql-3                                   0/2     Terminating         1               2m47s
mysql-2                                   2/2     Terminating         0               16m
mysql-2                                   2/2     Terminating         0               16m
mysql-2                                   0/2     Terminating         0               16m
mysql-2                                   0/2     Terminating         0               16m
mysql-2                                   0/2     Terminating         0               16m
```



> https://www.oiox.cn/   
> https://www.chenby.cn/  
> https://cby-chen.github.io/  
> https://blog.csdn.net/qq_33921750  
> https://my.oschina.net/u/3981543  
> https://www.zhihu.com/people/chen-bu-yun-2  
> https://segmentfault.com/u/hppyvyv6/articles  
> https://juejin.cn/user/3315782802482007  
> https://cloud.tencent.com/developer/column/93230  
> https://www.jianshu.com/u/0f894314ae2c  
> https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/
>
> CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、今日头条、个人博客、全网可搜《小陈运维》
>
> 文章主要发布于微信：《Linux运维交流社区》

