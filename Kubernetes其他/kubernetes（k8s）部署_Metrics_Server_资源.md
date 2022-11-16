---
layout: post
cid: 127
title: kubernetes（k8s）部署 Metrics Server 资源
slug: 127
date: 2022/03/28 15:42:00
updated: 2022/03/28 19:56:12
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


    资源使用指标，例如容器 CPU 和内存使用率，可通过 Metrics API 在 Kubernetes 中获得。这些指标可以直接被用户访问，比如使用 kubectl top 命令行，或者被集群中的控制器 （例如 Horizontal Pod Autoscalers) 使用来做决策。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff3fe4c5534f4058b1e49d88f5ae978c~tplv-k3u1fbpfcp-zoom-1.image)

  

**配置api聚合层**

  

```
添加配置api启动service文件

--enable-aggregator-routing=true 

ps -ef |grep apiserver|grep enable-aggregator-routing
root        1147       1 10 10:23 ?        00:30:13 /usr/local/bin/kube-apiserver --v=2 --logtostderr=true --allow-privileged=true --bind-address=0.0.0.0 --secure-port=6443 --insecure-port=0 --advertise-address=192.168.1.30 --service-cluster-ip-range=10.96.0.0/12 --service-node-port-range=30000-32767 --etcd-servers=https://192.168.1.30:2379,https://192.168.1.31:2379,https://192.168.1.32:2379 --etcd-cafile=/etc/etcd/ssl/etcd-ca.pem --etcd-certfile=/etc/etcd/ssl/etcd.pem --etcd-keyfile=/etc/etcd/ssl/etcd-key.pem --client-ca-file=/etc/kubernetes/pki/ca.pem --tls-cert-file=/etc/kubernetes/pki/apiserver.pem --tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem --kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem --kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-account-issuer=https://kubernetes.default.svc.cluster.local --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota --authorization-mode=Node,RBAC --enable-bootstrap-token-auth=true --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client-key.pem --requestheader-allowed-names=aggregator --requestheader-group-headers=X-Remote-Group --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-username-headers=X-Remote-User --enable-aggregator-routing=true
```

  

**创建应用权限 RBAC 资源文件**

  

```
root@hello:~/cby# cat metrics-rbac.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:aggregated-metrics-reader
  labels:
    rbac.authorization.k8s.io/aggregate-to-view: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
rules:
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods", "nodes"]
    verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:metrics-server
rules:
  - apiGroups:
      - ""
    resources:
      - pods
      - nodes
      - nodes/stats
      - namespaces
    verbs:
      - get
      - list
      - watch

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: kube-system
root@hello:~/cby#
```

  

**创建 APIService 资源文件**

  

    设置扩展 API Service 工作于聚合层，允许使用其 API 扩展 Kubernetes apiserver，而这些 API 并不是核心 Kubernetes API 的一部分。这里部署 APIservice 资源，来提供 Kubernetes Metrics 指标 API 数据。

  

```
root@hello:~/cby# cat metrics-api-service.yaml 
## APIService
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  name: v1beta1.metrics.k8s.io
spec:
  service:
    name: metrics-server
    namespace: kube-system
    port: 443
  group: metrics.k8s.io
  version: v1beta1
  insecureSkipTLSVerify: true
  groupPriorityMinimum: 100
  versionPriority: 100
root@hello:~/cby#
```

  

**创建 Metrics Server 应用资源文件**

  

```
root@hello:~/cby# cat metrics-server-deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
    spec:
      serviceAccountName: metrics-server
      volumes:
        # mount in tmp so we can safely use from-scratch images and/or read-only containers
        - name: tmp-dir
          emptyDir: {}
      hostNetwork: true
      containers:
        - name: metrics-server
          image: bitnami/metrics-server
          # command:
          # - /metrics-server
          # - --kubelet-insecure-tls
          # - --kubelet-preferred-address-types=InternalIP
          args:
            - --cert-dir=/tmp
            - --secure-port=4443
            - --kubelet-insecure-tls=true
            - --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,externalDNS
          ports:
            - name: main-port
              containerPort: 4443
              protocol: TCP
          securityContext:
            readOnlyRootFilesystem: true
            runAsNonRoot: true
            runAsUser: 1000
          imagePullPolicy: Always
          volumeMounts:
            - name: tmp-dir
              mountPath: /tmp
      nodeSelector:
        beta.kubernetes.io/os: linux

---
apiVersion: v1
kind: Service
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    kubernetes.io/name: "Metrics-server"
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    k8s-app: metrics-server
  ports:
    - port: 443
      protocol: TCP
      targetPort: 4443
root@hello:~/cby#
```

  

**通过 Kubectl 命令部署**

  

