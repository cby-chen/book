---
layout: post
cid: 228
title: 创建用户认证授权的 kubeconfig 文件
slug: 228
date: 2022/05/25 18:52:37
updated: 2022/05/25 18:52:37
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: 创建用户认证授权的 kubeconfig 文件
keywords: Kubernetes,k8s,二进制,
mode: default
thumb: 
video: 
---


# 创建用户认证授权的 kubeconfig 文件

当我们安装好集群后，如果想要把 kubectl 命令交给用户使用，就不得不对用户的身份进行认证和对其权限做出限制。

下面以创建一个 cby 用户并将其绑定到 cby 和 chenby 的 namespace 为例说明。

## 创建生成证书配置文件

```
详细见：https://github.com/cby-chen/Kubernetes#23%E5%88%9B%E5%BB%BA%E8%AF%81%E4%B9%A6%E7%9B%B8%E5%85%B3%E6%96%87%E4%BB%B6

cat > ca-config.json << EOF 
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "876000h"
      }
    }
  }
}
EOF


cat > cby-csr.json << EOF 
{
  "CN": "cby",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "system:masters",
      "OU": "Kubernetes-manual"
    }
  ]
}
EOF
```



## 生成 CA 证书和私钥

```
cfssl gencert \
   -ca=/etc/kubernetes/pki/ca.pem \
   -ca-key=/etc/kubernetes/pki/ca-key.pem \
   -config=ca-config.json \
   -profile=kubernetes \
   cby-csr.json | cfssljson -bare /etc/kubernetes/pki/cby




ll /etc/kubernetes/pki/cby*
-rw-r--r-- 1 root root 1021 May 25 17:36 /etc/kubernetes/pki/cby.csr
-rw------- 1 root root 1679 May 25 17:36 /etc/kubernetes/pki/cby-key.pem
-rw-r--r-- 1 root root 1440 May 25 17:36 /etc/kubernetes/pki/cby.pem
```



## 创建 kubeconfig 文件

```
kubectl config set-cluster kubernetes     \
  --certificate-authority=/etc/kubernetes/pki/ca.pem     \
  --embed-certs=true     \
  --server=https://10.0.0.89:8443     \
  --kubeconfig=/etc/kubernetes/cby.kubeconfig

kubectl config set-credentials cby  \
  --client-certificate=/etc/kubernetes/pki/cby.pem     \
  --client-key=/etc/kubernetes/pki/cby-key.pem     \
  --embed-certs=true     \
  --kubeconfig=/etc/kubernetes/cby.kubeconfig

kubectl config set-context cby@kubernetes    \
  --cluster=kubernetes     \
  --user=cby     \
  --kubeconfig=/etc/kubernetes/cby.kubeconfig

kubectl config use-context cby@kubernetes  --kubeconfig=/etc/kubernetes/cby.kubeconfig
```



## 添加用户并将配置其用户

```
useradd cby
su - cby
mkdir .kube/
exit 
cp /etc/kubernetes/cby.kubeconfig  /home/cby/.kube/config
chown cby.cby /home/cby/.kube/config
```



## RoleBinding

需要使用 RBAC创建角色绑定以将该用户的行为限制在某个或某几个 namespace 空间范围内

```
kubectl create namespace cby
kubectl create namespace chenby
kubectl create rolebinding cby --clusterrole=cluster-admin --user=cby --namespace=cby
kubectl create rolebinding cby --clusterrole=cluster-admin --user=cby --namespace=chenby

kubectl  describe -n chenby rolebindings.rbac.authorization.k8s.io cby 
Name:         cby
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind  Name  Namespace
  ----  ----  ---------
  User  cby   

kubectl  describe -n cby rolebindings.rbac.authorization.k8s.io cby 
Name:         cby
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind  Name  Namespace
  ----  ----  ---------
  User  cby   

su - cby
```



## 获取当前的 context

```
kubectl config get-contexts
CURRENT   NAME                        CLUSTER      AUTHINFO         NAMESPACE

*         kubernetes-cby@kubernetes   kubernetes   kubernetes-cby   cby
```



## 无法访问 default namespace

```
[cby@k8s-master01 ~]$ kubectl get pods --namespace default
Error from server (Forbidden): pods is forbidden: User "cby" cannot list resource "pods" in API group "" in the namespace "default"
[cby@k8s-master01 ~]$ 
```



## 可以访问 cby namespace

这样 cby 用户对 cby 和 chenby 两个 namespace 具有完全访问权限。

```
[cby@k8s-master01 ~]$ kubectl get pods --namespace cby
No resources found in cby namespace.
[cby@k8s-master01 ~]$ kubectl get pods --namespace chenby
No resources found in chenby namespace.
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