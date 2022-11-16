---
layout: post
cid: 302
title: Grafana Prometheus Altermanager
slug: 302
date: 2022/11/12 11:59:09
updated: 2022/11/12 11:59:09
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



# Grafana Prometheus Altermanager 监控系统 


### 基本概念

Prometheus 是一套开源的系统监控、报警、时间序列数据库的组合，最初有 SoundCloud 开发的，后来随着越来越多公司使用，于是便独立成开源项目。Alertmanager 主要用于接收 Prometheus 发送的告警信息，它支持丰富的告警通知渠道，例如邮件、微信、钉钉、Slack 等常用沟通工具，而且很容易做到告警信息进行去重，降噪，分组等，是一款很好用的告警通知系统。


Prometheus架构如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2e8d70d8a4c404c82cf95163a2dd113~tplv-k3u1fbpfcp-watermark.image?)


### 安装Grafana服务
```
root@cby:~# sudo apt-get install -y adduser libfontconfig1
root@cby:~# wget https://dl.grafana.com/enterprise/release/grafana-enterprise_9.2.4_amd64.deb
root@cby:~# sudo dpkg -i grafana-enterprise_9.2.4_amd64.deb

root@cby:~# systemctl enable --now grafana-server.service
Synchronizing state of grafana-server.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable grafana-server
Created symlink /etc/systemd/system/multi-user.target.wants/grafana-server.service → /lib/systemd/system/grafana-server.service.
root@cby:~# 
```


### 安装Prometheus服务
```
root@cby:~# wget https://github.com/prometheus/prometheus/releases/download/v2.40.1/prometheus-2.40.1.linux-amd64.tar.gz
root@cby:~# tar xvf prometheus-2.40.1.linux-amd64.tar.gz 
prometheus-2.40.1.linux-amd64/
prometheus-2.40.1.linux-amd64/NOTICE
prometheus-2.40.1.linux-amd64/prometheus
prometheus-2.40.1.linux-amd64/LICENSE
prometheus-2.40.1.linux-amd64/console_libraries/
prometheus-2.40.1.linux-amd64/console_libraries/menu.lib
prometheus-2.40.1.linux-amd64/console_libraries/prom.lib
prometheus-2.40.1.linux-amd64/promtool
prometheus-2.40.1.linux-amd64/prometheus.yml
prometheus-2.40.1.linux-amd64/consoles/
prometheus-2.40.1.linux-amd64/consoles/prometheus-overview.html
prometheus-2.40.1.linux-amd64/consoles/prometheus.html
prometheus-2.40.1.linux-amd64/consoles/node-cpu.html
prometheus-2.40.1.linux-amd64/consoles/node-overview.html
prometheus-2.40.1.linux-amd64/consoles/node-disk.html
prometheus-2.40.1.linux-amd64/consoles/index.html.example
prometheus-2.40.1.linux-amd64/consoles/node.html
root@cby:~# mv prometheus-2.40.1.linux-amd64 prometheus                   
root@cby:~# 
```

### 进行全局配置
```
root@cby:~# vim prometheus/prometheus.yml
root@cby:~# cat prometheus/prometheus.yml
# Prometheus全局配置项
global:
  scrape_interval:     15s # 设定抓取数据的周期，默认为1min
  evaluation_interval: 15s # 设定更新rules文件的周期，默认为1min
  scrape_timeout: 15s # 设定抓取数据的超时时间，默认为10s
  external_labels: # 额外的属性，会添加到拉取得数据并存到数据库中
   monitor: 'codelab_monitor'

# Alertmanager配置
alerting:
 alertmanagers:
 - static_configs:
   - targets: ["127.0.0.1:9093"] # 设定alertmanager和prometheus交互的接口，即alertmanager监听的ip地址和端口
     
# rule配置，首次读取默认加载，之后根据evaluation_interval设定的周期加载
rule_files:
  - "dist/*.yml"

# scape配置
scrape_configs:
- job_name: 'prometheus' # job_name默认写入timeseries的labels中，可以用于查询使用
  scrape_interval: 15s # 抓取周期，默认采用global配置
  static_configs: # 静态配置
  - targets: ['127.0.0.1:9090'] # prometheus所要抓取数据的地址，即instance实例项

- job_name: 'web'
  scrape_interval: 15s
  static_configs:
  - targets: ['10.0.0.10:9200']

- job_name: 'node-exporter'
  scrape_interval: 15s
  file_sd_configs:
    - files:
      - "static_conf/*.yaml"
      refresh_interval: 1s

root@cby:~# 
```

