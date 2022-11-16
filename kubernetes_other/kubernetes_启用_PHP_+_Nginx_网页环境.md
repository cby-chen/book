---
layout: post
cid: 243
title: kubernetes 启用 PHP + Nginx 网页环境
slug: 243
date: 2022/06/09 09:26:10
updated: 2022/06/09 09:26:10
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: kubernetes 启用 PHP + Nginx 网页环境
keywords: 小陈运维,小陈,小陈博客,文档,
mode: default
thumb: 
video: 
---




# kubernetes 启用 PHP + Nginx 网页环境



传统安装方式进行安装步骤较多，使用kubernetes可以实现快速启用环境，在测试或者线上都可以做到`快速` 启用



### 编写 yaml 文件

```
[root@k8s-master01 ~]# vim PHP-Nginx-Deployment-ConfMap-Service.yaml
[root@k8s-master01 ~]# cat PHP-Nginx-Deployment-ConfMap-Service.yaml
kind: Service # 对象类型
apiVersion: v1 # api 版本
metadata: # 元数据
  name: php-fpm-nginx #Service 服务名
spec:
  type: NodePort # 类型为nodeport
  selector: #标签选择器
    app: php-fpm-nginx 
  ports: #端口信息
    - port: 80  # 容器端口80
      protocol: TCP #tcp类型
      targetPort: 80 # Service 将 nginx 容器的 80 端口暴露出来
---
kind: ConfigMap # 对象类型
apiVersion: v1 # api 版本
metadata: # 元数据
  name: nginx-config # 对象名称
data: # key-value 数据集合
  nginx.conf: | # 将 nginx config 配置写入 ConfigMap 中，经典的 php-fpm 代理设置，这里就不再多说了
    user  nginx;
    worker_processes  auto;
    error_log  /var/log/nginx/error.log notice;
    pid        /var/run/nginx.pid;
    events {
        worker_connections  1024;
    }
    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
        access_log  /var/log/nginx/access.log  main;
        sendfile        on;
        keepalive_timeout  65;
        server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /var/www/html;
        index index.php;
        server_name _;
        if (-f $request_filename/index.html) {
        rewrite (.*) $1/index.html break;
        }
        if (-f $request_filename/index.php) {
        rewrite (.*) $1/index.php;
        }
        if (!-f $request_filename) {
        rewrite (.*) /index.php;
        }
        location / {
          try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
          include fastcgi_params;
          fastcgi_param REQUEST_METHOD $request_method;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          fastcgi_pass 127.0.0.1:9000;
        }
      }
        include /etc/nginx/conf.d/*.conf;
    }
---
kind: Deployment # 对象类型
apiVersion: apps/v1 # api 版本
metadata: # 元数据
  name: php-fpm-nginx # Deployment 对象名称
spec: # Deployment 对象规约
  selector: # 选择器
    matchLabels: # 标签匹配
      app: php-fpm-nginx
  replicas: 3 # 副本数量
  template: # 模版
    metadata: # Pod 对象的元数据
      labels: # Pod 对象的标签
        app: php-fpm-nginx
    spec: # Pod 对象规约
      containers: # 这里设置了两个容器
        - name: php-fpm # 第一个容器名称
          image: php:7.4.29-fpm # 容器镜像
          imagePullPolicy: IfNotPresent #镜像拉取策略
          livenessProbe: # 存活探测
            initialDelaySeconds: 5 # 容器启动后要等待多少秒后才启动存活和就绪探测器
            periodSeconds: 10 # 每多少秒执行一次存活探测
            tcpSocket: # 监测tcp端口
              port: 9000 #监测端口
          readinessProbe: # 就绪探测
            initialDelaySeconds: 5 # 容器启动后要等待多少秒后才启动存活和就绪探测器
            periodSeconds: 10 # 每多少秒执行一次存活探测
            tcpSocket: # 监测tcp端口
              port: 9000 #监测端口
          resources: # 资源约束
            requests: # 最小限制
              memory: "64Mi" # 内存最新64M 
              cpu: "250m" # CPU最大使用0.25核
            limits: # 最大限制
              memory: "128Mi" # 内存最新128M 
              cpu: "500m" # CPU最大使用0.5核
          ports:
            - containerPort: 9000 # php-fpm 端口
          volumeMounts: # 挂载数据卷
            - mountPath: /var/www/html # 挂载两个容器共享的 volume 
              name: nginx-www
          lifecycle: # 生命周期
            postStart: # 当容器处于 postStart 阶段时，执行一下命令
              exec:
                command: ["/bin/sh", "-c", "echo startup..."] # 将 /app/index.php 复制到挂载的 volume 
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5 && kill -SIGQUIT 1 # 优雅退出
        - name: nginx # 第二个容器名称
          image: nginx # 容器镜像
          imagePullPolicy: IfNotPresent
          livenessProbe: # 存活探测
            initialDelaySeconds: 5 # 容器启动后要等待多少秒后才启动存活和就绪探测器
            periodSeconds: 10 # 每多少秒执行一次存活探测
            httpGet: # 以httpGet方式进行探测
              path: / # 探测路径
              port: 80 # 探测端口
          readinessProbe:  # 就绪探测
            initialDelaySeconds: 5 # 容器启动后要等待多少秒后才启动存活和就绪探测器
            periodSeconds: 10 # 每多少秒执行一次存活探测
            httpGet: # 以httpGet方式进行探测
              path: / # 探测路径
              port: 80 # 探测端口
          resources: # 资源约束
            requests: # 最小限制
              memory: "64Mi" # 内存最新64M 
              cpu: "250m" # CPU最大使用0.25核
            limits: # 最大限制
              memory: "128Mi" # 内存最新128M 
              cpu: "500m" # CPU最大使用0.5核
          ports:
            - containerPort: 80 # nginx 端口
          volumeMounts: # nginx 容器挂载了两个 volume，一个是与 php-fpm 容器共享的 volume，另外一个是配置了 nginx.conf 的 volume
            - mountPath: /var/www/html # 挂载两个容器共享的 volume 
              name: nginx-www
            - mountPath: /etc/nginx/nginx.conf #  挂载配置了 nginx.conf 的 volume
              subPath: nginx.conf
              name: nginx-config
          lifecycle:
            preStop:
              exec:
                command:
                  - sh
                  - '-c'
                  - sleep 5 && /usr/sbin/nginx -s quit # 优雅退出
      volumes:
        - name: nginx-www # 网站文件通过nfs挂载
          nfs:
            path: /html/
            server: 192.168.1.123
        - name: nginx-config 
          configMap: # configMap 
            name: nginx-config
```



