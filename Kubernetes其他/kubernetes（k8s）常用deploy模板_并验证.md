---
layout: post
cid: 209
title: kubernetes（k8s）常用deploy模板 并验证
slug: 209
date: 2022/05/18 10:14:00
updated: 2022/06/03 15:17:14
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


kubernetes常用deploy模板，并验证
========================

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcf3087eeb74496cbe982ff78aef0ac3~tplv-k3u1fbpfcp-zoom-1.image)

编写deploy配置文件
============

```
root@hello:~# cat deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: hostname-test-cby
  labels:
    name: hostname-test-cby
spec:
  # 副本数
  replicas: 10
  # 标签选择器
  selector:
    matchLabels:
      name: hostname-test-cby
  # 更新策略
  strategy:
    rollingUpdate:
      maxSurge: 3    # 更新最大数量
      maxUnavailable: 3    #更新时最大不可用数量
    type: RollingUpdate  #滚动更新
  # 模板
  template:
    metadata:
      labels:
        name: hostname-test-cby
    spec:
      # 配置容器
      containers:
      - name: hostname-test-cby #容器名
        image: nginx #镜像
        imagePullPolicy: IfNotPresent # 拉取策略
        resources:
          requests:
            cpu: "100m" #CPU限制
            memory: "300M" #内存限制
        # 健康监测
        livenessProbe:
          httpGet:
            path: / # 探测路径
            port: 80 # 端口
          initialDelaySeconds: 15 # 第一次探测等待
          timeoutSeconds: 3 # 探测的超时后等待多少秒
        # 就绪探测
        readinessProbe:
          httpGet:
            path: / # 探测路径
            port: 80 # 端口
          initialDelaySeconds: 10 # 第一次探测等待
          timeoutSeconds: 3 # 探测的超时后等待多少秒
        #环境变量
        env:
        - name: cby
          value: chenby
        # 配置容器端口
        ports:
        - containerPort: 80
        # 配置挂载到目录
        volumeMounts:
        - mountPath: /usr/share/nginx/html/
          name: data
      # 配置目录挂载
      volumes:
      - name: data
        hostPath:
          path: /html/
          type: Directory
      # 配置指定解析
      hostAliases:
      - ip: "192.168.1.1" #IP地址
        hostnames:
        - "cby" #主机名
        - "cby.chenby.cn" #主机名
      - ip: "192.168.1.10"#IP地址
        hostnames:
        - "chenby" #主机名
        - "chenby.chenby.cn" #主机名
root@hello:~# 

```

执行deploy配置文件
============

```
root@hello:~# kubectl apply -f deploy.yaml 
deployment.apps/hostname-test-cby created

root@hello:~# mkdir /html
root@hello:~# echo 123 > /html/index.html

root@hello:~# kubectl  get pod  -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP              NODE         NOMINATED NODE   READINESS GATES
hostname-test-cby-86df45bf-9fx5n   1/1     Running   0          43s   172.17.125.38   k8s-node01   <none>           <none>
hostname-test-cby-86df45bf-cmv2b   1/1     Running   0          43s   172.17.125.37   k8s-node01   <none>           <none>
hostname-test-cby-86df45bf-f6drb   1/1     Running   0          43s   172.17.125.41   k8s-node01   <none>           <none>
hostname-test-cby-86df45bf-g79x2   1/1     Running   0          43s   172.27.14.232   k8s-node02   <none>           <none>
hostname-test-cby-86df45bf-h6blv   1/1     Running   0          43s   172.27.14.233   k8s-node02   <none>           <none>
hostname-test-cby-86df45bf-hqjnj   1/1     Running   0          43s   172.17.125.40   k8s-node01   <none>           <none>
hostname-test-cby-86df45bf-jt2rz   1/1     Running   0          43s   172.27.14.236   k8s-node02   <none>           <none>
hostname-test-cby-86df45bf-s5jjn   1/1     Running   0          43s   172.27.14.235   k8s-node02   <none>           <none>
hostname-test-cby-86df45bf-vfkbt   1/1     Running   0          43s   172.17.125.39   k8s-node01   <none>           <none>
hostname-test-cby-86df45bf-z2x2b   1/1     Running   0          43s   172.27.14.234   k8s-node02   <none>           <none>
root@hello:~# 

```

进入pod进行检查
=========

```
# 访问测试
root@hello:~# curl 172.17.125.38
123
root@hello:~# 

root@hello:~# kubectl  exec hostname-test-cby-86df45bf-9fx5n -it -- /bin/bash 
root@hostname-test-cby-86df45bf-9fx5n:/# 

# 查看dns解析
root@hostname-test-cby-86df45bf-9fx5n:/# cat /etc/resolv.conf 
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
root@hostname-test-cby-86df45bf-9fx5n:/# 

# 查看host配置已生效
root@hostname-test-cby-86df45bf-9fx5n:/# cat /etc/hosts 
# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
172.27.14.197   hostname-test-cby-86df45bf-9fx5n

# Entries added by HostAliases.
192.168.1.1     cby     cby.chenby.cn
192.168.1.10    chenby  chenby.chenby.cn
root@hostname-test-cby-86df45bf-9fx5n:/#

# 查看环境变量
root@hostname-test-cby-86df45bf-9fx5n:/# echo $cby
chenby
root@hostname-test-cby-86df45bf-9fx5n:/#

```

> https://www.oiox.cn/
> https://www.chenby.cn/
> https://cby-chen.github.io/
> https://blog.csdn.net/qq_33921750
> https://my.oschina.net/u/3981543
> https://www.zhihu.com/people/chen-bu-yun-2
> https://segmentfault.com/u/hppyvyv6/articles
> https://juejin.cn/user/3315782802482007
> https://cloud.tencent.com/developer/column/93230
> https://www.jianshu.com/u/0f894314ae2c
> https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/
> CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、今日头条、个人博客、全网可搜《小陈运维》
> 文章主要发布于微信：《Linux运维交流社区》

