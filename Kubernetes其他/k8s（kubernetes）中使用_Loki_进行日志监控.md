---
layout: post
cid: 63
title: 在 k8s（kubernetes）中使用 Loki 进行日志监控
slug: 63
date: 2021/12/30 17:14:09
updated: 2021/12/30 17:14:09
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3279789d71c64e61a0df1c24cb94b1dd~tplv-k3u1fbpfcp-zoom-1.image)

  

# 安装helm环境

  

```
[root@hello ~/yaml]#
[root@hello ~/yaml]# curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
[root@hello ~/yaml]# sudo apt-get install apt-transport-https --yes
[root@hello ~/yaml]# echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
deb https://baltocdn.com/helm/stable/debian/ all main
[root@hello ~/yaml]# sudo apt-get update
[root@hello ~/yaml]# sudo apt-get install helm
[root@hello ~/yaml]#
```

  

# 添加安装下载源

  

```
[root@hello ~/yaml]# helm repo add loki https://grafana.github.io/loki/charts && helm repo update
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
"loki" has been added to your repositories
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "loki" chart repository
Update Complete. ⎈Happy Helming!⎈
[root@hello ~/yaml]#
[root@hello ~/yaml]#




[root@hello ~/yaml]# helm pull loki/loki-stack
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
[root@hello ~/yaml]# ls
loki-stack-2.1.2.tgz  nfs-storage.yaml  nginx-ingress.yaml
[root@hello ~/yaml]# tar xf loki-stack-2.1.2.tgz
[root@hello ~/yaml]# ls
loki-stack  loki-stack-2.1.2.tgz  nfs-storage.yaml  nginx-ingress.yaml
```

  
  

# 安装loki日志系统

  

```
[root@hello ~/yaml]# helm install loki loki-stack/
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
WARNING: This chart is deprecated
W1203 07:31:04.751065  212245 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
W1203 07:31:04.754254  212245 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
W1203 07:31:04.833003  212245 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
W1203 07:31:04.833003  212245 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
NAME: loki
LAST DEPLOYED: Fri Dec  3 07:31:04 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
The Loki stack has been deployed to your cluster. Loki can now be added as a datasource in Grafana.


See http://docs.grafana.org/features/datasources/loki/ for more detail.
[root@hello ~/yaml]#
```

  

# 查看安装后是否完成

  

```
[root@hello ~/yaml]# helm list -A
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
NAME    NAMESPACE   REVISION    UPDATED                                 STATUS      CHART               APP VERSION
loki    default     1           2021-12-03 07:31:04.3324429 +0000 UTC   deployed    loki-stack-2.1.2    v2.0.0    
[root@hello ~/yaml]#




[root@hello ~/yaml]# kubectl  get pod
NAME                                     READY   STATUS    RESTARTS   AGE
loki-0                                   0/1     Running   0          68s
loki-promtail-79tn8                      1/1     Running   0          68s
loki-promtail-qzjjs                      1/1     Running   0          68s
loki-promtail-zlt7p                      1/1     Running   0          68s
nfs-client-provisioner-dc5789f74-jsrh7   1/1     Running   0          44m
[root@hello ~/yaml]#
```

  

# 查看svc并修改类型

  

```
[root@hello ~/yaml]# kubectl  get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.68.0.1       <none>        443/TCP    4h44m
loki            ClusterIP   10.68.140.107   <none>        3100/TCP   2m58s
loki-headless   ClusterIP   None            <none>        3100/TCP   2m58s
[root@hello ~/yaml]#
```

  

# 将svc设置为 type: NodePort

  

```
[root@hello ~/yaml]# kubectl  edit  svc loki
service/loki edited
[root@hello ~/yaml]#
[root@hello ~/yaml]# kubectl  get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP   10.68.0.1       <none>        443/TCP          4h46m
loki            NodePort    10.68.140.107   <none>        3100:31089/TCP   4m34s
loki-headless   ClusterIP   None            <none>        3100/TCP         4m34s
[root@hello ~/yaml]#
```

  

# 添加nginx应用  

  

```
[root@hello ~/yaml]# vim nginx-app.yaml
[root@hello ~/yaml]# cat nginx-app.yaml
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1 
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
    jobLabel: nginx
spec:
  ports:
  - name: nginx
    port: 80
    protocol: TCP
  selector:
    app: nginx
  type: NodePort
[root@hello ~/yaml]#

```

  

# 查看nginx的pod  

  

```
[root@hello ~/yaml]# kubectl apply -f nginx-app.yaml 
deployment.apps/nginx created
service/nginx created

[root@hello ~/yaml]# kubectl get pod  | grep nginx
nginx-5d59d67564-7fj4b                   1/1     Running   0          29s
[root@hello ~/yaml]#

```

  

# 测试访问  

  

```
[root@hello ~/yaml]# kubectl  get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP   10.68.0.1       <none>        443/TCP          4h57m
loki            NodePort    10.68.140.107   <none>        3100:31089/TCP   15m
loki-headless   ClusterIP   None            <none>        3100/TCP         15m
nginx           NodePort    10.68.150.95    <none>        80:31317/TCP     105s
[root@hello ~/yaml]# 
[root@hello ~/yaml]# while true; do curl --silent --output /dev/null --write-out '%{http_code}' http://192.168.1.12:31317; sleep 1; echo; done

```

  

# 在grafana中添加源  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb42d16252a748ce90bb5fa77945c038~tplv-k3u1fbpfcp-zoom-1.image)

  

  

# 查看日志  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed027a6ed9674ae98cedbc8f6a8249c5~tplv-k3u1fbpfcp-zoom-1.image)

  

# 添加面板

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c23a4d8a5afd459b93280f2e02506a2d~tplv-k3u1fbpfcp-zoom-1.image)

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb98af4b99594a2cbf0448e8685715b5~tplv-k3u1fbpfcp-zoom-1.image)

  

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