---
layout: post
cid: 37
title: KubeSphere 升级 &amp;&amp; 安装后启用插件
slug: 37
date: 2021/12/30 17:08:24
updated: 2021/12/30 17:08:24
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


**KubeSphere 升级**

  

```
root@master1:~# export KKZONE=cn
root@master1:~# kk upgrade --with-kubernetes v1.22.1 --with-kubesphere v3.2.0 -f sample.yaml

```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86d0648beb37491e99cece464f937e9c~tplv-k3u1fbpfcp-zoom-1.image)

  

**启用插件**

  

用户可以使用 KubeSphere Web 控制台查看和操作不同的资源。要在安装后启用可插拔组件，只需要在控制台中进行略微调整。对于那些习惯使用 Kubernetes 命令行工具 kubectl 的人来说，由于该工具已集成到控制台中，因此使用 KubeSphere 将毫无困难。

  
  

以 admin 身份登录控制台。点击左上角的平台管理 ，然后选择集群管理。

  

**集群管理**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c9d92a98e4a41948018cce16f0d797c~tplv-k3u1fbpfcp-zoom-1.image)

  

点击 CRD，然后在搜索栏中输入 clusterconfiguration，点击搜索结果进入其详情页面。

  

**CRD**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb05d28c223f40059dbf1f976b18ae5f~tplv-k3u1fbpfcp-zoom-1.image)

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7fc15c98332e452283636def960fbaf1~tplv-k3u1fbpfcp-zoom-1.image)

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0dd29db03665424fbb051f47d0b12e65~tplv-k3u1fbpfcp-zoom-1.image)

  

  

**编辑配置文件**

  

在该配置文件中，将对应组件 enabled 的 false 更改为 true，以启用要安装的组件。完成后，点击更新以保存配置。

  

**我的内容：**

  

```
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  labels:
    version: v3.2.0
  name: ks-installer
  namespace: kubesphere-system
spec:
  alerting:
    enabled: true
  auditing:
    enabled: true
  authentication:
    jwtSecret: ''
  common:
    core:
      console:
        enableMultiLogin: true
        port: 30880
        type: NodePort
    es:
      basicAuth:
        enabled: true
        password: ''
        username: ''
      data:
        volumeSize: 20Gi
      elkPrefix: logstash
      externalElasticsearchPort: ''
      externalElasticsearchUrl: ''
      logMaxAge: 7
      master:
        volumeSize: 4Gi
    gpu:
      kinds:
        - default: true
          resourceName: nvidia.com/gpu
          resourceType: GPU
    minio:
      volumeSize: 20Gi
    monitoring:
      GPUMonitoring:
        enabled: true
      endpoint: 'http://prometheus-operated.kubesphere-monitoring-system.svc:9090'
    openldap:
      enabled: true
    redis:
      enabled: true
  devops:
    enabled: true
    jenkinsJavaOpts_MaxRAM: 2g
    jenkinsJavaOpts_Xms: 512m
    jenkinsJavaOpts_Xmx: 512m
    jenkinsMemoryLim: 2Gi
    jenkinsMemoryReq: 1500Mi
    jenkinsVolumeSize: 8Gi
  etcd:
    endpointIps: 192.168.1.10
    monitoring: false
    port: 2379
    tlsEnable: true
  events:
    enabled: true
  kubeedge:
    cloudCore:
      cloudHub:
        advertiseAddress:
          - ''
        nodeLimit: '100'
      cloudhubHttpsPort: '10002'
      cloudhubPort: '10000'
      cloudhubQuicPort: '10001'
      cloudstreamPort: '10003'
      nodeSelector:
        node-role.kubernetes.io/worker: ''
      service:
        cloudhubHttpsNodePort: '30002'
        cloudhubNodePort: '30000'
        cloudhubQuicNodePort: '30001'
        cloudstreamNodePort: '30003'
        tunnelNodePort: '30004'
      tolerations: []
      tunnelPort: '10004'
    edgeWatcher:
      edgeWatcherAgent:
        nodeSelector:
          node-role.kubernetes.io/worker: ''
        tolerations: []
      nodeSelector:
        node-role.kubernetes.io/worker: ''
      tolerations: []
    enabled: true
  logging:
    containerruntime: docker
    enabled: true
    logsidecar:
      enabled: true
      replicas: 2
  metrics_server:
    enabled: true
  monitoring:
    gpu:
      nvidia_dcgm_exporter:
        enabled: true
    storageClass: ''
  multicluster:
    clusterRole: none
  network:
    ippool:
      type: none
    networkpolicy:
      enabled: true
    topology:
      type: none
  openpitrix:
    store:
      enabled: true
  persistence:
    storageClass: ''
  servicemesh:
    enabled: true

```

  

启用组件

  

执行以下命令，使用 Web kubectl 来检查安装过程：

  

```
root@master1:~# kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f

```

  

  

如果组件安装成功，输出将显示以下消息。

  

  

```
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################


Console: http://192.168.0.2:30880
Account: admin
Password: P@88w0rd


NOTES：
  1. After you log into the console, please check the
     monitoring status of service components in
     "Cluster Management". If any service is not
     ready, please wait patiently until all components
     are up and running.
  2. Please change the default password after login.


#####################################################
https://kubesphere.io             20xx-xx-xx xx:xx:xx
#####################################################
```

  

登录 KubeSphere 控制台，在系统组件中可以查看不同组件的状态。

  

服务组件

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95291f0fccea4148a035a4cfd005dbcb~tplv-k3u1fbpfcp-zoom-1.image)

  
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