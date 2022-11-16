---
layout: post_draft
cid: 221
title: kubernetes核心实战（四）--- Deployments
slug: @221
date: 2021/12/30 17:12:00
updated: 2022/05/19 15:59:24
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


#### 6、Deployments（重点）

一个 Deployment 控制器为 Pods和 ReplicaSets提供描述性的更新方式。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47311f0a2c0c4c7d885590895bd3c03d~tplv-k3u1fbpfcp-zoom-1.image)

描述 Deployment 中的 desired state，并且 Deployment 控制器以受控速率更改实际状态，以达到期望状态。可以定义 Deployments 以创建新的 ReplicaSets ，或删除现有 Deployments ，并通过新的 Deployments 使用其所有资源。

  

**用例**

以下是典型的 Deployments 用例：

  

创建 Deployment 以展开 ReplicaSet 。ReplicaSet 在后台创建 Pods。检查 ReplicaSet 展开的状态，查看其是否成功。

声明 Pod 的新状态 通过更新 Deployment 的 PodTemplateSpec。将创建新的 ReplicaSet ，并且 Deployment 管理器以受控速率将 Pod 从旧 ReplicaSet 移动到新 ReplicaSet 。每个新的 ReplicaSet 都会更新 Deployment 的修改历史。

回滚到较早的 Deployment 版本，如果 Deployment 的当前状态不稳定。每次回滚都会更新 Deployment 的修改。

扩展 Deployment 以承担更多负载.

暂停 Deployment 对其 PodTemplateSpec 进行修改，然后恢复它以启动新的展开。

使用 Deployment 状态 作为卡住展开的指示器。

清理较旧的 ReplicaSets ，那些不再需要的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6087ce839fea44a7b1eff7f2e317c366~tplv-k3u1fbpfcp-zoom-1.image)

##### 1）创建 Deployment

```
[root@k8s-master-node1 ~/yaml/test]# vim deployments.yaml
[root@k8s-master-node1 ~/yaml/test]# cat deployments.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl apply -f deployments.yaml 
deployment.apps/nginx-deployment created
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 含义介绍：

在该例中：将创建名为 nginx-deployment 的 Deployment ，由 .metadata.name 字段指示。

Deployment 创建三个复制的 Pods，由 replicas 字段指示。

selector 字段定义 Deployment 如何查找要管理的 Pods。在这种情况下，只需选择在 Pod 模板（app: nginx）中定义的标签。但是，更复杂的选择规则是可能的，只要 Pod 模板本身满足规则。

  

###### 说明：

`matchLabels` 字段是 {key,value} 的映射。单个 {key,value}在 `matchLabels` 映射中的值等效于 `matchExpressions` 的元素，其键字段是“key”，运算符为“In”，值数组仅包含“value”。所有要求，从 `matchLabels` 和 `matchExpressions`，必须满足才能匹配。

  

###### template 字段包含以下子字段：

Pod 标记为app: nginx，使用labels字段。

Pod 模板规范或 .template.spec 字段指示 Pods 运行一个容器， nginx，运行 nginx Docker Hub版本1.7.9的镜像 。

创建一个容器并使用name字段将其命名为 nginx。

  

###### 查看详细的字段解释：

```
[root@k8s-master-node1 ~]# kubectl  explain Deployment.spec
KIND:     Deployment
VERSION:  apps/v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the Deployment.

     DeploymentSpec is the specification of the desired behavior of the
     Deployment.

