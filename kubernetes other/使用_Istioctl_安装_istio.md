---
layout: post
cid: 38
title: 使用 Istioctl 安装 istio
slug: 38
date: 2021/12/30 17:08:34
updated: 2021/12/30 17:08:34
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


使用 Istioctl 安装 istio

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c57e2d07c5e47588442dd1cefd04dab~tplv-k3u1fbpfcp-zoom-1.image)

  

**下载 Istio**

转到 Istio 发布 页面，下载针对你操作系统的安装文件， 或用自动化工具下载并提取最新版本（Linux 或 macOS）：

  

```
[root@k8s-master-node1 ~]# curl -L https://istio.io/downloadIstio | sh -


```

  

若无法下载可以手动写入文件进行执行  

  

**脚本内容：**

  

  

```
#!/bin/sh

# Copyright Istio Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.


#
# This file will be fetched as: curl -L https://git.io/getLatestIstio | sh -
# so it should be pure bourne shell, not bash (and not reference other scripts)
#
# The script fetches the latest Istio release candidate and untars it.
# You can pass variables on the command line to download a specific version
# or to override the processor architecture. For example, to download
# Istio 1.6.8 for the x86_64 architecture,
# run curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.6.8 TARGET_ARCH=x86_64 sh -.


set -e


# Determines the operating system.
OS="$(uname)"
if [ "x${OS}" = "xDarwin" ] ; then
  OSEXT="osx"
else
  OSEXT="linux"
fi


# Determine the latest Istio version by version number ignoring alpha, beta, and rc versions.
if [ "x${ISTIO_VERSION}" = "x" ] ; then
  ISTIO_VERSION="$(curl -sL https://github.com/istio/istio/releases | \
                  grep -o 'releases/[0-9]*.[0-9]*.[0-9]*/' | sort -V | \
                  tail -1 | awk -F'/' '{ print $2}')"
  ISTIO_VERSION="${ISTIO_VERSION##*/}"
fi


LOCAL_ARCH=$(uname -m)
if [ "${TARGET_ARCH}" ]; then
    LOCAL_ARCH=${TARGET_ARCH}
fi


case "${LOCAL_ARCH}" in
  x86_64)
    ISTIO_ARCH=amd64
    ;;
  armv8*)
    ISTIO_ARCH=arm64
    ;;
  aarch64*)
    ISTIO_ARCH=arm64
    ;;
  armv*)
    ISTIO_ARCH=armv7
    ;;
  amd64|arm64)
    ISTIO_ARCH=${LOCAL_ARCH}
    ;;
  *)
    echo "This system's architecture, ${LOCAL_ARCH}, isn't supported"
    exit 1
    ;;
esac


if [ "x${ISTIO_VERSION}" = "x" ] ; then
  printf "Unable to get latest Istio version. Set ISTIO_VERSION env var and re-run. For example: export ISTIO_VERSION=1.0.4"
  exit 1;
fi


NAME="istio-$ISTIO_VERSION"
URL="https://github.com/istio/istio/releases/download/${ISTIO_VERSION}/istio-${ISTIO_VERSION}-${OSEXT}.tar.gz"
ARCH_URL="https://github.com/istio/istio/releases/download/${ISTIO_VERSION}/istio-${ISTIO_VERSION}-${OSEXT}-${ISTIO_ARCH}.tar.gz"


with_arch() {
  printf "\nDownloading %s from %s ...\n" "$NAME" "$ARCH_URL"
  if ! curl -o /dev/null -sIf "$ARCH_URL"; then
    printf "\n%s is not found, please specify a valid ISTIO_VERSION and TARGET_ARCH\n" "$ARCH_URL"
    exit 1
  fi
  curl -fsLO "$ARCH_URL"
  filename="istio-${ISTIO_VERSION}-${OSEXT}-${ISTIO_ARCH}.tar.gz"
  tar -xzf "${filename}"
  rm "${filename}"
}


without_arch() {
  printf "\nDownloading %s from %s ..." "$NAME" "$URL"
  if ! curl -o /dev/null -sIf "$URL"; then
    printf "\n%s is not found, please specify a valid ISTIO_VERSION\n" "$URL"
    exit 1
  fi
  curl -fsLO "$URL"
  filename="istio-${ISTIO_VERSION}-${OSEXT}.tar.gz"
  tar -xzf "${filename}"
  rm "${filename}"
}


# Istio 1.6 and above support arch
# Istio 1.5 and below do not have arch support
ARCH_SUPPORTED="1.6"


if [ "${OS}" = "Linux" ] ; then
  # This checks if ISTIO_VERSION is less than ARCH_SUPPORTED (version-sort's before it)
  if [ "$(printf '%s\n%s' "${ARCH_SUPPORTED}" "${ISTIO_VERSION}" | sort -V | head -n 1)" = "${ISTIO_VERSION}" ]; then
    without_arch
  else
    with_arch
  fi
elif [ "x${OS}" = "xDarwin" ] ; then
  without_arch
else
  printf "\n\n"
  printf "Unable to download Istio %s at this moment!\n" "$ISTIO_VERSION"
  printf "Please verify the version you are trying to download.\n\n"
  exit 1
fi


printf ""
printf "\nIstio %s Download Complete!\n" "$ISTIO_VERSION"
printf "\n"
printf "Istio has been successfully downloaded into the %s folder on your system.\n" "$NAME"
printf "\n"
BINDIR="$(cd "$NAME/bin" && pwd)"
printf "Next Steps:\n"
printf "See https://istio.io/latest/docs/setup/install/ to add Istio to your Kubernetes cluster.\n"
printf "\n"
printf "To configure the istioctl client tool for your workstation,\n"
printf "add the %s directory to your environment path variable with:\n" "$BINDIR"
printf "\t export PATH=\"\$PATH:%s\"\n" "$BINDIR"
printf "\n"
printf "Begin the Istio pre-installation check by running:\n"
printf "\t istioctl x precheck \n"
printf "\n"
printf "Need more information? Visit https://istio.io/latest/docs/setup/install/ \n"
```

