---
layout: post
cid: 286
title: 在Kubernetes部署GitLab
slug: 286
date: 2022/09/12 21:59:15
updated: 2022/09/12 22:00:58
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: 在Kubernetes部署GitLab
keywords: Kubernetes,k8s,二进制,
mode: default
thumb: 
video: 
---


## 在Kubernetes部署GitLab

### 前置条件

已安装Helm工具
已部署NFS自动创建PVC


### 使用HELM安装
```
[root@k8s-master01 ~]# helm repo add gitlab https://charts.gitlab.io/
"gitlab" has been added to your repositories

[root@k8s-master01 ~]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "gitlab" chart repository
...Successfully got an update from the "cilium" chart repository
Update Complete. ⎈Happy Helming!⎈


[root@k8s-master01 ~]# helm upgrade --install gitlab gitlab/gitlab \
  --timeout 600s \
  --set global.hosts.domain=git.oiox.cn \
  --set global.hosts.externalIP=192.168.1.61 \
  --set certmanager-issuer.email=cby@chenby.cn 
  
NAME: gitlab
LAST DEPLOYED: Mon Sep 12 19:49:30 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
=== NOTICE
The minimum required version of PostgreSQL is now 12. See https://gitlab.com/gitlab-org/charts/gitlab/-/blob/master/doc/installation/upgrade.md for more details.

=== NOTICE
You've installed GitLab Runner without the ability to use 'docker in docker'.
The GitLab Runner chart (gitlab/gitlab-runner) is deployed without the `privileged` flag by default for security purposes. This can be changed by setting `gitlab-runner.runners.privileged` to `true`. Before doing so, please read the GitLab Runner chart's documentation on why we
chose not to enable this by default. See https://docs.gitlab.com/runner/install/kubernetes.html#running-docker-in-docker-containers-with-gitlab-runners
Help us improve the installation experience, let us know how we did with a 1 minute survey:https://gitlab.fra1.qualtrics.com/jfe/form/SV_6kVqZANThUQ1bZb?installation=helm&release=15-3

=== NOTICE
The in-chart NGINX Ingress Controller has the following requirements:
    - Kubernetes version must be 1.19 or newer.
    - Ingress objects must be in group/version `networking.k8s.io/v1`.
[root@k8s-master01 ~]# 

```