### 进行写入动态配置文件
内容写需要监控的主机即可
```
root@cby:~# mkdir prometheus/static_conf/
root@cby:~# vim /prometheus/static_conf/file.yaml 
root@cby:~# cat /prometheus/static_conf/file.yaml                             
- targets: ['10.0.0.1:9200']
- targets: ['10.0.0.2:9200']
- targets: ['10.0.0.3:9200']
- targets: ['10.0.0.4:9200']
- targets: ['10.0.0.5:9200']
- targets: ['10.0.0.6:9200']
- targets: ['10.0.0.7:9200']
- targets: ['10.0.0.8:9200']
- targets: ['10.0.0.9:9200']
- targets: ['10.0.0.10:9200']
- targets: ['10.0.0.11:9200']
- targets: ['10.0.0.12:9200']
- targets: ['10.0.0.13:9200']
- targets: ['10.0.0.14:9200']
- targets: ['10.0.0.15:9200']
- targets: ['10.0.0.16:9200']
- targets: ['10.0.0.17:9200']
- targets: ['10.0.0.18:9200']
- targets: ['10.0.0.19:9200']
- targets: ['10.0.0.20:9200']
- targets: ['10.0.0.21:9200']
- targets: ['10.0.0.22:9200']
- targets: ['10.0.0.23:9200']
- targets: ['10.0.0.24:9200']
- targets: ['10.0.0.25:9200']
- targets: ['10.0.0.26:9200']
- targets: ['10.0.0.27:9200']
- targets: ['10.0.0.28:9200']
- targets: ['10.0.0.29:9200']
- targets: ['10.0.0.30:9200']
- targets: ['10.0.0.31:9200']
- targets: ['10.0.0.32:9200']
- targets: ['10.0.0.33:9200']
- targets: ['10.0.0.34:9200']
- targets: ['10.0.0.35:9200']
- targets: ['10.0.0.36:9200']
- targets: ['10.0.0.37:9200']
- targets: ['10.0.0.38:9200']
- targets: ['10.0.0.39:9200']
- targets: ['10.0.0.40:9200']
- targets: ['10.0.0.41:9200']
- targets: ['10.0.0.42:9200']
- targets: ['10.0.0.43:9200']
- targets: ['10.0.0.44:9200']
- targets: ['10.0.0.45:9200']
- targets: ['10.0.0.46:9200']
- targets: ['10.0.0.47:9200']
- targets: ['10.0.0.48:9200']
- targets: ['10.0.0.49:9200']
- targets: ['10.0.0.50:9200']
- targets: ['10.0.0.51:9200']
- targets: ['10.0.0.52:9200']
- targets: ['10.0.0.53:9200']
- targets: ['10.0.0.54:9200']
- targets: ['10.0.0.55:9200']
- targets: ['10.0.0.56:9200']
- targets: ['10.0.0.57:9200']
- targets: ['10.0.0.58:9200']
- targets: ['10.0.0.59:9200']
- targets: ['10.0.0.60:9200']
- targets: ['10.0.0.61:9200']
- targets: ['10.0.0.62:9200']
- targets: ['10.0.0.63:9200']
- targets: ['10.0.0.64:9200']
- targets: ['10.0.0.65:9200']
- targets: ['10.0.0.66:9200']
- targets: ['10.0.0.67:9200']
- targets: ['10.0.0.68:9200']
- targets: ['10.0.0.69:9200']
- targets: ['10.0.0.70:9200']
- targets: ['10.0.0.71:9200']
- targets: ['10.0.0.72:9200']
- targets: ['10.0.0.73:9200']
- targets: ['10.0.0.74:9200']
- targets: ['10.0.0.75:9200']
- targets: ['10.0.0.76:9200']
- targets: ['10.0.0.77:9200']
- targets: ['10.0.0.78:9200']
- targets: ['10.0.0.79:9200']
- targets: ['10.0.0.80:9200']
- targets: ['10.0.0.81:9200']
- targets: ['10.0.0.82:9200']
- targets: ['10.0.0.83:9200']
- targets: ['10.0.0.84:9200']
- targets: ['10.0.0.85:9200']
- targets: ['10.0.0.86:9200']
- targets: ['10.0.0.87:9200']
- targets: ['10.0.0.88:9200']
- targets: ['10.0.0.89:9200']
- targets: ['10.0.0.90:9200']
- targets: ['10.0.0.91:9200']
- targets: ['10.0.0.92:9200']
- targets: ['10.0.0.93:9200']
- targets: ['10.0.0.94:9200']
- targets: ['10.0.0.95:9200']
- targets: ['10.0.0.96:9200']
- targets: ['10.0.0.97:9200']
- targets: ['10.0.0.98:9200']
- targets: ['10.0.0.99:9200']
- targets: ['10.0.0.100:9200']
- targets: ['10.0.0.101:9200']
- targets: ['10.0.0.102:9200']
- targets: ['10.0.0.103:9200']
- targets: ['10.0.0.104:9200']
- targets: ['10.0.0.105:9200']
- targets: ['10.0.0.106:9200']
- targets: ['10.0.0.107:9200']
- targets: ['10.0.0.108:9200']
- targets: ['10.0.0.109:9200']
- targets: ['10.0.0.110:9200']
- targets: ['10.0.0.111:9200']
- targets: ['10.0.0.112:9200']
- targets: ['10.0.0.113:9200']
- targets: ['10.0.0.114:9200']
- targets: ['10.0.0.115:9200']
- targets: ['10.0.0.116:9200']
- targets: ['10.0.0.117:9200']
- targets: ['10.0.0.118:9200']
- targets: ['10.0.0.119:9200']
- targets: ['10.0.0.120:9200']
- targets: ['10.0.0.121:9200']
- targets: ['10.0.0.122:9200']
- targets: ['10.0.0.123:9200']
- targets: ['10.0.0.124:9200']
- targets: ['10.0.0.125:9200']
- targets: ['10.0.0.126:9200']
- targets: ['10.0.0.127:9200']
- targets: ['10.0.0.128:9200']
- targets: ['10.0.0.129:9200']
- targets: ['10.0.0.130:9200']
- targets: ['10.0.0.131:9200']
- targets: ['10.0.0.132:9200']
- targets: ['10.0.0.133:9200']
- targets: ['10.0.0.134:9200']
- targets: ['10.0.0.135:9200']
- targets: ['10.0.0.136:9200']
- targets: ['10.0.0.137:9200']
- targets: ['10.0.0.138:9200']
- targets: ['10.0.0.139:9200']
- targets: ['10.0.0.140:9200']
- targets: ['10.0.0.141:9200']
- targets: ['10.0.0.142:9200']
- targets: ['10.0.0.143:9200']
- targets: ['10.0.0.144:9200']
- targets: ['10.0.0.145:9200']
- targets: ['10.0.0.146:9200']
- targets: ['10.0.0.147:9200']
- targets: ['10.0.0.148:9200']
- targets: ['10.0.0.149:9200']
- targets: ['10.0.0.150:9200']
- targets: ['10.0.0.151:9200']
- targets: ['10.0.0.152:9200']
- targets: ['10.0.0.153:9200']
- targets: ['10.0.0.154:9200']
- targets: ['10.0.0.155:9200']
- targets: ['10.0.0.156:9200']
- targets: ['10.0.0.157:9200']
- targets: ['10.0.0.158:9200']
- targets: ['10.0.0.159:9200']
- targets: ['10.0.0.160:9200']
- targets: ['10.0.0.161:9200']
- targets: ['10.0.0.162:9200']
- targets: ['10.0.0.163:9200']
- targets: ['10.0.0.164:9200']
- targets: ['10.0.0.165:9200']
- targets: ['10.0.0.166:9200']
- targets: ['10.0.0.167:9200']
- targets: ['10.0.0.168:9200']
- targets: ['10.0.0.169:9200']
- targets: ['10.0.0.170:9200']
- targets: ['10.0.0.171:9200']
- targets: ['10.0.0.172:9200']
- targets: ['10.0.0.173:9200']
- targets: ['10.0.0.174:9200']
- targets: ['10.0.0.175:9200']
- targets: ['10.0.0.176:9200']
- targets: ['10.0.0.177:9200']
- targets: ['10.0.0.178:9200']
- targets: ['10.0.0.179:9200']
- targets: ['10.0.0.180:9200']
- targets: ['10.0.0.181:9200']
- targets: ['10.0.0.182:9200']
- targets: ['10.0.0.183:9200']
- targets: ['10.0.0.184:9200']
- targets: ['10.0.0.185:9200']
- targets: ['10.0.0.186:9200']
- targets: ['10.0.0.187:9200']
- targets: ['10.0.0.188:9200']
- targets: ['10.0.0.189:9200']
- targets: ['10.0.0.190:9200']
- targets: ['10.0.0.191:9200']
- targets: ['10.0.0.192:9200']
- targets: ['10.0.0.193:9200']
- targets: ['10.0.0.194:9200']
- targets: ['10.0.0.195:9200']
- targets: ['10.0.0.196:9200']
- targets: ['10.0.0.197:9200']
- targets: ['10.0.0.198:9200']
- targets: ['10.0.0.199:9200']
- targets: ['10.0.0.200:9200']
- targets: ['10.0.0.201:9200']
- targets: ['10.0.0.202:9200']
- targets: ['10.0.0.203:9200']
- targets: ['10.0.0.204:9200']
- targets: ['10.0.0.205:9200']
- targets: ['10.0.0.206:9200']
- targets: ['10.0.0.207:9200']
- targets: ['10.0.0.208:9200']
- targets: ['10.0.0.209:9200']
- targets: ['10.0.0.210:9200']
- targets: ['10.0.0.211:9200']
- targets: ['10.0.0.212:9200']
- targets: ['10.0.0.213:9200']
- targets: ['10.0.0.214:9200']
- targets: ['10.0.0.215:9200']
- targets: ['10.0.0.216:9200']
- targets: ['10.0.0.217:9200']
- targets: ['10.0.0.218:9200']
- targets: ['10.0.0.219:9200']
- targets: ['10.0.0.220:9200']
- targets: ['10.0.0.221:9200']
- targets: ['10.0.0.222:9200']
- targets: ['10.0.0.223:9200']
- targets: ['10.0.0.224:9200']
- targets: ['10.0.0.225:9200']
- targets: ['10.0.0.226:9200']
- targets: ['10.0.0.227:9200']
- targets: ['10.0.0.228:9200']
- targets: ['10.0.0.229:9200']
- targets: ['10.0.0.230:9200']
- targets: ['10.0.0.231:9200']
- targets: ['10.0.0.232:9200']
- targets: ['10.0.0.233:9200']
- targets: ['10.0.0.234:9200']
- targets: ['10.0.0.235:9200']
- targets: ['10.0.0.236:9200']
- targets: ['10.0.0.237:9200']
- targets: ['10.0.0.238:9200']
- targets: ['10.0.0.239:9200']
- targets: ['10.0.0.240:9200']
- targets: ['10.0.0.241:9200']
- targets: ['10.0.0.242:9200']
- targets: ['10.0.0.243:9200']
- targets: ['10.0.0.244:9200']
- targets: ['10.0.0.245:9200']
- targets: ['10.0.0.246:9200']
- targets: ['10.0.0.247:9200']
- targets: ['10.0.0.248:9200']
- targets: ['10.0.0.249:9200']
- targets: ['10.0.0.250:9200']
- targets: ['10.0.0.251:9200']
- targets: ['10.0.0.252:9200']
- targets: ['10.0.0.253:9200']
- targets: ['10.0.0.254:9200']
- targets: ['10.0.0.255:9200']
root@cby:~# 
```