FIELDS:
   minReadySeconds<integer>
     Minimum number of seconds for which a newly created pod should be ready
     without any of its container crashing, for it to be considered available.
     Defaults to 0 (pod will be considered available as soon as it is ready)

   paused<boolean>
     Indicates that the deployment is paused.

   progressDeadlineSeconds<integer>
     The maximum time in seconds for a deployment to make progress before it is
     considered to be failed. The deployment controller will continue to process
     failed deployments and a condition with a ProgressDeadlineExceeded reason
     will be surfaced in the deployment status. Note that progress will not be
     estimated during the time a deployment is paused. Defaults to 600s.

   replicas<integer>
     Number of desired pods. This is a pointer to distinguish between explicit
     zero and not specified. Defaults to 1.

   revisionHistoryLimit<integer>
     The number of old ReplicaSets to retain to allow rollback. This is a
     pointer to distinguish between explicit zero and not specified. Defaults to
     10.

   selector <Object> -required-
     Label selector for pods. Existing ReplicaSets whose pods are selected by
     this will be the ones affected by this deployment. It must match the pod
     template's labels.

   strategy<Object>
     The deployment strategy to use to replace existing pods with new ones.

   template <Object> -required-
     Template describes the pods that will be created.

[root@k8s-master-node1 ~]#
```

  

###### 查看pod

```
[root@k8s-master-node1 ~/yaml/test]# kubectl get pod 
NAME                                     READY   STATUS        RESTARTS     AGE
ingress-demo-app-694bf5d965-q4l7m        1/1     Terminating   0            23h
ingress-demo-app-694bf5d965-v28sl        1/1     Running       0            3m9s
ingress-demo-app-694bf5d965-v652j        1/1     Running       0            23h
nfs-client-provisioner-dc5789f74-nnk77   1/1     Running       1 (8h ago)   22h
nginx-deployment-66b6c48dd5-5hhjq        1/1     Running       0            3m9s
nginx-deployment-66b6c48dd5-9z2n5        1/1     Running       0            3m19s
nginx-deployment-66b6c48dd5-llq7c        1/1     Running       0            9m10s
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看deployments

```
[root@k8s-master-node1 ~/yaml/test]# kubectl get deployments.apps 
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
ingress-demo-app         2/2     2            2           23h
nfs-client-provisioner   1/1     1            1           22h
nginx-deployment         3/3     3            3           9m45s
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 解释说明：

  

检查集群中的 Deployments 时，将显示以下字段：

  

NAME 列出了集群中 Deployments 的名称。

DESIRED 显示应用程序的所需 副本 数，在创建 Deployment 时定义这些副本。这是 期望状态。

CURRENT显示当前正在运行的副本数。

UP-TO-DATE显示已更新以实现期望状态的副本数。

AVAILABLE显示应用程序可供用户使用的副本数。

AGE 显示应用程序运行的时间量。

  

###### 查看rs

```
[root@k8s-master-node1 ~/yaml/test]# kubectl get replicasets.apps 
NAME                               DESIRED   CURRENT   READY   AGE
ingress-demo-app-694bf5d965        2         2         2       23h
nfs-client-provisioner-dc5789f74   1         1         1       23h
nginx-deployment-66b6c48dd5        3         3         3       19m
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看pods的标签

```
[root@k8s-master-node1 ~/yaml/test]# kubectl get pods --show-labels
NAME                                     READY   STATUS        RESTARTS     AGE   LABELS
ingress-demo-app-694bf5d965-q4l7m        1/1     Terminating   0            23h   app=ingress-demo-app,pod-template-hash=694bf5d965
ingress-demo-app-694bf5d965-v28sl        1/1     Running       0            15m   app=ingress-demo-app,pod-template-hash=694bf5d965
ingress-demo-app-694bf5d965-v652j        1/1     Running       0            23h   app=ingress-demo-app,pod-template-hash=694bf5d965
nfs-client-provisioner-dc5789f74-nnk77   1/1     Running       1 (8h ago)   23h   app=nfs-client-provisioner,pod-template-hash=dc5789f74
nginx-deployment-66b6c48dd5-48k9j        0/1     Terminating   0            21m   app=nginx,pod-template-hash=66b6c48dd5
nginx-deployment-66b6c48dd5-5hhjq        1/1     Running       0            15m   app=nginx,pod-template-hash=66b6c48dd5
nginx-deployment-66b6c48dd5-9z2n5        1/1     Running       0            15m   app=nginx,pod-template-hash=66b6c48dd5
nginx-deployment-66b6c48dd5-kvzft        0/1     Terminating   0            21m   app=nginx,pod-template-hash=66b6c48dd5
nginx-deployment-66b6c48dd5-llq7c        1/1     Running       0            21m   app=nginx,pod-template-hash=66b6c48dd5
[root@k8s-master-node1 ~/yaml/test]#
```

  

