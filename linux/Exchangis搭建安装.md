---
layout: post
cid: 41
title: Exchangis搭建安装
slug: 41
date: 2021/12/30 17:09:00
updated: 2022/03/25 15:46:11
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


**项目简介**

Exchangis是一个轻量级的、高扩展性的数据交换平台，支持对结构化及无结构化的异构数据源之间的数据传输，在应用层上具有数据权限管控、节点服务高可用和多租户资源隔离等业务特性，而在数据层上又具有传输架构多样化、模块插件化和组件低耦合等架构特点。

  

Exchangis的传输交换能力依赖于其底层聚合的传输引擎，其顶层对各类数据源定义统一的参数模型，每种传输引擎对参数模型进行映射配置，转化为引擎的输入模型。每聚合一种引擎，都将增加Exchangis一类特性，对某类引擎的特性强化，都是对Exchangis特性的完善。默认聚合以及强化Alibaba的DataX传输引擎。

  

**核心特点**

数据源管理

以绑定项目的方式共享自己的数据源；

设置数据源对外权限，控制数据的流入和流出。

  

**多传输引擎支持**

传输引擎可横向扩展；

当前版本完整聚合了离线批量引擎DataX、部分聚合了大数据批量导数引擎SQOOP

  

**近实时任务管控**

快速抓取传输任务日志以及传输速率等信息，实时关闭任务；

可根据带宽状况对任务进行动态限流

  

**支持无结构化传输**

DataX框架改造，单独构建二进制流快速通道，适用于无数据转换的纯数据同步场景。

  

**任务状态自检**

监控长时间运行的任务和状态异常任务，及时释放占用的资源并发出告警。

  

**架构设计**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d0966aca5e64bd880036b4efa5bc2b3~tplv-k3u1fbpfcp-zoom-1.image)

  

  

**环境准备**

  

**基础软件安装**

  

MySQL (5.5+) 必选，对应客户端可以选装, Linux服务上若安装mysql的客户端可以通过部署脚本快速初始化数据库

JDK (1.8.0_141) 必选

Maven (3.6.1+) 必选

SQOOP (1.4.6) 可选，如果想要SQOOP做传输引擎，可以安装SQOOP，SQOOP安装依赖Hive,Hadoop环境，这里就不展开来讲

Python (2.x) 可选，主要用于调度执行底层DataX的启动脚本，默认的方式是以Java子进程方式执行DataX，用户可以选择以Python方式来做自定义的改造

  
  

**mysql 数据库安装**

  
  

```
[root@localhost ~]# mkdir mysql
[root@localhost ~]# cd mysql
[root@localhost mysql]# wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.35-1.el7.x86_64.rpm-bundle.tar
[root@localhost mysql]# tar xvf mysql-5.7.35-1.el7.x86_64.rpm-bundle.tar
[root@localhost mysql]# yum install ./*.rpm
[root@localhost mysql]# systemctl start mysqld.service
[root@localhost mysql]#
[root@localhost mysql]# systemctl enable mysqld.service
[root@localhost mysql]#
[root@localhost mysql]#
[root@localhost mysql]# sudo grep 'temporary password' /var/log/mysqld.log
2021-10-25T06:57:46.569037Z 1 [Note] A temporary password is generated for root@localhost: (l5aFfIxfNuu
[root@localhost mysql]#
[root@localhost mysql]#
[root@localhost mysql]#
[root@localhost mysql]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.35 MySQL Community Server (GPL)


Copyright (c) 2000, 2021, Oracle and/or its affiliates.


Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.


Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.


mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Cby123..';
Query OK, 0 rows affected (0.00 sec)


mysql>
mysql>
mysql>
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A


Database changed
mysql>
mysql>
mysql> update user set host='%' where user ='root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0


mysql> set global validate_password_policy=0;
Query OK, 0 rows affected (0.00 sec)


mysql> set global validate_password_mixed_case_count=0;
Query OK, 0 rows affected (0.01 sec)


mysql> set global validate_password_number_count=3;
Query OK, 0 rows affected (0.00 sec)


mysql> set global validate_password_special_char_count=0;
Query OK, 0 rows affected (0.00 sec)


mysql> set global validate_password_length=3;
Query OK, 0 rows affected (0.00 sec)


mysql>
```

  

  