### 部署网站

```

# 下载网站代码
wget https://typecho.org/downloads/1.1-17.10.30-release.tar.gz

# 解压源码包
tar xvf 1.1-17.10.30-release.tar.gz

#移动到当前目录下
mv build/* .

#设置权限
chmod 777 -R *
```



### 创建资源
```
kubectl  apply -f PHP-Nginx-Deployment-ConfMap-Service.yaml
```



### 测试环境

```
kubectl  get pod -l app=php-fpm-nginx
NAME                            READY   STATUS    RESTARTS        AGE
php-fpm-nginx-8b4bfb457-24bpd   2/2     Running   1 (6m34s ago)   16m
php-fpm-nginx-8b4bfb457-fvqd6   2/2     Running   2 (5m39s ago)   16m
php-fpm-nginx-8b4bfb457-kmzsc   2/2     Running   1 (6m34s ago)   16m

kubectl get configmaps  | grep nginx
NAME               DATA   AGE
nginx-config       1      17m

kubectl get svc | grep nginx
php-fpm-nginx   NodePort    10.98.66.104     <none>        80:31937/TCP   16m
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea12c829c0e34ca1839b623334197ca9~tplv-k3u1fbpfcp-zoom-1.image)



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
>
> CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、今日头条、个人博客、全网可搜《小陈运维》
>
> 文章主要发布于微信：《Linux运维交流社区》