##### 2）更新回滚 Deployment

  

###### 命令行行升级使用镜像

```
[root@k8s-master-node1 ~/yaml/test]# kubectl get deployments -o wide
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS               IMAGES                                                                                    SELECTOR
ingress-demo-app         2/2     2            2           23h   whoami                   traefik/whoami:v1.6.1                                                                     app=ingress-demo-app
nfs-client-provisioner   1/1     1            1           23h   nfs-client-provisioner   registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/nfs-subdir-external-provisioner:v4.0.2   app=nfs-client-provisioner
nginx-deployment         3/3     3            3           18m   nginx                    nginx:1.14.2                                                                              app=nginx
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.20.1
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/nginx-deployment image updated
deployment.apps/nginx-deployment image updated
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl get deployments -o wide
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS               IMAGES                                                                                    SELECTOR
ingress-demo-app         2/2     2            2           23h   whoami                   traefik/whoami:v1.6.1                                                                     app=ingress-demo-app
nfs-client-provisioner   1/1     1            1           23h   nfs-client-provisioner   registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/nfs-subdir-external-provisioner:v4.0.2   app=nfs-client-provisioner
nginx-deployment         3/3     1            3           24m   nginx                    nginx:1.20.1                                                                              app=nginx
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### yaml方式修改镜像

```
[root@k8s-master-node1 ~/yaml/test]# kubectl edit deployments.apps nginx-deployment
Edit cancelled, no changes made.
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看更新过程

```
[root@k8s-master-node1 ~/yaml/test]# kubectl rollout status deployment.v1.apps/nginx-deployment
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看详细信息

```
[root@k8s-master-node1 ~/yaml/test]# kubectl describe deployments
Name:                   ingress-demo-app
Namespace:              default
CreationTimestamp:      Tue, 16 Nov 2021 13:28:26 +0800
Labels:                 app=ingress-demo-app
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=ingress-demo-app
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=ingress-demo-app
  Containers:
   whoami:
    Image:        traefik/whoami:v1.6.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   ingress-demo-app-694bf5d965 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  23h   deployment-controller  Scaled up replica set ingress-demo-app-694bf5d965 to 2