### 配置开机自启服务
```
root@cby:~# vim /lib/systemd/system/prometheus.service
root@cby:~# cat /lib/systemd/system/prometheus.service
[Unit]
Description=Prometheus
After=network-online.target

[Service]
Type=simple
ExecStart=/prometheus/prometheus  --config.file=/prometheus/prometheus.yml
Restart=on-failur
ExecStop=/bin/kill -9 $MAINPID

[Install]
WantedBy=multi-user.target
root@cby:~# 
root@cby:~# systemctl daemon-reload
root@cby:~# 
root@cby:~# systemctl enable --now prometheus.service 
Created symlink /etc/systemd/system/multi-user.target.wants/prometheus.service → /lib/systemd/system/prometheus.service.
root@cby:~# 
root@cby:~# systemctl status prometheus.service  

```

### 安装Node_exporter监控组件
```
root@cby:~# wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
root@cby:~# tar xvf node_exporter-1.4.0.linux-amd64.tar.gz                                                                  
node_exporter-1.4.0.linux-amd64/
node_exporter-1.4.0.linux-amd64/LICENSE
node_exporter-1.4.0.linux-amd64/NOTICE
node_exporter-1.4.0.linux-amd64/node_exporter
root@cby:~# 


root@cby:~# mv node_exporter-1.4.0.linux-amd64 node_exporter
root@cby:~# mv prometheus /
root@cby:~# mv node_exporter /

```

### 设置为开机自启
```
root@cby:~# vim /lib/systemd/system/node_exporter.service
root@cby:~# cat /lib/systemd/system/node_exporter.service
[Unit]
Description=node_exporter
After=network-online.target

[Service]
Type=simple
ExecStart=/node_exporter/node_exporter  --web.listen-address=":9200"
Restart=on-failur
ExecStop=/bin/kill -9 $MAINPID

[Install]
WantedBy=multi-user.target
root@cby:~# systemctl daemon-reload
root@cby:~# 
root@cby:~# systemctl enable --now node_exporter.service 
Created symlink /etc/systemd/system/multi-user.target.wants/prometheus.service → /lib/systemd/system/prometheus.service.
root@cby:~# 
root@cby:~# systemctl status node_exporter.service 
```


