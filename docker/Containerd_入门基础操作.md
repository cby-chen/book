---
layout: post
cid: 103
title: Containerd 入门基础操作
slug: 103
date: 2022/03/21 17:35:00
updated: 2022/03/23 10:03:15
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


Containerd 被 Docker、Kubernetes  CRI 和其他一些项目使用

Containerd 旨在轻松嵌入到更大的系统中。Docker 在后台使用 containerd来运行容器。Kubernetes 可以通过 CRI 使用 containerd来管理单个节点上的容器。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fc6fe64ea794d388dd0abbb196c8ac0~tplv-k3u1fbpfcp-zoom-1.image)

  

  

生成默认配置

  

```
root@hello:~# containerd config default > /etc/containerd/config.toml


root@hello:~# vim /etc/containerd/config.toml 


root@hello:~# cat /etc/containerd/config.toml
version = 2
root = "/var/lib/containerd"
state = "/run/containerd"
plugin_dir = ""
disabled_plugins = []
required_plugins = []
oom_score = 0


[grpc]
  address = "/run/containerd/containerd.sock"
  tcp_address = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216


[ttrpc]
  address = ""
  uid = 0
  gid = 0


[debug]
  address = ""
  uid = 0
  gid = 0
  level = ""


[metrics]
  address = ""
  grpc_histogram = false


[cgroup]
  path = ""


[timeouts]
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"


[plugins]
  [plugins."io.containerd.gc.v1.scheduler"]
    pause_threshold = 0.02
    deletion_threshold = 0
    mutation_threshold = 100
    schedule_delay = "0s"
    startup_delay = "100ms"
  [plugins."io.containerd.grpc.v1.cri"]
    disable_tcp_service = true
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    stream_idle_timeout = "4h0m0s"
    enable_selinux = false
    selinux_category_range = 1024
    sandbox_image = "k8s.gcr.io/pause:3.2"
    stats_collect_period = 10
    systemd_cgroup = false
    enable_tls_streaming = false
    max_container_log_line_size = 16384
    disable_cgroup = false
    disable_apparmor = false
    restrict_oom_score_adj = false
    max_concurrent_downloads = 3
    disable_proc_mount = false
    unset_seccomp_profile = ""
    tolerate_missing_hugetlb_controller = true
    disable_hugetlb_controller = true
    ignore_image_defined_volumes = false
    [plugins."io.containerd.grpc.v1.cri".containerd]
      snapshotter = "overlayfs"
      default_runtime_name = "runc"
      no_pivot = false
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
        runtime_type = ""
        runtime_engine = ""
        runtime_root = ""
        privileged_without_host_devices = false
        base_runtime_spec = ""
      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        runtime_type = ""
        runtime_engine = ""
        runtime_root = ""
        privileged_without_host_devices = false
        base_runtime_spec = ""
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          runtime_engine = ""
          runtime_root = ""
          privileged_without_host_devices = false
          base_runtime_spec = ""
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      max_conf_num = 1
      conf_template = ""
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://registry-1.docker.io"]
    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = ""
    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""
  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"
  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"
  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"
  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false
  [plugins."io.containerd.runtime.v1.linux"]
    shim = "containerd-shim"
    runtime = "runc"
    runtime_root = ""
    no_shim = false
    shim_debug = false
  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]
  [plugins."io.containerd.snapshotter.v1.devmapper"]
    root_path = ""
    pool_name = ""
    base_image_size = ""
    async_remove = false
root@hello:~# 
```

  

配置镜像加速器

  

```
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://ted9wxpi.mirror.aliyuncs.com"]
```

  

  

ctr 相当于核心组件，通过加载插件的方式来支持各种操作。

  

使用 ctr plugins ls 来查看当前加载的插件和支持的操作。

  

