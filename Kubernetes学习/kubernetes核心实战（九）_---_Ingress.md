---
layout: post
cid: 59
title: kubernetes核心实战（九） --- Ingress
slug: 59
date: 2021/12/30 17:13:00
updated: 2022/05/19 15:51:37
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


#### 14、Ingress

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90d8f35932b0479b850193b4a02195a0~tplv-k3u1fbpfcp-zoom-1.image)

  

##### 检查是否有安装

```
[root@k8s-master-node1 ~/yaml/test]# kubectl get pod,svc -n ingress-nginx
NAME                                           READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create--1-74mtg    0/1     Completed   0          172m
pod/ingress-nginx-admission-patch--1-5qrct     0/1     Completed   0          172m
pod/ingress-nginx-controller-f97bd58b5-vr8c2   1/1     Running     0          172m

NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.96.109.80    <none>        80:30127/TCP,443:36903/TCP   172m
service/ingress-nginx-controller-admission   ClusterIP   10.96.215.201   <none>        443/TCP                      172m
[root@k8s-master-node1 ~/yaml/test]#
```

  

若未安装可以查看官网文档：https://kubernetes.github.io/ingress-nginx/deploy/

  

##### 创建环境：

```
[root@k8s-master-node1 ~/yaml/test]# vim ingress.yaml
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# cat ingress.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-server
  template:
    metadata:
      labels:
        app: hello-server
    spec:
      containers:
      - name: hello-server
        image: registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/hello-server
        ports:
        - containerPort: 9000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - image: nginx
        name: nginx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
spec:
  selector:
    app: nginx-demo
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-server
  name: hello-server
spec:
  selector:
    app: hello-server
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 9000
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  apply  -f ingress.yaml 
deployment.apps/hello-server created
deployment.apps/nginx-demo created
service/nginx-demo created
service/hello-server created
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看四层负载并测试

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  get service
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-server       ClusterIP   10.96.246.46    <none>        8000/TCP         21s
ingress-demo-app   ClusterIP   10.96.145.40    <none>        80/TCP           3h1m
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP          3h3m
my-dep             NodePort    10.96.241.162   <none>        8000:32306/TCP   23m
nginx              ClusterIP   None            <none>        80/TCP           115m
nginx-demo         ClusterIP   10.96.162.193   <none>        8000/TCP         21s
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# curl  -I 10.96.162.193:8000
HTTP/1.1 200 OK
Server: nginx/1.21.4
Date: Wed, 17 Nov 2021 09:49:40 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 02 Nov 2021 14:49:22 GMT
Connection: keep-alive
ETag: "61814ff2-267"
Accept-Ranges: bytes

[root@k8s-master-node1 ~/yaml/test]# curl  -I 10.96.246.46:8000
HTTP/1.1 200 OK
Date: Wed, 17 Nov 2021 09:49:52 GMT
Content-Length: 12
Content-Type: text/plain; charset=utf-8

[root@k8s-master-node1 ~/yaml/test]#
```

  

##### 创建七层

```
[root@k8s-master-node1 ~/yaml/test]# vim ingress-7.yaml
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# cat ingress-7.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  name: ingress-host-bar
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.chenby.cn"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-server
            port:
              number: 8000
  - host: "demo.chenby.cn"
    http:
      paths:
      - pathType: Prefix
        path: "/nginx"  # 把请求会转给下面的服务，下面的服务一定要能处理这个路径，不能处理就是404
        backend:
          service:
            name: nginx-demo  ## java，比如使用路径重写，去掉前缀nginx
            port:
              number: 8000
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  apply  -f ingress-7.yaml 
ingress.networking.k8s.io/ingress-host-bar created
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  get ingress
NAME               CLASS    HOSTS                            ADDRESS        PORTS   AGE
ingress-demo-app   <none>   app.demo.com                     192.168.1.62   80      3h14m
ingress-host-bar   nginx    hello.chenby.cn,demo.chenby.cn   192.168.1.62   80      9m50s
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  describe ingress  ingress-host-bar 
Name:             ingress-host-bar
Namespace:        default
Address:          192.168.1.62
Default backend:  default-http-backend:80 (10.244.2.7:8080)
Rules:
  Host             Path  Backends
  ----             ----  --------
  hello.chenby.cn  
                   /   hello-server:8000 (10.244.2.47:9000,10.244.2.48:9000)
  demo.chenby.cn   
                   /nginx   nginx-demo:8000 (10.244.0.13:80,10.244.1.34:80)
Annotations:       <none>
Events:
  Type    Reason  Age                    From                      Message
  ----    ------  ----                   ----                      -------
  Normal  Sync    6m26s (x2 over 6m50s)  nginx-ingress-controller  Scheduled for sync
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 电脑上写死hosts

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  get service ingress-demo-app 
NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
ingress-demo-app   ClusterIP   10.96.145.40   <none>        80/TCP    3h15m
[root@k8s-master-node1 ~/yaml/test]# 


10.96.145.40 hello.chenby.cn 
10.96.145.40 demo.chenby.cn
```

  

