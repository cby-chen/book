---
layout: post
cid: 69
title: kubernetes（k8s）中部署 efk
slug: 69
date: 2021/12/30 17:15:17
updated: 2021/12/30 17:15:17
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


Kubernetes 开发了一个 Elasticsearch 附加组件来实现集群的日志管理。这是一个 Elasticsearch、Fluentd 和 Kibana 的组合。

  

Elasticsearch 是一个搜索引擎，负责存储日志并提供查询接口；

Fluentd 负责从 Kubernetes 搜集日志，每个node节点上面的fluentd监控并收集该节点上面的系统日志，并将处理过后的日志信息发送给Elasticsearch；

Kibana 提供了一个 Web GUI，用户可以浏览和搜索存储在 Elasticsearch 中的日志。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/963904f901324291806e8cad4d701f2c~tplv-k3u1fbpfcp-zoom-1.image)

  

从官方github仓库下载yaml文件

  

```
[root@hello ~/efk]# git clone https://github.com/kubernetes/kubernetes.git
[root@hello ~/efk]# kubectl create namespace logging
[root@hello ~/efk]#
```

  

执行所有yaml文件

  

```
[root@hello ~/efk]# cd kubernetes/cluster/addons/fluentd-elasticsearch/
[root@hello ~/efk/kubernetes/cluster/addons/fluentd-elasticsearch]# kubectl apply -f ./
namespace/logging created
service/elasticsearch-logging created
serviceaccount/elasticsearch-logging created
clusterrole.rbac.authorization.k8s.io/elasticsearch-logging created
clusterrolebinding.rbac.authorization.k8s.io/elasticsearch-logging created
statefulset.apps/elasticsearch-logging created
configmap/fluentd-es-config-v0.2.1 created
serviceaccount/fluentd-es created
clusterrole.rbac.authorization.k8s.io/fluentd-es created
clusterrolebinding.rbac.authorization.k8s.io/fluentd-es created
daemonset.apps/fluentd-es-v3.1.1 created
deployment.apps/kibana-logging created
service/kibana-logging created
```

  

查看pod状态：

  

```
[root@hello ~]# kubectl  get pod -n logging
NAME                              READY   STATUS    RESTARTS       AGE
elasticsearch-logging-0           1/1     Running   0              2m17s
elasticsearch-logging-1           1/1     Running   0              96s
fluentd-es-v3.1.1-qw9dj           1/1     Running   1 (97s ago)    2m16s
kibana-logging-75bd6cccf5-pskrr   1/1     Running   1 (106s ago)   2m16s
[root@hello ~]#


[root@hello ~]# kubectl  get service -n logging
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
elasticsearch-logging   ClusterIP   None            <none>        9200/TCP,9300/TCP   2m41s
kibana-logging          ClusterIP   10.68.145.186   <none>        5601/TCP            2m40s
[root@hello ~]#
```

  

访问 kibana

  

```
[root@hello ~]# kubectl proxy --address='192.168.1.11' --port=8086 --accept-hosts='^*$'
#访问
http://192.168.1.11:8086//api/v1/namespaces/logging/services/kibana-logging/proxy/
```

  

创建一个index-pattern索引

  
  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc588e1fd00747d49642989fe24226b4~tplv-k3u1fbpfcp-zoom-1.image)

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb93320695564020a6d77ddb1e4e1ee8~tplv-k3u1fbpfcp-zoom-1.image)

  

  

默认为 logstash-\* 即可，之后这里会看到日志

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03541d56f5b943cdb720641f0792dd15~tplv-k3u1fbpfcp-zoom-1.image)

  

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