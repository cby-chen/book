---
layout: post
cid: 268
title: 在Kubernetes（k8s）中部署 jenkins
slug: 268
date: 2022/06/21 09:54:20
updated: 2022/06/21 09:54:20
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: 在Kubernetes（k8s）中部署 jenkins
keywords: Kubernetes,k8s,二进制,
mode: default
thumb: 
video: 
---


在Kubernetes（k8s）中部署 jenkins
===========================

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fd830306fba4d5eb8aac7ce67221e8b~tplv-k3u1fbpfcp-zoom-1.image)

### YAML配置文件

由于jenkins需要持久化存储，通过nfs动态供给pvc存储卷。

可以参考我之前的文档：https://cloud.tencent.com/developer/article/1902519

```
vim jenkins-deploy.yaml
cat jenkins-deploy.yaml
###############使用 storageClass 创建 pvc ###################
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-data-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi

###############创建一个ServiceAccount 名称为：jenkins-admin###################
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin
  namespace: default
  labels:
    name: jenkins

###############绑定账户jenkins-admin 为集群管理员角色，为了控制权限建议绑定自定义角色###################
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: jenkins-admin
  labels:
    name: jenkins
subjects:
  - kind: ServiceAccount
    name: jenkins-admin
    namespace: default
roleRef:
  kind: ClusterRole
  # cluster-admin 是 k8s 集群中默认的管理员角色
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io


############### 在 default 命名空间创建 deployment ###################
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      terminationGracePeriodSeconds: 10
      # 注意：k8s 1.21.x 中 serviceAccount 改名为 serviceAccountName
      # 这里填写上面创建的 serviceAccount 的 name
      serviceAccount: jenkins-admin
      containers:
        - name: jenkins
          image: jenkins/jenkins:latest
          imagePullPolicy: IfNotPresent
          env:
            - name: JAVA_OPTS
              value: -Duser.timezone=Asia/Shanghai
          ports:
            - containerPort: 8080
              name: web
              protocol: TCP
            - containerPort: 50000
              name: agent
              protocol: TCP
          resources:
            limits:
              cpu: 1000m
              memory: 1Gi
            requests:
              cpu: 500m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 60
            timeoutSeconds: 5
            failureThreshold: 12
          readinessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 60
            timeoutSeconds: 5
            failureThreshold: 12
          volumeMounts:
            - name: jenkinshome
              mountPath: /var/jenkins_home
      volumes:
        - name: jenkinshome
          persistentVolumeClaim:
            claimName: jenkins-data-pvc

############### 在 default 命名空间创建 service ###################
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: default
  labels:
    app: jenkins
spec:
  selector:
    app: jenkins
  type: ClusterIP
  ports:
    - name: web
      port: 8080
      targetPort: 8080


---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-agent
  namespace: default
  labels:
    app: jenkins
spec:
  selector:
    app: jenkins
  type: ClusterIP
  ports:
    - name: agent
      port: 50000
      targetPort: 50000
```



### 执行部署

```
kubectl apply -f jenkins-deploy.yaml
persistentvolumeclaim/jenkins-data-pvc created
serviceaccount/jenkins-admin created
clusterrolebinding.rbac.authorization.k8s.io/jenkins-admin created
deployment.apps/jenkins created
service/jenkins created
service/jenkins-agent created
```



### 访问测试

```
# 查看svc
kubectl  get svc | grep jenkins
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
jenkins         ClusterIP   10.99.124.103   <none>        8080/TCP       3m7s
jenkins-agent   ClusterIP   10.98.21.139    <none>        50000/TCP      3m6s

# 修改为NodePort
kubectl  edit svc jenkins
type: NodePort

# 查看修改后的svc端口
kubectl  get svc | grep jenkins
jenkins         NodePort    10.99.124.103   <none>        8080:31613/TCP   4m24s
jenkins-agent   ClusterIP   10.98.21.139    <none>        50000/TCP        4m23s
```



### 查看密码

```
# 查看pod名称
kubectl get pod -n default | grep jenkins
jenkins-7db75dbcb9-76l7l                  1/1     Running   0          5m11s

# 查看默认密码
kubectl  exec jenkins-7db75dbcb9-76l7l -- cat /var/jenkins_home/secrets/initialAdminPassword
a9b2d13bc4c9453f93bb83e43a780f7c
```



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