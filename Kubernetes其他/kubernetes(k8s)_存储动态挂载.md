---
layout: post
cid: 32
title: kubernetes(k8s) 存储动态挂载
slug: 32
date: 2021/12/30 17:07:00
updated: 2022/06/07 15:24:47
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


使用 nfs 文件系统 实现kubernetes存储动态挂载

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dba1e41aadeb45279e8a921040b9cdd1~tplv-k3u1fbpfcp-zoom-1.image)

  

1\. 安装服务端和客户端

  

```
root@hello:~# apt install nfs-kernel-server nfs-common

```

其中 nfs-kernel-server 为服务端，　nfs-common 为客户端。

  

2\. 配置 nfs 共享目录

  

```
root@hello:~# mkdir /nfs
root@hello:~# sudo vim /etc/exports
/nfs *(rw,sync,no_root_squash,no_subtree_check)
```

  

```
各字段解析如下：
/nfs: 要共享的目录
：指定可以访问共享目录的用户 ip, * 代表所有用户。192.168.3.　指定网段。192.168.3.29 指定 ip。
rw：可读可写。如果想要只读的话，可以指定 ro。
sync：文件同步写入到内存与硬盘中。
async：文件会先暂存于内存中，而非直接写入硬盘。
no_root_squash：登入 nfs 主机使用分享目录的使用者，如果是 root 的话，那么对于这个分享的目录来说，他就具有 root 的权限！这个项目『极不安全』，不建议使用！但如果你需要在客户端对 nfs 目录进行写入操作。你就得配置 no_root_squash。方便与安全不可兼得。
root_squash：在登入 nfs 主机使用分享之目录的使用者如果是 root 时，那么这个使用者的权限将被压缩成为匿名使用者，通常他的 UID 与 GID 都会变成 nobody 那个系统账号的身份。
subtree_check：强制 nfs 检查父目录的权限（默认）
no_subtree_check：不检查父目录权限
```

  

配置完成后，执行以下命令导出共享目录，并重启 nfs 服务:

  

```
root@hello:~# exportfs -a      
root@hello:~# systemctl restart nfs-kernel-server
root@hello:~#
root@hello:~# systemctl enable nfs-kernel-server
```

  

客户端挂载

  

```
root@hello:~# apt install nfs-common
root@hello:~# mkdir -p /nfs/
root@hello:~# mount -t nfs 192.168.1.66:/nfs/ /nfs/
```

  

```
root@hello:~# df -hT
Filesystem                        Type      Size  Used Avail Use% Mounted on
udev                              devtmpfs  7.8G     0  7.8G   0% /dev
tmpfs                             tmpfs     1.6G  2.9M  1.6G   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4       97G  9.9G   83G  11% /
tmpfs                             tmpfs     7.9G     0  7.9G   0% /dev/shm
tmpfs                             tmpfs     5.0M     0  5.0M   0% /run/lock
tmpfs                             tmpfs     7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/loop0                        squashfs   56M   56M     0 100% /snap/core18/2128
/dev/loop1                        squashfs   56M   56M     0 100% /snap/core18/2246
/dev/loop3                        squashfs   33M   33M     0 100% /snap/snapd/12704
/dev/loop2                        squashfs   62M   62M     0 100% /snap/core20/1169
/dev/loop4                        squashfs   33M   33M     0 100% /snap/snapd/13640
/dev/loop6                        squashfs   68M   68M     0 100% /snap/lxd/21835
/dev/loop5                        squashfs   71M   71M     0 100% /snap/lxd/21029
/dev/sda2                         ext4      976M  107M  803M  12% /boot
tmpfs                             tmpfs     1.6G     0  1.6G   0% /run/user/0
192.168.1.66:/nfs                 nfs4       97G  6.4G   86G   7% /nfs
```

  

创建配置默认存储

  

```
[root@k8s-master-node1 ~/yaml]# vim nfs-storage.yaml
[root@k8s-master-node1 ~/yaml]#
[root@k8s-master-node1 ~/yaml]# cat nfs-storage.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
parameters:
  archiveOnDelete: "true"  ## 删除pv的时候，pv的内容是否要备份


---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: registry.cn-hangzhou.aliyuncs.com/chenby/nfs-subdir-external-provisioner:v4.0.2
          # resources:
          #    limits:
          #      cpu: 10m
          #    requests:
          #      cpu: 10m
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: 192.168.1.66 ## 指定自己nfs服务器地址
            - name: NFS_PATH  
              value: /nfs/  ## nfs服务器共享的目录
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.1.66
            path: /nfs/
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

  

创建

  

```
[root@k8s-master-node1 ~/yaml]# kubectl apply -f nfs-storage.yaml
storageclass.storage.k8s.io/nfs-storage created
deployment.apps/nfs-client-provisioner created
serviceaccount/nfs-client-provisioner created
clusterrole.rbac.authorization.k8s.io/nfs-client-provisioner-runner created
clusterrolebinding.rbac.authorization.k8s.io/run-nfs-client-provisioner created
role.rbac.authorization.k8s.io/leader-locking-nfs-client-provisioner created
rolebinding.rbac.authorization.k8s.io/leader-locking-nfs-client-provisioner created
[root@k8s-master-node1 ~/yaml]#
```

  
  

查看是否创建默认存储

  

```
[root@k8s-master-node1 ~/yaml]# kubectl get storageclasses.storage.k8s.io
NAME                    PROVISIONER                                   RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-storage (default)   k8s-sigs.io/nfs-subdir-external-provisioner   Delete          Immediate           false                  100s
[root@k8s-master-node1 ~/yaml]#
```

  

创建pvc进行测试

  

```
[root@k8s-master-node1 ~/yaml]# vim pvc.yaml
[root@k8s-master-node1 ~/yaml]# cat pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
[root@k8s-master-node1 ~/yaml]#
[root@k8s-master-node1 ~/yaml]# kubectl apply -f pvc.yaml
persistentvolumeclaim/nginx-pvc created
[root@k8s-master-node1 ~/yaml]#
```

  

查看pvc

  

```
[root@k8s-master-node1 ~/yaml]#
[root@k8s-master-node1 ~/yaml]# kubectl get pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-pvc   Bound    pvc-8a4b6065-904a-4bae-bef9-1f3b5612986c   200Mi      RWX            nfs-storage    4s
[root@k8s-master-node1 ~/yaml]#
```

  

查看pv

  

```
[root@k8s-master-node1 ~/yaml]# kubectl  get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
pvc-8a4b6065-904a-4bae-bef9-1f3b5612986c   200Mi      RWX            Delete           Bound    default/nginx-pvc   nfs-storage             103s
[root@k8s-master-node1 ~/yaml]#
```

  
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