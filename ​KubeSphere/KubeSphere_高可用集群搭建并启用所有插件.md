---
layout: post
cid: 33
title: KubeSphere 高可用集群搭建并启用所有插件
slug: 33
date: 2021/12/30 17:07:33
updated: 2021/12/30 17:07:33
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


**介绍**

大多数情况下，单主节点集群大致足以供开发和测试环境使用。但是，对于生产环境，您需要考虑集群的高可用性。如果关键组件（例如 kube-apiserver、kube-scheduler 和 kube-controller-manager）都在同一个主节点上运行，一旦主节点宕机，Kubernetes 和 KubeSphere 都将不可用。因此，您需要为多个主节点配置负载均衡器，以创建高可用集群。您可以使用任意云负载均衡器或者任意硬件负载均衡器（例如 F5）。此外，也可以使用 Keepalived 和 HAproxy，或者 Nginx 来创建高可用集群。

  

**架构**

在您开始操作前，请确保准备了 6 台 Linux 机器，其中 3 台充当主节点，另外 3 台充当工作节点。下图展示了这些机器的详情，包括它们的私有 IP 地址和角色。

  

**配置负载均衡器**

您必须在您的环境中创建一个负载均衡器来监听（在某些云平台也称作监听器）关键端口。建议监听下表中的端口。

  

服务  协议  端口

apiserver TCP 6443

ks-console  TCP 30880

http  TCP 80

https TCP 443

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6d53d4c347947de82600070f4a2474c~tplv-k3u1fbpfcp-zoom-1.image)

  

**配置免密**

  

```
root@hello:~# ssh-keygen


root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.10
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.11  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.12  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.13  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.14  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.15  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.16  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.51  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.52  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.53  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.54  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.55  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.56  
root@hello:~# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.57
```

  
  

**下载 KubeKey**

Kubekey 是新一代安装程序，可以简单、快速和灵活地安装 Kubernetes 和 KubeSphere。

  

  

```
root@cby:~# export KKZONE=cn
root@cby:~# curl -sfL https://get-kk.kubesphere.io | VERSION=v1.2.0 sh -


Downloading kubekey v1.2.0 from https://kubernetes.pek3b.qingstor.com/kubekey/releases/download/v1.2.0/kubekey-v1.2.0-linux-amd64.tar.gz ...



Kubekey v1.2.0 Download Complete!
```

  

**为 kk 添加可执行权限**

  

  

```
root@cby:~# chmod +x kk
root@cby:~# ./kk create config --with-kubesphere v3.2.0 --with-kubernetes v1.22.1
```

  

  

**部署 KubeSphere 和 Kubernetes**

  

  