```
[root@k8s-master01 ~]# ctr plugins ls
TYPE                            ID                       PLATFORMS      STATUS    
io.containerd.content.v1        content                  -              ok        
io.containerd.snapshotter.v1    aufs                     linux/amd64    error     
io.containerd.snapshotter.v1    devmapper                linux/amd64    error     
io.containerd.snapshotter.v1    native                   linux/amd64    ok        
io.containerd.snapshotter.v1    overlayfs                linux/amd64    ok        
io.containerd.snapshotter.v1    zfs                      linux/amd64    error     
io.containerd.metadata.v1       bolt                     -              ok        
io.containerd.differ.v1         walking                  linux/amd64    ok        
io.containerd.gc.v1             scheduler                -              ok        
io.containerd.service.v1        introspection-service    -              ok        
io.containerd.service.v1        containers-service       -              ok        
io.containerd.service.v1        content-service          -              ok        
io.containerd.service.v1        diff-service             -              ok        
io.containerd.service.v1        images-service           -              ok        
io.containerd.service.v1        leases-service           -              ok        
io.containerd.service.v1        namespaces-service       -              ok        
io.containerd.service.v1        snapshots-service        -              ok        
io.containerd.runtime.v1        linux                    linux/amd64    ok        
io.containerd.runtime.v2        task                     linux/amd64    ok        
io.containerd.monitor.v1        cgroups                  linux/amd64    ok        
io.containerd.service.v1        tasks-service            -              ok        
io.containerd.internal.v1       restart                  -              ok        
io.containerd.grpc.v1           containers               -              ok        
io.containerd.grpc.v1           content                  -              ok        
io.containerd.grpc.v1           diff                     -              ok        
io.containerd.grpc.v1           events                   -              ok        
io.containerd.grpc.v1           healthcheck              -              ok        
io.containerd.grpc.v1           images                   -              ok        
io.containerd.grpc.v1           leases                   -              ok        
io.containerd.grpc.v1           namespaces               -              ok        
io.containerd.internal.v1       opt                      -              ok        
io.containerd.grpc.v1           snapshots                -              ok        
io.containerd.grpc.v1           tasks                    -              ok        
io.containerd.grpc.v1           version                  -              ok        
io.containerd.grpc.v1           cri                      linux/amd64    ok        
[root@k8s-master01 ~]#
```

  

ctr plugins ls 命令会展示三列 ，第二列 ID 就是对应的命令。

  

  

例如 plugins 的 id 为 content 可使用 ctr content --help 来查看帮助，以及其他命令来执行操作。

  

```
[root@k8s-master01 ~]# ctr content --help
NAME:
   ctr content - manage content


USAGE:
   ctr content [global options] command [command options] [arguments...]


VERSION:
   1.4.13


COMMANDS:
   active                   display active transfers
   delete, del, remove, rm  permanently delete one or more blobs
   edit                     edit a blob and return a new digest
   fetch                    fetch all content for an image into containerd
   fetch-object             retrieve objects from a remote
   get                      get the data for an object
   ingest                   accept content into the store
   list, ls                 list all blobs in the store
   push-object              push an object to a remote
   label                    add labels to content


GLOBAL OPTIONS:
   --help, -h  show help
[root@k8s-master01 ~]# 
```

  

  

查看有哪些命名空间

  

```
[root@k8s-master01 ~]# ctr namespace ls
NAME    LABELS 
default        
k8s.io         
[root@k8s-master01 ~]# 
```

  

  

查看 k8s.io 空间下的镜像有哪些

  

```
[root@k8s-master01 ~]# ctr -n k8s.io  images ls
REF                                                                                                                                 TYPE                                                      DIGEST                                                                  SIZE      PLATFORMS                                                                    LABELS                          
k8s.gcr.io/ingress-nginx/kube-webhook-certgen@sha256:64d8c73dca984af206adf9d6d7e46aa550362b1d7a01f3a0a91b20cc67868660               application/vnd.docker.distribution.manifest.list.v2+json sha256:64d8c73dca984af206adf9d6d7e46aa550362b1d7a01f3a0a91b20cc67868660 18.0 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/s390x
```

  

接下来 从 容器的 生命周期流程 来说明 ctr 命令的使用。

  

ctr images ls 查看镜像

  

```
[root@k8s-master01 ~]# ctr images ls
REF                            TYPE                                                      DIGEST                                                                  SIZE    PLATFORMS                                                                                LABELS 
docker.io/library/nginx:alpine application/vnd.docker.distribution.manifest.list.v2+json sha256:77cc350019d0188d3115084265483dcefdd8489ccf719ff4e4c956b48de8ff6a 9.7 MiB linux/386,linux/amd64,linux/arm/v6,linux/arm/v7,linux/arm64/v8,linux/ppc64le,linux/s390x -      
[root@k8s-master01 ~]#
```

  

  

  

