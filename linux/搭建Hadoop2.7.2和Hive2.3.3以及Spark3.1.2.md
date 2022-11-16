---
layout: post
cid: 44
title: 搭建Hadoop2.7.2和Hive2.3.3以及Spark3.1.2
slug: 44
date: 2021/12/30 17:09:00
updated: 2022/03/25 15:47:41
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


**Hadoop 简介**

  

Hadoop是一个用Java编写的Apache开源框架，允许使用简单的编程模型跨计算机集群分布式处理大型数据集。Hadoop框架工作的应用程序在跨计算机集群提供分布式存储和计算的环境中工作。Hadoop旨在从单个服务器扩展到数千个机器，每个都提供本地计算和存储。

  

**Hive简介**

  

Apache Hive是一个构建于Hadoop顶层的数据仓库，可以将结构化的数据文件映射为一张数据库表，并提供简单的SQL查询功能，可以将SQL语句转换为MapReduce任务进行运行。需要注意的是，Hive它并不是数据库。

  

Hive依赖于HDFS和MapReduce，其对HDFS的操作类似于SQL，我们称之为HQL，它提供了丰富的SQL查询方式来分析存储在HDFS中的数据。HQL可以编译转为MapReduce作业，完成查询、汇总、分析数据等工作。这样一来，即使不熟悉MapReduce 的用户也可以很方便地利用SQL 语言查询、汇总、分析数据。而MapReduce开发人员可以把己写的mapper 和reducer 作为插件来支持Hive 做更复杂的数据分析。

  

  

**Apache Spark 简介**

  

用于大数据工作负载的分布式开源处理系统

  

Apache Spark 是一种用于大数据工作负载的分布式开源处理系统。它使用内存中缓存和优化的查询执行方式，可针对任何规模的数据进行快速分析查询。它提供使用 Java、Scala、Python 和 R 语言的开发 API，支持跨多个工作负载重用代码—批处理、交互式查询、实时分析、机器学习和图形处理等。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6afdfd92a4324cb1bfe1d8f726d59201~tplv-k3u1fbpfcp-zoom-1.image)

  

**本文将先搭建 jdk1.8 + MySQL5.7基础环境**

**之后搭建Hadoop2.7.2和Hive2.3.3以及Spark3.1.2**

**此文章搭建为单机版**  

  

**1.创建目录并解压jdk安装包**  

  

```
[root@localhost ~]# mkdir  jdk
[root@localhost ~]# cd jdk/
[root@localhost jdk]# ls
jdk-8u202-linux-x64.tar.gz
[root@localhost jdk]# ll
total 189496
-rw-r--r--. 1 root root 194042837 Oct 18 12:05 jdk-8u202-linux-x64.tar.gz
[root@localhost jdk]# 
[root@localhost jdk]# tar xvf jdk-8u202-linux-x64.tar.gz
```

  

**2.配置环境变量**  

  

```
[root@localhost ~]# vim /etc/profile
[root@localhost ~]# tail -n 3 /etc/profile
export JAVA_HOME=/root/jdk/jdk1.8.0_202/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
[root@localhost ~]# 
[root@localhost ~]# source /etc/profile
```

  

**3.下载安装MySQL并设置为开机自启**  

  

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
```

  

**4.查看MySQL默认密码，并修改默认密码，同时创建新的用户，将其设置为可以远程登录**  

  

```
[root@localhost mysql]# sudo grep 'temporary password' /var/log/mysqld.log
2021-10-18T06:12:35.519726Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: eNHu<sXHt3rq
[root@localhost mysql]# 
[root@localhost mysql]# 
[root@localhost mysql]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.25


Copyright (c) 2000, 2021, Oracle and/or its affiliates.


Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.


Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.


mysql> 
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Cby123..';
Query OK, 0 rows affected (0.02 sec)


mysql> 
mysql> 
mysql> use mysql;
Database changed
mysql> 
mysql> update user set host='%' where user ='root';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0


mysql> set global validate_password_policy=0;
Query OK, 0 rows affected (0.01 sec)


mysql> set global validate_password_mixed_case_count=0;
Query OK, 0 rows affected (0.00 sec)


mysql>  set global validate_password_number_count=3;
Query OK, 0 rows affected (0.00 sec)


