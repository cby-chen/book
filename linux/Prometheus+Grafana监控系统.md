---
layout: post
cid: 26
title: Prometheus+Grafana监控系统
slug: 26
date: 2021/12/30 17:03:00
updated: 2022/11/11 21:36:43
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


### Prometheus vs Zabbix

    Zabbix的客户端更多是只做上报的事情，push模式。而Prometheus则是客户端本地也会存储监控数据，服务端定时来拉取想要的数据。

  

 Zabbix的客户端agent可以比较方便的通过脚本来读取机器内数据库、日志等文件来做上报。zabbix的客户端agent可以比较方便的通过脚本来读取机器内数据库、日志等文件来做上报。Prometheus的上报客户端则分为不同语言的SDK和不同用途的exporter两种，比如如果你要监控机器状态、mysql性能等，有大量已经成熟的exporter来直接开箱使用，通过http通信来对服务端提供信息上报（server去pull信息）；

  

Zabbix's client is more of only reporting things, push mode. In Prometheus, the client also stores monitoring data locally, and the server regularly pulls the desired data.

  

  

     Zabbix's client agent can easily read the database, log and other files in the machine through scripts for reporting. The zabbix client agent can easily read the database, log and other files in the machine through scripts for reporting. Prometheus reporting clients are divided into SDKs in different languages and exporters for different purposes. For example, if you want to monitor machine status, mysql performance, etc., there are a large number of mature exporters to use directly out of the box, and serve through HTTP communication. The terminal provides information reporting (server to pull information);

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1528e9872de49549f77699874d9de9a~tplv-k3u1fbpfcp-zoom-1.image)

### 安装Prometheus：

**install Prometheus**

  

  

    官网下载地址：

Official website download address

```
https://prometheus.io/download/
```

  

    下载您想要的版本后，进行安装使用即可。

After downloading the version you want, install it and use it

```
cby@cby-Inspiron-7577:~$ wget https://github.com/prometheus/prometheus/releases/download/v2.21.0/prometheus-2.21.0.linux-amd64.tar.gz
cby@cby-Inspiron-7577:~$ tar xvf prometheus-2.21.0.linux-amd64.tar.gz 
prometheus-2.21.0.linux-amd64/
prometheus-2.21.0.linux-amd64/LICENSE
prometheus-2.21.0.linux-amd64/prometheus
prometheus-2.21.0.linux-amd64/promtool
prometheus-2.21.0.linux-amd64/prometheus.yml
prometheus-2.21.0.linux-amd64/NOTICE
prometheus-2.21.0.linux-amd64/console_libraries/
prometheus-2.21.0.linux-amd64/console_libraries/menu.lib
prometheus-2.21.0.linux-amd64/console_libraries/prom.lib
prometheus-2.21.0.linux-amd64/consoles/
prometheus-2.21.0.linux-amd64/consoles/node-overview.html
prometheus-2.21.0.linux-amd64/consoles/node.html
prometheus-2.21.0.linux-amd64/consoles/prometheus.html
prometheus-2.21.0.linux-amd64/consoles/node-cpu.html
prometheus-2.21.0.linux-amd64/consoles/index.html.example
prometheus-2.21.0.linux-amd64/consoles/prometheus-overview.html
prometheus-2.21.0.linux-amd64/consoles/node-disk.html
```

  

    解压后进入文件夹内即可看到该程序。同时即可使用。

After decompression, enter the folder to see the program. Can be used at the same time

```
cby@cby-Inspiron-7577:~/prometheus-2.21.0.linux-amd64$ ll
总用量 161140
drwxr-xr-x  4 cby cby     4096 9月  11 21:30 ./
drwxr-xr-x 22 cby cby     4096 10月  5 00:18 ../
drwxr-xr-x  2 cby cby     4096 9月  11 21:29 console_libraries/
drwxr-xr-x  2 cby cby     4096 9月  11 21:29 consoles/
-rw-r--r--  1 cby cby    11357 9月  11 21:29 LICENSE
-rw-r--r--  1 cby cby     3420 9月  11 21:29 NOTICE
-rwxr-xr-x  1 cby cby 88471209 9月  11 19:37 prometheus\*
-rw-r--r--  1 cby cby      926 9月  11 21:29 prometheus.yml
-rwxr-xr-x  1 cby cby 76493104 9月  11 19:39 promtool\*

```

  

    查看一下版本：  

Check the version