```
[root@k8s-master-node1 ~]# bash istio.sh


```

  

转到 Istio 包目录。例如，如果包是 istio-1.11.4：

  

```
[root@k8s-master-node1 ~]# cd istio-1.11.4/
[root@k8s-master-node1 ~/istio-1.11.4]# ll
total 28
drwxr-x---.  2 root root    22 Oct 13 22:50 bin
-rw-r--r--.  1 root root 11348 Oct 13 22:50 LICENSE
drwxr-xr-x.  5 root root    52 Oct 13 22:50 manifests
-rw-r-----.  1 root root   854 Oct 13 22:50 manifest.yaml
-rw-r--r--.  1 root root  5866 Oct 13 22:50 README.md
drwxr-xr-x. 21 root root  4096 Oct 13 22:50 samples
drwxr-xr-x.  3 root root    57 Oct 13 22:50 tools
[root@k8s-master-node1 ~/istio-1.11.4]#
```

  

**安装目录包含：**

  

samples/ 目录下的示例应用程序

bin/ 目录下的 istioctl 客户端二进制文件 .

将 istioctl 客户端加入搜索路径（Linux or macOS）:

  

```
$ export PATH=$PWD/bin:$PATH



export PATH=/root/istio-1.11.4/bin:$PATH


[root@k8s-master-node1 ~/istio-1.11.4]# export PATH=$PWD/bin:$PATH
[root@k8s-master-node1 ~/istio-1.11.4]#
[root@k8s-master-node1 ~/istio-1.11.4]# vim /etc/profile
[root@k8s-master-node1 ~/istio-1.11.4]# tail -n 2 /etc/profile


export PATH=/root/istio-1.11.4/bin:$PATH
[root@k8s-master-node1 ~/istio-1.11.4]#
```

  

**使用默认配置档安装 Istio**

最简单的选择是用下面命令安装 Istio 默认 配置档：

  

```
[root@k8s-master-node1 ~]# istioctl version
no running Istio pods in "istio-system"
1.11.4
[root@k8s-master-node1 ~]#
[root@k8s-master-node1 ~]#
[root@k8s-master-node1 ~]# istioctl install --set profile=demo -y
✔ Istio core installed                                                                                                                                                                                                                                                        
✔ Istiod installed                                                                                                                                                                                                                                                            
✔ Egress gateways installed                                                                                                                                                                                                                                                  
✔ Ingress gateways installed                                                                                                                                                                                                                                                  
✔ Installation complete                                                                                                                                                                                                                                                      
Thank you for installing Istio 1.11.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/kWULBRjUv7hHci7T6
[root@k8s-master-node1 ~]#
```

  

