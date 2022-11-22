# 在k8s上安装Harbor

### 先前条件
《kubernetes(k8s) 存储动态挂载》
《在k8s（kubernetes）上安装 ingress V1.1.3》 
参考我之前的文档进行部署
`https://www.oiox.cn/index.php/archives/32/`
`https://www.oiox.cn/index.php/archives/142/`

### 我用到的批量将dockerhub导入阿里云
```shell
#!/bin/bash

for((i=0;i<n;i++)); do
    echo "${i}"
done

export docker_images="goharbor/harbor-db:v2.6.2 goharbor/harbor-jobservice:v2.6.2 goharbor/harbor-portal:v2.6.2 goharbor/harbor-registryctl:v2.6.2 goharbor/notary-server-photon:v2.6.2 goharbor/notary-signer-photon:v2.6.2 goharbor/redis-photon:v2.6.2 goharbor/registry-photon:v2.6.2 goharbor/trivy-adapter-photon:v2.6.2"


export aliyun_image="registry.cn-hangzhou.aliyuncs.com/chenby/"


for images in $docker_images;do
    export end_image=`echo "$images" | awk -F "/" '{print $NF}'`
    docker pull "$images"
    docker tag "$images" "$aliyun_image""$end_image"
    docker push "$aliyun_image""$end_image"
    docker rmi "$images"
    docker rmi "$aliyun_image""$end_image"
done
```

### 安装helm工具
```shell
# 安装helm工具
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### 添加Harbor 官方Helm Chart仓库
```shell
# 添加Harbor 官方Helm Chart仓库
root@cby:~# helm repo add harbor  https://helm.goharbor.io
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
"harbor" has been added to your repositories
```

### 查看源列表
```shell
# 查看源列表
root@cby:~# helm repo list
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
NAME    URL                     
devtron https://helm.devtron.ai 
harbor  https://helm.goharbor.io
root@cby:~# 
```

### 列出最新版本的包 
```shell
# 列出最新版本的包 
root@cby:~# helm search repo harbor -l |  grep harbor/harbor  | head  -4
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
harbor/harbor   1.10.2          2.6.2           An open source trusted cloud native registry th...
harbor/harbor   1.10.1          2.6.1           An open source trusted cloud native registry th...
harbor/harbor   1.10.0          2.6.0           An open source trusted cloud native registry th...
harbor/harbor   1.9.4           2.5.4           An open source trusted cloud native registry th...
root@cby:~# 
```

### 下载Chart包到本地
```shell
# 下载Chart包到本地
root@cby:~# helm pull harbor/harbor --version 1.10.2
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
root@cby:~# 
root@cby:~# ls harbor-1.10.2.tgz 
harbor-1.10.2.tgz
root@cby:~# 
root@cby:~# tar zxvf harbor-1.10.2.tgz

root@cby:~# cd harbor/
root@cby:~/harbor# ll
total 276
drwxr-xr-x  5 root root   4096 Nov 22 10:35 ./
drwx------ 12 root root   4096 Nov 22 10:35 ../
drwxr-xr-x  2 root root   4096 Nov 22 10:35 cert/
-rw-r--r--  1 root root    567 Nov 10 09:08 Chart.yaml
drwxr-xr-x  2 root root   4096 Nov 22 10:35 conf/
-rw-r--r--  1 root root     57 Nov 10 09:08 .helmignore
-rw-r--r--  1 root root  11357 Nov 10 09:08 LICENSE
-rw-r--r--  1 root root 202142 Nov 10 09:08 README.md
drwxr-xr-x 16 root root   4096 Nov 22 10:35 templates/
-rw-r--r--  1 root root  33779 Nov 10 09:08 values.yaml
root@cby:~/harbor# 
```

### 修改values.yaml配置
```shell
# 修改values.yaml配置
root@cby:~/harbor# sed -i "s#harbor.domain#oiox.cn#g" values.yaml

# 设置为我的阿里云仓库
root@cby:~/harbor# sed -i "s#repository: goharbor#repository: registry.cn-hangzhou.aliyuncs.com/chenby#g" values.yaml

# 修改字段 externalURL  
# 注意 30785 是我的ingress端口，各位的端口应该和我的不一样
root@cby:~/harbor# vim values.yaml
externalURL: https://core.oiox.cn:30785

