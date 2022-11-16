---
layout: post
cid: 146
title: kubernetes（k8s） 安装 Prometheus + Grafana
slug: 146
date: 2022/04/24 10:01:00
updated: 2022/05/09 17:19:50
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


kubernetes（k8s） 安装 Prometheus + Grafana
=======================================

组件说明
====

**MetricServer：是kubernetes集群资源使用情况的聚合器，收集数据给kubernetes集群内使用，如 kubectl,hpa,scheduler等。**

**PrometheusOperator：是一个系统监测和警报工具箱，用来存储监控数据。**

**NodeExporter：用于各node的关键度量指标状态数据。**

**KubeStateMetrics：收集kubernetes集群内资源对象数 据，制定告警规则。**

**Prometheus：采用pull方式收集apiserver，scheduler，controller-manager，kubelet组件数 据，通过http协议传输。**

**Grafana：是可视化数据统计和监控平台。**

**![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a041fa6a689f48c3a45317369b2d13c5~tplv-k3u1fbpfcp-zoom-1.image)**

克隆代码
====

```
root@hello:~#   git clone -b release-0.10 https://github.com/prometheus-operator/kube-prometheus.git
Cloning into 'kube-prometheus'...
remote: Enumerating objects: 16026, done.
remote: Counting objects: 100% (2639/2639), done.
remote: Compressing objects: 100% (165/165), done.
remote: Total 16026 (delta 2524), reused 2485 (delta 2470), pack-reused 13387
Receiving objects: 100% (16026/16026), 7.81 MiB | 2.67 MiB/s, done.
Resolving deltas: 100% (10333/10333), done.
root@hello:~# 

```

进入目录修改镜像地址
==========

若访问Google畅通无阻即可无需修改，跳过即可

```
root@hello:~# cd kube-prometheus/manifests
root@hello:~/kube-prometheus/manifests# sed -i "s#quay.io/prometheus/#registry.cn-hangzhou.aliyuncs.com/chenby/#g" *.yaml
root@hello:~/kube-prometheus/manifests# sed -i "s#quay.io/brancz/#registry.cn-hangzhou.aliyuncs.com/chenby/#g" *.yaml
root@hello:~/kube-prometheus/manifests# sed -i "s#k8s.gcr.io/prometheus-adapter/#registry.cn-hangzhou.aliyuncs.com/chenby/#g" *.yaml
root@hello:~/kube-prometheus/manifests# sed -i "s#quay.io/prometheus-operator/#registry.cn-hangzhou.aliyuncs.com/chenby/#g" *.yaml
root@hello:~/kube-prometheus/manifests# sed -i "s#k8s.gcr.io/kube-state-metrics/#registry.cn-hangzhou.aliyuncs.com/chenby/#g" *.yaml
root@hello:~/kube-prometheus/manifests# 

```

修改svc为NodePort
==============

```
root@hello:~/kube-prometheus/manifests# sed -i  "/ports:/i\  type: NodePort" grafana-service.yaml
root@hello:~/kube-prometheus/manifests# sed -i  "/targetPort: http/i\    nodePort: 31100" grafana-service.yaml
root@hello:~/kube-prometheus/manifests# cat grafana-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: grafana
    app.kubernetes.io/name: grafana
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 8.3.3
  name: grafana
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: http
    port: 3000
    nodePort: 31100
    targetPort: http
  selector:
    app.kubernetes.io/component: grafana
    app.kubernetes.io/name: grafana
    app.kubernetes.io/part-of: kube-prometheus
root@hello:~/kube-prometheus/manifests# sed -i  "/ports:/i\  type: NodePort" prometheus-service.yaml
root@hello:~/kube-prometheus/manifests# sed -i  "/targetPort: web/i\    nodePort: 31200" prometheus-service.yaml
root@hello:~/kube-prometheus/manifests# sed -i  "/targetPort: reloader-web/i\    nodePort: 31300" prometheus-service.yaml
root@hello:~/kube-prometheus/manifests# cat prometheus-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.32.1
  name: prometheus-k8s
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: web
    port: 9090
    nodePort: 31200
    targetPort: web
  - name: reloader-web
    port: 8080
    nodePort: 31300
    targetPort: reloader-web
  selector:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
  sessionAffinity: ClientIP
root@hello:~/kube-prometheus/manifests# sed -i  "/ports:/i\  type: NodePort" alertmanager-service.yaml 
root@hello:~/kube-prometheus/manifests# sed -i  "/targetPort: web/i\    nodePort: 31400" alertmanager-service.yaml 
root@hello:~/kube-prometheus/manifests# sed -i  "/targetPort: reloader-web/i\    nodePort: 31500" alertmanager-service.yaml 
root@hello:~/kube-prometheus/manifests# cat alertmanager-service.yaml 
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: alert-router
    app.kubernetes.io/instance: main
    app.kubernetes.io/name: alertmanager
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 0.23.0
  name: alertmanager-main
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: web
    port: 9093
    nodePort: 31400
    targetPort: web
  - name: reloader-web
    port: 8080
    nodePort: 31500
    targetPort: reloader-web
  selector:
    app.kubernetes.io/component: alert-router
    app.kubernetes.io/instance: main
    app.kubernetes.io/name: alertmanager
    app.kubernetes.io/part-of: kube-prometheus
  sessionAffinity: ClientIP

```

