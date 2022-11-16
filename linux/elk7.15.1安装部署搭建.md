---
layout: post
cid: 43
title: elk7.15.1安装部署搭建
slug: 43
date: 2021/12/30 17:09:00
updated: 2022/03/25 15:47:06
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


**ELK简介**

ELK是Elasticsearch、Logstash、Kibana三大开源框架首字母大写简称（但是后期出现的Filebeat（beats中的一种）可以用来替代Logstash的数据收集功能，比较轻量级）。市面上也被成为Elastic Stack。

  

**Filebeat**是用于转发和集中日志数据的轻量级传送工具。Filebeat监视您指定的日志文件或位置，收集日志事件，并将它们转发到Elasticsearch或 Logstash进行索引。Filebeat的工作方式如下：启动Filebeat时，它将启动一个或多个输入，这些输入将在为日志数据指定的位置中查找。对于Filebeat所找到的每个日志，Filebeat都会启动收集器。每个收集器都读取单个日志以获取新内容，并将新日志数据发送到libbeat，libbeat将聚集事件，并将聚集的数据发送到为Filebeat配置的输出。

  

**Logstash**是免费且开放的服务器端数据处理管道，能够从多个来源采集数据，转换数据，然后将数据发送到您最喜欢的“存储库”中。Logstash能够动态地采集、转换和传输数据，不受格式或复杂度的影响。利用Grok从非结构化数据中派生出结构，从IP地址解码出地理坐标，匿名化或排除敏感字段，并简化整体处理过程。

  

**Elasticsearch**是Elastic Stack核心的分布式搜索和分析引擎，是一个基于Lucene、分布式、通过Restful方式进行交互的近实时搜索平台框架。Elasticsearch为所有类型的数据提供近乎实时的搜索和分析。无论您是结构化文本还是非结构化文本，数字数据或地理空间数据，Elasticsearch都能以支持快速搜索的方式有效地对其进行存储和索引。

  

**Kibana**是一个针对Elasticsearch的开源分析及可视化平台，用来搜索、查看交互存储在Elasticsearch索引中的数据。使用Kibana，可以通过各种图表进行高级数据分析及展示。并且可以为Logstash和ElasticSearch提供的日志分析友好的 Web 界面，可以汇总、分析和搜索重要数据日志。还可以让海量数据更容易理解。它操作简单，基于浏览器的用户界面可以快速创建仪表板（Dashboard）实时显示Elasticsearch查询动态

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04339a2911744c46b7fac726e2c764fa~tplv-k3u1fbpfcp-zoom-1.image)

  

完整日志系统基本特征

收集：能够采集多种来源的日志数据

传输：能够稳定的把日志数据解析过滤并传输到存储系统

存储：存储日志数据

分析：支持UI分析

警告：能够提供错误报告，监控机制

  

  

**安装jdk17环境**

  

```
root@elk:~# mkdir jdk
root@elk:~# cd jdk
root@elk:~/jdk# wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz
root@elk:~/jdk# tar xf jdk-17_linux-x64_bin.tar.gz
root@elk:~/jdk# cd ..
root@elk:~#
root@elk:~# mv jdk/ /
root@elk:~# vim /etc/profile
root@elk:~#
root@elk:~#
root@elk:~# tail -n 4 /etc/profile

export JAVA_HOME=/jdk/jdk-17.0.1/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
root@elk:~#
root@elk:~# source /etc/profile
root@elk:~# chmod -R 777  /jdk/
```

  

**创建elk文件夹，并下载所需包**

  

```
root@elk:~# mkdir elk
root@elk:~# cd elk
root@elk:~/elk# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.1-linux-x86_64.tar.gz
root@elk:~/elk# wget https://artifacts.elastic.co/downloads/kibana/kibana-7.15.1-linux-x86_64.tar.gz
root@elk:~/elk# wget https://artifacts.elastic.co/downloads/logstash/logstash-7.15.1-linux-x86_64.tar.gz
```

  

**解压安装包**

  

```
root@elk:~/elk# tar xf elasticsearch-7.15.1-linux-x86_64.tar.gz
root@elk:~/elk# tar xf kibana-7.15.1-linux-x86_64.tar.gz
root@elk:~/elk# tar xf logstash-7.15.1-linux-x86_64.tar.gz
root@elk:~/elk# ll
total 970288
drwxr-xr-x  5 root root      4096 Oct 20 06:09 ./
drwx------  7 root root      4096 Oct 20 06:04 ../
drwxr-xr-x  9 root root      4096 Oct  7 22:00 elasticsearch-7.15.1/
-rw-r--r--  1 root root 340849929 Oct 14 13:28 elasticsearch-7.15.1-linux-x86_64.tar.gz
drwxr-xr-x 10 root root      4096 Oct 20 06:09 kibana-7.15.1-linux-x86_64/
-rw-r--r--  1 root root 283752241 Oct 14 13:34 kibana-7.15.1-linux-x86_64.tar.gz
drwxr-xr-x 13 root root      4096 Oct 20 06:09 logstash-7.15.1/
-rw-r--r--  1 root root 368944379 Oct 14 13:38 logstash-7.15.1-linux-x86_64.tar.gz
```

