---
layout: post
cid: 58
title: kubernetes核心实战（八）--- service
slug: 58
date: 2021/12/30 17:13:00
updated: 2022/05/19 15:54:15
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


#### 13、service

四层网络负载

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d46bc36e09de4cbb93b68b066aa3ea31~tplv-k3u1fbpfcp-zoom-1.image)

##### 创建

```
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# vim my-app.yaml
[root@k8s-master-node1 ~/yaml/test]# cat my-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-dep
  template:
    metadata:
      labels:
        app: my-dep
    spec:
      containers:
      - image: nginx
        name: nginx
[root@k8s-master-node1 ~/yaml/test]# kubectl  apply -f my-app.yaml 
deployment.apps/my-dep created
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  get deployments.apps 
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
ingress-demo-app         2/2     2            2           155m
my-dep                   3/3     3            3           71s
nfs-client-provisioner   1/1     1            1           140m
[root@k8s-master-node1 ~/yaml/test]#

```

###### 使用标签查找

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  get pod --show-labels 
NAME                                     READY   STATUS              RESTARTS   AGE     LABELS
hello-27285682--1-q2p6x                  0/1     Completed           0          3m15s   controller-uid=5e700c2e-29b3-4099-be17-b005d8077284,job-name=hello-27285682
hello-27285683--1-b2qgn                  0/1     Completed           0          2m15s   controller-uid=d7455e28-bd37-4fdf-bf47-de7ae6b4b7bb,job-name=hello-27285683
hello-27285684--1-glsmp                  0/1     Completed           0          75s     controller-uid=9cc7f28d-e780-49fb-a23a-ab725413ea8a,job-name=hello-27285684
hello-27285685--1-s7ws5                  0/1     ContainerCreating   0          15s     controller-uid=169e3631-6981-4df8-bfee-6a4f4632b713,job-name=hello-27285685
ingress-demo-app-694bf5d965-8rh7f        1/1     Running             0          157m    app=ingress-demo-app,pod-template-hash=694bf5d965
ingress-demo-app-694bf5d965-swkpb        1/1     Running             0          157m    app=ingress-demo-app,pod-template-hash=694bf5d965
my-dep-5b7868d854-kzflw                  1/1     Running             0          2m34s   app=my-dep,pod-template-hash=5b7868d854
my-dep-5b7868d854-pfhps                  1/1     Running             0          2m34s   app=my-dep,pod-template-hash=5b7868d854
my-dep-5b7868d854-v67ll                  1/1     Running             0          2m34s   app=my-dep,pod-template-hash=5b7868d854
nfs-client-provisioner-dc5789f74-5bznq   1/1     Running             0          141m    app=nfs-client-provisioner,pod-template-hash=dc5789f74
pi--1-k5cbq                              0/1     Completed           0          25m     controller-uid=2ecfcafd-f848-403b-b37f-9c145a0dc8cc,job-name=pi
redis-app-86g4q                          1/1     Running             0          27m     controller-revision-hash=77c8899f5d,name=fluentd-redis,pod-template-generation=1
redis-app-rt92n                          1/1     Running             0          27m     controller-revision-hash=77c8899f5d,name=fluentd-redis,pod-template-generation=1
redis-app-vkzft                          1/1     Running             0          27m     controller-revision-hash=77c8899f5d,name=fluentd-redis,pod-template-generation=1
web-0                                    1/1     Running             0          91m     app=nginx,controller-revision-hash=web-57c5cc66df,statefulset.kubernetes.io/pod-name=web-0
web-1                                    1/1     Running             0          91m     app=nginx,controller-revision-hash=web-57c5cc66df,statefulset.kubernetes.io/pod-name=web-1
web-2                                    1/1     Running             0          90m     app=nginx,controller-revision-hash=web-57c5cc66df,statefulset.kubernetes.io/pod-name=web-2
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl get pod -l app=my-dep
NAME                      READY   STATUS    RESTARTS   AGE
my-dep-5b7868d854-kzflw   1/1     Running   0          2m42s
my-dep-5b7868d854-pfhps   1/1     Running   0          2m42s
my-dep-5b7868d854-v67ll   1/1     Running   0          2m42s
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 命令行暴露端口

```
[root@k8s-master-node1 ~/yaml/test]# kubectl expose deployment my-dep --port=8000 --target-port=80
service/my-dep exposed
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  get service
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
ingress-demo-app   ClusterIP   10.96.145.40    <none>        80/TCP     158m
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP    160m
my-dep             ClusterIP   10.96.241.162   <none>        8000/TCP   11s
nginx              ClusterIP   None            <none>        80/TCP     92m
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### yaml文件暴露端口

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  selector:
    app: my-dep
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
```

  

##### ClusterIP方式暴露

###### 命令行

```
kubectl expose deployment my-dep --port=8000 --target-port=80 --type=ClusterIP
```

###### yaml方式

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
  selector:
    app: my-dep
  type: ClusterIP
```

  

##### 访问测试

```
[root@k8s-master-node1 ~/yaml/test]# curl -I  10.96.241.162:8000
HTTP/1.1 200 OK
Server: nginx/1.21.4
Date: Wed, 17 Nov 2021 09:30:27 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 02 Nov 2021 14:49:22 GMT
Connection: keep-alive
ETag: "61814ff2-267"
Accept-Ranges: bytes

[root@k8s-master-node1 ~/yaml/test]#
```

  

##### NodePort方式暴露

###### 命令行

```
kubectl expose deployment my-dep --port=8000 --target-port=80 --type=NodePort
```

  

###### yaml方式

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
  selector:
    app: my-dep
  type: NodePort
```

  

##### 查看service

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  get service
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
ingress-demo-app   ClusterIP   10.96.145.40    <none>        80/TCP           165m
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP          167m
my-dep             NodePort    10.96.241.162   <none>        8000:32306/TCP   7m13s
nginx              ClusterIP   None            <none>        80/TCP           99m
[root@k8s-master-node1 ~/yaml/test]#
```

  

##### 访问测试

使用kubernetes任何node的ip都可以访问

```
[root@k8s-master-node1 ~/yaml/test]# curl -I 192.168.1.62:32306
HTTP/1.1 200 OK
Server: nginx/1.21.4
Date: Wed, 17 Nov 2021 09:36:50 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 02 Nov 2021 14:49:22 GMT
Connection: keep-alive
ETag: "61814ff2-267"
Accept-Ranges: bytes

[root@k8s-master-node1 ~/yaml/test]# curl -I 192.168.1.61:32306
HTTP/1.1 200 OK
Server: nginx/1.21.4
Date: Wed, 17 Nov 2021 09:36:53 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 02 Nov 2021 14:49:22 GMT
Connection: keep-alive
ETag: "61814ff2-267"
Accept-Ranges: bytes

[root@k8s-master-node1 ~/yaml/test]# curl -I 192.168.1.63:32306
HTTP/1.1 200 OK
Server: nginx/1.21.4
Date: Wed, 17 Nov 2021 09:36:56 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 02 Nov 2021 14:49:22 GMT
Connection: keep-alive
ETag: "61814ff2-267"
Accept-Ranges: bytes

[root@k8s-master-node1 ~/yaml/test]#
```

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f717fc23b7a40ff9aab2637fa649ead~tplv-k3u1fbpfcp-zoom-1.image)  

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