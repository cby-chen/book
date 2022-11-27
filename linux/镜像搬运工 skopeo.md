# 镜像搬运工 skopeo

### 介绍
skopeo 是一个命令行工具，可对容器镜像和容器存储进行操作。 在没有dockerd的环境下，使用 skopeo 操作镜像是非常方便的。

### 安装
```shell

# 安装 skopeo
https://github.com/containers/skopeo/blob/main/install.md

root@cby:~# . /etc/os-release
root@cby:~# echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
root@cby:~# curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/Release.key | sudo apt-key add -
root@cby:~# sudo apt-get update
root@cby:~# sudo apt-get -y upgrade
root@cby:~# sudo apt-get -y install skopeo
root@cby:~# skopeo --version

root@cby:~# skopeo --help    # 子命令可采用如下命令 skopeo [command] --help 命令
Usage:
 skopeo [flags]
 skopeo [command]
Available Commands:  
 copy          # 复制一个镜像从 A 到 B，这里的 A 和 B 可以为本地 docker 镜像或者 registry 上的镜像；
 delete        # 删除一个镜像 tag，可以是本地 docker 镜像或者 registry 上的镜像；
 help          # 帮助查看
 inspect       # 查看一个镜像的 manifest 或者 image config 详细信息；
 list-tags     # 列出存储库名称指定的镜像的tag
 login           # 登陆某个镜像仓库,类似于 docker login 命令
 logout          # 退出某个已认证的镜像仓库, 类似于 docker logout 命令
 manifest-digest # 计算文件的清单摘要是一个sha256sum 值
 standalone-sign   # 使用本地文件创建签名
 standalone-verify # 验证本地文件的签名
 sync              # 将一个或多个图像从一个位置同步到另一个位置 (该功能非常Nice)
Flags:
   --command-timeout duration   # 命令超时时间(单位秒)
   --debug                      # 启用debug模式
   --insecure-policy            # 在不进行任何策略检查的情况下运行该工具（如果没有配置 policy 的话需要加上该参数）
   --override-arch ARCH         # 处理镜像时覆盖客户端 CPU 体系架构，如在 amd64 的机器上用 skopeo 处理 arm64 的镜像
   --override-os OS             # 处理镜像时覆盖客户端 OS
   --override-variant VARIANT   # 处理镜像时使用VARIANT而不是运行架构变量
   --policy string              # 信任策略文件的路径 (为镜像配置安全策略情况下使用)
   --registries.d DIR           # 在目录中使用Registry配置文件（例如，用于容器签名存储）
   --tmpdir string              # 用于存储临时文件的目录
-h, --help                       help for skopeo  
-v, --version                    Version for Skopeo

# 查看已有的认证信息
root@cby:~# cat ~/.docker/config.json
{
        "auths": {
                "core.oiox.cn:30785": {
                        "auth": "XXXX"
                },
                "hb.oiox.cn": {
                        "auth": "XXXX"
                },
                "swr.cn-north-1.myhuaweicloud.com": {
                        "auth": "XXXX"
                }
        }
}root@cby:~# 

```
### 使用
```shell
# 从一个仓库拷贝到另一个仓库
root@cby:~# skopeo copy docker://docker.io/busybox:latest docker://hb.oiox.cn/cby/busybox:latest --dest-authfile /root/.docker/config.json --src-tls-verify=false --dest-tls-verify=false
Getting image source signatures
Copying blob 405fecb6a2fa done  
Copying config 9d5226e6ce done  
Writing manifest to image destination
Storing signatures
root@cby:~# 

# 从一个仓库同步所以版本到另一个仓库
root@cby:~# skopeo sync --src docker --dest docker k8s.gcr.io/etcd hb.oiox.cn/cby/ --src-tls-verify=false --dest-tls-verify=false
INFO[0000] Tag presence check                            imagename=k8s.gcr.io/etcd tagged=false
INFO[0000] Getting tags                                  image=k8s.gcr.io/etcd
INFO[0004] Copying image ref 1/106                       from="docker://k8s.gcr.io/etcd:2.0.12" to="docker://hb.oiox.cn/cby/etcd:2.0.12"
Getting image source signatures
Copying blob a3ed95caeb02 done  
Copying blob a3ed95caeb02 done  
Copying blob a3ed95caeb02 done  
Copying blob 35c8bf5fd6cd done  
Copying blob a7e0d6960478 done  
Copying blob 3109a5487eac done  
Copying config 8c32a2c999 done  
Writing manifest to image destination
Storing signatures
INFO[0020] Copying image ref 2/106                       from="docker://k8s.gcr.io/etcd:2.0.13" to="docker://hb.oiox.cn/cby/etcd:2.0.13"
Getting image source signatures
Copying blob a3ed95caeb02 [--------------------------------------] 0.0b / 0.0b
Copying blob a3ed95caeb02 skipped: already exists  
Copying blob 35c8bf5fd6cd skipped: already exists  
Copying blob a3ed95caeb02 skipped: already exists
...
root@cby:~#

# 删除镜像
root@cby:~# skopeo delete docker://hb.oiox.cn/cby/etcd:2.0.12 --tls-verify=false --debug
DEBU[0000] Loading registries configuration "/etc/containers/registries.conf" 
DEBU[0000] Loading registries configuration "/etc/containers/registries.conf.d/shortnames.conf" 
DEBU[0000] Found credentials for hb.oiox.cn in credential helper containers-auth.json 
DEBU[0000] Using registries.d directory /etc/containers/registries.d for sigstore configuration 
DEBU[0000]  No signature storage configuration found for hb.oiox.cn/cby/etcd:2.0.12, using built-in default file:///var/lib/containers/sigstore 
DEBU[0000] Looking for TLS certificates and private keys in /etc/docker/certs.d/hb.oiox.cn 
DEBU[0000] GET https://hb.oiox.cn/v2/                   
DEBU[0000] Ping https://hb.oiox.cn/v2/ status 401       
DEBU[0000] GET https://hb.oiox.cn/service/token?account=admin&scope=repository%3Acby%2Fetcd%3A%2A&service=harbor-registry 
DEBU[0000] GET https://hb.oiox.cn/v2/cby/etcd/manifests/2.0.12 
DEBU[0000] DELETE https://hb.oiox.cn/v2/cby/etcd/manifests/sha256:24cf1202eea3953f9a8c44b0930d03666019ff8c277a0f6cd6190645eb1f7ba5 
DEBU[0000] Deleting /var/lib/containers/sigstore/cby/etcd@sha256=24cf1202eea3953f9a8c44b0930d03666019ff8c277a0f6cd6190645eb1f7ba5/signature-1 
root@cby:~# 


# 查看有哪些tags
root@cby:~# skopeo list-tags docker://k8s.gcr.io/pause
{
    "Repository": "k8s.gcr.io/pause",
    "Tags": [
        "0.8.0",
        "1.0",
        "2.0",
        "3.0",
        "3.1",
        "3.2",
        "3.3",
        "3.4.1",
        "3.5",
        "3.6",
        "3.7",
        "3.8",
        "3.9",
        "go",
        "latest",
        "sha256-7031c1b283388d2c2e09b57badb803c05ebed362dc88d84b480cc47f72a21097.sig",
        "sha256-9001185023633d17a2f98ff69b6ff2615b8ea02a825adffa40422f51dfdcde9d.sig",
        "test",
        "test2"
    ]
}
root@cby:~# 

```
### 实际应用
```shell
# 实际应用

root@cby:~# vim config.sh 
root@cby:~# cat config.sh 
echo "gcr.io:" >> images.yaml
echo "  images:" >> images.yaml
echo "    kaniko-project/executor:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://gcr.io/kaniko-project/executor | grep \"v | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    google-samples/xtrabackup:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://gcr.io/google-samples/xtrabackup | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | awk -F '"' '{print "    - "$2}' >> images.yaml
echo "docker.io:" >> images.yaml
echo "  images:" >> images.yaml
echo "    calico/typha:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.io/calico/typha | grep \"v | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    calico/cni:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.io/calico/cni | grep \"v | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    calico/node:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.io/calico/node | grep \"v | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    calico/kube-controllers:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.io/calico/kube-controllers | grep \"v | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | awk -F '"' '{print "    - "$2}' >> images.yaml
echo "docker.elastic.co:" >> images.yaml
echo "  images:" >> images.yaml
echo "    elasticsearch/elasticsearch:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/elasticsearch/elasticsearch | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    kibana/kibana:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/kibana/kibana | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    logstash/logstash:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/logstash/logstash | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    beats/filebeat:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/beats/filebeat | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    beats/heartbeat:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/beats/heartbeat | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    beats/packetbeat:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/beats/packetbeat | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    beats/auditbeat:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/beats/auditbeat | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    beats/journalbeat:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/beats/journalbeat | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    beats/metricbeat:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/beats/metricbeat | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    apm/apm-server:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/apm/apm-server | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    app-search/app-search:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://docker.elastic.co/app-search/app-search | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "quay.io:" >> images.yaml
echo "  images:" >> images.yaml
echo "    coreos/flannel:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/coreos/flannel | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    ceph/ceph:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/ceph/ceph | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    cephcsi/cephcsi:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/cephcsi/cephcsi | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    csiaddons/k8s-sidecar:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/csiaddons/k8s-sidecar | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    csiaddons/volumereplication-operator:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/csiaddons/volumereplication-operator | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    prometheus/prometheus:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/prometheus/prometheus | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    prometheus/alertmanager:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/prometheus/alertmanager | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    prometheus/pushgateway:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/prometheus/pushgateway | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    prometheus/blackbox-exporter:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/prometheus/blackbox-exporter | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    prometheus/node-exporter:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/prometheus/node-exporter | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    prometheus-operator/prometheus-config-reloader:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/prometheus-operator/prometheus-config-reloader | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    prometheus-operator/prometheus-operator:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/prometheus-operator/prometheus-operator | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    brancz/kube-rbac-proxy:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/brancz/kube-rbac-proxy | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    cilium/cilium:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/cilium/cilium | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    cilium/operator-generic:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://quay.io/cilium/operator-generic | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "k8s.gcr.io:" >> images.yaml
echo "  images:" >> images.yaml
echo "    etcd:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/etcd | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    pause:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/pause | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    kube-proxy:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/kube-proxy | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    kube-apiserver:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/kube-apiserver | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    kube-scheduler:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/kube-scheduler | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    kube-controller-manager:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/kube-controller-manager | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    coredns/coredns:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/coredns/coredns | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    dns/k8s-dns-node-cache:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/dns/k8s-dns-node-cache | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    metrics-server/metrics-server:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/metrics-server/metrics-server | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    ingress-nginx/controller:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/ingress-nginx/controller | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    ingress-nginx/kube-webhook-certgen:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/ingress-nginx/kube-webhook-certgen | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    kube-state-metrics/kube-state-metrics:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/kube-state-metrics/kube-state-metrics | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    prometheus-adapter/prometheus-adapter:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/prometheus-adapter/prometheus-adapter | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    isig-storage/nfs-subdir-external-provisioner:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    sig-storage/csi-node-driver-registrar:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/sig-storage/csi-node-driver-registrar | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    sig-storage/csi-provisioner:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/sig-storage/csi-provisioner | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    sig-storage/csi-resizer:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/sig-storage/csi-resizer | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    sig-storage/csi-snapshotter:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/sig-storage/csi-snapshotter | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    sig-storage/csi-attacher:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/sig-storage/csi-attacher | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    sig-storage/nfsplugin:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/sig-storage/nfsplugin | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
echo "    defaultbackend-amd64:" >> images.yaml
skopeo list-tags --tls-verify=false  docker://k8s.gcr.io/defaultbackend-amd64 | grep -v alpha | grep -v beta | grep -v rc | grep -v amd64 | grep -v ppc64le | grep -v arm64 | grep -v arm | grep -v s390x | grep -v SNAPSHOT | grep -v debug | grep -v master | grep -v main | grep -v \} | grep -v \] | grep -v \{ | grep -v Repository | grep -v Tags | grep -v dev | grep -v g | grep -v '-'| awk -F '"' '{print "    - "$2}' >> images.yaml
root@cby:~# 
root@cby:~# 


root@cby:~#  vim skopeo.sh
root@cby:~#  cat skopeo.sh 
#!/bin/bash

HUB_USERNAME="xxxx"
HUB_PASSWORD="xxxx"

hub="swr.cn-north-1.myhuaweicloud.com"
repo="$hub/chenby"

rm -rf images.yaml
bash config.sh

if [ -f images.yaml ]; then
   echo "[Start] sync......."
   
   sudo skopeo login --tls-verify=false -u ${HUB_USERNAME} -p ${HUB_PASSWORD} ${hub} \
   && sudo skopeo --tls-verify=false --insecure-policy sync --src yaml --dest docker images.yaml $repo
   
   echo "[End] done."
   
else
    echo "[Error]not found images.yaml!"
fi
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