```
cby@cby-Inspiron-7577:~/prometheus-2.21.0.linux-amd64$ ./prometheus --version
prometheus, version 2.21.0 (branch: HEAD, revision: e83ef207b6c2398919b69cd87d2693cfc2fb4127)
  build user:       root@a4d9bea8479e
  build date:       20200911-11:35:02
  go version:       go1.15.2
```

  

    查看启动sever文件：  

View startup sever file

```
cby@cby-Inspiron-7577:~/prometheus-2.21.0.linux-amd64$ cat prometheus.yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label \`job=<job_name>\` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: \['localhost:9090'\]
```

  

    其大致分为四部分：

It is roughly divided into four parts:

*   global：全局配置，其中scrape_interval表示抓取一次数据的间隔时间，evaluation_interval表示进行告警规则检测的间隔时间；
    
*   global: global configuration, in which scrape_interval represents the interval of data capture, evaluation_interval represents the interval of alarm rule detection;
    
*     
    
*   alerting：告警管理器（Alertmanager）的配置，目前还没有安装Alertmanager；
    
*   alerting: The configuration of the alert manager (Alertmanager), Alertmanager is not installed yet;
    
*     
    
*   rule_files：告警规则有哪些；
    
*   rule_files: what are the alarm rules;
    
*     
    
*   scrape_configs：抓取监控信息的目标。一个job_name就是一个目标，其targets就是采集信息的IP和端口。这里默认监控了Prometheus自己，可以通过修改这里来修改Prometheus的监控端口。Prometheus的每个exporter都会是一个目标，它们可以上报不同的监控信息，比如机器状态，或者mysql性能等等，不同语言sdk也会是一个目标，它们会上报你自定义的业务监控信息。
    
*   scrape_configs: The goal of grabbing monitoring information. A job_name is a target, and its targets are the IP and port for collecting information. Prometheus itself is monitored by default here, and the monitoring port of Prometheus can be modified by modifying this. Each exporter of Prometheus will be a target, they can report different monitoring information, such as machine status, or mysql performance, etc., different language SDK will also be a target, they will report your customized business monitoring information.  
    

  

    启动运行sever：

Start running sever

```
cby@cby-Inspiron-7577:~/prometheus-2.21.0.linux-amd64$ ./prometheus --config.file=prometheus.yml
```

  

    运行后，使用默认9090端口即可进行访问，若无法访问您可以查看一下是否有防火墙的限制，若没有限制，那就看一下是否正常启动，有端口的监听。  

    After running, you can use the default port 9090 to access it. If you can't access it, you can check if there is a firewall restriction. If there is no restriction, check if it is started normally and there is port monitoring.

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c2fcbebeecc41da9186e09c2721645b~tplv-k3u1fbpfcp-zoom-1.image)

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/836e9218798543c78b610fe580c510f0~tplv-k3u1fbpfcp-zoom-1.image)

**添加机器的监控器：**

**Add machine monitor**

  

    在官网的下载页面中，可以找到 node_exporter 这个tar包，这个监空插件可以监控基础的硬件信息，例如CPU内存硬盘等信息，node_exporter本身也是一个http服务可以进行直接调用使用哦。

    On the download page of the official website, you can find the tar package of node_exporter. This plug-in can monitor basic hardware information, such as CPU memory and hard disk information. The node_exporter itself is also an http service that can be used directly.

  

    下载最新的此插件，同时进行解压，并运行：

Download the latest plug-in, unzip at the same time, and run

```
cby@cby-Inspiron-7577:~$ wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
cby@cby-Inspiron-7577:~$ cd node_exporter-1.0.1.linux-amd64/
cby@cby-Inspiron-7577:~/node_exporter-1.0.1.linux-amd64$ ls
LICENSE  node_exporter  NOTICE
cby@cby-Inspiron-7577:~/node_exporter-1.0.1.linux-amd64$ ./node_exporter 
level=info ts=2020-10-04T16:31:41.858Z caller=node_exporter.go:177 msg="Starting node_exporter" version="(version=1.0.1, branch=HEAD, revision=3715be6ae899f2a9b9dbfd9c39f3e09a7bd4559f)"
level=info ts=2020-10-04T16:31:41.858Z caller=node_exporter.go:178 msg="Build context" build_context="(go=go1.14.4, user=root@1f76dbbcfa55, date=20200616-12:44:12)"
level=info ts=2020-10-04T16:31:41.859Z caller=node_exporter.go:105 msg="Enabled collectors"

```

  

    可以使用curl进行测试一下是否正常启动

 You can use curl to test whether it starts normally

```
cby@cby-Inspiron-7577:~$ curl http://localhost:9100/metrics
```

  

    若可以正常访问，那就可以在prometheus.yml文件中添加一个target