mysql> set global validate_password_special_char_count=0;
Query OK, 0 rows affected (0.00 sec)


mysql> set global validate_password_length=3;
Query OK, 0 rows affected (0.00 sec)


mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password_check_user_name    | OFF   |
| validate_password_dictionary_file    |       |
| validate_password_length             | 3     |
| validate_password_mixed_case_count   | 0     |
| validate_password_number_count       | 3     |
| validate_password_policy             | LOW   |
| validate_password_special_char_count | 0     |
+--------------------------------------+-------+
7 rows in set (0.00 sec)


mysql> create user 'cby'@'%' identified by 'cby';
Query OK, 0 rows affected (0.00 sec)


mysql> grant all on *.* to 'cby'@'%';
Query OK, 0 rows affected (0.00 sec)


mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)


mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)


mysql> CREATE DATABASE dss_dev;
Query OK, 1 row affected (0.00 sec)


mysql> 
mysql> select host,user,plugin from user;
+-----------+---------------+-----------------------+
| host      | user          | plugin                |
+-----------+---------------+-----------------------+
| %         | root          | mysql_native_password |
| localhost | mysql.session | mysql_native_password |
| localhost | mysql.sys     | mysql_native_password |
+-----------+---------------+-----------------------+
3 rows in set (0.01 sec)




mysql>
```

  

  

**注：若上面root不是mysql_native_password使用以下命令将其改掉**

  

```
update user set plugin='mysql_native_password' where user='root';


