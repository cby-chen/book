---
layout: post
cid: 250
title: 在 Kubernetes 集群上部署 VSCode
slug: 250
date: 2022/06/13 09:17:56
updated: 2022/06/13 09:17:56
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: 在 Kubernetes 集群上部署 VSCode
keywords: 小陈运维,小陈,小陈博客,文档,
mode: default
thumb: 
video: 
---


# 在 Kubernetes 集群上部署 VSCode



### Visual Studio Code 

Visual Studio Code 是一个轻量级但功能强大的源代码编辑器，可在您的桌面上运行，适用于 Windows、macOS 和 Linux。它内置了对 JavaScript、TypeScript 和 Node.js 的支持，并为其他语言（如 C++、C#、Java、Python、PHP、Go）和运行时（如 .NET 和 Unity）提供了丰富的扩展生态系统.



开发工具来说云端 IDE 也逐渐受到大家重视，Visual Studio Code 有官方web版本，由于访问不太稳定可以借助Code-Server部署在本地环境。

官方地址：https://vscode.dev/ 



### 传统方式安装

```
# 安装
curl -fsSL https://code-server.dev/install.sh | sh

# 查看配置
cat .config/code-server/config.yaml 
bind-addr: 0.0.0.0:8080
auth: password
password: c5d4b8deec690d04e81ef0d5
cert: false
```



### docker方式安装

```
# 启用容器
mkdir -p ~/.config
docker run -d --name code-server  \
-p 8080:8080   \
-v "$HOME/.config:/home/coder/.config"   \
-v "$PWD:/home/coder/project"   \
-u "$(id -u):$(id -g)"   \
-e "DOCKER_USER=$USER"  \
codercom/code-server:latest  

# 查看密码
docker exec -it code-server  cat ~/.config/code-server/config.yaml
bind-addr: 127.0.0.1:8080
auth: password
password: cca029c905426a228d46d3ea
cert: false
```



### kubernetes方式安装

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: code-server
---
apiVersion: v1
kind: Service
metadata:
  name: code-server
  namespace: code-server
spec:
  type: NodePort
  selector:
    app: code-server
  ports:
  - port: 80
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: code-server
  namespace: code-server
  labels:
    app: code-server
spec:
  replicas: 3
  strategy:
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 3
    type: RollingUpdate
  selector:
    matchLabels:
      app: code-server
  template:
    metadata:
      labels:
        app: code-server
    spec:
      containers:
      - name: code-server
        image: codercom/code-server
        imagePullPolicy: IfNotPresent
        env:
        - name: PASSWORD
          value: "123123"
        resources:
          limits:
            memory: "512Mi"
            cpu: "4096m"
        ports:
        - containerPort: 8080
```

### kubernetes方式验证测试
```
kubectl  get svc -n code-server 
NAME          TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
code-server   NodePort   10.97.52.132   <none>        80:31274/TCP   2d21h

curl -I 192.168.1.61:31274
HTTP/1.1 302 Found
Location: ./login
Vary: Accept, Accept-Encoding
Content-Type: text/plain; charset=utf-8
Content-Length: 29
Date: Mon, 13 Jun 2022 01:11:16 GMT
Connection: keep-alive
Keep-Alive: timeout=5

```


> **关于**
>
> https://www.oiox.cn/
>
> https://www.oiox.cn/index.php/start-page.html
>
> **CSDN、GitHub、知乎、微信、开源中国、思否、掘金、简书、华为云、阿里云、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客、全网可搜《小陈运维》**
>
> **文章主要发布于微信**