If you can access normally, you can add a target in the prometheus.yml file

```
\# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label \`job=<job_name>\` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: \['localhost:9090'\]

  - job_name: 'server'
    static_configs:
    - targets: \['localhost:9100'\]

```

  

    在标签栏的 Status --> Targets 中可以：

In Status --> Targets in the tab bar, you can

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/548ef0ace1b941d6ac554e5064c5bbce~tplv-k3u1fbpfcp-zoom-1.image)

  

### 安装Grafana：

**Install Grafana**

```
cby@cby-Inspiron-7577:~$ sudo apt-get install -y adduser libfontconfig1
cby@cby-Inspiron-7577:~$ wget https://dl.grafana.com/oss/release/grafana_7.2.0_amd64.deb

cby@cby-Inspiron-7577:~$ sudo dpkg -i grafana_7.2.0_amd64.deb
正在选中未选择的软件包 grafana。
(正在读取数据库 ... 系统当前共安装有 211277 个文件和目录。)
准备解压 grafana_7.2.0_amd64.deb  ...
正在解压 grafana (7.2.0) ...
正在设置 grafana (7.2.0) ...
正在添加系统用户"grafana" (UID 130)...
正在将新用户"grafana" (UID 130)添加到组"grafana"...
无法创建主目录"/usr/share/grafana"。
### NOT starting on installation, please execute the following statements to configure grafana to start automatically using systemd
 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable grafana-server
### You can start grafana-server by executing
 sudo /bin/systemctl start grafana-server
正在处理用于 systemd (245.4-4ubuntu3.2) 的触发器 ...

```

  

    安装完成后，进行启动：  

After the installation is complete, start

```
cby@cby-Inspiron-7577:~$ sudo systemctl start grafana-server.service
cby@cby-Inspiron-7577:~$ sudo systemctl status grafana-server.service
\[sudo\] cby 的密码：
对不起，请重试。
\[sudo\] cby 的密码：
对不起，请重试。
\[sudo\] cby 的密码：
● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2020-10-05 00:02:59 CST; 40min ago
       Docs: http://docs.grafana.org
   Main PID: 1521572 (grafana-server)
      Tasks: 14 (limit: 18689)
     Memory: 25.3M
     CGroup: /system.slice/grafana-server.service
             └─1521572 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini --pidfile=/var/run/grafa
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca97d6954de84f6bb1b5df3b80b7912f~tplv-k3u1fbpfcp-zoom-1.image)

    默认端口为3000 ，使用IP加端口即可进行访问，默认用户名密码是admin，登录后即可看到首页。在设置中进行添加Prometheus监控数据。  

    The default port is 3000, you can access by using IP plus port, the default user name and password is admin, you can see the home page after logging in. Add Prometheus monitoring data in the settings.

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c3edba91a864cd998e1e0e53150c403~tplv-k3u1fbpfcp-zoom-1.image)

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05bbc9bcf4bc46f7b15d7a630a1c8a4d~tplv-k3u1fbpfcp-zoom-1.image)

  

    添加监控数据后，导入一个监控面板，或者勤劳的人们可以自行进行配置面板，哇哈哈哈，同时可以在官方的面板界面中寻找到一个心仪的面板

地址为：https://grafana.com/dashboards

下载面板的json后，可以进行导入面板。

    After adding monitoring data, import a monitoring panel, or industrious people can configure the panel by themselves, wow ha ha ha, and you can find a favorite panel in the official panel interface

The address is: https://grafana.com/dashboards

After downloading the json of the panel, you can import the panel.

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eadf9f2650ae4a34a923e622219ff3f6~tplv-k3u1fbpfcp-zoom-1.image)

  

导入后即可显示看到花里胡哨的面版了

After importing, you can see the bells and whistles

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8188207dea0a440481d12f89601f93d7~tplv-k3u1fbpfcp-zoom-1.image)

  

    面板添加后，必然需要报警。可以使用onealert，进行告警。  

https://caweb.aiops.com/#/Application/newBuild/grafana/0

  

    After the panel is added, an alarm is necessary. You can use onealert to alert.

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4eece143b6e245859c112c2cf08a5d4f~tplv-k3u1fbpfcp-zoom-1.image)

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b202d06e2f964dbebc2528faa54c4779~tplv-k3u1fbpfcp-zoom-1.image)

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61f421538f5b48a9bf5398a25c510e8c~tplv-k3u1fbpfcp-zoom-1.image)

到这里环境已经配置完成

The environment has been configured here

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc597a7d80f54efe8b755c0b6e530c51~tplv-k3u1fbpfcp-zoom-1.image)