### 下载安装alertmanager服务
```
root@cby:~# wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
root@cby:~# tar xvf alertmanager-0.24.0.linux-amd64.tar.gz 
alertmanager-0.24.0.linux-amd64/
alertmanager-0.24.0.linux-amd64/alertmanager.yml
alertmanager-0.24.0.linux-amd64/LICENSE
alertmanager-0.24.0.linux-amd64/NOTICE
alertmanager-0.24.0.linux-amd64/alertmanager
alertmanager-0.24.0.linux-amd64/amtool
root@cby:~# 
root@cby:~# mv alertmanager-0.24.0.linux-amd64 alertmanager
root@cby:~# mv alertmanager /
root@cby:~# 
```

### 全局配置
```
root@cby:~# vim /alertmanager/alertmanager.yml
root@cby:~# cat /alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m
  smtp_from: 'cby@chenby.cn'
  smtp_smarthost: 'smtp.qiye.aliyun.com:465'
  smtp_auth_username: 'cby@chenby.cn'
  smtp_auth_password: 'xxxxxxxx'
  smtp_require_tls: false
  smtp_hello: 'chenby.cn'
route:
  group_by: ['alertname']
  group_wait: 5s
  group_interval: 5s
  repeat_interval: 5m
  receiver: 'email'
receivers:
- name: 'email'
  email_configs:
  - to: 'cby@chenby.cn'
    send_resolved: true
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']

root@cby:~#
root@cby:~#
```

### 配置告警规则

规则模板建议在此网站找适合自己的  
https://awesome-prometheus-alerts.grep.to/

举例
```
groups:
 - name: test-rules
   rules:
   - alert: InstanceDown # 告警名称
     expr: up == 0 # 告警的判定条件，参考Prometheus高级查询来设定
     for: 2m # 满足告警条件持续时间多久后，才会发送告警
     labels: #标签项
      team: node
     annotations: # 解析项，详细解释告警信息
      summary: "{{$labels.instance}}: has been down"
      description: "{{$labels.instance}}: job {{$labels.job}} has been down "
      value: {{$value}}
```

