---
layout: post
cid: 62
title: Kubernetes（k8s）集群安装JupyterHub以及Lab
slug: 62
date: 2021/12/30 17:13:00
updated: 2022/03/25 15:40:53
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


**背景**

  

JupyterHub 为用户组带来了笔记本的强大功能。它使用户能够访问计算环境和资源，而不会给用户带来安装和维护任务的负担。用户——包括学生、研究人员和数据科学家——可以在他们自己的工作空间中完成他们的工作，共享资源可以由系统管理员有效管理。

  

JupyterHub 在云端或您自己的硬件上运行，可以为世界上的任何用户提供预先配置的数据科学环境。它是可定制和可扩展的，适用于小型和大型团队、学术课程和大型基础设施。

  

**第一步**、参考：https://cloud.tencent.com/developer/article/1902519 创建动态挂载存储

  

**第二步**、安装helm

  

```
root@hello:~# curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
root@hello:~# sudo apt-get install apt-transport-https --yes
root@hello:~# echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
root@hello:~# sudo apt-get update
root@hello:~# sudo apt-get install helm
```

  

**第三步**、导入镜像

  

```
root@hello:~# docker load -i pause-3.5.tar
root@hello:~# docker load -i kube-scheduler.tar
```

  
  

**第四步**、安装jupyterhub

  

```
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/


helm repo update


helm upgrade --cleanup-on-fail \
  --install ju jupyterhub/jupyterhub \
  --namespace ju \
  --create-namespace \
  --version=1.2.0 \
  --values config.yaml
```

  
注：此文件可以自定义内容，具体看注释，如下开启lab功能

```
root@hello:~# vim config.yaml
root@hello:~# cat config.yaml 
# This file can update the JupyterHub Helm chart's default configuration values.
# #
# # For reference see the configuration reference and default values, but make
# # sure to refer to the Helm chart version of interest to you!
# #
# # Introduction to YAML:     https://www.youtube.com/watch?v=cdLNKUoMc6c
# # Chart config reference:   https://zero-to-jupyterhub.readthedocs.io/en/stable/resources/reference.html
# # Chart default values:     https://github.com/jupyterhub/zero-to-jupyterhub-k8s/blob/HEAD/jupyterhub/values.yaml
# # Available chart versions: https://jupyterhub.github.io/helm-chart/
# #
singleuser:
  defaultUrl: "/lab"
  extraEnv:
    JUPYTERHUB_SINGLEUSER_APP: "jupyter_server.serverapp.ServerApp"

#singleuser:
#  defaultUrl: "/lab"
#  extraEnv:
#    JUPYTERHUB_SINGLEUSER_APP: "notebook.notebookapp.NotebookApp"
root@hello:~# 
root@hello:~# 
root@hello:~#

```

  

第五步、修改svc为nodeport

  

```
root@hello:~# kubectl get svc  -A
NAMESPACE     NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes                  ClusterIP      10.68.0.1       <none>        443/TCP                  16h
ju            hub                         ClusterIP      10.68.60.16     <none>        8081/TCP                 114s
ju            proxy-api                   ClusterIP      10.68.239.54    <none>        8001/TCP                 114s
ju            proxy-public                LoadBalancer   10.68.62.47     <pending>     80:32070/TCP             114s
kube-system   dashboard-metrics-scraper   ClusterIP      10.68.244.241   <none>        8000/TCP                 16h
kube-system   kube-dns                    ClusterIP      10.68.0.2       <none>        53/UDP,53/TCP,9153/TCP   16h
kube-system   kube-dns-upstream           ClusterIP      10.68.221.104   <none>        53/UDP,53/TCP            16h
kube-system   kubernetes-dashboard        NodePort       10.68.206.196   <none>        443:32143/TCP            16h
kube-system   metrics-server              ClusterIP      10.68.1.149     <none>        443/TCP                  16h
kube-system   node-local-dns              ClusterIP      None            <none>        9253/TCP                 16h
root@hello:~# kubectl edit svc proxy-public -n ju
service/proxy-public edited
root@hello:~# 
root@hello:~# 
root@hello:~# kubectl get svc  -A
NAMESPACE     NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes                  ClusterIP   10.68.0.1       <none>        443/TCP                  16h
ju            hub                         ClusterIP   10.68.60.16     <none>        8081/TCP                 2m19s
ju            proxy-api                   ClusterIP   10.68.239.54    <none>        8001/TCP                 2m19s
ju            proxy-public                NodePort    10.68.62.47     <none>        80:32070/TCP             2m19s
kube-system   dashboard-metrics-scraper   ClusterIP   10.68.244.241   <none>        8000/TCP                 16h
kube-system   kube-dns                    ClusterIP   10.68.0.2       <none>        53/UDP,53/TCP,9153/TCP   16h
kube-system   kube-dns-upstream           ClusterIP   10.68.221.104   <none>        53/UDP,53/TCP            16h
kube-system   kubernetes-dashboard        NodePort    10.68.206.196   <none>        443:32143/TCP            16h
kube-system   metrics-server              ClusterIP   10.68.1.149     <none>        443/TCP                  16h
kube-system   node-local-dns              ClusterIP   None            <none>        9253/TCP                 16h
root@hello:~#

```

  

![](https://mmbiz.qpic.cn/mmbiz_png/KYOkQ49zfMf4BicLuGydibIKb2tftJtCBW5lzdULRlqlmrzDgOkbMIByevhzh2dRtLabALJicjqtmtm2GYzQkpwXw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/KYOkQ49zfMf4BicLuGydibIKb2tftJtCBWcphA0lCzZVZJL2X5piaXcMJpMNoh0yPba7Ric8qE1btuQK7Jlam7TNkA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/KYOkQ49zfMdA6AqIqjlTKC2oKfCKRNFLaTibVZXQ5hq0VLsrfDAn1pLLCNneOVrpCIF2JXy5U9HlU6H5tRgbB4g/640?wx_fmt=jpeg)  

  

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