执行部署
====

```
root@hello:~# 
root@hello:~#  kubectl create -f /root/kube-prometheus/manifests/setup
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com created
namespace/monitoring created
root@hello:~# 
root@hello:~# 


root@hello:~# 
root@hello:~#  kubectl create -f /root/kube-prometheus/manifests/
alertmanager.monitoring.coreos.com/main created
networkpolicy.networking.k8s.io/alertmanager-main created
poddisruptionbudget.policy/alertmanager-main created
prometheusrule.monitoring.coreos.com/alertmanager-main-rules created
secret/alertmanager-main created
service/alertmanager-main created
serviceaccount/alertmanager-main created
servicemonitor.monitoring.coreos.com/alertmanager-main created
clusterrole.rbac.authorization.k8s.io/blackbox-exporter created
clusterrolebinding.rbac.authorization.k8s.io/blackbox-exporter created
configmap/blackbox-exporter-configuration created
deployment.apps/blackbox-exporter created
networkpolicy.networking.k8s.io/blackbox-exporter created
service/blackbox-exporter created
serviceaccount/blackbox-exporter created
servicemonitor.monitoring.coreos.com/blackbox-exporter created
secret/grafana-config created
secret/grafana-datasources created
configmap/grafana-dashboard-alertmanager-overview created
configmap/grafana-dashboard-apiserver created
configmap/grafana-dashboard-cluster-total created
configmap/grafana-dashboard-controller-manager created
configmap/grafana-dashboard-grafana-overview created
configmap/grafana-dashboard-k8s-resources-cluster created
configmap/grafana-dashboard-k8s-resources-namespace created
configmap/grafana-dashboard-k8s-resources-node created
configmap/grafana-dashboard-k8s-resources-pod created
configmap/grafana-dashboard-k8s-resources-workload created
configmap/grafana-dashboard-k8s-resources-workloads-namespace created
configmap/grafana-dashboard-kubelet created
configmap/grafana-dashboard-namespace-by-pod created
configmap/grafana-dashboard-namespace-by-workload created
configmap/grafana-dashboard-node-cluster-rsrc-use created
configmap/grafana-dashboard-node-rsrc-use created
configmap/grafana-dashboard-nodes created
configmap/grafana-dashboard-persistentvolumesusage created
configmap/grafana-dashboard-pod-total created
configmap/grafana-dashboard-prometheus-remote-write created
configmap/grafana-dashboard-prometheus created
configmap/grafana-dashboard-proxy created
configmap/grafana-dashboard-scheduler created
configmap/grafana-dashboard-workload-total created
configmap/grafana-dashboards created
deployment.apps/grafana created
networkpolicy.networking.k8s.io/grafana created
prometheusrule.monitoring.coreos.com/grafana-rules created
service/grafana created
serviceaccount/grafana created
servicemonitor.monitoring.coreos.com/grafana created
prometheusrule.monitoring.coreos.com/kube-prometheus-rules created
clusterrole.rbac.authorization.k8s.io/kube-state-metrics created
clusterrolebinding.rbac.authorization.k8s.io/kube-state-metrics created
deployment.apps/kube-state-metrics created
networkpolicy.networking.k8s.io/kube-state-metrics created
prometheusrule.monitoring.coreos.com/kube-state-metrics-rules created
service/kube-state-metrics created
serviceaccount/kube-state-metrics created
servicemonitor.monitoring.coreos.com/kube-state-metrics created
prometheusrule.monitoring.coreos.com/kubernetes-monitoring-rules created
servicemonitor.monitoring.coreos.com/kube-apiserver created
servicemonitor.monitoring.coreos.com/coredns created
servicemonitor.monitoring.coreos.com/kube-controller-manager created
servicemonitor.monitoring.coreos.com/kube-scheduler created
servicemonitor.monitoring.coreos.com/kubelet created
clusterrole.rbac.authorization.k8s.io/node-exporter created
clusterrolebinding.rbac.authorization.k8s.io/node-exporter created
daemonset.apps/node-exporter created
networkpolicy.networking.k8s.io/node-exporter created
prometheusrule.monitoring.coreos.com/node-exporter-rules created
service/node-exporter created
serviceaccount/node-exporter created
servicemonitor.monitoring.coreos.com/node-exporter created
clusterrole.rbac.authorization.k8s.io/prometheus-k8s created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-k8s created
networkpolicy.networking.k8s.io/prometheus-k8s created
poddisruptionbudget.policy/prometheus-k8s created
prometheus.monitoring.coreos.com/k8s created
prometheusrule.monitoring.coreos.com/prometheus-k8s-prometheus-rules created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s-config created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s-config created
role.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s created
service/prometheus-k8s created
serviceaccount/prometheus-k8s created
servicemonitor.monitoring.coreos.com/prometheus-k8s created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
clusterrole.rbac.authorization.k8s.io/prometheus-adapter created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-adapter created
clusterrolebinding.rbac.authorization.k8s.io/resource-metrics:system:auth-delegator created
clusterrole.rbac.authorization.k8s.io/resource-metrics-server-resources created
configmap/adapter-config created
deployment.apps/prometheus-adapter created
networkpolicy.networking.k8s.io/prometheus-adapter created
poddisruptionbudget.policy/prometheus-adapter created
rolebinding.rbac.authorization.k8s.io/resource-metrics-auth-reader created
service/prometheus-adapter created
serviceaccount/prometheus-adapter created
servicemonitor.monitoring.coreos.com/prometheus-adapter created
clusterrole.rbac.authorization.k8s.io/prometheus-operator created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-operator created
deployment.apps/prometheus-operator created
networkpolicy.networking.k8s.io/prometheus-operator created
prometheusrule.monitoring.coreos.com/prometheus-operator-rules created
service/prometheus-operator created
serviceaccount/prometheus-operator created
servicemonitor.monitoring.coreos.com/prometheus-operator created
root@hello:~# 
root@hello:~# 

```