**jdk安装**

  
  

```
[root@localhost ~]# mkdir jdk
[root@localhost ~]# cd jdk
[root@localhost jdk]#
[root@localhost jdk]# tar xf jdk-8u141-linux-x64.tar.gz
[root@localhost jdk]#
[root@localhost jdk]# ll
total 181172
drwxr-xr-x. 8   10  143       255 Jul 12  2017 jdk1.8.0_141
-rw-r--r--. 1 root root 185516505 Jul 25  2017 jdk-8u141-linux-x64.tar.gz
[root@localhost jdk]#
[root@localhost jdk]# vim /etc/profile
[root@localhost jdk]#  tail -n 3 /etc/profile
export JAVA_HOME=/root/jdk/jdk1.8.0_141/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
[root@localhost jdk]#
[root@localhost jdk]# source /etc/profile
[root@localhost jdk]# java -version
java version "1.8.0_141"
Java(TM) SE Runtime Environment (build 1.8.0_141-b15)
Java HotSpot(TM) 64-Bit Server VM (build 25.141-b15, mixed mode)
[root@localhost jdk]#
```

  

  

maven 安装

  

```
[root@localhost ~]#
[root@localhost ~]# mkdir maven
[root@localhost ~]# cd maven
[root@localhost maven]# wget https://archive.apache.org/dist/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz
[root@localhost maven]#
[root@localhost maven]# tar xf apache-maven-3.6.1-bin.tar.gz
[root@localhost maven]# ll
total 8924
drwxr-xr-x. 6 root root      99 Oct 25 15:08 apache-maven-3.6.1
-rw-r--r--. 1 root root 9136463 Sep  4  2019 apache-maven-3.6.1-bin.tar.gz
[root@localhost maven]# vim /etc/profile
[root@localhost maven]#
[root@localhost maven]# tail -n 3 /etc/profile


export MAVEN_HOME=/root/maven/apache-maven-3.6.1
export PATH=$MAVEN_HOME/bin:$PATH:$HOME/bin
[root@localhost maven]#
[root@localhost maven]# source /etc/profile
[root@localhost maven]# mvn -version
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
Maven home: /root/maven/apache-maven-3.6.1
Java version: 1.8.0_141, vendor: Oracle Corporation, runtime: /root/jdk/jdk1.8.0_141/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-1160.31.1.el7.x86_64", arch: "amd64", family: "unix"
[root@localhost maven]#
```

  