```
root@cby:~# vim config-sample.yaml
root@cby:~# 
root@cby:~# 
root@cby:~# cat config-sample.yaml
apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: master1, address: 192.168.1.10, internalAddress: 192.168.1.10, user: root, password: Cby123..}
  - {name: master2, address: 192.168.1.11, internalAddress: 192.168.1.11, user: root, password: Cby123..}
  - {name: master3, address: 192.168.1.12, internalAddress: 192.168.1.12, user: root, password: Cby123..}
  - {name: node1, address: 192.168.1.13, internalAddress: 192.168.1.13, user: root, password: Cby123..}
  - {name: node2, address: 192.168.1.14, internalAddress: 192.168.1.14, user: root, password: Cby123..}
  - {name: node3, address: 192.168.1.15, internalAddress: 192.168.1.15, user: root, password: Cby123..}
  - {name: node4, address: 192.168.1.16, internalAddress: 192.168.1.16, user: root, password: Cby123..}
  - {name: node5, address: 192.168.1.51, internalAddress: 192.168.1.51, user: root, password: Cby123..}
  - {name: node6, address: 192.168.1.52, internalAddress: 192.168.1.52, user: root, password: Cby123..}
  - {name: node7, address: 192.168.1.53, internalAddress: 192.168.1.53, user: root, password: Cby123..}
  - {name: node8, address: 192.168.1.54, internalAddress: 192.168.1.54, user: root, password: Cby123..}
  - {name: node9, address: 192.168.1.55, internalAddress: 192.168.1.55, user: root, password: Cby123..}
  - {name: node10, address: 192.168.1.56, internalAddress: 192.168.1.56, user: root, password: Cby123..}
  - {name: node11, address: 192.168.1.57, internalAddress: 192.168.1.57, user: root, password: Cby123..}
  roleGroups:
    etcd:
    - master1
    - master2
    - master3
    master:
    - master1
    - master2
    - master3
    worker:
    - node1
    - node2
    - node3
    - node4
    - node5
    - node6
    - node7
    - node8
    - node9
    - node10
    - node11
  controlPlaneEndpoint:
    ##Internal loadbalancer for apiservers
    #internalLoadbalancer: haproxy


    domain: lb.kubesphere.local
    address: "192.168.1.20"
    port: 6443
  kubernetes:
    version: v1.22.1
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: []
    insecureRegistries: []
  addons: []




---
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  name: ks-installer
  namespace: kubesphere-system
  labels:
    version: v3.2.0
spec:
  persistence:
    storageClass: ""
  authentication:
    jwtSecret: ""
  local_registry: ""
  # dev_tag: ""
  etcd:
    monitoring: false
    endpointIps: localhost
    port: 2379
    tlsEnable: true
  common:
    core:
      console:
        enableMultiLogin: true
        port: 30880
        type: NodePort
    # apiserver:
    #  resources: {}
    # controllerManager:
    #  resources: {}
    redis:
      enabled: false
      volumeSize: 2Gi
    openldap:
      enabled: false
      volumeSize: 2Gi
    minio:
      volumeSize: 20Gi
    monitoring:
      # type: external
      endpoint: http://prometheus-operated.kubesphere-monitoring-system.svc:9090
      GPUMonitoring:
        enabled: false
    gpu:
      kinds:        
      - resourceName: "nvidia.com/gpu"
        resourceType: "GPU"
        default: true
    es:
      # master:
      #   volumeSize: 4Gi
      #   replicas: 1
      #   resources: {}
      # data:
      #   volumeSize: 20Gi
      #   replicas: 1
      #   resources: {}
      logMaxAge: 7
      elkPrefix: logstash
      basicAuth:
        enabled: false
        username: ""
        password: ""
      externalElasticsearchUrl: ""
      externalElasticsearchPort: ""
  alerting:
    enabled: false
    # thanosruler:
    #   replicas: 1
    #   resources: {}
  auditing:
    enabled: false
    # operator:
    #   resources: {}
    # webhook:
    #   resources: {}
  devops:
    enabled: false
    jenkinsMemoryLim: 2Gi
    jenkinsMemoryReq: 1500Mi
    jenkinsVolumeSize: 8Gi
    jenkinsJavaOpts_Xms: 512m
    jenkinsJavaOpts_Xmx: 512m
    jenkinsJavaOpts_MaxRAM: 2g
  events:
    enabled: false
    # operator:
    #   resources: {}
    # exporter:
    #   resources: {}
    # ruler:
    #   enabled: false
    #   replicas: 2
    #   resources: {}
  logging:
    enabled: false
    containerruntime: docker
    logsidecar:
      enabled: false
      replicas: 2
      # resources: {}
  metrics_server:
    enabled: false
  monitoring:
    storageClass: ""
    # kube_rbac_proxy:
    #   resources: {}
    # kube_state_metrics:
    #   resources: {}
    # prometheus:
    #   replicas: 1
    #   volumeSize: 20Gi
    #   resources: {}
    #   operator:
    #     resources: {}
    #   adapter:
    #     resources: {}
    # node_exporter:
    #   resources: {}
    # alertmanager:
    #   replicas: 1
    #   resources: {}
    # notification_manager:
    #   resources: {}
    #   operator:
    #     resources: {}
    #   proxy:
    #     resources: {}
    gpu:
      nvidia_dcgm_exporter:
        enabled: false
        # resources: {}
  multicluster:
    clusterRole: none 
  network:
    networkpolicy:
      enabled: false
    ippool:
      type: none
    topology:
      type: none
  openpitrix:
    store:
      enabled: false
  servicemesh:
    enabled: false
  kubeedge:
    enabled: false  
    cloudCore:
      nodeSelector: {"node-role.kubernetes.io/worker": ""}
      tolerations: []
      cloudhubPort: "10000"
      cloudhubQuicPort: "10001"
      cloudhubHttpsPort: "10002"
      cloudstreamPort: "10003"
      tunnelPort: "10004"
      cloudHub:
        advertiseAddress:
          - ""
        nodeLimit: "100"
      service:
        cloudhubNodePort: "30000"
        cloudhubQuicNodePort: "30001"
        cloudhubHttpsNodePort: "30002"
        cloudstreamNodePort: "30003"
        tunnelNodePort: "30004"
    edgeWatcher:
      nodeSelector: {"node-role.kubernetes.io/worker": ""}
      tolerations: []
      edgeWatcherAgent:
        nodeSelector: {"node-role.kubernetes.io/worker": ""}
        tolerations: []



root@cby:~#
```

  

