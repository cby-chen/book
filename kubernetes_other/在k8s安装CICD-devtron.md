
# 在k8s安装CICD-devtron

### 先前条件
《kubernetes(k8s) 存储动态挂载》
参考我之前的文档进行部署
`https://www.oiox.cn/index.php/archives/32/`

### 安装helm工具
```shell
root@cby:~# curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
root@cby:~# chmod 700 get_helm.sh
root@cby:~# ./get_helm.sh
Downloading https://get.helm.sh/helm-v3.10.2-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
root@cby:~# 
```

### 使用 helm 安装
```shell
root@cby:~# helm repo add devtron https://helm.devtron.ai
"devtron" has been added to your repositories
root@cby:~# 
root@cby:~# 
root@cby:~# 
root@cby:~# helm install devtron devtron/devtron-operator --create-namespace --namespace devtroncd --set installer.modules={cicd}

NAME: devtron
LAST DEPLOYED: Fri Nov 18 05:22:13 2022
NAMESPACE: devtroncd
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Run the following command to get the password for the default admin user:

   kubectl -n devtroncd get secret devtron-secret -o jsonpath='{.data.ADMIN_PASSWORD}' | base64 -d
2. Run the following command to get the dashboard URL for the service type:
   LoadBalancer
   
   kubectl get svc -n devtroncd devtron-service -o jsonpath='{.status.loadBalancer.ingress}'
3. To track the progress of Devtron microservices installation, run the following command:

   kubectl -n devtroncd get installers installer-devtron -o jsonpath='{.status.sync.status}'
root@cby:~# 

```

### 查看验证
```shell
root@cby:~# kubectl get pod -n devtroncd
NAME                                     READY   STATUS      RESTARTS        AGE
app-sync-cronjob-27815700-lz565          0/1     Completed   0               2d5h
app-sync-cronjob-27817140-6wsj6          0/1     Completed   0               29h
app-sync-cronjob-27818580-kzjdb          0/1     Completed   0               5h33m
argo-rollouts-68dc6f5b75-949x9           1/1     Running     2 (152m ago)    4d10h
argocd-application-controller-0          1/1     Running     2 (152m ago)    4d9h
argocd-dex-server-54c8d7cbdf-nfjj2       1/1     Running     2 (153m ago)    4d10h
argocd-redis-7967b6b9f7-6c69j            1/1     Running     2 (152m ago)    4d9h
argocd-repo-server-6f9d65d87f-9p9p8      1/1     Running     2 (152m ago)    4d9h
argocd-server-7cf98cdffb-4qxgm           1/1     Running     2 (152m ago)    4d9h
clair-8cd58cdd9-nhglm                    1/1     Running     46 (152m ago)   4d9h
dashboard-777c9bb5f9-zz4b5               1/1     Running     2 (152m ago)    4d10h
devtron-d74cf8958-2x7sb                  1/1     Running     4 (151m ago)    4d8h
devtron-grafana-6657cbc8f9-9j7fp         2/2     Running     2 (153m ago)    4d8h
devtron-grafana-test                     0/1     Completed   6               4d8h
devtron-housekeeping-qp59k               0/1     Completed   0               4d10h
devtron-nats-0                           3/3     Running     6 (152m ago)    4d10h
devtron-nats-test-request-reply          0/1     Completed   0               4d10h
git-sensor-0                             1/1     Running     6 (152m ago)    4d10h
grafana-org-job-jgzjp                    0/1     Completed   0               4d8h
image-scanner-8679b48b66-t7bd2           1/1     Running     8 (151m ago)    4d9h
inception-846694f944-5hjtq               1/1     Running     2 (152m ago)    4d10h
kubelink-67985f58d5-xmds2                1/1     Running     2 (152m ago)    4d10h
kubewatch-655f8669dd-xrx5q               1/1     Running     8 (152m ago)    4d10h
lens-6c86975478-vwpq2                    1/1     Running     9 (151m ago)    4d10h
notifier-5b4b48b677-dkcls                1/1     Running     1 (152m ago)    4d8h
postgresql-migrate-casbin-2lz42          0/1     Completed   0               4d10h
postgresql-migrate-casbin-bnzdb-954p6    0/1     Completed   0               4d8h
postgresql-migrate-devtron-t2w25         0/1     Completed   0               4d10h
postgresql-migrate-devtron-vlym3-jnvmf   0/1     Completed   0               4d8h
postgresql-migrate-gitsensor-sxpcr       0/1     Completed   0               4d10h
postgresql-migrate-lens-tmvt5            0/1     Completed   0               4d10h
postgresql-postgresql-0                  2/2     Running     4 (152m ago)    4d10h
root@cby:~# 
root@cby:~# kubectl get svc -n devtroncd
NAME                             TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                 AGE
argo-rollouts-metrics            ClusterIP      10.98.113.34     <none>        8090/TCP                                                4d10h
argocd-application-controller    ClusterIP      10.107.155.128   <none>        8082/TCP                                                4d9h
argocd-dex-server                ClusterIP      10.97.14.200     <none>        5556/TCP,5557/TCP,5558/TCP                              4d10h
argocd-redis                     ClusterIP      10.102.166.243   <none>        6379/TCP                                                4d9h
argocd-repo-server               ClusterIP      10.111.245.9     <none>        8081/TCP                                                4d9h
argocd-server                    ClusterIP      10.106.6.25      <none>        80/TCP,443/TCP                                          4d9h
clair                            ClusterIP      10.109.97.107    <none>        6060/TCP,6061/TCP                                       4d9h
dashboard-service                ClusterIP      10.110.239.18    <none>        80/TCP                                                  4d10h
devtron-grafana                  ClusterIP      10.111.200.165   <none>        80/TCP                                                  4d8h
devtron-nats                     ClusterIP      None             <none>        4222/TCP,6222/TCP,8222/TCP,7777/TCP,7422/TCP,7522/TCP   4d10h
devtron-service                  LoadBalancer   10.100.28.2      <pending>     80:32489/TCP                                            4d10h
git-sensor-service               ClusterIP      10.99.53.176     <none>        80/TCP                                                  4d10h
image-scanner-service            ClusterIP      10.103.97.46     <none>        80/TCP                                                  4d9h
kubelink-service                 ClusterIP      10.97.172.63     <none>        50051/TCP                                               4d10h
lens-service                     ClusterIP      10.100.239.205   <none>        80/TCP                                                  4d10h
notifier-service                 ClusterIP      10.102.67.212    <none>        80/TCP                                                  4d8h
postgresql-postgresql            ClusterIP      10.104.194.12    <none>        5432/TCP                                                4d10h
postgresql-postgresql-headless   ClusterIP      None             <none>        5432/TCP                                                4d10h
postgresql-postgresql-metrics    ClusterIP      10.103.17.122    <none>        9187/TCP                                                4d10h
root@cby:~# 
```

### 访问测试
```shell
# 使用用户名：admin和下面提到的密码运行命令。
root@cby:~# kubectl -n devtroncd get secret devtron-secret -o jsonpath='{.data.ADMIN_PASSWORD}' | base64 -d
Qn7GuI26j4HcuVW2


# 访问地址
http://192.168.8.61:32489/

# 用户名：admin
# 密码：Qn7GuI26j4HcuVW2
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