ctr images pull 拉取镜像 

  

```
[root@k8s-master01 ~]# ctr images pull docker.io/library/nginx:alpine
docker.io/library/nginx:alpine:                                                   resolved       |++++++++++++++++++++++++++++++++++++++| 
index-sha256:77cc350019d0188d3115084265483dcefdd8489ccf719ff4e4c956b48de8ff6a:    done           |++++++++++++++++++++++++++++++++++++++| 
manifest-sha256:1e3458b8841319dec826a9a63b66f98c0bb260d50454dcdbdfe414eed362a3c4: done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:9a9d990f68b82fceea08b4b08a0549e3de8ba7840ac721e0b8cc4d2d27e33ccf:    done           |++++++++++++++++++++++++++++++++++++++| 
config-sha256:7d73f57a7cf733ff46e22c3d60cb237f7b29e8e7ec6753922f2daa7f5af5d186:   done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:6c53e58c6af6338b6ea1ddeb46b638a719e4afdd2cffb5cf80362af3e61099d1:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:bda3fba8f6c468c5b9f60cec056498ebdedf711410c8864f956f0b8d3408428c:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:e07cc103cea6f44382a40ffe1f7d893781521aa2723765c069f23480e674dd0c:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:3d243047344378e9b7136d552d48feb7ea8b6fe14ce0990e0cc011d5e369626a:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:4ba4f346920eaf3fd54877cf123ac46a7bbea16f23d4b0bdc210988ebe7969f0:    done           |++++++++++++++++++++++++++++++++++++++| 
elapsed: 14.8s                                                                    total:  9.7 Mi (671.0 KiB/s)                                     
unpacking linux/amd64 sha256:77cc350019d0188d3115084265483dcefdd8489ccf719ff4e4c956b48de8ff6a...
done
[root@k8s-master01 ~]# 
```

  

只有通过 crictl 或者 Kubernetes 调用时 mirror 才会生效，通过 ctr 拉取是不会生效的。

  

  

ctr images rm 删除镜像

  

```
[root@k8s-master01 ~]# ctr images rm docker.io/library/nginx:alpine
docker.io/library/nginx:alpine


[root@k8s-master01 ~]# 
[root@k8s-master01 ~]# ctr images ls
REF TYPE DIGEST SIZE PLATFORMS LABELS 
[root@k8s-master01 ~]# 
```

  

ctr images mount 挂载

  

```
[root@k8s-master01 ~]# ctr images ls
REF                            TYPE                                                      DIGEST                                                                  SIZE    PLATFORMS                                                                                LABELS 
docker.io/library/nginx:alpine application/vnd.docker.distribution.manifest.list.v2+json sha256:77cc350019d0188d3115084265483dcefdd8489ccf719ff4e4c956b48de8ff6a 9.7 MiB linux/386,linux/amd64,linux/arm/v6,linux/arm/v7,linux/arm64/v8,linux/ppc64le,linux/s390x -      


[root@k8s-master01 ~]# ctr images mount docker.io/library/nginx:alpine /cby
sha256:7a7cbbee0f17b403a980a36ae708bbd9ee428511a7219da36c50ce7e33662d43
/cby
[root@k8s-master01 ~]# 


[root@k8s-master01 ~]# ls /cby/
bin  docker-entrypoint.d   etc   lib    mnt  proc  run   srv  tmp  var
dev  docker-entrypoint.sh  home  media  opt  root  sbin  sys  usr
[root@k8s-master01 ~]# 
```

  

ctr images unmount 卸载

  

```
[root@k8s-master01 ~]# ctr images  unmount  /cby
/cby
[root@k8s-master01 ~]#
```

  

ctr images  export 导出镜像

  

```
root@hello:~# ctr images  export nginx.tar docker.io/library/nginx:alpine
root@hello:~# 
root@hello:~# ls nginx.tar 
nginx.tar
root@hello:~# 
```

  