**若是haproxy配置如下：**

  

```
frontend kube-apiserver
  bind *:6443
  mode tcp
  option tcplog
  default_backend kube-apiserver

backend kube-apiserver
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server kube-apiserver-1 192.168.1.10:6443 check
    server kube-apiserver-2 192.168.1.11:6443 check
    server kube-apiserver-3 192.168.1.12:6443 check 
```

  

**安装所需环境**

  

```
root@hello:~# bash -x 1.sh 
root@hello:~# cat 1.sh
ssh root@192.168.1.10 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.11 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.12 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.13 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.14 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.15 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.16 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.51 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.52 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.53 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.54 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.55 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.56 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
ssh root@192.168.1.57 "apt update && apt install sudo curl openssl ebtables socat ipset conntrack nfs-common -y"
```

  

  

**开始安装**

**配置完成后，您可以执行以下命令来开始安装：**

  

  

```
root@hello:~# ./kk create cluster -f config-sample.yaml
+---------+------+------+---------+----------+-------+-------+-----------+--------+------------+-------------+------------------+--------------+
| name    | sudo | curl | openssl | ebtables | socat | ipset | conntrack | docker | nfs client | ceph client | glusterfs client | time         |
+---------+------+------+---------+----------+-------+-------+-----------+--------+------------+-------------+------------------+--------------+
| node3   | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:01 |
| node1   | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:01 |
| node4   | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:01 |
| node8   | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:01 |
| node11  | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:01 |
| master1 | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:00 |
| node5   | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:00 |
| master2 | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:00 |
| node2   | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:01 |
| node7   | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:00 |
| master3 | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:01 |
| node6   | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:01 |
| node9   | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:01 |
| node10  | y    | y    | y       | y        | y     | y     | y         |        | y          |             |                  | UTC 13:26:01 |
+---------+------+------+---------+----------+-------+-------+-----------+--------+------------+-------------+------------------+--------------+


This is a simple check of your environment.
Before installation, you should ensure that your machines meet all requirements specified at
https://github.com/kubesphere/kubekey#requirements-and-recommendations


Continue this installation? [yes/no]: yes
INFO[13:26:06 UTC] Downloading Installation Files              
INFO[13:26:06 UTC] Downloading kubeadm ...                      




---略---
```

  

**验证安装**

**运行以下命令查看安装日志。**

  