### 查看POD情况
```
[root@k8s-master01 ~]# kubectl get pod -A
NAMESPACE           NAME                                                     READY   STATUS      RESTARTS         AGE
cilium-monitoring   grafana-59957b9549-6zzqh                                 1/1     Running     1 (6m28s ago)    8h
cilium-monitoring   prometheus-7c8c9684bb-4v9cl                              1/1     Running     1 (4m49s ago)    8h
default             chenby-75b5d7fbfb-7zjsr                                  1/1     Running     1 (6m15s ago)    35h
default             chenby-75b5d7fbfb-hbvr8                                  1/1     Running     1 (5m27s ago)    35h
default             chenby-75b5d7fbfb-ppbzg                                  1/1     Running     1 (5m57s ago)    35h
default             cm-acme-http-solver-8b6lg                                1/1     Running     1 (4m49s ago)    11m
default             cm-acme-http-solver-9sd7r                                1/1     Running     1 (4m49s ago)    11m
default             cm-acme-http-solver-tx5x2                                1/1     Running     1 (5m27s ago)    11m
default             cm-acme-http-solver-w74zd                                1/1     Running     1 (4m49s ago)    11m
default             echo-a-6799dff547-pnx6w                                  1/1     Running     1 (6m28s ago)    8h
default             echo-b-fc47b659c-4bdg9                                   1/1     Running     1 (4m49s ago)    8h
default             echo-b-host-67fcfd59b7-28r9s                             1/1     Running     1 (4m49s ago)    8h
default             gitlab-certmanager-7cb7797848-fgdff                      1/1     Running     1 (5m27s ago)    12m
default             gitlab-certmanager-cainjector-5968cb88f9-qw4d7           1/1     Running     2 (5m57s ago)    12m
default             gitlab-certmanager-webhook-797bcff548-t266p              1/1     Running     1 (6m15s ago)    12m
default             gitlab-gitaly-0                                          1/1     Running     1 (6m28s ago)    12m
default             gitlab-gitlab-exporter-58fc5779d7-lbl4s                  1/1     Running     1 (5m27s ago)    12m
default             gitlab-gitlab-runner-5484688b78-d5gmt                    0/1     Running     3 (2m8s ago)     12m
default             gitlab-gitlab-shell-7578c56d55-p5fvp                     1/1     Running     1 (5m27s ago)    12m
default             gitlab-gitlab-shell-7578c56d55-vzbrb                     1/1     Running     1 (4m49s ago)    12m
default             gitlab-issuer-1-sw7nm                                    0/1     Completed   0                12m
default             gitlab-kas-85f677867b-sjxqv                              1/1     Running     1 (4m49s ago)    12m
default             gitlab-kas-85f677867b-wwlsl                              1/1     Running     1 (6m28s ago)    12m
default             gitlab-migrations-1-hpsc8                                0/1     Completed   2                12m
default             gitlab-minio-74467697bb-76xcb                            1/1     Running     1 (4m49s ago)    12m
default             gitlab-minio-create-buckets-1-nwzh2                      0/1     Completed   0                12m
default             gitlab-nginx-ingress-controller-77589fdd6f-7rk5f         1/1     Running     1 (5m27s ago)    12m
default             gitlab-nginx-ingress-controller-77589fdd6f-lk96x         1/1     Running     1 (4m49s ago)    12m
default             gitlab-postgresql-0                                      2/2     Running     2 (5m27s ago)    12m
default             gitlab-prometheus-server-6bf4fffc55-ww59q                2/2     Running     2 (6m14s ago)    12m
default             gitlab-redis-master-0                                    2/2     Running     2 (4m49s ago)    12m
default             gitlab-registry-54899b8c96-gkmm2                         1/1     Running     1 (5m27s ago)    12m
default             gitlab-registry-54899b8c96-pzxcd                         1/1     Running     1 (5m57s ago)    12m
default             gitlab-sidekiq-all-in-1-v2-64cbbc8cd8-4pmm9              1/1     Running     1 (5m57s ago)    12m
default             gitlab-sidekiq-all-in-1-v2-64cbbc8cd8-fr2wn              1/1     Running     0                81s
default             gitlab-sidekiq-all-in-1-v2-64cbbc8cd8-sx8b6              1/1     Running     0                81s
default             gitlab-toolbox-746c98d8f6-cxwl9                          1/1     Running     1 (5m27s ago)    12m
default             gitlab-webservice-default-6998494449-9hrtc               2/2     Running     1 (6m28s ago)    12m
default             gitlab-webservice-default-6998494449-kdbbq               2/2     Running     2 (6m14s ago)    12m
default             host-to-b-multi-node-clusterip-69c57975d6-z4j2z          1/1     Running     3 (4m6s ago)     8h
default             host-to-b-multi-node-headless-865899f7bb-frrmc           1/1     Running     2 (4m16s ago)    8h
default             nfs-client-provisioner-665598d599-4xwmf                  1/1     Running     3 (5m57s ago)    52m
default             pod-to-a-allowed-cnp-5f9d7d4b9d-hcd8x                    1/1     Running     4 (3m54s ago)    8h
default             pod-to-a-denied-cnp-65cc5ff97b-2rzb8                     1/1     Running     1 (6m28s ago)    8h
default             pod-to-a-dfc64f564-p7xcn                                 1/1     Running     3 (4m6s ago)     8h
default             pod-to-b-intra-node-nodeport-677868746b-trk2l            1/1     Running     1 (4m49s ago)    8h
default             pod-to-b-multi-node-clusterip-76bbbc677b-knfq2           1/1     Running     2 (4m2s ago)     8h
default             pod-to-b-multi-node-headless-698c6579fd-mmvd7            1/1     Running     2 (4m48s ago)    8h
default             pod-to-b-multi-node-nodeport-5dc4b8cfd6-8dxmz            1/1     Running     2 (4m48s ago)    8h
default             pod-to-external-1111-8459965778-pjt9b                    1/1     Running     13 (5m57s ago)   8h
default             pod-to-external-fqdn-allow-google-cnp-64df9fb89b-l9l4q   1/1     Running     15 (4m39s ago)   8h
kube-system         cilium-7rfj6                                             1/1     Running     1 (5m27s ago)    8h
kube-system         cilium-d4cch                                             1/1     Running     1 (6m28s ago)    8h
kube-system         cilium-h5x8r                                             1/1     Running     1 (5m57s ago)    8h
kube-system         cilium-operator-5dbddb6dbf-flpl5                         1/1     Running     1 (6m28s ago)    8h
kube-system         cilium-operator-5dbddb6dbf-gcznc                         1/1     Running     2 (4m49s ago)    8h
kube-system         cilium-t2xlz                                             1/1     Running     1 (4m49s ago)    8h
kube-system         cilium-z65z7                                             1/1     Running     1 (6m15s ago)    8h
kube-system         coredns-665475b9f8-jkqn8                                 1/1     Running     2 (4m49s ago)    44h
kube-system         hubble-relay-59d8575-9pl9z                               1/1     Running     1 (6m28s ago)    8h
kube-system         hubble-ui-64d4995d57-nsv9j                               2/2     Running     2 (6m28s ago)    8h
kube-system         metrics-server-776f58c94b-c6zgs                          1/1     Running     2 (6m14s ago)    45h
[root@k8s-master01 ~]# 
```