```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/732afa3a5c5a4b23bc1fdfba94105597~tplv-k3u1fbpfcp-zoom-1.image)

  

**5.添加hosts解析，同时设置免密登录**

  

```
[root@localhost ~]# mkdir Hadoop
[root@localhost ~]# 
[root@localhost ~]# vim /etc/hosts
[root@localhost ~]# 
[root@localhost ~]# 
[root@localhost ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
127.0.0.1 namenode
[root@localhost ~]# ssh-keygen
[root@localhost ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@127.0.0.1
[root@localhost ~]#
```

  

**6.下载Hadoop，解压后创建所需目录**

  

```
[root@localhost ~]# cd Hadoop/
[root@localhost Hadoop]# ls
[root@localhost Hadoop]# wget https://archive.apache.org/dist/hadoop/core/hadoop-2.7.2/hadoop-2.7.2.tar.gz
[root@localhost Hadoop]# tar xvf hadoop-2.7.2.tar.gz 
[root@localhost Hadoop]# 
[root@localhost Hadoop]# mkdir  -p /root/Hadoop/hadoop-2.7.2/hadoopinfra/hdfs/namenode
[root@localhost Hadoop]# 
[root@localhost Hadoop]# 
[root@localhost Hadoop]# mkdir  -p /root/Hadoop/hadoop-2.7.2/hadoopinfra/hdfs/datanode
```

  

**7.添加Hadoop环境变量**

  

```
[root@localhost ~]# vim /etc/profile
[root@localhost ~]# tail -n 8 /etc/profile
export HADOOP_HOME=/root/Hadoop/hadoop-2.7.2/
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
[root@localhost ~]# 
[root@localhost ~]# source /etc/profile
```

  

**8.修改Hadoop配置**

  

```
[root@localhost ~]# cd Hadoop/hadoop-2.7.2/
[root@localhost hadoop]# vim /root/Hadoop/hadoop-2.7.2/etc/hadoop/core-site.xml
[root@localhost hadoop]# 
[root@localhost hadoop]# 
[root@localhost hadoop]# tail /root/Hadoop/hadoop-2.7.2/etc/hadoop/core-site.xml
<!-- Put site-specific property overrides in this file. -->


<configuration>
    <!-- 指定HDFS中NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://127.0.0.1:9000</value>
    </property>


    <!-- 指定Hadoop运行时产生文件的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/hadoop-2.7.2/data/tmp</value>
    </property>
</configuration>
```

  

**9.修改Hadoop的hdfs目录配置**

  

```
[root@localhost hadoop]# vim /root/Hadoop/hadoop-2.7.2/etc/hadoop/hdfs-site.xml 
[root@localhost hadoop]# 
[root@localhost hadoop]# 
[root@localhost hadoop]# 
[root@localhost hadoop]# tail -n 15  /root/Hadoop/hadoop-2.7.2/etc/hadoop/hdfs-site.xml 
<configuration>
   <property> 
      <name>dfs.replication</name> 
      <value>1</value> 
   </property> 
   <property> 
      <name>dfs.name.dir</name> 
      <value>/root/Hadoop/hadoop-2.7.2/hadoopinfra/hdfs/namenode</value> 
   </property> 
   <property> 
      <name>dfs.data.dir</name>
      <value>/root/Hadoop/hadoop-2.7.2/hadoopinfra/hdfs/datanode</value> 
   </property>
</configuration>
```

  

**10.修改Hadoop的yarn配置**

  

```
[root@localhost hadoop]# 
[root@localhost hadoop]# vim  /root/Hadoop/hadoop-2.7.2/etc/hadoop/yarn-site.xml
[root@localhost hadoop]# 
[root@localhost hadoop]# 
[root@localhost hadoop]# tail -n 6 /root/Hadoop/hadoop-2.7.2/etc/hadoop/yarn-site.xml
<configuration>


   <property> 
      <name>yarn.nodemanager.aux-services</name> 
      <value>mapreduce_shuffle</value> 
   </property>


</configuration>
```

  

```
[root@localhost hadoop]# 
[root@localhost hadoop]# cp /root/Hadoop/hadoop-2.7.2/etc/hadoop/mapred-site.xml.template /root/Hadoop/hadoop-2.7.2/etc/hadoop/mapred-site.xml
[root@localhost hadoop]# vim /root/Hadoop/hadoop-2.7.2/etc/hadoop/mapred-site.xml
[root@localhost hadoop]# 
[root@localhost hadoop]# 
[root@localhost hadoop]# tail /root/Hadoop/hadoop-2.7.2/etc/hadoop/mapred-site.xml
<configuration>
   <property> 
      <name>mapreduce.framework.name</name> 
      <value>yarn</value> 
   </property>
</configuration>
[root@localhost hadoop]#
```

  

**11.修改Hadoop环境配置文件**  

  

```
[root@localhost hadoop]# vim /root/Hadoop/hadoop-2.7.2/etc/hadoop/hadoop-env.sh


修改JAVA_HOME
export JAVA_HOME=/root/jdk/jdk1.8.0_202/


[root@localhost ~]# hdfs namenode -format
[root@localhost ~]# start-dfs.sh 
[root@localhost ~]# start-yarn.sh
```

  

  

**若重置太多次会导致clusterID不匹配，datanode起不来，删除版本后在初始化启动**

  

```
[root@localhost ~]# rm -rf /root/Hadoop/hadoop-2.7.2/hadoopinfra/hdfs/datanode/current/VERSION 
[root@localhost ~]# hadoop namenode -format
[root@localhost ~]# hdfs namenode -format
[root@localhost ~]# start-dfs.sh 
[root@localhost ~]# start-yarn.sh
```

  

**在浏览器访问Hadoop**

  

访问Hadoop的默认端口号为50070.使用以下网址，以获取浏览器Hadoop服务。

**http://localhost:50070/**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6e5d35c40ba4ab1824e48f88a081536~tplv-k3u1fbpfcp-zoom-1.image)

  

 **验证集群的所有应用程序**

  

访问集群中的所有应用程序的默认端口号为8088。使用以下URL访问该服务。

  

**http://localhost:8088/**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8ef9393ee3c422c8c1a3e1df487ef62~tplv-k3u1fbpfcp-zoom-1.image)

  

**12.创建hive目录并解压**

  

```
[root@localhost ~]# mkdir hive
[root@localhost ~]# cd hive
[root@localhost hive]# wget https://archive.apache.org/dist/hive/hive-2.3.3/apache-hive-2.3.3-bin.tar.gz
[root@localhost hive]# tar xvf apache-hive-2.3.3-bin.tar.gz 
[root@localhost hive]#
```

  

**13.备份hive配置文件**  

  

```
[root@localhost hive]# cd /root/hive/apache-hive-2.3.3-bin/conf/
[root@localhost conf]# cp hive-env.sh.template hive-env.sh
[root@localhost conf]# cp hive-default.xml.template hive-site.xml
[root@localhost conf]# cp hive-log4j2.properties.template hive-log4j2.properties
[root@localhost conf]# cp hive-exec-log4j2.properties.template hive-exec-log4j2.properties
```

  

**14.在Hadoop中创建文件夹并设置权限**  

  

```
[root@localhost conf]# hadoop fs -mkdir -p /data/hive/warehouse
21/10/18 14:27:03 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
[root@localhost conf]# 
[root@localhost conf]# hadoop fs -mkdir /data/hive/tmp
21/10/18 14:27:12 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
[root@localhost conf]# 
[root@localhost conf]# hadoop fs -mkdir /data/hive/log
21/10/18 14:27:18 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
[root@localhost conf]# 
[root@localhost conf]# hadoop fs -chmod -R 777 /data/hive/warehouse
21/10/18 14:27:49 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
[root@localhost conf]# hadoop fs -chmod -R 777 /data/hive/tmp
21/10/18 14:27:50 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
[root@localhost conf]# hadoop fs -chmod -R 777 /data/hive/log
21/10/18 14:27:51 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
[root@localhost conf]#
```

  

**15.修改hive配置文件**

  

```
[root@localhost conf]# vim hive-site.xml


hive 配置入下：
<property>
  <name>hive.exec.scratchdir</name>
  <value>hdfs://127.0.0.1:9000/data/hive/tmp</value>
</property>
<property>
   <name>hive.metastore.warehouse.dir</name>
  <value>hdfs://127.0.0.1:9000/data/hive/warehouse</value>
</property>
<property>
  <name>hive.querylog.location</name>
  <value>hdfs://127.0.0.1:9000/data/hive/log</value>
</property>


<!—该配置是关闭hive元数据版本认证，否则会在启动spark程序时报错-->
<property>
  <name>hive.metastore.schema.verification</name>
  <value>false</value>
</property>




配置mysql IP 端口以及放元数据的库名称


<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://127.0.0.1:3306/hive?createDatabaseIfNotExist=true</value>
</property>
<!—配置mysql启动器名称 -->
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
   <value>com.mysql.jdbc.Driver</value>
</property>
<!—配置连接mysql用户名 -->
<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>root</value>
</property>
<!—配置连接mysql用户名登录密码-->
<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>Cby123..</value>
</property>
```

  

  

  

**修改配置文件中 system:java.io.tmpdir 和 system:user.name 相关信息，改为实际目录和用户名，或者加入如下配置**

  

```
  <property>
    <name>system:java.io.tmpdir</name>
    <value>/tmp/hive/java</value>
  </property>
  <property>
    <name>system:user.name</name>
    <value>${user.name}</value>
  </property>
```

  

**并修改临时路径 ：**

  

```
 <property>
    <name>hive.exec.local.scratchdir</name>
    <value>/root/hive/apache-hive-2.3.3-bin/tmp/${system:user.name}</value>
    <description>Local scratch space for Hive jobs</description>
  </property>
  <property>
    <name>hive.downloaded.resources.dir</name>
    <value>/root/hive/apache-hive-2.3.3-bin/tmp/${hive.session.id}_resources</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>


<property>
    <name>hive.server2.logging.operation.log.location</name>
    <value>/root/hive/apache-hive-2.3.3-bin/tmp/root/operation_logs</value>
    <description>Top level directory where operation logs are stored if logging functionality is enabled</description>
  </property>
```

  

**16.配置hive中jdbc的MySQL驱动**  

  

```
[root@localhost lib]# cd /root/hive/apache-hive-2.3.3-bin/lib/
[root@localhost lib]# wget https://downloads.mysql.com/archives/get/p/3/file/mysql-connector-java-5.1.49.tar.gz
[root@localhost lib]# tar xvf mysql-connector-java-5.1.49.tar.gz 
[root@localhost lib]# cp mysql-connector-java-5.1.49/mysql-connector-java-5.1.49.jar .
[root@localhost bin]# 
[root@localhost bin]# vim /root/hive/apache-hive-2.3.3-bin/conf/hive-env.sh
[root@localhost bin]# tail -n 3 /root/hive/apache-hive-2.3.3-bin/conf/hive-env.sh
export HADOOP_HOME=/root/Hadoop/hadoop-2.7.2/
export HIVE_CONF_DIR=/root/hive/apache-hive-2.3.3-bin/conf
export HIVE_AUX_JARS_PATH=/root/hive/apache-hive-2.3.3-bin/lib
[root@localhost bin]#
```

  

**17.配置hive环境变量**

  

```
[root@localhost ~]# vim /etc/profile
[root@localhost ~]# tail -n 6 /etc/profile
export HADOOP_HOME=/root/Hadoop/hadoop-2.7.2/
export HIVE_CONF_DIR=/root/hive/apache-hive-2.3.3-bin/conf
export HIVE_AUX_JARS_PATH=/root/hive/apache-hive-2.3.3-bin/lib
export HIVE_PATH=/root/hive/apache-hive-2.3.3-bin/
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$HIVE_PATH/bin


[root@localhost bin]# ./schematool -dbType mysql -initSchema
```

  

  

**初始化完成后修改MySQL链接信息，之后配置mysql IP 端口以及放元数据的库名称**

  

```
[root@localhost conf]# vim hive-site.xml

<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://127.0.0.1:3306/hive?characterEncoding=utf8&amp;useSSL=false</value>
</property>

[root@localhost bin]# nohup hive --service metastore &
[root@localhost bin]# nohup hive --service hiveserver2 &
```

  

**18.创建spark目录并下载所需文件  
**

  

```
[root@localhost ~]# mkdir  spark
[root@localhost ~]# 
[root@localhost ~]# cd spark
[root@localhost spark]# 
[root@localhost spark]# wget https://dlcdn.apache.org/spark/spark-3.1.2/spark-3.1.2-bin-without-hadoop.tgz --no-check-certificate
[root@localhost spark]# tar xvf  spark-3.1.2-bin-without-hadoop.tgz
```

  

**19.配置spark环境变量以及备份配置文件**

  

```
[root@localhost ~]# vim /etc/profile
[root@localhost ~]# 
[root@localhost ~]# tail -n 3 /etc/profile
export SPARK_HOME=/root/spark/spark-3.1.2-bin-without-hadoop/
export PATH=$PATH:$SPARK_HOME/bin


[root@localhost spark]# cd /root/spark/spark-3.1.2-bin-without-hadoop/conf/
[root@localhost conf]# cp spark-env.sh.template spark-env.sh
[root@localhost conf]# cp spark-defaults.conf.template spark-defaults.conf
[root@localhost conf]# cp metrics.properties.template metrics.properties
```

  

**20.配置程序的环境变量**  

  

```
[root@localhost conf]# cp workers.template workers
[root@localhost conf]# vim spark-env.sh


export JAVA_HOME=/root/jdk/jdk1.8.0_202
export HADOOP_HOME=/root/Hadoop/hadoop-2.7.2
export HADOOP_CONF_DIR=/root/Hadoop/hadoop-2.7.2/etc/hadoop
export SPARK_DIST_CLASSPATH=$(/root/Hadoop/hadoop-2.7.2/bin/hadoop classpath)
export SPARK_MASTER_HOST=127.0.0.1
export SPARK_MASTER_PORT=7077
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -
Dspark.history.retainedApplications=50 -
Dspark.history.fs.logDirectory=hdfs://127.0.0.1:9000/spark-eventlog"
```

  

**21.修改默认的配置文件**  

  

```
[root@localhost conf]# vim spark-defaults.conf


spark.master                     spark://127.0.0.1:7077
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://127.0.0.1:9000/spark-eventlog
spark.serializer                 org.apache.spark.serializer.KryoSerializer
spark.driver.memory              3g
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://127.0.0.1:9000/spark-eventlog
spark.eventLog.compress          true
```

  

**22.配置工作节点**  

  

```
[root@localhost conf]# vim workers


[root@localhost conf]# cat workers
127.0.0.1


[root@localhost conf]# 


[root@localhost sbin]# /root/spark/spark-3.1.2-bin-without-hadoop/sbin/start-all.sh
```

  

 **验证应用程序**

  

访问集群中的所有应用程序的默认端口号为8080。使用以下URL访问该服务。

  

**http://localhost:8080/**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35679b2bcb784216a1ec18a153319e61~tplv-k3u1fbpfcp-zoom-1.image)

  

  

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