```
root@cby:~# kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f


**************************************************
Collecting installation results ...
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################


Console: http://192.168.1.10:30880
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
https://kubesphere.io             2021-11-10 10:24:00
#####################################################


root@hello:~# kubectl get node 
NAME      STATUS   ROLES                  AGE   VERSION
master1   Ready    control-plane,master   30m   v1.22.1
master2   Ready    control-plane,master   29m   v1.22.1
master3   Ready    control-plane,master   29m   v1.22.1
node1     Ready    worker                 29m   v1.22.1
node10    Ready    worker                 29m   v1.22.1
node11    Ready    worker                 29m   v1.22.1
node2     Ready    worker                 29m   v1.22.1
node3     Ready    worker                 29m   v1.22.1
node4     Ready    worker                 29m   v1.22.1
node5     Ready    worker                 29m   v1.22.1
node6     Ready    worker                 29m   v1.22.1
node7     Ready    worker                 30m   v1.22.1
node8     Ready    worker                 30m   v1.22.1
node9     Ready    worker                 29m   v1.22.1
```

  

  

**在安装后启用插件**

  

使用 admin 用户登录控制台。点击左上角的平台管理，然后选择集群管理。

  

点击 CRD，然后在搜索栏中输入 clusterconfiguration。点击搜索结果查看其详情页。

  

在自定义资源中，点击 ks-installer 右侧的 ，然后选择编辑 YAML。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/267930fc6cf6476e8770003867ec01d5~tplv-k3u1fbpfcp-zoom-1.image)

  

```
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: >
      {"apiVersion":"installer.kubesphere.io/v1alpha1","kind":"ClusterConfiguration","metadata":{"annotations":{},"labels":{"version":"v3.2.0"},"name":"ks-installer","namespace":"kubesphere-system"},"spec":{"alerting":{"enabled":false},"auditing":{"enabled":false},"authentication":{"jwtSecret":""},"common":{"core":{"console":{"enableMultiLogin":true,"port":30880,"type":"NodePort"}},"es":{"basicAuth":{"enabled":false,"password":"","username":""},"elkPrefix":"logstash","externalElasticsearchPort":"","externalElasticsearchUrl":"","logMaxAge":7},"gpu":{"kinds":[{"default":true,"resourceName":"nvidia.com/gpu","resourceType":"GPU"}]},"minio":{"volumeSize":"20Gi"},"monitoring":{"GPUMonitoring":{"enabled":false},"endpoint":"http://prometheus-operated.kubesphere-monitoring-system.svc:9090"},"openldap":{"enabled":false,"volumeSize":"2Gi"},"redis":{"enabled":false,"volumeSize":"2Gi"}},"devops":{"enabled":false,"jenkinsJavaOpts_MaxRAM":"2g","jenkinsJavaOpts_Xms":"512m","jenkinsJavaOpts_Xmx":"512m","jenkinsMemoryLim":"2Gi","jenkinsMemoryReq":"1500Mi","jenkinsVolumeSize":"8Gi"},"etcd":{"endpointIps":"192.168.1.10,192.168.1.11,192.168.1.12","monitoring":false,"port":2379,"tlsEnable":true},"events":{"enabled":false},"kubeedge":{"cloudCore":{"cloudHub":{"advertiseAddress":[""],"nodeLimit":"100"},"cloudhubHttpsPort":"10002","cloudhubPort":"10000","cloudhubQuicPort":"10001","cloudstreamPort":"10003","nodeSelector":{"node-role.kubernetes.io/worker":""},"service":{"cloudhubHttpsNodePort":"30002","cloudhubNodePort":"30000","cloudhubQuicNodePort":"30001","cloudstreamNodePort":"30003","tunnelNodePort":"30004"},"tolerations":[],"tunnelPort":"10004"},"edgeWatcher":{"edgeWatcherAgent":{"nodeSelector":{"node-role.kubernetes.io/worker":""},"tolerations":[]},"nodeSelector":{"node-role.kubernetes.io/worker":""},"tolerations":[]},"enabled":false},"logging":{"containerruntime":"docker","enabled":false,"logsidecar":{"enabled":false,"replicas":2}},"metrics_server":{"enabled":false},"monitoring":{"gpu":{"nvidia_dcgm_exporter":{"enabled":false}},"storageClass":""},"multicluster":{"clusterRole":"none"},"network":{"ippool":{"type":"none"},"networkpolicy":{"enabled":false},"topology":{"type":"none"}},"openpitrix":{"store":{"enabled":false}},"persistence":{"storageClass":""},"servicemesh":{"enabled":false}}}
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
      elkPrefix: logstash
      externalElasticsearchPort: ''
      externalElasticsearchUrl: ''
      logMaxAge: 7
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
      volumeSize: 2Gi
    redis:
      enabled: true
      volumeSize: 2Gi
  devops:
    enabled: true
    jenkinsJavaOpts_MaxRAM: 2g
    jenkinsJavaOpts_Xms: 512m
    jenkinsJavaOpts_Xmx: 512m
    jenkinsMemoryLim: 2Gi
    jenkinsMemoryReq: 1500Mi
    jenkinsVolumeSize: 8Gi
  etcd:
    endpointIps: '192.168.1.10,192.168.1.11,192.168.1.12'
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
      type: weave-scope
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

  

**批量将所有服务器设置阿里云加速**

  

```
root@hello:~# vim 8 
root@hello:~# cat 8 
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ted9wxpi.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker



root@hello:~# vim 7 
root@hello:~# cat 7
scp 8 root@192.168.1.11:
scp 8 root@192.168.1.12:
scp 8 root@192.168.1.13:
scp 8 root@192.168.1.14:
scp 8 root@192.168.1.15:
scp 8 root@192.168.1.16:
scp 8 root@192.168.1.51:
scp 8 root@192.168.1.52:
scp 8 root@192.168.1.53:
scp 8 root@192.168.1.54:
scp 8 root@192.168.1.55:
scp 8 root@192.168.1.56:
scp 8 root@192.168.1.57:
root@hello:~# bash -x 7



root@hello:~# vim 6 
root@hello:~# cat 6
ssh root@192.168.1.10 "bash -x 8"
ssh root@192.168.1.11 "bash -x 8"
ssh root@192.168.1.12 "bash -x 8"
ssh root@192.168.1.13 "bash -x 8"
ssh root@192.168.1.14 "bash -x 8"
ssh root@192.168.1.15 "bash -x 8"
ssh root@192.168.1.16 "bash -x 8"
ssh root@192.168.1.51 "bash -x 8"
ssh root@192.168.1.52 "bash -x 8"
ssh root@192.168.1.53 "bash -x 8"
ssh root@192.168.1.54 "bash -x 8"
ssh root@192.168.1.55 "bash -x 8"
ssh root@192.168.1.56 "bash -x 8"
ssh root@192.168.1.57 "bash -x 8"
root@hello:~# bash -x 6
```

  

**查看node节点**

  

```
root@hello:~# kubectl  get node
NAME      STATUS   ROLES                  AGE   VERSION
master1   Ready    control-plane,master   11h   v1.22.1
master2   Ready    control-plane,master   11h   v1.22.1
master3   Ready    control-plane,master   11h   v1.22.1
node1     Ready    worker                 11h   v1.22.1
node10    Ready    worker                 11h   v1.22.1
node11    Ready    worker                 11h   v1.22.1
node2     Ready    worker                 11h   v1.22.1
node3     Ready    worker                 11h   v1.22.1
node4     Ready    worker                 11h   v1.22.1
node5     Ready    worker                 11h   v1.22.1
node6     Ready    worker                 11h   v1.22.1
node7     Ready    worker                 11h   v1.22.1
node8     Ready    worker                 11h   v1.22.1
node9     Ready    worker                 11h   v1.22.1
root@hello:~#  
root@hello:~#
```

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17c387d141a443db9630d3ff27430cc5~tplv-k3u1fbpfcp-zoom-1.image)

  

  

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