查看验证
====

```
root@hello:~# kubectl  get pod -n monitoring 
NAME                                   READY   STATUS    RESTARTS   AGE
alertmanager-main-0                    2/2     Running   0          69s
alertmanager-main-1                    2/2     Running   0          69s
alertmanager-main-2                    2/2     Running   0          69s
blackbox-exporter-6c559c5c66-kw6vd     3/3     Running   0          83s
grafana-7fd69887fb-jmpmp               1/1     Running   0          81s
kube-state-metrics-867b64476b-h84g4    3/3     Running   0          81s
node-exporter-576bm                    2/2     Running   0          80s
node-exporter-94gn9                    2/2     Running   0          80s
node-exporter-cbjqk                    2/2     Running   0          80s
node-exporter-mhlh7                    2/2     Running   0          80s
node-exporter-pdc6k                    2/2     Running   0          80s
node-exporter-pqqds                    2/2     Running   0          80s
node-exporter-s9cz4                    2/2     Running   0          80s
node-exporter-tdlnt                    2/2     Running   0          81s
prometheus-adapter-8f88b5b45-rrsh4     1/1     Running   0          78s
prometheus-adapter-8f88b5b45-wh6pf     1/1     Running   0          78s
prometheus-k8s-0                       2/2     Running   0          68s
prometheus-k8s-1                       2/2     Running   0          68s
prometheus-operator-7f9d9c77f8-h5gkt   2/2     Running   0          78s
root@hello:~# 



root@hello:~# kubectl  get svc -n monitoring 
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
alertmanager-main       NodePort    10.103.47.160    <none>        9093:31400/TCP,8080:31500/TCP   92s
alertmanager-operated   ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP      78s
blackbox-exporter       ClusterIP   10.102.108.160   <none>        9115/TCP,19115/TCP              92s
grafana                 NodePort    10.106.2.21      <none>        3000:31100/TCP                  90s
kube-state-metrics      ClusterIP   None             <none>        8443/TCP,9443/TCP               90s
node-exporter           ClusterIP   None             <none>        9100/TCP                        90s
prometheus-adapter      ClusterIP   10.108.65.108    <none>        443/TCP                         87s
prometheus-k8s          NodePort    10.100.227.174   <none>        9090:31200/TCP,8080:31300/TCP   88s
prometheus-operated     ClusterIP   None             <none>        9090/TCP                        77s
prometheus-operator     ClusterIP   None             <none>        8443/TCP                        87s
root@hello:~# 



http://192.168.1.81:31400/
http://192.168.1.81:31200/
http://192.168.1.81:31100/

```