**创建用户并设置权限**

  

```
root@elk:~/elk# cd
root@elk:~# useradd elk
root@elk:~# mkdir /home/elk
root@elk:~# cp -r elk/ /home/elk/
root@elk:~# chown -R elk:elk /home/elk/
```

  

**修改系统配置文件**

  

```
root@elk:~# vim /etc/security/limits.conf
root@elk:~#
root@elk:~#
root@elk:~# tail -n 3 /etc/security/limits.conf
*       soft    nofile          65536
*       hard    nofile          65536
root@elk:~#

root@elk:~# vim /etc/sysctl.conf
root@elk:~#
root@elk:~# tail -n 2 /etc/sysctl.conf

vm.max_map_count=262144
root@elk:~#
root@elk:~# sysctl -p
vm.max_map_count = 262144
root@elk:~#
```

**修改elk配置文件**

  

```
root@elk:~# su - elk
$ bash
elk@elk:~$ cd /elk/elasticsearch-7.15.1/config
elk@elk:~/elk/elasticsearch-7.15.1/config$ vim elasticsearch.yml
elk@elk:~/elk/elasticsearch-7.15.1/config$
elk@elk:~/elk/elasticsearch-7.15.1/config$ tail -n 20 elasticsearch.yml

#设置data存放的路径为/data/es-data
path.data: /home/elk/data/
#设置logs日志的路径为/log/es-log
path.logs: /home/elk/data/
#设置内存不使用交换分区
bootstrap.memory_lock: false
#配置了bootstrap.memory_lock为true时反而会引发9200不会被监听，原因不明
#设置允许所有ip可以连接该elasticsearch
network.host: 0.0.0.0
#开启监听的端口为9200
http.port: 9500
#增加新的参数，为了让elasticsearch-head插件可以访问es (5.x版本，如果没有可以自己手动加)
http.cors.enabled: true
http.cors.allow-origin: "*"
cluster.initial_master_nodes: ["elk"]
node.name: elk

root@elk:~/elk/elasticsearch-7.15.1/config#
```

  

**使用elk用户去启动elasticsearch**

  
  

```
root@elk:~# su - elk
$ bash
elk@elk:~$
elk@elk:~$ mkdir data
elk@elk:~/elk/elasticsearch-7.15.1/bin$ cd
elk@elk:~$ cd /home/elk/elk/elasticsearch-7.15.1/bin
elk@elk:~/elk/elasticsearch-7.15.1/bin$ ./elasticsearch
```

  

**启动之后访问测试：**

  

```
root@elk:~# curl -I http://192.168.1.19:9500/
HTTP/1.1 200 OK
X-elastic-product: Elasticsearch
Warning: 299 Elasticsearch-7.15.1-83c34f456ae29d60e94d886e455e6a3409bba9ed "Elasticsearch built-in security features are not enabled. Without authentication, your cluster could be accessible to anyone. See https://www.elastic.co/guide/en/elasticsearch/reference/7.15/security-minimal-setup.html to enable security."
content-type: application/json; charset=UTF-8
content-length: 532

root@elk:~#
```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92edd7720aa34cbda6c7acd791f88856~tplv-k3u1fbpfcp-zoom-1.image)

  

**放到后台运行**

  

```
elk@elk:~/elk/elasticsearch-7.15.1/bin$ nohup /home/elk/elk/elasticsearch-7.15.1/bin/elasticsearch >> /home/elk/elk/elasticsearch-7.15.1/output.log 2>&1 &
[1] 8811
elk@elk:~/elk/elasticsearch-7.15.1/bin$
```

  

```
elk@elk:~$ cd elk/kibana-7.15.1-linux-x86_64/config/
elk@elk:~/elk/kibana-7.15.1-linux-x86_64/config$ vim kibana.yml
elk@elk:~/elk/kibana-7.15.1-linux-x86_64/config$ tail -n 18 kibana.yml
#设置监听端口为5601
server.port: 5601
#设置可访问的主机地址
server.host: "0.0.0.0"
#设置elasticsearch主机地址
elasticsearch.hosts: ["http://localhost:9500"]
#如果elasticsearch设置了用户名密码,那么需要配置该两项,如果没配置,那就不用管
#elasticsearch.username: "user"
#elasticsearch.password: "pass"
elk@elk:~/elk/kibana-7.15.1-linux-x86_64/config$

elk@elk:~$ cd /home/elk/elk/kibana-7.15.1-linux-x86_64/bin

elk@elk:~/elk/kibana-7.15.1-linux-x86_64/bin$ ./kibana
```

**测试访问**

  