**查看istio相应的 namespace 和 pod 是否已经正常创建**

  

```
[root@k8s-master-node1 ~]#
[root@k8s-master-node1 ~]# kubectl get pods -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
istio-egressgateway-756d4db566-wh949    1/1     Running   0          2m
istio-ingressgateway-8577c57fb6-2vrtg   1/1     Running   0          2m
istiod-5847c59c69-l2dt2                 1/1     Running   0          2m39s
[root@k8s-master-node1 ~]#
[root@k8s-master-node1 ~]#
[root@k8s-master-node1 ~]#
[root@k8s-master-node1 ~]#
```

  

**检查 istio 的 CRD 和 API 资源**

  

```
[root@k8s-master-node1 ~]#
[root@k8s-master-node1 ~]# kubectl get crd |grep istio
authorizationpolicies.security.istio.io    2021-11-01T09:43:55Z
destinationrules.networking.istio.io       2021-11-01T09:43:55Z
envoyfilters.networking.istio.io           2021-11-01T09:43:55Z
gateways.networking.istio.io               2021-11-01T09:43:55Z
istiooperators.install.istio.io            2021-11-01T09:43:55Z
peerauthentications.security.istio.io      2021-11-01T09:43:55Z
requestauthentications.security.istio.io   2021-11-01T09:43:55Z
serviceentries.networking.istio.io         2021-11-01T09:43:55Z
sidecars.networking.istio.io               2021-11-01T09:43:55Z
telemetries.telemetry.istio.io             2021-11-01T09:43:55Z
virtualservices.networking.istio.io        2021-11-01T09:43:55Z
workloadentries.networking.istio.io        2021-11-01T09:43:55Z
workloadgroups.networking.istio.io         2021-11-01T09:43:55Z
[root@k8s-master-node1 ~]#



[root@k8s-master-node1 ~]# kubectl api-resources |grep istio
istiooperators                    iop,io       install.istio.io/v1alpha1              true         IstioOperator
destinationrules                  dr           networking.istio.io/v1beta1            true         DestinationRule
envoyfilters                                   networking.istio.io/v1alpha3           true         EnvoyFilter
gateways                          gw           networking.istio.io/v1beta1            true         Gateway
serviceentries                    se           networking.istio.io/v1beta1            true         ServiceEntry
sidecars                                       networking.istio.io/v1beta1            true         Sidecar
virtualservices                   vs           networking.istio.io/v1beta1            true         VirtualService
workloadentries                   we           networking.istio.io/v1beta1            true         WorkloadEntry
workloadgroups                    wg           networking.istio.io/v1alpha3           true         WorkloadGroup
authorizationpolicies                          security.istio.io/v1beta1              true         AuthorizationPolicy
peerauthentications               pa           security.istio.io/v1beta1              true         PeerAuthentication
requestauthentications            ra           security.istio.io/v1beta1              true         RequestAuthentication
telemetries                       telemetry    telemetry.istio.io/v1alpha1            true         Telemetry
[root@k8s-master-node1 ~]#
```

  

**安装 dashboard 组件。命令如下**

  

```
[root@k8s-master-node1 ~]# kubectl apply -f /root/istio-1.11.4/samples/addons/ -n istio-system
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
deployment.apps/jaeger created
service/tracing created
service/zipkin created
service/jaeger-collector created
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
[root@k8s-master-node1 ~]#



[root@k8s-master-node1 ~]# kubectl get pods -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
grafana-68cc7d6d78-792cw                1/1     Running   0          88s
istio-egressgateway-756d4db566-wh949    1/1     Running   0          6m9s
istio-ingressgateway-8577c57fb6-2vrtg   1/1     Running   0          6m9s
istiod-5847c59c69-l2dt2                 1/1     Running   0          6m48s
jaeger-5d44bc5c5d-n6zjq                 1/1     Running   0          88s
kiali-fd9f88575-svz7g                   1/1     Running   0          87s
prometheus-77b49cb997-7d4s9             2/2     Running   0          86s
[root@k8s-master-node1 ~]#
```

  

**将istio-ingressgateway改为NodePort方式，方便访问**

  

```
[root@k8s-master-node1 ~]# kubectl patch service istio-ingressgateway -n istio-system -p '{"spec":{"type":"NodePort"}}'
service/istio-ingressgateway patched
[root@k8s-master-node1 ~]#
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