### 查看INGRESS情况
```
[root@k8s-master01 ~]# kubectl  get svc -A | grep ingress
default             gitlab-nginx-ingress-controller           LoadBalancer   10.111.0.148     <pending>     80:32002/TCP,443:31390/TCP,22:30887/TCP   26m
default             gitlab-nginx-ingress-controller-metrics   ClusterIP      10.104.165.192   <none>        10254/TCP                                 26m

# 修改为NodePort
[root@k8s-master01 ~]# kubectl  edit svc gitlab-nginx-ingress-controller
service/gitlab-nginx-ingress-controller edited
[root@k8s-master01 ~]# 
[root@k8s-master01 ~]# kubectl  get svc -A | grep ingress
default             gitlab-nginx-ingress-controller           NodePort    10.111.0.148     <none>        80:32002/TCP,443:31390/TCP,22:30887/TCP   26m
default             gitlab-nginx-ingress-controller-metrics   ClusterIP   10.104.165.192   <none>        10254/TCP                                 26m
[root@k8s-master01 ~]# 
[root@k8s-master01 ~]# 

# 查看有哪些域名
[root@k8s-master01 ~]# kubectl  get ingress
NAME                        CLASS          HOSTS                  ADDRESS        PORTS     AGE
cm-acme-http-solver-84tql   gitlab-nginx   minio.git.oiox.cn      10.111.0.148   80        25m
cm-acme-http-solver-c4n6s   gitlab-nginx   kas.git.oiox.cn        10.111.0.148   80        25m
cm-acme-http-solver-vwn4s   gitlab-nginx   gitlab.git.oiox.cn     10.111.0.148   80        25m
cm-acme-http-solver-zccvm   gitlab-nginx   registry.git.oiox.cn   10.111.0.148   80        25m
gitlab-kas                  gitlab-nginx   kas.git.oiox.cn        10.111.0.148   80, 443   27m
gitlab-minio                gitlab-nginx   minio.git.oiox.cn      10.111.0.148   80, 443   27m
gitlab-registry             gitlab-nginx   registry.git.oiox.cn   10.111.0.148   80, 443   27m
gitlab-webservice-default   gitlab-nginx   gitlab.git.oiox.cn     10.111.0.148   80, 443   27m
[root@k8s-master01 ~]# 

```

### 本地写入域名
```
[root@k8s-master01 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# 没有IPv6选择不配置即可
2409:8a10:9e10:8700::10 k8s-master01
2409:8a10:9e10:8700::20 k8s-master02
2409:8a10:9e10:8700::30 k8s-master03
2409:8a10:9e10:8700::40 k8s-node01
2409:8a10:9e10:8700::50 k8s-node02

192.168.1.61 k8s-master01
192.168.1.62 k8s-master02
192.168.1.63 k8s-master03
192.168.1.64 k8s-node01
192.168.1.65 k8s-node02
192.168.1.66 lb-vip

192.168.1.61 kas.git.oiox.cn
192.168.1.61 minio.git.oiox.cn
192.168.1.61 registry.git.oiox.cn
192.168.1.61 gitlab.git.oiox.cn
[root@k8s-master01 ~]# 
```

### 测试访问
```
# 查看密码
[root@k8s-master01 ~]# kubectl get secret gitlab-gitlab-initial-root-password -ojsonpath='{.data.password}' | base64 --decode ; echo
Hh7EjzH01T7DJw7TutWG6ynAU8yoGYcxNcV0cADCIpRCPeuFA5DBTC1I5V4T4gz4
[root@k8s-master01 ~]# 

# 访问
https://gitlab.git.oiox.cn:31390/

```
123
> **关于**
>
> https://www.oiox.cn/
>
> https://www.oiox.cn/index.php/start-page.html
>
> **CSDN、GitHub、知乎、开源中国、思否、掘金、简书、华为云、阿里云、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客**
>
> **全网可搜《小陈运维》**
>
> **文章主要发布于微信**