我的告警配置
```
root@cby:~# mkdir /prometheus/dist/
root@cby:~# vim /prometheus/dist/123.yml 
root@cby:~# cat /prometheus/dist/123.yml 
groups:
  - name: generals.rules
    rules:
    - alert: PrometheusJobMissing
      expr: absent(up{job="prometheus"})
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Prometheus job missing (instance {{ $labels.instance }})
        description: "A Prometheus job has disappeared\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTargetMissing
      expr: up == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus target missing (instance {{ $labels.instance }})
        description: "A Prometheus target has disappeared. An exporter might be crashed.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusAllTargetsMissing
      expr: sum by (job) (up) == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus all targets missing (instance {{ $labels.instance }})
        description: "A Prometheus job does not have living target anymore.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTargetMissingWithWarmupTime
      expr: sum by (instance, job) ((up == 0) * on (instance) group_right(job) (node_time_seconds - node_boot_time_seconds > 600))
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus target missing with warmup time (instance {{ $labels.instance }})
        description: "Allow a job time to start up (10 minutes) before alerting that it's down.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusConfigurationReloadFailure
      expr: prometheus_config_last_reload_successful != 1
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Prometheus configuration reload failure (instance {{ $labels.instance }})
        description: "Prometheus configuration reload error\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTooManyRestarts
      expr: changes(process_start_time_seconds{job=~"prometheus|pushgateway|alertmanager"}[15m]) > 2
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Prometheus too many restarts (instance {{ $labels.instance }})
        description: "Prometheus has restarted more than twice in the last 15 minutes. It might be crashlooping.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusAlertmanagerJobMissing
      expr: absent(up{job="alertmanager"})
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Prometheus AlertManager job missing (instance {{ $labels.instance }})
        description: "A Prometheus AlertManager job has disappeared\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusAlertmanagerConfigurationReloadFailure
      expr: alertmanager_config_last_reload_successful != 1
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Prometheus AlertManager configuration reload failure (instance {{ $labels.instance }})
        description: "AlertManager configuration reload error\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusAlertmanagerConfigNotSynced
      expr: count(count_values("config_hash", alertmanager_config_hash)) > 1
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Prometheus AlertManager config not synced (instance {{ $labels.instance }})
        description: "Configurations of AlertManager cluster instances are out of sync\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusAlertmanagerE2eDeadManSwitch
      expr: vector(1)
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus AlertManager E2E dead man switch (instance {{ $labels.instance }})
        description: "Prometheus DeadManSwitch is an always-firing alert. It's used as an end-to-end test of Prometheus through the Alertmanager.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusNotConnectedToAlertmanager
      expr: prometheus_notifications_alertmanagers_discovered < 1
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus not connected to alertmanager (instance {{ $labels.instance }})
        description: "Prometheus cannot connect the alertmanager\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusRuleEvaluationFailures
      expr: increase(prometheus_rule_evaluation_failures_total[3m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus rule evaluation failures (instance {{ $labels.instance }})
        description: "Prometheus encountered {{ $value }} rule evaluation failures, leading to potentially ignored alerts.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTemplateTextExpansionFailures
      expr: increase(prometheus_template_text_expansion_failures_total[3m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus template text expansion failures (instance {{ $labels.instance }})
        description: "Prometheus encountered {{ $value }} template text expansion failures\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusRuleEvaluationSlow
      expr: prometheus_rule_group_last_duration_seconds > prometheus_rule_group_interval_seconds
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Prometheus rule evaluation slow (instance {{ $labels.instance }})
        description: "Prometheus rule evaluation took more time than the scheduled interval. It indicates a slower storage backend access or too complex query.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusNotificationsBacklog
      expr: min_over_time(prometheus_notifications_queue_length[10m]) > 0
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Prometheus notifications backlog (instance {{ $labels.instance }})
        description: "The Prometheus notification queue has not been empty for 10 minutes\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusAlertmanagerNotificationFailing
      expr: rate(alertmanager_notifications_failed_total[1m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus AlertManager notification failing (instance {{ $labels.instance }})
        description: "Alertmanager is failing sending notifications\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTargetEmpty
      expr: prometheus_sd_discovered_targets == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus target empty (instance {{ $labels.instance }})
        description: "Prometheus has no target in service discovery\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTargetScrapingSlow
      expr: prometheus_target_interval_length_seconds{quantile="0.9"} / on (interval, instance, job) prometheus_target_interval_length_seconds{quantile="0.5"} > 1.05
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Prometheus target scraping slow (instance {{ $labels.instance }})
        description: "Prometheus is scraping exporters slowly since it exceeded the requested interval time. Your Prometheus server is under-provisioned.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusLargeScrape
      expr: increase(prometheus_target_scrapes_exceeded_sample_limit_total[10m]) > 10
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Prometheus large scrape (instance {{ $labels.instance }})
        description: "Prometheus has many scrapes that exceed the sample limit\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTargetScrapeDuplicate
      expr: increase(prometheus_target_scrapes_sample_duplicate_timestamp_total[5m]) > 0
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Prometheus target scrape duplicate (instance {{ $labels.instance }})
        description: "Prometheus has many samples rejected due to duplicate timestamps but different values\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTsdbCheckpointCreationFailures
      expr: increase(prometheus_tsdb_checkpoint_creations_failed_total[1m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus TSDB checkpoint creation failures (instance {{ $labels.instance }})
        description: "Prometheus encountered {{ $value }} checkpoint creation failures\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTsdbCheckpointDeletionFailures
      expr: increase(prometheus_tsdb_checkpoint_deletions_failed_total[1m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus TSDB checkpoint deletion failures (instance {{ $labels.instance }})
        description: "Prometheus encountered {{ $value }} checkpoint deletion failures\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTsdbCompactionsFailed
      expr: increase(prometheus_tsdb_compactions_failed_total[1m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus TSDB compactions failed (instance {{ $labels.instance }})
        description: "Prometheus encountered {{ $value }} TSDB compactions failures\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTsdbHeadTruncationsFailed
      expr: increase(prometheus_tsdb_head_truncations_failed_total[1m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus TSDB head truncations failed (instance {{ $labels.instance }})
        description: "Prometheus encountered {{ $value }} TSDB head truncation failures\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTsdbReloadFailures
      expr: increase(prometheus_tsdb_reloads_failures_total[1m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus TSDB reload failures (instance {{ $labels.instance }})
        description: "Prometheus encountered {{ $value }} TSDB reload failures\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTsdbWalCorruptions
      expr: increase(prometheus_tsdb_wal_corruptions_total[1m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus TSDB WAL corruptions (instance {{ $labels.instance }})
        description: "Prometheus encountered {{ $value }} TSDB WAL corruptions\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTsdbWalTruncationsFailed
      expr: increase(prometheus_tsdb_wal_truncations_failed_total[1m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Prometheus TSDB WAL truncations failed (instance {{ $labels.instance }})
        description: "Prometheus encountered {{ $value }} TSDB WAL truncation failures\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: PrometheusTimeserieCardinality
      expr: label_replace(count by(__name__) ({__name__=~".+"}), "name", "$1", "__name__", "(.+)") > 10000
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Prometheus timeserie cardinality (instance {{ $labels.instance }})
        description: "The \"{{ $labels.name }}\" timeserie cardinality is getting very high: {{ $value }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"




    - alert: HostOutOfMemory
      expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 10
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host out of memory (instance {{ $labels.instance }})
        description: "Node memory is filling up (< 10% left)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostMemoryUnderMemoryPressure
      expr: rate(node_vmstat_pgmajfault[1m]) > 1000
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host memory under memory pressure (instance {{ $labels.instance }})
        description: "The node is under heavy memory pressure. High rate of major page faults\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostUnusualNetworkThroughputIn
      expr: sum by (instance) (rate(node_network_receive_bytes_total[2m])) / 1024 / 1024 > 100
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Host unusual network throughput in (instance {{ $labels.instance }})
        description: "Host network interfaces are probably receiving too much data (> 100 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostUnusualNetworkThroughputOut
      expr: sum by (instance) (rate(node_network_transmit_bytes_total[2m])) / 1024 / 1024 > 100
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Host unusual network throughput out (instance {{ $labels.instance }})
        description: "Host network interfaces are probably sending too much data (> 100 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostUnusualDiskReadRate
      expr: sum by (instance) (rate(node_disk_read_bytes_total[2m])) / 1024 / 1024 > 50
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Host unusual disk read rate (instance {{ $labels.instance }})
        description: "Disk is probably reading too much data (> 50 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostUnusualDiskWriteRate
      expr: sum by (instance) (rate(node_disk_written_bytes_total[2m])) / 1024 / 1024 > 50
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host unusual disk write rate (instance {{ $labels.instance }})
        description: "Disk is probably writing too much data (> 50 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    # Please add ignored mountpoints in node_exporter parameters like
    # "--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|run)($|/)".
    # Same rule using "node_filesystem_free_bytes" will fire when disk fills for non-root users.
    - alert: HostOutOfDiskSpace
      expr: (node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes < 10 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host out of disk space (instance {{ $labels.instance }})
        description: "Disk is almost full (< 10% left)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    # Please add ignored mountpoints in node_exporter parameters like
    # "--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|run)($|/)".
    # Same rule using "node_filesystem_free_bytes" will fire when disk fills for non-root users.
    - alert: HostDiskWillFillIn24Hours
      expr: (node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes < 10 and ON (instance, device, mountpoint) predict_linear(node_filesystem_avail_bytes{fstype!~"tmpfs"}[1h], 24 * 3600) < 0 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host disk will fill in 24 hours (instance {{ $labels.instance }})
        description: "Filesystem is predicted to run out of space within the next 24 hours at current write rate\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostOutOfInodes
      expr: node_filesystem_files_free / node_filesystem_files * 100 < 10 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host out of inodes (instance {{ $labels.instance }})
        description: "Disk is almost running out of available inodes (< 10% left)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostInodesWillFillIn24Hours
      expr: node_filesystem_files_free / node_filesystem_files * 100 < 10 and predict_linear(node_filesystem_files_free[1h], 24 * 3600) < 0 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host inodes will fill in 24 hours (instance {{ $labels.instance }})
        description: "Filesystem is predicted to run out of inodes within the next 24 hours at current write rate\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostUnusualDiskReadLatency
      expr: rate(node_disk_read_time_seconds_total[1m]) / rate(node_disk_reads_completed_total[1m]) > 0.1 and rate(node_disk_reads_completed_total[1m]) > 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host unusual disk read latency (instance {{ $labels.instance }})
        description: "Disk latency is growing (read operations > 100ms)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostUnusualDiskWriteLatency
      expr: rate(node_disk_write_time_seconds_total[1m]) / rate(node_disk_writes_completed_total[1m]) > 0.1 and rate(node_disk_writes_completed_total[1m]) > 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host unusual disk write latency (instance {{ $labels.instance }})
        description: "Disk latency is growing (write operations > 100ms)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostHighCpuLoad
      expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 80
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Host high CPU load (instance {{ $labels.instance }})
        description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostCpuStealNoisyNeighbor
      expr: avg by(instance) (rate(node_cpu_seconds_total{mode="steal"}[5m])) * 100 > 10
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Host CPU steal noisy neighbor (instance {{ $labels.instance }})
        description: "CPU steal is > 10%. A noisy neighbor is killing VM performances or a spot instance may be out of credit.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostCpuHighIowait
      expr: avg by (instance) (rate(node_cpu_seconds_total{mode="iowait"}[5m])) * 100 > 5
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Host CPU high iowait (instance {{ $labels.instance }})
        description: "CPU iowait > 5%. A high iowait means that you are disk or network bound.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    # 1000 context switches is an arbitrary number.
    # Alert threshold depends on nature of application.
    # Please read: https://github.com/samber/awesome-prometheus-alerts/issues/58
    - alert: HostContextSwitching
      expr: (rate(node_context_switches_total[5m])) / (count without(cpu, mode) (node_cpu_seconds_total{mode="idle"})) > 1000
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Host context switching (instance {{ $labels.instance }})
        description: "Context switching is growing on node (> 1000 / s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostSwapIsFillingUp
      expr: (1 - (node_memory_SwapFree_bytes / node_memory_SwapTotal_bytes)) * 100 > 80
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host swap is filling up (instance {{ $labels.instance }})
        description: "Swap is filling up (>80%)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostSystemdServiceCrashed
      expr: node_systemd_unit_state{state="failed"} == 1
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Host systemd service crashed (instance {{ $labels.instance }})
        description: "systemd service crashed\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostPhysicalComponentTooHot
      expr: node_hwmon_temp_celsius > 75
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Host physical component too hot (instance {{ $labels.instance }})
        description: "Physical hardware component too hot\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostNodeOvertemperatureAlarm
      expr: node_hwmon_temp_crit_alarm_celsius == 1
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Host node overtemperature alarm (instance {{ $labels.instance }})
        description: "Physical node temperature alarm triggered\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostRaidArrayGotInactive
      expr: node_md_state{state="inactive"} > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Host RAID array got inactive (instance {{ $labels.instance }})
        description: "RAID array {{ $labels.device }} is in degraded state due to one or more disks failures. Number of spare drives is insufficient to fix issue automatically.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostRaidDiskFailure
      expr: node_md_disks{state="failed"} > 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host RAID disk failure (instance {{ $labels.instance }})
        description: "At least one device in RAID array on {{ $labels.instance }} failed. Array {{ $labels.md_device }} needs attention and possibly a disk swap\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostKernelVersionDeviations
      expr: count(sum(label_replace(node_uname_info, "kernel", "$1", "release", "([0-9]+.[0-9]+.[0-9]+).*")) by (kernel)) > 1
      for: 6h
      labels:
        severity: warning
      annotations:
        summary: Host kernel version deviations (instance {{ $labels.instance }})
        description: "Different kernel versions are running\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostOomKillDetected
      expr: increase(node_vmstat_oom_kill[1m]) > 0
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Host OOM kill detected (instance {{ $labels.instance }})
        description: "OOM kill detected\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostEdacCorrectableErrorsDetected
      expr: increase(node_edac_correctable_errors_total[1m]) > 0
      for: 0m
      labels:
        severity: info
      annotations:
        summary: Host EDAC Correctable Errors detected (instance {{ $labels.instance }})
        description: "Host {{ $labels.instance }} has had {{ printf \"%.0f\" $value }} correctable memory errors reported by EDAC in the last 5 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostEdacUncorrectableErrorsDetected
      expr: node_edac_uncorrectable_errors_total > 0
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Host EDAC Uncorrectable Errors detected (instance {{ $labels.instance }})
        description: "Host {{ $labels.instance }} has had {{ printf \"%.0f\" $value }} uncorrectable memory errors reported by EDAC in the last 5 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostNetworkReceiveErrors
      expr: rate(node_network_receive_errs_total[2m]) / rate(node_network_receive_packets_total[2m]) > 0.01
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host Network Receive Errors (instance {{ $labels.instance }})
        description: "Host {{ $labels.instance }} interface {{ $labels.device }} has encountered {{ printf \"%.0f\" $value }} receive errors in the last two minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostNetworkTransmitErrors
      expr: rate(node_network_transmit_errs_total[2m]) / rate(node_network_transmit_packets_total[2m]) > 0.01
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host Network Transmit Errors (instance {{ $labels.instance }})
        description: "Host {{ $labels.instance }} interface {{ $labels.device }} has encountered {{ printf \"%.0f\" $value }} transmit errors in the last two minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostNetworkInterfaceSaturated
      expr: (rate(node_network_receive_bytes_total{device!~"^tap.*|^vnet.*|^veth.*|^tun.*"}[1m]) + rate(node_network_transmit_bytes_total{device!~"^tap.*|^vnet.*|^veth.*|^tun.*"}[1m])) / node_network_speed_bytes{device!~"^tap.*|^vnet.*|^veth.*|^tun.*"} > 0.8 < 10000
      for: 1m
      labels:
        severity: warning
      annotations:
        summary: Host Network Interface Saturated (instance {{ $labels.instance }})
        description: "The network interface \"{{ $labels.device }}\" on \"{{ $labels.instance }}\" is getting overloaded.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostNetworkBondDegraded
      expr: (node_bonding_active - node_bonding_slaves) != 0
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host Network Bond Degraded (instance {{ $labels.instance }})
        description: "Bond \"{{ $labels.device }}\" degraded on \"{{ $labels.instance }}\".\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostConntrackLimit
      expr: node_nf_conntrack_entries / node_nf_conntrack_entries_limit > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Host conntrack limit (instance {{ $labels.instance }})
        description: "The number of conntrack is approaching limit\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostClockSkew
      expr: (node_timex_offset_seconds > 0.05 and deriv(node_timex_offset_seconds[5m]) >= 0) or (node_timex_offset_seconds < -0.05 and deriv(node_timex_offset_seconds[5m]) <= 0)
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host clock skew (instance {{ $labels.instance }})
        description: "Clock skew detected. Clock is out of sync. Ensure NTP is configured correctly on this host.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostClockNotSynchronising
      expr: min_over_time(node_timex_sync_status[1m]) == 0 and node_timex_maxerror_seconds >= 16
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Host clock not synchronising (instance {{ $labels.instance }})
        description: "Clock not synchronising. Ensure NTP is configured on this host.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: HostRequiresReboot
      expr: node_reboot_required > 0
      for: 4h
      labels:
        severity: info
      annotations:
        summary: Host requires reboot (instance {{ $labels.instance }})
        description: "{{ $labels.instance }} requires a reboot.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesNodeReady
      expr: kube_node_status_condition{condition="Ready",status="true"} == 0
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes Node ready (instance {{ $labels.instance }})
        description: "Node {{ $labels.node }} has been unready for a long time\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesMemoryPressure
      expr: kube_node_status_condition{condition="MemoryPressure",status="true"} == 1
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes memory pressure (instance {{ $labels.instance }})
        description: "{{ $labels.node }} has MemoryPressure condition\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesDiskPressure
      expr: kube_node_status_condition{condition="DiskPressure",status="true"} == 1
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes disk pressure (instance {{ $labels.instance }})
        description: "{{ $labels.node }} has DiskPressure condition\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesNetworkUnavailable
      expr: kube_node_status_condition{condition="NetworkUnavailable",status="true"} == 1
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes network unavailable (instance {{ $labels.instance }})
        description: "{{ $labels.node }} has NetworkUnavailable condition\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesOutOfCapacity
      expr: sum by (node) ((kube_pod_status_phase{phase="Running"} == 1) + on(uid) group_left(node) (0 * kube_pod_info{pod_template_hash=""})) / sum by (node) (kube_node_status_allocatable{resource="pods"}) * 100 > 90
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes out of capacity (instance {{ $labels.instance }})
        description: "{{ $labels.node }} is out of capacity\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesContainerOomKiller
      expr: (kube_pod_container_status_restarts_total - kube_pod_container_status_restarts_total offset 10m >= 1) and ignoring (reason) min_over_time(kube_pod_container_status_last_terminated_reason{reason="OOMKilled"}[10m]) == 1
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes container oom killer (instance {{ $labels.instance }})
        description: "Container {{ $labels.container }} in pod {{ $labels.namespace }}/{{ $labels.pod }} has been OOMKilled {{ $value }} times in the last 10 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesJobFailed
      expr: kube_job_status_failed > 0
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes Job failed (instance {{ $labels.instance }})
        description: "Job {{$labels.namespace}}/{{$labels.exported_job}} failed to complete\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesCronjobSuspended
      expr: kube_cronjob_spec_suspend != 0
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes CronJob suspended (instance {{ $labels.instance }})
        description: "CronJob {{ $labels.namespace }}/{{ $labels.cronjob }} is suspended\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesPersistentvolumeclaimPending
      expr: kube_persistentvolumeclaim_status_phase{phase="Pending"} == 1
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes PersistentVolumeClaim pending (instance {{ $labels.instance }})
        description: "PersistentVolumeClaim {{ $labels.namespace }}/{{ $labels.persistentvolumeclaim }} is pending\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesVolumeOutOfDiskSpace
      expr: kubelet_volume_stats_available_bytes / kubelet_volume_stats_capacity_bytes * 100 < 10
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes Volume out of disk space (instance {{ $labels.instance }})
        description: "Volume is almost full (< 10% left)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesVolumeFullInFourDays
      expr: predict_linear(kubelet_volume_stats_available_bytes[6h], 4 * 24 * 3600) < 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes Volume full in four days (instance {{ $labels.instance }})
        description: "{{ $labels.namespace }}/{{ $labels.persistentvolumeclaim }} is expected to fill up within four days. Currently {{ $value | humanize }}% is available.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesPersistentvolumeError
      expr: kube_persistentvolume_status_phase{phase=~"Failed|Pending", job="kube-state-metrics"} > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes PersistentVolume error (instance {{ $labels.instance }})
        description: "Persistent volume is in bad state\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesStatefulsetDown
      expr: (kube_statefulset_status_replicas_ready / kube_statefulset_status_replicas_current) != 1
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes StatefulSet down (instance {{ $labels.instance }})
        description: "A StatefulSet went down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesHpaScalingAbility
      expr: kube_horizontalpodautoscaler_status_condition{status="false", condition="AbleToScale"} == 1
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes HPA scaling ability (instance {{ $labels.instance }})
        description: "Pod is unable to scale\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesHpaMetricAvailability
      expr: kube_horizontalpodautoscaler_status_condition{status="false", condition="ScalingActive"} == 1
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes HPA metric availability (instance {{ $labels.instance }})
        description: "HPA is not able to collect metrics\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesHpaScaleCapability
      expr: kube_horizontalpodautoscaler_status_desired_replicas >= kube_horizontalpodautoscaler_spec_max_replicas
      for: 2m
      labels:
        severity: info
      annotations:
        summary: Kubernetes HPA scale capability (instance {{ $labels.instance }})
        description: "The maximum number of desired Pods has been hit\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesPodNotHealthy
      expr: min_over_time(sum by (namespace, pod) (kube_pod_status_phase{phase=~"Pending|Unknown|Failed"})[15m:1m]) > 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes Pod not healthy (instance {{ $labels.instance }})
        description: "Pod has been in a non-ready state for longer than 15 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesPodCrashLooping
      expr: increase(kube_pod_container_status_restarts_total[1m]) > 3
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes pod crash looping (instance {{ $labels.instance }})
        description: "Pod {{ $labels.pod }} is crash looping\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesReplicassetMismatch
      expr: kube_replicaset_spec_replicas != kube_replicaset_status_ready_replicas
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes ReplicasSet mismatch (instance {{ $labels.instance }})
        description: "Deployment Replicas mismatch\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesDeploymentReplicasMismatch
      expr: kube_deployment_spec_replicas != kube_deployment_status_replicas_available
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes Deployment replicas mismatch (instance {{ $labels.instance }})
        description: "Deployment Replicas mismatch\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesStatefulsetReplicasMismatch
      expr: kube_statefulset_status_replicas_ready != kube_statefulset_status_replicas
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes StatefulSet replicas mismatch (instance {{ $labels.instance }})
        description: "A StatefulSet does not match the expected number of replicas.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesDeploymentGenerationMismatch
      expr: kube_deployment_status_observed_generation != kube_deployment_metadata_generation
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes Deployment generation mismatch (instance {{ $labels.instance }})
        description: "A Deployment has failed but has not been rolled back.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesStatefulsetGenerationMismatch
      expr: kube_statefulset_status_observed_generation != kube_statefulset_metadata_generation
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes StatefulSet generation mismatch (instance {{ $labels.instance }})
        description: "A StatefulSet has failed but has not been rolled back.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesStatefulsetUpdateNotRolledOut
      expr: max without (revision) (kube_statefulset_status_current_revision unless kube_statefulset_status_update_revision) * (kube_statefulset_replicas != kube_statefulset_status_replicas_updated)
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes StatefulSet update not rolled out (instance {{ $labels.instance }})
        description: "StatefulSet update has not been rolled out.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesDaemonsetRolloutStuck
      expr: kube_daemonset_status_number_ready / kube_daemonset_status_desired_number_scheduled * 100 < 100 or kube_daemonset_status_desired_number_scheduled - kube_daemonset_status_current_number_scheduled > 0
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes DaemonSet rollout stuck (instance {{ $labels.instance }})
        description: "Some Pods of DaemonSet are not scheduled or not ready\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesDaemonsetMisscheduled
      expr: kube_daemonset_status_number_misscheduled > 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes DaemonSet misscheduled (instance {{ $labels.instance }})
        description: "Some DaemonSet Pods are running where they are not supposed to run\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesCronjobTooLong
      expr: time() - kube_cronjob_next_schedule_time > 3600
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes CronJob too long (instance {{ $labels.instance }})
        description: "CronJob {{ $labels.namespace }}/{{ $labels.cronjob }} is taking more than 1h to complete.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesJobSlowCompletion
      expr: kube_job_spec_completions - kube_job_status_succeeded > 0
      for: 12h
      labels:
        severity: critical
      annotations:
        summary: Kubernetes job slow completion (instance {{ $labels.instance }})
        description: "Kubernetes Job {{ $labels.namespace }}/{{ $labels.job_name }} did not complete in time.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesApiServerErrors
      expr: sum(rate(apiserver_request_total{job="apiserver",code=~"^(?:5..)$"}[1m])) / sum(rate(apiserver_request_total{job="apiserver"}[1m])) * 100 > 3
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes API server errors (instance {{ $labels.instance }})
        description: "Kubernetes API server is experiencing high error rate\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesApiClientErrors
      expr: (sum(rate(rest_client_requests_total{code=~"(4|5).."}[1m])) by (instance, job) / sum(rate(rest_client_requests_total[1m])) by (instance, job)) * 100 > 1
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes API client errors (instance {{ $labels.instance }})
        description: "Kubernetes API client is experiencing high error rate\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesClientCertificateExpiresNextWeek
      expr: apiserver_client_certificate_expiration_seconds_count{job="apiserver"} > 0 and histogram_quantile(0.01, sum by (job, le) (rate(apiserver_client_certificate_expiration_seconds_bucket{job="apiserver"}[5m]))) < 7*24*60*60
      for: 0m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes client certificate expires next week (instance {{ $labels.instance }})
        description: "A client certificate used to authenticate to the apiserver is expiring next week.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesClientCertificateExpiresSoon
      expr: apiserver_client_certificate_expiration_seconds_count{job="apiserver"} > 0 and histogram_quantile(0.01, sum by (job, le) (rate(apiserver_client_certificate_expiration_seconds_bucket{job="apiserver"}[5m]))) < 24*60*60
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: Kubernetes client certificate expires soon (instance {{ $labels.instance }})
        description: "A client certificate used to authenticate to the apiserver is expiring in less than 24.0 hours.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

    - alert: KubernetesApiServerLatency
      expr: histogram_quantile(0.99, sum(rate(apiserver_request_latencies_bucket{subresource!="log",verb!~"^(?:CONNECT|WATCHLIST|WATCH|PROXY)$"} [10m])) WITHOUT (instance, resource)) / 1e+06 > 1
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: Kubernetes API server latency (instance {{ $labels.instance }})
        description: "Kubernetes API server has a 99th percentile latency of {{ $value }} seconds for {{ $labels.verb }} {{ $labels.resource }}.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
root@cby:~# 
```


