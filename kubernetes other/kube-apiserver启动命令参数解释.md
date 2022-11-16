---
layout: post
cid: 98
title: kube-apiserver启动命令参数解释
slug: 98
date: 2022/03/07 13:20:00
updated: 2022/03/07 14:34:37
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


在apiserver启动时候会有很多参数来配置启动命令，有些时候不是很明白这些参数具体指的是什么意思。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5dec6da363b54737bf341d947489e7e8~tplv-k3u1fbpfcp-zoom-1.image)

  

我的kube-apiserver启动命令参数：

  

```
cat > /usr/lib/systemd/system/kube-apiserver.service << EOF


[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target


[Service]
ExecStart=/usr/local/bin/kube-apiserver \
      --v=2  \
      --logtostderr=true  \
      --allow-privileged=true  \
      --bind-address=0.0.0.0  \
      --secure-port=6443  \
      --insecure-port=0  \
      --advertise-address=192.168.1.30 \
      --service-cluster-ip-range=10.96.0.0/12  \
      --service-node-port-range=30000-32767  \
      --etcd-servers=https://192.168.1.30:2379,https://192.168.1.31:2379,https://192.168.1.32:2379 \
      --etcd-cafile=/etc/etcd/ssl/etcd-ca.pem  \
      --etcd-certfile=/etc/etcd/ssl/etcd.pem  \
      --etcd-keyfile=/etc/etcd/ssl/etcd-key.pem  \
      --client-ca-file=/etc/kubernetes/pki/ca.pem  \
      --tls-cert-file=/etc/kubernetes/pki/apiserver.pem  \
      --tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem  \
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem  \
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem  \
      --service-account-key-file=/etc/kubernetes/pki/sa.pub  \
      --service-account-signing-key-file=/etc/kubernetes/pki/sa.key  \
      --service-account-issuer=https://kubernetes.default.svc.cluster.local \
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname  \
      --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota  \
      --authorization-mode=Node,RBAC  \
      --enable-bootstrap-token-auth=true  \
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem  \
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem  \
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client-key.pem  \
      --requestheader-allowed-names=aggregator  \
      --requestheader-group-headers=X-Remote-Group  \
      --requestheader-extra-headers-prefix=X-Remote-Extra-  \
      --requestheader-username-headers=X-Remote-User
      # --token-auth-file=/etc/kubernetes/token.csv


Restart=on-failure
RestartSec=10s
LimitNOFILE=65535


[Install]
WantedBy=multi-user.target


EOF
```

  

解释

  