```
root@hello:~/cby# kubectl apply -f metrics-rbac.yaml -n kube-system
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
serviceaccount/metrics-server created


root@hello:~/cby# kubectl apply -f metrics-api-service.yaml -n kube-system
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created


root@hello:~/cby# kubectl apply -f metrics-server-deploy.yaml -n kube-system
Warning: spec.template.spec.nodeSelector[beta.kubernetes.io/os]: deprecated since v1.14; use "kubernetes.io/os" instead
deployment.apps/metrics-server created
service/metrics-server created
root@hello:~/cby# 

验证

root@hello:~/cby# kubectl  get pod -A | grep metrics-server
kube-system   metrics-server-5c69d5d5b7-b6246              1/1     Running   0               2m25s
root@hello:~/cby# 

查看日志

root@hello:~/cby# kubectl  logs -n kube-system   metrics-server-5c69d5d5b7-b6246
I0328 07:11:37.676490       1 serving.go:341] Generated self-signed cert (/tmp/apiserver.crt, /tmp/apiserver.key)
I0328 07:11:38.148457       1 requestheader_controller.go:169] Starting RequestHeaderAuthRequestController
I0328 07:11:38.148472       1 configmap_cafile_content.go:202] Starting client-ca::kube-system::extension-apiserver-authentication::client-ca-file
I0328 07:11:38.148507       1 shared_informer.go:240] Waiting for caches to sync for client-ca::kube-system::extension-apiserver-authentication::client-ca-file
I0328 07:11:38.148475       1 configmap_cafile_content.go:202] Starting client-ca::kube-system::extension-apiserver-authentication::requestheader-client-ca-file
I0328 07:11:38.148550       1 shared_informer.go:240] Waiting for caches to sync for client-ca::kube-system::extension-apiserver-authentication::requestheader-client-ca-file
I0328 07:11:38.148490       1 shared_informer.go:240] Waiting for caches to sync for RequestHeaderAuthRequestController
I0328 07:11:38.149073       1 dynamic_serving_content.go:130] Starting serving-cert::/tmp/apiserver.crt::/tmp/apiserver.key
I0328 07:11:38.149428       1 secure_serving.go:202] Serving securely on [::]:4443
I0328 07:11:38.149535       1 tlsconfig.go:240] Starting DynamicServingCertificateController
I0328 07:11:38.248713       1 shared_informer.go:247] Caches are synced for RequestHeaderAuthRequestController 
I0328 07:11:38.248732       1 shared_informer.go:247] Caches are synced for client-ca::kube-system::extension-apiserver-authentication::requestheader-client-ca-file 
I0328 07:11:38.248754       1 shared_informer.go:247] Caches are synced for client-ca::kube-system::extension-apiserver-authentication::client-ca-file 
root@hello:~/cby#

查看node资源信息

root@hello:~/cby# kubectl  top node 
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
192.168.1.50   184m         2%     4354Mi          56%       
192.168.1.51   207m         2%     3892Mi          50%       
192.168.1.52   197m         2%     3881Mi          50%       
192.168.1.53   185m         2%     3528Mi          46%       
192.168.1.54   109m         1%     3427Mi          44%       
root@hello:~/cby# 

查看pod资源信息

root@hello:~/cby# kubectl  top pod -n kube-system 
NAME                                         CPU(cores)   MEMORY(bytes)   
calico-kube-controllers-754966f84c-jm7f7     5m           25Mi            
calico-node-9tvck                            43m          69Mi            
calico-node-kt2pk                            41m          68Mi            
calico-node-skm82                            45m          70Mi            
calico-node-t4lhb                            44m          65Mi            
calico-node-tz5k9                            45m          66Mi            
coredns-596755dbff-7ggzl                     3m           15Mi            
dashboard-metrics-scraper-799d786dbf-s6r5f   1m           7Mi             
kubernetes-dashboard-9f8c8b989-57fhz         1m           13Mi            
metrics-server-5c69d5d5b7-b6246              4m           16Mi            
node-local-dns-4hzvh                         5m           17Mi            
node-local-dns-6zpdh                         3m           17Mi            
node-local-dns-9jmzz                         5m           16Mi            
node-local-dns-q8pcw                         5m           17Mi            
node-local-dns-tpm6b                         5m           29Mi            
root@hello:~/cby# 

查看单个pod资源信息

root@hello:~/cby# kubectl  top pod  -n kube-system metrics-server-5c69d5d5b7-b6246
NAME                              CPU(cores)   MEMORY(bytes)   
metrics-server-5c69d5d5b7-b6246   4m           13Mi            
root@hello:~/cby# 

```

  

https://www.oiox.cn/

https://www.chenby.cn/

https://cby-chen.github.io/

https://weibo.com/u/5982474121

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

https://www.jianshu.com/u/0f894314ae2c

https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/

CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客、全网可搜《小陈运维》