###### 测试访问

```
[root@k8s-master-node1 ~/yaml/test]# curl hello.chenby.cn
Hostname: ingress-demo-app-694bf5d965-8rh7f
IP: 127.0.0.1
IP: 10.244.1.6
RemoteAddr: 192.168.1.61:49809
GET / HTTP/1.1
Host: hello.chenby.cn
User-Agent: curl/7.68.0
Accept: */*

[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# curl demo.chenby.cn
Hostname: ingress-demo-app-694bf5d965-swkpb
IP: 127.0.0.1
IP: 10.244.2.4
RemoteAddr: 192.168.1.61:57797
GET / HTTP/1.1
Host: demo.chenby.cn
User-Agent: curl/7.68.0
Accept: */*

[root@k8s-master-node1 ~/yaml/test]#
```

  

##### url 重写

```
[root@k8s-master-node1 ~/yaml/test]# vim ingress-url
[root@k8s-master-node1 ~/yaml/test]# cat ingress-url.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: ingress-host-bar
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.chenby.cn"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-server
            port:
              number: 8000
  - host: "demo.chenby.cn"
    http:
      paths:
      - pathType: Prefix
        path: "/nginx(/|$)(.*)"  # 把请求会转给下面的服务，下面的服务一定要能处理这个路径，不能处理就是404
        backend:
          service:
            name: nginx-demo  ## java，比如使用路径重写，去掉前缀nginx
            port:
              number: 8000
[root@k8s-master-node1 ~/yaml/test]# kubectl  apply -f ingress-url.yaml 
ingress.networking.k8s.io/ingress-host-bar created
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# curl hello.chenby.cn
Hostname: ingress-demo-app-694bf5d965-8rh7f
IP: 127.0.0.1
IP: 10.244.1.6
RemoteAddr: 192.168.1.61:42303
GET / HTTP/1.1
Host: hello.chenby.cn
User-Agent: curl/7.68.0
Accept: */*

[root@k8s-master-node1 ~/yaml/test]# curl demo.chenby.cn
Hostname: ingress-demo-app-694bf5d965-swkpb
IP: 127.0.0.1
IP: 10.244.2.4
RemoteAddr: 192.168.1.61:1108
GET / HTTP/1.1
Host: demo.chenby.cn
User-Agent: curl/7.68.0
Accept: */*

[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 流量限制

```
[root@k8s-master-node1 ~/yaml/test]# vim ingress-limit
[root@k8s-master-node1 ~/yaml/test]# cat ingress-limit.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-limit-rate
  annotations:
    nginx.ingress.kubernetes.io/limit-rps: "1"
spec:
  ingressClassName: nginx
  rules:
  - host: "haha.chenby.cn"
    http:
      paths:
      - pathType: Exact
        path: "/"
        backend:
          service:
            name: nginx-demo
            port:
              number: 8000
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl apply -f ingress-limit.yaml 
ingress.networking.k8s.io/ingress-limit-rate created
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# vim /etc/hosts
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# curl haha.chenby.cn
Hostname: ingress-demo-app-694bf5d965-8rh7f
IP: 127.0.0.1
IP: 10.244.1.6
RemoteAddr: 192.168.1.61:1676
GET / HTTP/1.1
Host: haha.chenby.cn
User-Agent: curl/7.68.0
Accept: */*

[root@k8s-master-node1 ~/yaml/test]#
```

  

注：可以将server 改为nodeport 即可在外部访问，将 type 改为 NodePort

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  get service ingress-demo-app 
NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
ingress-demo-app   ClusterIP   10.96.145.40   <none>        80/TCP    6h27m
[root@k8s-master-node1 ~/yaml/test]# kubectl  edit service ingress-demo-app 
service/ingress-demo-app edited
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  get service ingress-demo-app 
NAME               TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
ingress-demo-app   NodePort   10.96.145.40   <none>        80:30975/TCP   6h28m
[root@k8s-master-node1 ~/yaml/test]#
```

  


  

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

知乎、CSDN、开源中国、思否、掘金、哔哩哔哩、腾讯云

  

  

```