Name:               nfs-client-provisioner
Namespace:          default
CreationTimestamp:  Tue, 16 Nov 2021 14:07:33 +0800
Labels:             app=nfs-client-provisioner
Annotations:        deployment.kubernetes.io/revision: 1
Selector:           app=nfs-client-provisioner
Replicas:           1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:       Recreate
MinReadySeconds:    0
Pod Template:
  Labels:           app=nfs-client-provisioner
  Service Account:  nfs-client-provisioner
  Containers:
   nfs-client-provisioner:
    Image:      registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/nfs-subdir-external-provisioner:v4.0.2
    Port:       <none>
    Host Port:  <none>
    Environment:
      PROVISIONER_NAME:  k8s-sigs.io/nfs-subdir-external-provisioner
      NFS_SERVER:        192.168.1.66
      NFS_PATH:          /nfs/
    Mounts:
      /persistentvolumes from nfs-client-root (rw)
  Volumes:
   nfs-client-root:
    Type:      NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    192.168.1.66
    Path:      /nfs/
    ReadOnly:  false
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nfs-client-provisioner-dc5789f74 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  23h   deployment-controller  Scaled up replica set nfs-client-provisioner-dc5789f74 to 1


Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Wed, 17 Nov 2021 12:54:46 +0800
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 3
                        kubernetes.io/change-cause:
                          kubectl deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.20.1 --record=true
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.16.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-559d658b74 (3/3 replicas created)
Events:
  Type    Reason             Age                From                   Message
  ----    ------             ----               ----                   -------
  Normal  ScalingReplicaSet  30m                deployment-controller  Scaled up replica set nginx-deployment-66b6c48dd5 to 3
  Normal  ScalingReplicaSet  5m55s              deployment-controller  Scaled up replica set nginx-deployment-58b9b8ff79 to 1
  Normal  ScalingReplicaSet  5m27s              deployment-controller  Scaled down replica set nginx-deployment-66b6c48dd5 to 2
  Normal  ScalingReplicaSet  5m27s              deployment-controller  Scaled up replica set nginx-deployment-58b9b8ff79 to 2
  Normal  ScalingReplicaSet  5m                 deployment-controller  Scaled down replica set nginx-deployment-66b6c48dd5 to 1
  Normal  ScalingReplicaSet  5m                 deployment-controller  Scaled up replica set nginx-deployment-58b9b8ff79 to 3
  Normal  ScalingReplicaSet  4m56s              deployment-controller  Scaled down replica set nginx-deployment-66b6c48dd5 to 0
  Normal  ScalingReplicaSet  78s                deployment-controller  Scaled up replica set nginx-deployment-559d658b74 to 1
  Normal  ScalingReplicaSet  63s                deployment-controller  Scaled down replica set nginx-deployment-58b9b8ff79 to 2
  Normal  ScalingReplicaSet  63s                deployment-controller  Scaled up replica set nginx-deployment-559d658b74 to 2
  Normal  ScalingReplicaSet  49s (x3 over 61s)  deployment-controller  (combined from similar events): Scaled down replica set nginx-deployment-58b9b8ff79 to 0
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]#
```

  

##### 3） Deployment历史记录

  

###### 查看历史

```
[root@k8s-master-node1 ~/yaml/test]# kubectl rollout history deployment.v1.apps/nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.20.1 --record=true
3         kubectl deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.20.1 --record=true

[root@k8s-master-node1 ~/yaml/test]#
```

  

  

###### 回滚到上次

```
[root@k8s-master-node1 ~/yaml/test]# kubectl rollout undo deployment.v1.apps/nginx-deployment
deployment.apps/nginx-deployment rolled back
[root@k8s-master-node1 ~/yaml/test]# kubectl rollout status deployment.v1.apps/nginx-deployment
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 回滚到指定版本

```
[root@k8s-master-node1 ~/yaml/test]# kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=3
deployment.apps/nginx-deployment rolled back
[root@k8s-master-node1 ~/yaml/test]# kubectl rollout status deployment.v1.apps/nginx-deployment
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
[root@k8s-master-node1 ~/yaml/test]#
```

  

##### 4）缩放 Deployment

  

###### 扩容到十个pod

```
[root@k8s-master-node1 ~/yaml/test]# kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
deployment.apps/nginx-deployment scaled
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  get deployments.apps
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
ingress-demo-app         0/2     2            0           24h
nfs-client-provisioner   0/1     1            0           23h
nginx-deployment         5/10    10           5           45m
[root@k8s-master-node1 ~/yaml/test]#
```

  

假设启用水平自动缩放 Pod在集群中，可以为 Deployment 设置自动缩放器，并选择最小和最大 要基于现有 Pods 的 CPU 利用率运行的 Pods。

```
[root@k8s-master-node1 ~/yaml/test]# kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
horizontalpodautoscaler.autoscaling/nginx-deployment autoscaled
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]#
```

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fa52ad147be4e6fba60dc8c149f734b~tplv-k3u1fbpfcp-zoom-1.image)  

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

知乎、CSDN、开源中国、思否、掘金、哔哩哔哩、腾讯云

  

```