ctr images import 导入镜像

  

```
root@hello:~# ctr images import nginx.tar
unpacking docker.io/library/nginx:alpine (sha256:77cc350019d0188d3115084265483dcefdd8489ccf719ff4e4c956b48de8ff6a)...done
root@hello:~# 
```

  

  

ctr中 containers 是镜像实例化的一个虚拟环境，提供一个磁盘，模拟空间，就好比你电脑处于关机状态一样。

  

ctr中 tasks 是将容器运行起来，电脑开机了 ，初始化进程等 ，task就是的这么个形式。

  

  

ctr containers ls 查看容器

  

```
root@hello:~# ctr containers  ls
CONTAINER    IMAGE                             RUNTIME                  
nginx        docker.io/library/nginx:alpine    io.containerd.runc.v2    
root@hello:~# 
```

  

ctr containers create 创建容器

  

```
root@hello:~# ctr containers create docker.io/library/nginx:alpine nginx
root@hello:~# 
```

  

ctr containers rm 删除容器

  

```
root@hello:~# ctr containers rm nginx

root@hello:~# ctr containers  ls
CONTAINER    IMAGE    RUNTIME    
root@hello:~# 
```

  

ctr containers info 查看详细信息

  

```
root@hello:~# ctr containers info nginx
{
    "ID": "nginx",
    "Labels": {
        "io.containerd.image.config.stop-signal": "SIGQUIT"
    },
    "Image": "docker.io/library/nginx:alpine",
    "Runtime": {
        "Name": "io.containerd.runc.v2",
        "Options": {
            "type_url": "containerd.runc.v1.Options"
        }
    },
    "SnapshotKey": "nginx",
    "Snapshotter": "overlayfs",
    "CreatedAt": "2022-03-21T08:51:45.127872097Z",
    "UpdatedAt": "2022-03-21T08:51:45.127872097Z",
    "Extensions": null,
    "Spec": {
---略---
```

  

  

create 的命令创建了容器后，并没有处于运行状态，只是一个静态的容器。一个 container 对象只是包含了运行一个容器所需的资源及配置的数据结构，这意味着 namespaces、rootfs 和容器的配置都已经初始化成功了，只是用户进程(这里是 nginx)还没有启动。

  

  

ctr tasks start -d 在后台运行容器

  

```
root@hello:~# ctr tasks start -d  nginx


root@hello:~# ctr tasks ls
TASK     PID       STATUS    
nginx    118454    RUNNING
root@hello:~# 
```

  

ctr task exec 进入容器，id随便写就行，需要将其唯一

  

```
root@hello:~# ctr task exec --exec-id 1 -t nginx sh
/ # 
```

  

ctr task pause 暂停容器

  

```
root@hello:~# ctr task pause nginx
root@hello:~# ctr task ls
TASK     PID       STATUS    
nginx    118454    PAUSED
root@hello:~#
```

  

ctr task resume 恢复容器

  

```
root@hello:~# ctr task resume nginx
root@hello:~# ctr task ls
TASK     PID       STATUS    
nginx    118454    RUNNING
root@hello:~# 
```

  

ctr task kill 杀死容器

  

```
root@hello:~# ctr task kill nginx
root@hello:~# ctr task ls
TASK     PID       STATUS    
nginx    118454    STOPPED
root@hello:~# 
```

  

ctr task metrics获取容器信息

  

```
root@hello:~# ctr task metrics nginx
ID       TIMESTAMP                                  
nginx    2022-03-21 09:05:49.949321537 +0000 UTC    


METRIC                   VALUE                                                                       
memory.usage_in_bytes    3821568                                                                     
memory.limit_in_bytes    9223372036854771712                                                         
memory.stat.cache        135168                                                                      
cpuacct.usage            176641571                                                                   
cpuacct.usage_percpu     [24856408 21740008 12150472 37947198 31775746 28169704 7366623 12635412]    
pids.current             0                                                                           
pids.limit               0                                                                           
root@hello:~# 
```

  

ctr tasks rm 删除容器

  

```
root@hello:~# ctr tasks rm nginx
root@hello:~# ctr tasks ls
TASK    PID    STATUS    
root@hello:~#
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