Exchangis安装

  
```
[root@localhost ~]#
[root@localhost ~]# mkdir  Exchangis
[root@localhost ~]# cd Exchangis
[root@localhost Exchangis]# wget https://github.com/WeBankFinTech/Exchangis/releases/download/release-0.5.0/wedatasphere-exchangis-0.5.0.RELEASE.tar.gz
[root@localhost Exchangis]# ll
total 552904
-rw-r--r--. 1 root root 566172217 Oct 25 15:14 wedatasphere-exchangis-0.5.0.RELEASE.tar.gz
[root@localhost Exchangis]# tar xf wedatasphere-exchangis-0.5.0.RELEASE.tar.gz
[root@localhost Exchangis]# ll
total 552904
drwxr-xr-x. 6 root root        91 Oct 25 15:14 wedatasphere-exchangis-0.5.0.RELEASE
-rw-r--r--. 1 root root 566172217 Oct 25 15:14 wedatasphere-exchangis-0.5.0.RELEASE.tar.gz
[root@localhost Exchangis]# cd wedatasphere-exchangis-0.5.0.RELEASE
[root@localhost wedatasphere-exchangis-0.5.0.RELEASE]# ll
total 20
drwxrwxrwx. 2 root root   120 Oct 29  2020 bin
drwxrwxrwx. 4 root root    32 May 12  2020 docs
drwxrwxrwx. 4 root root    57 May 12  2020 images
-rwxrwxrwx. 1 root root 11357 Oct 29  2020 LICENSE
drwxr-xr-x. 2 root root   198 Oct 25 15:14 packages
-rwxrwxrwx. 1 root root  4582 Oct 29  2020 README.md
[root@localhost wedatasphere-exchangis-0.5.0.RELEASE]# cd bin/
[root@localhost bin]# ./install.sh
2021-10-25 15:16:19.723 [INFO] (12476) Creating directory: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/bin/../modules].
2021-10-25 15:16:19.728 [INFO] (12476)  ####### Start To Uncompress Packages ######
2021-10-25 15:16:19.730 [INFO] (12476) Uncompressing....
Do you want to decompress this package: [exchangis-eureka_0.5.0.RELEASE_1.tar.gz]? (Y/N)y
2021-10-25 15:16:22.691 [INFO] (12476)  Uncompress package: [exchangis-eureka_0.5.0.RELEASE_1.tar.gz] to modules directory
Do you want to decompress this package: [exchangis-executor_0.5.0.RELEASE_1.tar.gz]? (Y/N)y
2021-10-25 15:16:24.798 [INFO] (12476)  Uncompress package: [exchangis-executor_0.5.0.RELEASE_1.tar.gz] to modules directory
Do you want to decompress this package: [exchangis-gateway_0.5.0.RELEASE_1.tar.gz]? (Y/N)y
2021-10-25 15:16:31.947 [INFO] (12476)  Uncompress package: [exchangis-gateway_0.5.0.RELEASE_1.tar.gz] to modules directory
Do you want to decompress this package: [exchangis-service_0.5.0.RELEASE_1.tar.gz]? (Y/N)y
2021-10-25 15:16:35.029 [INFO] (12476)  Uncompress package: [exchangis-service_0.5.0.RELEASE_1.tar.gz] to modules directory
2021-10-25 15:16:36.537 [INFO] (12476)  ####### Finish To Umcompress Packages ######
Scan modules directory: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/bin/../modules] to find server under exchangis
2021-10-25 15:16:36.542 [INFO] (12476)  ####### Start To Install Modules ######
2021-10-25 15:16:36.545 [INFO] (12476) Module servers could be installed:
 [exchangis-eureka]  [exchangis-executor]  [exchangis-gateway]  [exchangis-service]
Do you want to confiugre and install [exchangis-eureka]? (Y/N)y
2021-10-25 15:16:37.676 [INFO] (12476)  Install module server: [exchangis-eureka]
2021-10-25 15:16:37.706 [INFO] (12527)  Start to build directory
2021-10-25 15:16:37.709 [INFO] (12527) Creating directory: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-eureka/bin/../logs].
2021-10-25 15:16:37.779 [INFO] (12527) Directory or file: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-eureka/bin/../conf] has been exist
2021-10-25 15:16:37.782 [INFO] (12527) Creating directory: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-eureka/bin/../data].
Do you want to confiugre and install [exchangis-executor]? (Y/N)y
2021-10-25 15:16:38.529 [INFO] (12476)  Install module server: [exchangis-executor]
2021-10-25 15:16:38.558 [INFO] (12565)  Start to build directory
2021-10-25 15:16:38.561 [INFO] (12565) Creating directory: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-executor/bin/../logs].
2021-10-25 15:16:38.596 [INFO] (12565) Directory or file: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-executor/bin/../conf] has been exist
2021-10-25 15:16:38.599 [INFO] (12565) Creating directory: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-executor/bin/../data].
Do you want to confiugre and install [exchangis-gateway]? (Y/N)y
2021-10-25 15:16:39.291 [INFO] (12476)  Install module server: [exchangis-gateway]
2021-10-25 15:16:39.317 [INFO] (12603)  Start to build directory
2021-10-25 15:16:39.320 [INFO] (12603) Creating directory: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-gateway/bin/../logs].
2021-10-25 15:16:39.354 [INFO] (12603) Directory or file: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-gateway/bin/../conf] has been exist
2021-10-25 15:16:39.356 [INFO] (12603) Creating directory: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-gateway/bin/../data].
Do you want to confiugre and install [exchangis-service]? (Y/N)y
2021-10-25 15:16:39.991 [INFO] (12476)  Install module server: [exchangis-service]
2021-10-25 15:16:40.017 [INFO] (12641)  Start to build directory
2021-10-25 15:16:40.020 [INFO] (12641) Creating directory: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-service/bin/../logs].
2021-10-25 15:16:40.056 [INFO] (12641) Directory or file: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-service/bin/../conf] has been exist
2021-10-25 15:16:40.059 [INFO] (12641) Creating directory: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/modules/exchangis-service/bin/../data].
2021-10-25 15:16:40.099 [INFO] (12641)  Scan out mysql command, so begin to initalize the database
Do you want to initalize database with sql: [/root/Exchangis/wedatasphere-exchangis-0.5.0.RELEASE/bin/exchangis-init.sql]? (Y/N)y
Please input the db host(default: 127.0.0.1):
Please input the db port(default: 3306):
Please input the db username(default: root):
Please input the db password(default: ): Cby123..
Please input the db name(default: exchangis)
mysql: [Warning] Using a password on the command line interface can be insecure.
2021-10-25 15:16:55.665 [INFO] (12476)  ####### Finish To Install Modules ######
[root@localhost bin]#



[root@localhost bin]# ./start-all.sh
2021-10-25 15:18:22.181 [INFO] (12691)  Try To Start Modules In Order
2021-10-25 15:18:22.189 [INFO] (12699)  ####### Begin To Start Module: [exchangis-eureka] ######
2021-10-25 15:18:22.199 [INFO] (12707) load environment variables
2021-10-25 15:18:22.717 [INFO] (12707) /root/jdk/jdk1.8.0_141//bin/java
2021-10-25 15:18:22.721 [INFO] (12707) Waiting EXCHANGIS-EUREKA to start complete ...
2021-10-25 15:18:22.994 [INFO] (12707) EXCHANGIS-EUREKA start success
2021-10-25 15:18:23.003 [INFO] (13009)  ####### Begin To Start Module: [exchangis-gateway] ######
2021-10-25 15:18:23.012 [INFO] (13017) load environment variables
2021-10-25 15:18:23.493 [INFO] (13017) /root/jdk/jdk1.8.0_141//bin/java
2021-10-25 15:18:23.497 [INFO] (13017) Waiting EXCHANGIS-GATEWAY to start complete ...
2021-10-25 15:18:24.081 [INFO] (13017) EXCHANGIS-GATEWAY start success
2021-10-25 15:18:24.091 [INFO] (13321)  ####### Begin To Start Module: [exchangis-service] ######
2021-10-25 15:18:24.099 [INFO] (13329) load environment variables
2021-10-25 15:18:24.933 [INFO] (13329) /root/jdk/jdk1.8.0_141//bin/java
2021-10-25 15:18:24.936 [INFO] (13329) Waiting EXCHANGIS-SERVICE to start complete ...
2021-10-25 15:18:26.398 [INFO] (13329) EXCHANGIS-SERVICE start success
2021-10-25 15:18:26.410 [INFO] (13634)  ####### Begin To Start Module: [exchangis-executor] ######
2021-10-25 15:18:26.423 [INFO] (13643) load environment variables
2021-10-25 15:18:27.677 [INFO] (13643) /root/jdk/jdk1.8.0_141//bin/java
2021-10-25 15:18:27.681 [INFO] (13643) Waiting EXCHANGIS-EXECUTOR to start complete ...
2021-10-25 15:18:28.441 [INFO] (13643) EXCHANGIS-EXECUTOR start success
[root@localhost bin]#
```

  

  
**登陆访问  
**  

注册中心：http://192.168.1.161:8500/

访问地址：http://192.168.1.161:9503/

账号：admin

密码：admin

  

  

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