# debug看看配置与自己的环境是否匹配，是否需要修改
root@cby:~/harbor# helm install harbor ./ --dry-run | grep oiox.cn
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
  EXT_ENDPOINT: "https://core.oiox.cn:30785"
    - core.oiox.cn
    host: core.oiox.cn
    - notary.oiox.cn
    host: notary.oiox.cn
Then you should be able to visit the Harbor portal at https://core.oiox.cn:30785
root@cby:~/harbor# 
```

### 安装
```shell
# 创建命名空间
root@cby:~/harbor# kubectl create namespace harbor
namespace/harbor created
root@cby:~/harbor# 

# 进行安装
root@cby:~/harbor# helm install  harbor . -n harbor
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
NAME: harbor
LAST DEPLOYED: Tue Nov 22 10:56:50 2022
NAMESPACE: harbor
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Please wait for several minutes for Harbor deployment to complete.
Then you should be able to visit the Harbor portal at https://core.oiox.cn
For more details, please visit https://github.com/goharbor/harbor
root@cby:~/harbor# 

```


### 编辑ingress配置
```shell
root@cby:~# kubectl edit ingress -n harbor harbor-ingress
root@cby:~# kubectl edit ingress -n harbor harbor-ingress-notary

# 添加字段  ingressClassName: nginx
spec:
  ingressClassName: nginx
  rules:
  - host: core.oiox.cn
    http:

# 查看
root@cby:~# kubectl get ingress -n harbor harbor-ingress -o yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    ingress.kubernetes.io/proxy-body-size: "0"
    ingress.kubernetes.io/ssl-redirect: "true"
    meta.helm.sh/release-name: harbor
    meta.helm.sh/release-namespace: harbor
    nginx.ingress.kubernetes.io/proxy-body-size: "0"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  creationTimestamp: "2022-11-22T15:21:35Z"
  generation: 3
  labels:
    app: harbor
    app.kubernetes.io/managed-by: Helm
    chart: harbor
    heritage: Helm
    release: harbor
  name: harbor-ingress
  namespace: harbor
  resourceVersion: "2070090"
  uid: def0b549-3a00-49a4-8ece-b5ce18205427
spec:
  ingressClassName: nginx
  rules:
  - host: core.oiox.cn
    http:
      paths:
      - backend:
          service:
            name: harbor-core
            port:
              number: 80
        path: /api/
        pathType: Prefix
      - backend:
          service:
            name: harbor-core
            port:
              number: 80
        path: /service/
        pathType: Prefix
      - backend:
          service:
            name: harbor-core
            port:
              number: 80
        path: /v2/
        pathType: Prefix
      - backend:
          service:
            name: harbor-core
            port:
              number: 80
        path: /chartrepo/
        pathType: Prefix
      - backend:
          service:
            name: harbor-core
            port:
              number: 80
        path: /c/
        pathType: Prefix
      - backend:
          service:
            name: harbor-portal
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - core.oiox.cn
    secretName: harbor-ingress
status:
  loadBalancer:
    ingress:
    - ip: 192.168.8.65
root@cby:~# 


root@cby:~# kubectl get ingress -n harbor 
NAME                    CLASS   HOSTS            ADDRESS        PORTS     AGE
harbor-ingress          nginx   core.oiox.cn     192.168.8.65   80, 443   9m8s
harbor-ingress-notary   nginx   notary.oiox.cn   192.168.8.65   80, 443   9m8s
root@cby:~# 

```


### 访问测试
```shell
# 查看管理员密码
root@cby:~# kubectl get secret -n harbor harbor-core -o jsonpath='{.data.HARBOR_ADMIN_PASSWORD}'|base64 --decode
Harbor12345

# 写入本地hosts配置
root@cby:~# echo "192.168.8.65 core.oiox.cn" >> /etc/hosts


root@cby:~# sudo mkdir -p /etc/docker
root@cby:~# sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ],
  "insecure-registries": [
    "hb.oiox.cn",
    "core.oiox.cn:30785"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
root@cby:~# sudo systemctl daemon-reload
root@cby:~# sudo systemctl restart docker

root@cby:~# docker login -uadmin -pHarbor12345 core.oiox.cn:30785
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
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
