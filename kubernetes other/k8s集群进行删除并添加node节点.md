---
layout: post
cid: 7
title: k8s集群进行删除并添加node节点
slug: 7
date: 2021/12/30 16:58:32
updated: 2021/12/30 16:58:32
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


    在已建立好的k8s集群中删除节点后，进行添加新的节点，可参考用于添加全新node节点，若新的node需要安装docker和k8s基础组件。  

  

    **建立集群可以参考曾经的文章：**[**CentOS8 搭建Kubernetes**](http://mp.weixin.qq.com/s?__biz=MzI0MzA4NTM2NQ==&mid=2247483901&idx=1&sn=54414b634fbeb5b4c411164672b352c0&chksm=e97338a7de04b1b127b93321c5820316beae1c309b1de0672d9e1f093b79cfb4ab9b747c6254&scene=21#wechat_redirect)

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91245c6a9f3640c582c4c84ab2d27296~tplv-k3u1fbpfcp-zoom-1.image)Linux运维交流社区推荐搜索

k8s集群

k8s集群添加节点

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e7741b987ac4028a80b0d0971e17fa4~tplv-k3u1fbpfcp-zoom-1.image)

  

  

    1. 在master中，查看节点数和要删除的节点数，因集群ip进行了修改，节点出现了异常。  

  

        \[root@k8s-master ~\]# kubectl  get nodes

        NAME         STATUS     ROLES    AGE   VERSION

        k8s-master   Ready      master   13d   v1.19.3

        k8s-node1    NotReady   <none>   13d   v1.19.3

        k8s-node2    NotReady   <none>   13d   v1.19.3

  

    2. 进行删除节点操作。  

  

        \[root@k8s-master ~\]# kubectl  delete nodes k8s-node1

        node "k8s-node1" deleted

        \[root@k8s-master ~\]# kubectl  delete nodes k8s-node2

        node "k8s-node2" deleted

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/edb1bb8ede5e416088ec79d27c2666e7~tplv-k3u1fbpfcp-zoom-1.image)

  

    3. 在被删除的node节点中清空集群数据信息。  

```
\[root@k8s-node1 ~\]# kubeadm reset
\[reset\] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
\[reset\] Are you sure you want to proceed? \[y/N\]: y
\[preflight\] Running pre-flight checks
W1121 05:40:44.876393    9649 removeetcdmember.go:79\] \[reset\] No kubeadm config, using etcd pod spec to get data directory
\[reset\] No etcd config found. Assuming external etcd
\[reset\] Please, manually reset etcd to prevent further issues
\[reset\] Stopping the kubelet service
\[reset\] Unmounting mounted directories in "/var/lib/kubelet"
\[reset\] Deleting contents of config directories: \[/etc/kubernetes/manifests /etc/kubernetes/pki\]
\[reset\] Deleting files: \[/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf\]
\[reset\] Deleting contents of stateful directories: \[/var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni\]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.

```

  

    4. 在集群中查看集群的token值

  

```
\[root@k8s-master ~\]# kubeadm token create --print-join-command
W1121 05:38:27.405833   12512 configset.go:348\] WARNING: kubeadm cannot validate component configs for API groups \[kubelet.config.k8s.io kubeproxy.config.k8s.io\]
kubeadm join 10.0.1.48:6443 --token 8xwcaq.qxekio9xd02ed936     --discovery-token-ca-cert-hash sha256:d988ba566675095ae25255d63b21cc4d5a9a69bee9905dc638f58b217c651c14 
```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75c22d7e80d94e34ab6c949f6245c676~tplv-k3u1fbpfcp-zoom-1.image)

  

    5. 将node节点重新添加到k8s集群中

  

```
\[root@k8s-node1 ~\]# kubeadm join 10.0.1.48:6443 --token 8xwcaq.qxekio9xd02ed936     --discovery-token-ca-cert-hash sha256:d988ba566675095ae25255d63b21cc4d5a9a69bee9905dc638f58b217c651c14
\[preflight\] Running pre-flight checks
  \[WARNING IsDockerSystemdCheck\]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
\[preflight\] Reading configuration from the cluster...
\[preflight\] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
\[kubelet-start\] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
\[kubelet-start\] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
\[kubelet-start\] Starting the kubelet
\[kubelet-start\] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
\* Certificate signing request was sent to apiserver and a response was received.
\* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```

  

    6. 查看pod情况  

```
\[root@k8s-master ~\]# kubectl get pods -n kube-system -o wide
NAME                                 READY   STATUS              RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES
coredns-f9fd979d6-c6qrl              0/1     ContainerCreating   1          13d   <none>       k8s-node1    <none>           <none>
coredns-f9fd979d6-hmpbj              1/1     Running             0          13d   10.244.2.2   k8s-node2    <none>           <none>
etcd-k8s-master                      1/1     Running             5          13d   10.0.1.48    k8s-master   <none>           <none>
kube-apiserver-k8s-master            1/1     Running             6          13d   10.0.1.48    k8s-master   <none>           <none>
kube-controller-manager-k8s-master   1/1     Running             5          13d   10.0.1.48    k8s-master   <none>           <none>
kube-flannel-ds-5ftj9                1/1     Running             4          13d   10.0.1.48    k8s-master   <none>           <none>
kube-flannel-ds-bwh28                1/1     Running             0          23m   10.0.1.50    k8s-node2    <none>           <none>
kube-flannel-ds-ttx7c                0/1     Init:0/1            0          23m   10.0.1.49    k8s-node1    <none>           <none>
kube-proxy-4xxxh                     0/1     ContainerCreating   2          13d   10.0.1.49    k8s-node1    <none>           <none>
kube-proxy-7rs4w                     1/1     Running             0          13d   10.0.1.50    k8s-node2    <none>           <none>
kube-proxy-d5hrv                     1/1     Running             4          13d   10.0.1.48    k8s-master   <none>           <none>
kube-scheduler-k8s-master            1/1     Running             5          13d   10.0.1.48    k8s-master   <none>           <none>
```

  

    7.查看node情况  

```
\[root@k8s-master ~\]# kubectl  get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   13d   v1.19.3
k8s-node1    Ready    <none>   24m   v1.19.3
k8s-node2    Ready    <none>   24m   v1.19.3
```

  

  

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19a338fd86bc46e5bee3ff2d4b8e86de~tplv-k3u1fbpfcp-zoom-1.image)

高新科技园