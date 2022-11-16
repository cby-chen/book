---
layout: post
cid: 57
title: kubernetes核心实战（七）--- job、CronJob、Secret
slug: 57
date: 2021/12/30 17:12:00
updated: 2022/05/19 15:55:04
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


#### 10、job任务

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3388e96ac1e48e08eb909949fda06ad~tplv-k3u1fbpfcp-zoom-1.image)

  

###### 使用perl，做pi的圆周率计算

```
[root@k8s-master-node1 ~/yaml/test]# vim job.yaml 
[root@k8s-master-node1 ~/yaml/test]# cat job.yaml 
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl  apply  -f job.yaml 
job.batch/pi created
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看任务

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  get pod
NAME                                     READY   STATUS      RESTARTS   AGE
ingress-demo-app-694bf5d965-8rh7f        1/1     Running     0          134m
ingress-demo-app-694bf5d965-swkpb        1/1     Running     0          134m
nfs-client-provisioner-dc5789f74-5bznq   1/1     Running     0          118m
pi--1-k5cbq                              0/1     Completed   0          115s
redis-app-86g4q                          1/1     Running     0          4m14s
redis-app-rt92n                          1/1     Running     0          4m14s
redis-app-vkzft                          1/1     Running     0          4m14s
web-0                                    1/1     Running     0          67m
web-1                                    1/1     Running     0          67m
web-2                                    1/1     Running     0          67m
[root@k8s-master-node1 ~/yaml/test]# kubectl  get job
NAME   COMPLETIONS   DURATION   AGE
pi     1/1           84s        2m
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看计算结果

```
[root@k8s-master-node1 ~/yaml/test]# kubectl logs pi--1-k5cbq 3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901[root@k8s-master-node1 ~/yaml/test]#
```

  

#### 11、CronJob

  

CronJobs 对于创建周期性的、反复重复的任务很有用，例如执行数据备份或者发送邮件。CronJobs 也可以用来计划在指定时间来执行的独立任务，例如计划当集群看起来很空闲时 执行某个 Job。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af94ef2c0fb3472a9551f2133a63cd98~tplv-k3u1fbpfcp-zoom-1.image)

###### 创建任务

```
[root@k8s-master-node1 ~/yaml/test]# vim cronjob.yaml
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# cat cronjob.yaml 
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]# kubectl apply -f cronjob.yaml 
Warning: batch/v1beta1 CronJob is deprecated in v1.21+, unavailable in v1.25+; use batch/v1 CronJob
cronjob.batch/hello created
[root@k8s-master-node1 ~/yaml/test]# kubectl  get cronjobs.batch 
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        <none>          21s
[root@k8s-master-node1 ~/yaml/test]# 
[root@k8s-master-node1 ~/yaml/test]#
```

  

###### 查看结果

```
[root@k8s-master-node1 ~/yaml/test]# kubectl  logs  hello-27285668--1-zqg92 
Wed Nov 17 09:08:18 UTC 2021
Hello from the Kubernetes cluster
[root@k8s-master-node1 ~/yaml/test]#
```

  

#### 12、Secret

Secret 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 SSH 密钥。将这些信息放在 secret 中比放在 Pod 的定义或者 容‍器镜像 中来说更加安全和灵活。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c2e0106b4354300a0affc8066ab0fcb~tplv-k3u1fbpfcp-zoom-1.image)

```
kubectl create secret docker-registry regcred \
  --docker-server=<你的镜像仓库服务器> \
  --docker-username=<你的用户名> \
  --docker-password=<你的密码> \
  --docker-email=<你的邮箱地址>
```

  

```
apiVersion: v1
kind: Pod
metadata:
  name: private-nginx
spec:
  containers:
  - name: private-nginx
    image: chenbuyun/my-app:v1.0
  imagePullSecrets:
  - name: regcred
```

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2489298766624ab1ac7836a6f49340f4~tplv-k3u1fbpfcp-zoom-1.image)  

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
> **文章主要发布于微信**