```
root@elk:~# curl -I http://192.168.1.19:5601/app/home#/tutorial_directory
HTTP/1.1 200 OK
content-security-policy: script-src 'unsafe-eval' 'self'; worker-src blob: 'self'; style-src 'unsafe-inline' 'self'
x-content-type-options: nosniff
referrer-policy: no-referrer-when-downgrade
kbn-name: elk
kbn-license-sig: aaa69ea6a0792153cde61e88d0cd9bbad7ddcdaec87b613f281dd275e9dbad47
content-type: text/html; charset=utf-8
cache-control: private, no-cache, no-store, must-revalidate
content-length: 144351
vary: accept-encoding
Date: Wed, 20 Oct 2021 07:11:10 GMT
Connection: keep-alive
Keep-Alive: timeout=120
root@elk:~#
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d77de83c3a84101857b57d6ebbe1199~tplv-k3u1fbpfcp-zoom-1.image)

  

**放到后台运行**

  

```
elk@elk:~/elk/kibana-7.15.1-linux-x86_64/bin$ nohup /home/elk/elk/kibana-7.15.1-linux-x86_64/bin/kibana >> /home/elk/elk/kibana-7.15.1-linux-x86_64/output.log 2>&1 &
[2] 9378
elk@elk:~/elk/kibana-7.15.1-linux-x86_64/bin$
```

  

**将日志信息输出到屏幕上**

  

```
elk@elk:~$ cd elk/logstash-7.15.1/bin/
elk@elk:~/elk/logstash-7.15.1/bin$ ./logstash -e 'input {stdin{}} output{stdout{}}'

输入个123然后回车,会把结果输出到屏幕上

{
          "host" => "elk",
    "@timestamp" => 2021-10-20T07:15:54.230Z,
      "@version" => "1",
       "message" => ""
}
123
{
          "host" => "elk",
    "@timestamp" => 2021-10-20T07:15:56.453Z,
      "@version" => "1",
       "message" => "123"
}

elk@elk:~/elk/logstash-7.15.1/bin$ cd ../config/
elk@elk:~/elk/logstash-7.15.1/config$ vim logstash
elk@elk:~/elk/logstash-7.15.1/config$ cat logstash
input {
    # 从文件读取日志信息
      file {
          path => "/var/log/messages"
          type => "system"
          start_position => "beginning"
           }
}

filter {
}

output {
      # 标准输出
      stdout {}
}
elk@elk:~/elk/logstash-7.15.1/config$ mv logstash logstash.conf
elk@elk:~/elk/logstash-7.15.1/config$
```

  

**启动测试**

  

```
elk@elk:~/elk/logstash-7.15.1/config$ cd ../bin/
elk@elk:~/elk/logstash-7.15.1/bin$ ./logstash -f ../config/logstash.conf
```

  
  

**后台启动**

  

```
elk@elk:~$ nohup /home/elk/elk/logstash-7.15.1/bin/logstash -f /home/elk/elk/logstash-7.15.1/config/logstash.conf >> /home/elk/elk/logstash-7.15.1/output.log 2>&1 &
[3] 10177
elk@elk:~$
```

  
  

**设置开机自启**

  

```
elk@elk:~$ vim startup.sh
elk@elk:~$
elk@elk:~$ cat startup.sh
#!/bin/bash
nohup /home/elk/elk/elasticsearch-7.15.1/bin/elasticsearch >> /home/elk/elk/elasticsearch-7.15.1/output.log 2>&1 &
nohup /home/elk/elk/kibana-7.15.1-linux-x86_64/bin/kibana >> /home/elk/elk/kibana-7.15.1-linux-x86_64/output.log 2>&1 &
nohup /home/elk/elk/logstash-7.15.1/bin/logstash -f /home/elk/elk/logstash-7.15.1/config/logstash.conf >> /home/elk/elk/logstash-7.15.1/output.log 2>&1 &

elk@elk:~$
elk@elk:~$ crontab -e
no crontab for elk - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]: 2
crontab: installing new crontab
elk@elk:~$
elk@elk:~$
elk@elk:~$ crontab -l

@reboot /home/elk/startup.sh

elk@elk:~$
```

  

**logstash插件**

logstash是通过插件对其功能进行加强

  

插件分类:

  

inputs 输入

codecs 解码

filters 过滤

outputs 输出

在Gemfile文件里记录了logstash的插件

  

```
elk@elk:~$ cd elk/logstash-7.15.1
elk@elk:~/elk/logstash-7.15.1$ ls Gemfile
Gemfile
elk@elk:~/elk/logstash-7.15.1$
```

  
  

去其github上下载插件,地址为:**https://github.com/logstash-plugins**

  
  

使用filter插件logstash-filter-mutate

  

```
elk@elk:~/elk/logstash-7.15.1/config$  vim logstash2.conf
#创建一个新的配置文件用来过滤
input {
    stdin {
    }
}
filter {
   mutate {
        split => ["message", "|"]
    }
}
output {
    stdout {
    }
}
```

  

当输入sss|sssni|akok223|23即会按照|分隔符进行分隔

  

其数据处理流程:input–>解码–>filter–>解码–>output

  

**启动服务**

然后去启动logstash服务

  

```shell
elk@elk:~$ nohup /home/elk/elk/logstash-7.15.1/bin/logstash -f /home/elk/elk/logstash-7.15.1/config/logstash2.conf >> /home/elk/elk/logstash-7.15.1/output.log 2>&1 &

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