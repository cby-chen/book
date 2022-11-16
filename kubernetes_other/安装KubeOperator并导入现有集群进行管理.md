---
layout: post
cid: 275
title: 安装KubeOperator并导入现有集群进行管理
slug: 275
date: 2022/08/05 22:33:32
updated: 2022/08/05 22:38:14
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: 安装KubeOperator并导入现有集群进行管理
keywords: 小陈运维,小陈,小陈博客,文档,
mode: default
thumb: 
video: 
---


# 安装KubeOperator并导入现有集群进行管理



### 介绍

[KubeOperator](https://kubeoperator.io/) 是一个开源的轻量级 [Kubernetes](https://kubernetes.io/) 发行版，专注于帮助企业规划、部署和运营生产级别的 [Kubernetes](https://kubernetes.io/) 集群。

[KubeOperator](https://kubeoperator.io/) 提供可视化的 Web UI，支持离线环境，支持物理机、[VMware](https://www.vmware.com/)、[OpenStack](https://www.openstack.org/) 和 [FusionCompute](https://support.huawei.com/enterprise/zh/cloud-computing/fusioncompute-pid-8576912) 等 IaaS 平台，支持 x86 和 ARM64 架构，支持 GPU，内置应用商店，已通过 CNCF 的 [Kubernetes](https://kubernetes.io/) 软件一致性认证。

[KubeOperator](https://kubeoperator.io/) 使用 [Terraform](https://www.terraform.io/) 在 IaaS 平台上自动创建主机（用户也可以自行准备主机，比如物理机或者虚机），通过 [Ansible](https://www.ansible.com/) 完成自动化部署和变更操作，支持 [Kubernetes](https://kubernetes.io/) 集群 从 Day 0 规划，到 Day 1 部署，到 Day 2 运营的全生命周期管理。



### 安装

```
root@hello:~# curl -sSL https://github.com/KubeOperator/KubeOperator/releases/latest/download/quick_start.sh | sh


...略...

======================= KubeOperator 安装完成 =======================

请开放防火墙或安全组的80,8081-8083端口,通过以下方式访问:
 URL:  http://$LOCAL_IP:80 
 用户名:  admin  
 初始密码:  kubeoperator@admin123 
root@hello:~# 
root@hello:~# koctl status
         Name                        Command                  State                                       Ports                                
-----------------------------------------------------------------------------------------------------------------------------------------------
kubeoperator_kobe         sh /root/entrypoint.sh           Up (healthy)   8080/tcp                                                             
kubeoperator_kotf         kotf-server                      Up (healthy)   8080/tcp                                                             
kubeoperator_kubepi       kubepi-server                    Up (healthy)   80/tcp                                                               
kubeoperator_mysql        /entrypoint.sh mysqld            Up (healthy)   3306/tcp, 33060/tcp                                                  
kubeoperator_nexus        sh -c ${SONATYPE_DIR}/star ...   Up (healthy)   0.0.0.0:8081->8081/tcp,:::8081->8081/tcp,                            
                                                                          0.0.0.0:8082->8082/tcp,:::8082->8082/tcp,                            
                                                                          0.0.0.0:8083->8083/tcp,:::8083->8083/tcp                             
kubeoperator_nginx        /docker-entrypoint.sh /bin ...   Up (healthy)   0.0.0.0:80->80/tcp,:::80->80/tcp                                     
kubeoperator_server       ko-server                        Up (healthy)   8080/tcp                                                             
kubeoperator_ui           /docker-entrypoint.sh ngin ...   Up (healthy)   80/tcp                                                               
kubeoperator_webkubectl   sh /opt/webkubectl/start-w ...   Up (healthy)                                                                        
root@hello:~# 
```



### 登陆

```
地址: http://<ko服务器_ip>:80
用户名: admin
密码: kubeoperator@admin123
```



### 导入集群

```
# 获取 Api Server
[root@k8s-master01 ~]# cat ~/.kube/config | grep server: | awk '{print $2}'
https://192.168.1.69:8443

# 获取 Router
# 若使用kubeadm安装可以使用如下命令进行查看 ，若二进制安装使用节点IP即可
[root@k8s-master01 ~]# kubectl -n kube-system get pod -o wide | grep kube-proxy
[root@k8s-master01 ~]# 

# 获取 Token
[root@k8s-master01 ~]# vim 123.yaml
[root@k8s-master01 ~]# cat 123.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kubeoperator-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubeoperator-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: kubeoperator-user
    namespace: kube-system
[root@k8s-master01 ~]# 
[root@k8s-master01 ~]# kubectl  apply -f 123.yaml 
serviceaccount/kubeoperator-user created
clusterrolebinding.rbac.authorization.k8s.io/kubeoperator-user created
[root@k8s-master01 ~]# 
[root@k8s-master01 ~]# 

# 1.23 以及以下可以使用如下命令查看
[root@k8s-master01 ~]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kubeoperator-user | awk '{print $1}') | grep token: | awk '{print $2}'
[root@k8s-master01 ~]# 
[root@k8s-master01 ~]# 

# 1.24 版本使用如下命令创建token
[root@k8s-master01 ~]# kubectl -n kube-system create token kubeoperator-user
eyJhbGciOiJSUzI1NiIsImtpZCI6Ik9fdmIzY3ZjU2w0V3ZuUXl2bExBN2tZYlh3bFV2MTliZElSd0hvMnN6SXMifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjU4ODk4MTE5LCJpYXQiOjE2NTg4OTQ1MTksImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJrdWJlb3BlcmF0b3ItdXNlciIsInVpZCI6ImZhOGJmZjJjLWIyYjYtNDAxMS1iODAzLTY4MDVmZDYwZjMxOSJ9fSwibmJmIjoxNjU4ODk0NTE5LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06a3ViZW9wZXJhdG9yLXVzZXIifQ.HvLQlMW_aJ2TDlyE-aM9UiDKl3QHAod9oUQZaHBI97-nLc3aoUdKsGrhICD42ud_Qcn_vFhUvJkPvBi_5esqKCB9LPF-cUhyyj0TxRIH_rTfUdzmDeYUVn3rfg0jlGkXRhzpJMLIRpsK_RB0StbDR4WxfhdnpRkFz-7YgtsRUfRZXG4AF6HNzt1ZWEA3ZVv779TqJemBUTmwJGB9OdyYkKTnGNy4tDGfryZsfW7zN-FhdVugd_7-_lNlFrLZWwrN3fUYPSZLGqulvy7BBpIBO16pBtIA0Qi0bkNdkSpu5a2RNjpMtXKVRYy7M--mQ4EaEod4aCZDuDhMz2S-75VwDA
[root@k8s-master01 ~]# 
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