### 设置开机自启

```
root@cby:~# vim /lib/systemd/system/alertmanager.service
root@cby:~# cat /lib/systemd/system/alertmanager.service
[Unit]
Description=Alertmanager for Prometheus
After=network-online.target

[Service]
Type=simple
ExecStart=/alertmanager/alertmanager --config.file=/alertmanager/alertmanager.yml
Restart=on-failur
ExecStop=/bin/kill -9 $MAINPID

[Install]
WantedBy=multi-user.target
root@cby:~# systemctl daemon-reload
root@cby:~# 
root@cby:~# systemctl enable --now alertmanager.service 
Created symlink /etc/systemd/system/multi-user.target.wants/prometheus.service → /lib/systemd/system/prometheus.service.
root@cby:~# 
root@cby:~# systemctl status alertmanager.service 
```


访问地址
```
#  Node
http://10.0.0.2:9090/
# Grafana
http://10.0.0.2:3000/
# Prometheus
http://10.0.0.2:9200/
# Altermanager
http://10.0.0.2:9093/
```

Grafana 链接 Prometheus
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/deef01f21e194a6fb0a19301a56c82bd~tplv-k3u1fbpfcp-watermark.image?)

异常记录
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d50168fd05849bbb9a6a29d918ea457~tplv-k3u1fbpfcp-watermark.image?)

邮件内容
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03f254722a9f4b4d9d1bbd8f7db9ca2c~tplv-k3u1fbpfcp-watermark.image?)

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

