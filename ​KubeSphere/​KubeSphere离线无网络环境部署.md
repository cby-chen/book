---
layout: post
cid: 82
title: ​KubeSphere离线无网络环境部署
slug: 82
date: 2022/01/13 09:16:10
updated: 2022/01/13 09:16:10
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


# **KubeSphere离线无网络环境部署**

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79d332c424e348aa972488f3ae51f1d1~tplv-k3u1fbpfcp-zoom-1.image)

KubeSphere 是 GitHub 上的一个开源项目，是成千上万名社区用户的聚集地。很多用户都在使用 KubeSphere 运行工作负载。对于在 Linux 上的安装，KubeSphere 既可以部署在云端，也可以部署在本地环境中，例如 AWS EC2、Azure VM 和裸机等。

KubeSphere 为用户提供轻量级安装程序 KubeKey（该程序支持安装 Kubernetes、KubeSphere 及相关插件），安装过程简单而友好。KubeKey 不仅能帮助用户在线创建集群，还能作为离线安装解决方案。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbe0a5301f4b4f85a59db30ba39716f8~tplv-k3u1fbpfcp-zoom-1.image)

  

# **前期准备所需包**

```
#前期准备所需包
root@hello:~# wget https://github.com/kubesphere/kubekey/releases/download/v1.2.1/kubekey-v1.2.1-linux-amd64.tar.gz
root@hello:~# tar xvf kubekey-v1.2.1-linux-amd64.tar.gz 
root@hello:~# ls kk 
kk
root@hello:~#


root@hello:~# curl -L -O https://github.com/kubesphere/ks-installer/releases/download/v3.2.1/images-list.txt
root@hello:~# curl -L -O https://github.com/kubesphere/ks-installer/releases/download/v3.2.1/offline-installation-tool.sh

root@hello:~# chmod +x offline-installation-tool.sh

root@hello:~# export KKZONE=cn
root@hello:~# ./offline-installation-tool.sh -b

root@hello:~# ./offline-installation-tool.sh -s -l images-list.txt -d ./kubesphere-images

root@hello:~# curl -L -o /root/kubekey/v1.21.5/amd64/docker-20.10.8.tgz https://download.docker.com/linux/static/stable/x86_64/docker-20.10.8.tgz

root@hello:~# curl -L -o /root/kubekey/v1.21.5/amd64/crictl-v1.22.0-linux-amd64.tar.gz https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.22.0/crictl-v1.22.0-linux-amd64.tar.gz
```

  

# **离线环境安装**

```
#创建证书，注意“Common Name” 需要写域名

root@cby:~# mkdir -p certs
root@cby:~# openssl req \
> -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
> -x509 -days 36500 -out certs/domain.crt
Generating a RSA private key
............++++
.......++++
writing new private key to 'certs/domain.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:dockerhub.kubekey.local
Email Address []:
root@cby:~#
```

  

# **安装docker**

```
#安装docker

root@cby:~# 
root@cby:~/package# ll
total 94776
drwxr-xr-x 2 root root     4096 Jan 12 07:17 ./
drwx------ 7 root root     4096 Jan 12 07:16 ../
-rw-r--r-- 1 root root 23703726 Jan 12 07:17 containerd.io_1.4.12-1_amd64.deb
-rw-r--r-- 1 root root 21234738 Jan 12 07:16 docker-ce_5%3a20.10.12~3-0~ubuntu-focal_amd64.deb
-rw-r--r-- 1 root root 40652850 Jan 12 07:16 docker-ce-cli_5%3a20.10.12~3-0~ubuntu-focal_amd64.deb
-rw-r--r-- 1 root root  7921036 Jan 12 07:16 docker-ce-rootless-extras_5%3a20.10.12~3-0~ubuntu-focal_amd64.deb
-rw-r--r-- 1 root root  3517780 Jan 12 07:16 docker-scan-plugin_0.12.0~ubuntu-focal_amd64.deb
root@cby:~/package# 
root@cby:~/package# apt install ./*
```

  

# **部署镜像仓库**

```
# 导入镜像
root@cby:~/cby# docker load -i registry.tar

# 启动 Docker 仓库
root@cby:~# docker run -d   --restart=always   --name registry   -v "$(pwd)"/certs:/certs   -v /mnt/registry:/var/lib/registry   -e REGISTRY_HTTP_ADDR=0.0.0.0:443   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt   -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key   -p 443:443   registry:2


#配置仓库

#在 /etc/hosts 中添加一个条目
root@cby:~# vim /etc/hosts
root@cby:~# cat /etc/hosts
3.7.191.234 dockerhub.kubekey.local

#配置免证书
root@cby:~# mkdir -p  /etc/docker/certs.d/dockerhub.kubekey.local
root@cby:~# cp certs/domain.crt  /etc/docker/certs.d/dockerhub.kubekey.local/ca.crt
root@cby:~# 

#配置免验证
root@cby:~# cat /etc/docker/daemon.json
{
   "insecure-registries":["https://dockerhub.kubekey.local"]
}

#重载配置，并重启
root@cby:~# systemctl daemon-reload
root@cby:~# systemctl restart docker

```

  

# **部署 KubeSphere 和 kubernetes**

**注意添加字段“privateRegistry”**  

```
#添加执行权限
root@cby:~# 
root@cby:~# chmod +x kk
root@cby:~# chmod +x offline-installation-tool.sh

#推送镜像到私有仓库
root@cby:~# ./offline-installation-tool.sh -l images-list.txt -d ./kubesphere-images -r dockerhub.kubekey.local



root@cby:~# apt install conntrack


root@cby:~# ./kk create config --with-kubernetes v1.21.5 --with-kubesphere v3.2.1 -f config-sample.yaml
root@cby:~# 
root@cby:~# vim config-sample.yaml 
root@cby:~# cat config-sample.yaml
apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: master, address: 3.7.191.234, internalAddress: 3.7.191.234, user: root, password: Cby23..}
  - {name: node1, address: 3.7.191.235, internalAddress: 3.7.191.235, user: root, password: Cby23..}
  - {name: node2, address: 3.7.191.238, internalAddress: 3.7.191.238, user: root, password: Cby23..}
  roleGroups:
    etcd:
    - master
    master: 
    - node1
    worker:
    - node1
    - node2
  controlPlaneEndpoint:
    ##Internal loadbalancer for apiservers 
    #internalLoadbalancer: haproxy

    domain: lb.kubesphere.local
    address: ""
    port: 6443
  kubernetes:
    version: v1.21.5
    clusterName: cluster.local
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: []
    insecureRegistries: []
    privateRegistry: dockerhub.kubekey.local
  addons: []



---
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  name: ks-installer
  namespace: kubesphere-system
  labels:
    version: v3.2.1
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
      externalElasticsearchHost: ""
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
    #   enabled: true
    #   replicas: 2
    #   resources: {}
  logging:
    enabled: false
    containerruntime: docker
    logsidecar:
      enabled: true
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
root@cby:~# 
root@cby:~# 
root@cby:~# ./kk create cluster -f config-sample.yaml

----略

#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://3.7.191.235:30880
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
https://kubesphere.io             2022-01-12 09:42:36
#####################################################
INFO[09:42:45 UTC] Installation is complete.

Please check the result using the command:

       kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
 
root@cby:~#

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