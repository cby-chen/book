---
layout: post
cid: 48
title: kubernetes 安装 Prometheus + Grafana
slug: 48
date: 2021/12/30 17:10:00
updated: 2022/03/25 15:34:49
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


**kubernetes 安装 Prometheus + Grafana**

kubernetes install Prometheus + Grafana

  

**官网**

Official website

  

```
https://prometheus.io/
```

  

**GitHub**

GitHub

  

```
https://github.com/coreos/kube-prometheus
```

  

  

**组件说明**

Component description

  

**MetricServer：是kubernetes集群资源使用情况的聚合器，收集数据给kubernetes集群内使用，如 kubectl,hpa,scheduler等。**

**PrometheusOperator：是一个系统监测和警报工具箱，用来存储监控数据。**

**NodeExporter：用于各node的关键度量指标状态数据。**

**KubeStateMetrics：收集kubernetes集群内资源对象数 据，制定告警规则。**

**Prometheus：采用pull方式收集apiserver，scheduler，controller-manager，kubelet组件数 据，通过http协议传输。**

**Grafana：是可视化数据统计和监控平台。**

  

MetricServer: It is an aggregator of the resource usage of the kubernetes cluster, collecting data for use in the kubernetes cluster, such as kubectl, hpa, scheduler, etc.

PrometheusOperator: is a system monitoring and alerting toolbox used to store monitoring data.

NodeExporter: Used for the key metric status data of each node.

KubeStateMetrics: Collect resource object data in the kubernetes cluster and formulate alarm rules.

Prometheus: collect data from apiserver, scheduler, controller-manager, and kubelet components in a pull mode, and transmit it through the http protocol.

Grafana: It is a platform for visual data statistics and monitoring.

  

  

**安装**

Install

  

**配置Google上网环境下的docker，docker会去外网进行下载部分镜像**

Configure docker in Google's Internet environment, docker will go to the external network to download part of the image

  

  

```
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/proxy.conf
```

  

```
[root@k8s-master-node1 ~]# cat /etc/systemd/system/docker.service.d/proxy.conf
[Service]
Environment="HTTP_PROXY=http://192.168.1.6:7890/" 
Environment="HTTPS_PROXY=http://192.168.1.6:7890/" 
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
```

  

  

**dockerd代理的修改比较特殊，它实际上是改systemd的配置，因此需要重载systemd并重启dockerd才能生效。**

The modification of the dockerd agent is quite special. It actually changes the configuration of systemd, so systemd needs to be reloaded and dockerd restarted to take effect.

  

  

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13d4725ba08148eea28a3ef5a252f1f3~tplv-k3u1fbpfcp-zoom-1.image)

  

  

**下载**

download

  

```
[root@k8s-master-node1 ~]# git clone https://github.com/coreos/kube-prometheus.git
Cloning into 'kube-prometheus'...
remote: Enumerating objects: 13409, done.
remote: Counting objects: 100% (1908/1908), done.
remote: Compressing objects: 100% (801/801), done.
remote: Total 13409 (delta 1184), reused 1526 (delta 947), pack-reused 11501
Receiving objects: 100% (13409/13409), 6.65 MiB | 5.21 MiB/s, done.
Resolving deltas: 100% (8313/8313), done.
[root@k8s-master-node1 ~]# 
[root@k8s-master-node1 ~]# cd kube-prometheus/manifests
[root@k8s-master-node1 ~/kube-prometheus/manifests]# 
```

  

  

  

**修改 grafana-service.yaml 文件，使用 nodepode 方式访问 grafana：**

Modify the grafana-service.yaml file and use nodepode to access grafana:

  

```
[root@k8s-master-node1 ~/kube-prometheus/manifests]# cat grafana-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: grafana
    app.kubernetes.io/name: grafana
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 8.1.3
  name: grafana
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: http
    port: 3000
    targetPort: http
    nodePort: 31100
  selector:
    app.kubernetes.io/component: grafana
    app.kubernetes.io/name: grafana
    app.kubernetes.io/part-of: kube-prometheus
```

  

  

  

**修改 prometheus-service.yaml，改为 nodepode：**

Modify prometheus-service.yaml to nodepode:

  

```
[root@k8s-master-node1 ~/kube-prometheus/manifests]# cat prometheus-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.30.0
    prometheus: k8s
  name: prometheus-k8s
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: web
    port: 9090
    targetPort: web
    nodePort: 31200
  - name: reloader-web
    port: 8080
    targetPort: reloader-web
    nodePort: 31300
  selector:
    app: prometheus
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    prometheus: k8s
  sessionAffinity: ClientIP
```

  

**修改 alertmanager-service.yaml，改为 nodepode**

Modify alertmanager-service.yaml to nodepode

  

  

```
[root@k8s-master-node1 ~/kube-prometheus/manifests]# cat alertmanager-service.yaml 
apiVersion: v1
kind: Service
metadata:
  labels:
    alertmanager: main
    app.kubernetes.io/component: alert-router
    app.kubernetes.io/name: alertmanager
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 0.23.0
  name: alertmanager-main
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: web
    port: 9093
    targetPort: web
    nodePort: 31400
  - name: reloader-web
    port: 8080
    targetPort: reloader-web
    nodePort: 31500
  selector:
    alertmanager: main
    app: alertmanager
    app.kubernetes.io/component: alert-router
    app.kubernetes.io/name: alertmanager
    app.kubernetes.io/part-of: kube-prometheus
  sessionAffinity: ClientIP
[root@k8s-master-node1 ~/kube-prometheus/manifests]# 
```

  

  

  

  

  

**创建名称空间和CRD**

Create namespace and CRD

  

```
[root@k8s-master-node1 ~/kube-prometheus]# kubectl create -f /root/kube-prometheus/manifests/setup
namespace/monitoring created
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com created
clusterrole.rbac.authorization.k8s.io/prometheus-operator created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-operator created
deployment.apps/prometheus-operator created
service/prometheus-operator created
serviceaccount/prometheus-operator created
```

  

  

  

**等待资源可用后，安装**

After waiting for resources to be available, install

  

  

```
[root@k8s-master-node1 ~/kube-prometheus]# 
[root@k8s-master-node1 ~/kube-prometheus]# 
[root@k8s-master-node1 ~/kube-prometheus]# kubectl create -f /root/kube-prometheus/manifests/


---略---


[root@k8s-master-node1 ~/kube-prometheus]# 
```

  

  

  

**访问 Prometheus**

Visit Prometheus

  

http://192.168.1.10:31200/targets

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ad294f11c4e4fe686c2072db2dca411~tplv-k3u1fbpfcp-zoom-1.image)

  

**访问 Grafana**

Visit Grafana

  

http://192.168.1.10:31100/

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2423377395c46f98d5da67306eb2e98~tplv-k3u1fbpfcp-zoom-1.image)

  

**访问报警平台 AlertManager**

Visit the alert platform AlertManager

  

http://192.168.1.10:31400/#/status

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/514e8ba382c1456ab570d5083d5ab55c~tplv-k3u1fbpfcp-zoom-1.image)

  

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