```
-v, --v int
日志级别详细程度的数字。

--logtostderr     默认值：true
在标准错误而不是文件中输出日志记录。
--bind-address string     默认值："0.0.0.0"
用来监听 --secure-port 端口的 IP 地址。集群的其余部分以及 CLI/web 客户端必须可以访问所关联的接口。如果为空白或未指定地址（0.0.0.0 或 ::），则将使用所有接口。

--secure-port int     默认值：6443
带身份验证和鉴权机制的 HTTPS 服务端口。不能用 0 关闭。

--advertise-address string
向集群成员通知 apiserver 消息的 IP 地址。这个地址必须能够被集群中其他成员访问。如果 IP 地址为空，将会使用 --bind-address， 如果未指定 --bind-address，将会使用主机的默认接口地址。

--service-cluster-ip-range string
CIDR 表示的 IP 范围用来为服务分配集群 IP。此地址不得与指定给节点或 Pod 的任何 IP 范围重叠。

--service-node-port-range <形式为 'N1-N2' 的字符串>     默认值：30000-32767
保留给具有 NodePort 可见性的服务的端口范围。例如："30000-32767"。范围的两端都包括在内。

--etcd-servers strings
要连接的 etcd 服务器列表（scheme://ip:port），以逗号分隔。

--etcd-cafile string
用于保护 etcd 通信的 SSL 证书颁发机构文件。

--etcd-certfile string
用于保护 etcd 通信的 SSL 证书文件。

--etcd-keyfile string
用于保护 etcd 通信的 SSL 密钥文件。

--client-ca-file string
如果已设置，则使用与客户端证书的 CommonName 对应的标识对任何出示由 client-ca 文件中的授权机构之一签名的客户端证书的请求进行身份验证。

--tls-cert-file string
包含用于 HTTPS 的默认 x509 证书的文件。（CA 证书（如果有）在服务器证书之后并置）。如果启用了 HTTPS 服务，并且未提供 --tls-cert-file 和 --tls-private-key-file， 为公共地址生成一个自签名证书和密钥，并将其保存到 --cert-dir 指定的目录中。

--tls-private-key-file string
包含匹配 --tls-cert-file 的 x509 证书私钥的文件。

--kubelet-client-certificate string
TLS 的客户端证书文件的路径。

--kubelet-client-key string
TLS 客户端密钥文件的路径。

--service-account-key-file strings
包含 PEM 编码的 x509 RSA 或 ECDSA 私钥或公钥的文件，用于验证 ServiceAccount 令牌。指定的文件可以包含多个键，并且可以使用不同的文件多次指定标志。如果未指定，则使用 --tls-private-key-file。提供 --service-account-signing-key 时必须指定。

--service-account-signing-key-file string
包含服务帐户令牌颁发者当前私钥的文件的路径。颁发者将使用此私钥签署所颁发的 ID 令牌。

--service-account-issuer strings
服务帐号令牌颁发者的标识符。颁发者将在已办法令牌的 "iss" 声明中检查此标识符。此值为字符串或 URI。如果根据 OpenID Discovery 1.0 规范检查此选项不是有效的 URI，则即使特性门控设置为 true， ServiceAccountIssuerDiscovery 功能也将保持禁用状态。强烈建议该值符合 OpenID 规范：https://openid.net/specs/openid-connect-discovery-1_0.html。实践中，这意味着 service-account-issuer 取值必须是 HTTPS URL。还强烈建议此 URL 能够在 {service-account-issuer}/.well-known/openid-configuration 处提供 OpenID 发现文档。当此值被多次指定时，第一次的值用于生成令牌，所有的值用于确定接受哪些发行人。

--kubelet-preferred-address-types strings     默认值：Hostname,InternalDNS,InternalIP,ExternalDNS,ExternalIP
用于 kubelet 连接的首选 NodeAddressTypes 列表。

--enable-admission-plugins stringSlice
除了默认启用的插件（NamespaceLifecycle、LimitRanger、ServiceAccount、TaintNodesByCondition、PodSecurity、Priority、DefaultTolerationSeconds、DefaultStorageClass、StorageObjectInUseProtection、PersistentVolumeClaimResize、RuntimeClass、CertificateApproval、CertificateSigning、CertificateSubjectRestriction、DefaultIngressClass、MutatingAdmissionWebhook、ValidatingAdmissionWebhook、ResourceQuota）之外要启用的插件
取值为逗号分隔的准入插件列表：AlwaysAdmit、AlwaysDeny、AlwaysPullImages、CertificateApproval、CertificateSigning、CertificateSubjectRestriction、DefaultIngressClass、DefaultStorageClass、DefaultTolerationSeconds、DenyServiceExternalIPs、EventRateLimit、ExtendedResourceToleration、ImagePolicyWebhook、LimitPodHardAntiAffinityTopology、LimitRanger、MutatingAdmissionWebhook、NamespaceAutoProvision、NamespaceExists、NamespaceLifecycle、NodeRestriction、OwnerReferencesPermissionEnforcement、PersistentVolumeClaimResize、PersistentVolumeLabel、PodNodeSelector、PodSecurity、PodSecurityPolicy、PodTolerationRestriction、Priority、ResourceQuota、RuntimeClass、SecurityContextDeny、ServiceAccount、StorageObjectInUseProtection、TaintNodesByCondition、ValidatingAdmissionWebhook
该标志中插件的顺序无关紧要。

--authorization-mode stringSlice     默认值："AlwaysAllow"
在安全端口上进行鉴权的插件的顺序列表。逗号分隔的列表：AlwaysAllow、AlwaysDeny、ABAC、Webhook、RBAC、Node。

--enable-bootstrap-token-auth
启用以允许将 "kube-system" 名字空间中类型为 "bootstrap.kubernetes.io/token" 的 Secret 用于 TLS 引导身份验证。

--requestheader-client-ca-file string
在信任请求头中以 --requestheader-username-headers 指示的用户名之前， 用于验证接入请求中客户端证书的根证书包。警告：一般不要假定传入请求已被授权。

--proxy-client-cert-file string
当必须调用外部程序以处理请求时，用于证明聚合器或者 kube-apiserver 的身份的客户端证书。包括代理转发到用户 api-server 的请求和调用 Webhook 准入控制插件的请求。Kubernetes 期望此证书包含来自于 --requestheader-client-ca-file 标志中所给 CA 的签名。该 CA 在 kube-system 命名空间的 "extension-apiserver-authentication" ConfigMap 中公开。从 kube-aggregator 收到调用的组件应该使用该 CA 进行各自的双向 TLS 验证。

--proxy-client-key-file string
当必须调用外部程序来处理请求时，用来证明聚合器或者 kube-apiserver 的身份的客户端私钥。这包括代理转发给用户 api-server 的请求和调用 Webhook 准入控制插件的请求。

--requestheader-allowed-names strings
此值为客户端证书通用名称（Common Name）的列表；表中所列的表项可以用来提供用户名， 方式是使用 --requestheader-username-headers 所指定的头部。如果为空，能够通过 --requestheader-client-ca-file 中机构 认证的客户端证书都是被允许的。

--requestheader-group-headers strings
用于查验用户组的请求头部列表。建议使用 X-Remote-Group。

--requestheader-extra-headers-prefix strings
用于查验请求头部的前缀列表。建议使用 X-Remote-Extra-。

--requestheader-username-headers strings
用于查验用户名的请求头头列表。建议使用 X-Remote-User。

--token-auth-file string
如果设置该值，这个文件将被用于通过令牌认证来保护 API 服务的安全端口。

官方文档：
https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/
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