一条命令执行
======

```
cd /root ; git clone -b release-0.10 https://github.com/prometheus-operator/kube-prometheus.git ;cd kube-prometheus/manifests ;sed -i "s#quay.io/prometheus/#registry.cn-hangzhou.aliyuncs.com/chenby/#g" *.yaml ;sed -i "s#quay.io/brancz/#registry.cn-hangzhou.aliyuncs.com/chenby/#g" *.yaml ;sed -i "s#k8s.gcr.io/prometheus-adapter/#registry.cn-hangzhou.aliyuncs.com/chenby/#g" *.yaml ;sed -i "s#quay.io/prometheus-operator/#registry.cn-hangzhou.aliyuncs.com/chenby/#g" *.yaml ;sed -i "s#k8s.gcr.io/kube-state-metrics/#registry.cn-hangzhou.aliyuncs.com/chenby/#g" *.yaml ;sed -i  "/ports:/i\  type: NodePort" grafana-service.yaml ;sed -i  "/targetPort: http/i\    nodePort: 31100" grafana-service.yaml ;sed -i  "/ports:/i\  type: NodePort" prometheus-service.yaml ;sed -i  "/targetPort: web/i\    nodePort: 31200" prometheus-service.yaml ;sed -i  "/targetPort: reloader-web/i\    nodePort: 31300" prometheus-service.yaml ;sed -i  "/ports:/i\  type: NodePort" alertmanager-service.yaml  ;sed -i  "/targetPort: web/i\    nodePort: 31400" alertmanager-service.yaml  ;sed -i  "/targetPort: reloader-web/i\    nodePort: 31500" alertmanager-service.yaml  ;kubectl create -f /root/kube-prometheus/manifests/setup ;kubectl create -f /root/kube-prometheus/manifests/ ; sleep 30 ; kubectl  get pod -n monitoring  ; kubectl  get svc -n monitoring  ;

```

> https://www.oiox.cn/
> 
> https://www.chenby.cn/
> 
> https://cby-chen.github.io/
> 
> https://blog.csdn.net/qq_33921750
> 
> https://my.oschina.net/u/3981543
> 
> https://www.zhihu.com/people/chen-bu-yun-2
> 
> https://segmentfault.com/u/hppyvyv6/articles
> 
> https://juejin.cn/user/3315782802482007
> 
> https://cloud.tencent.com/developer/column/93230
> 
> https://www.jianshu.com/u/0f894314ae2c
> 
> https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/
> 
> CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、今日头条、个人博客、全网可搜《小陈运维》
> 
> 文章主要发布于微信：《Linux运维交流社区》