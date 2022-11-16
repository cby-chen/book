---
layout: post
cid: 75
title: 为kubernetes（k8s）单独配置kubectl工具
slug: 75
date: 2022/01/06 15:31:21
updated: 2022/01/06 15:31:21
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


# 介绍

Kubernetes API 是一个 HTTP REST API。这个 API 是真正的 Kubernetes 用户界面，通过它可以完全控制它。这意味着每个 Kubernetes 操作都作为 API 端点公开，并且可以通过对该端点的 HTTP 请求进行。因此，kubectl 的主要目的是向 Kubernetes API 发出 HTTP 请求：![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3dd62e455489487985d282b6867f10c6~tplv-k3u1fbpfcp-zoom-1.image)

# 配置apt软件源

```
root@hello:~# apt-get update && apt-get install -y apt-transport-https
Hit:1 http://192.168.1.104:81/ubuntu focal InRelease
Hit:2 http://192.168.1.104:81/ubuntu focal-security InRelease
Hit:3 http://192.168.1.104:81/ubuntu focal-updates InRelease
Hit:4 http://192.168.1.104:81/ubuntu focal-proposed InRelease
Hit:5 https://mirrors.aliyun.com/docker-ce/linux/ubuntu focal InRelease
Reading package lists... Done        
Reading package lists... Done
Building dependency tree       
Reading state information... Done
apt-transport-https is already the newest version (2.0.6).
0 upgraded, 0 newly installed, 0 to remove and 54 not upgraded.
root@hello:~# 
root@hello:~# 
root@hello:~# curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2537  100  2537    0     0  26989      0 --:--:-- --:--:-- --:--:-- 26989
OK
root@hello:~# 
root@hello:~# cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
> deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
> EOF
root@hello:~# apt-get update
Hit:1 http://192.168.1.104:81/ubuntu focal InRelease
Hit:2 http://192.168.1.104:81/ubuntu focal-security InRelease
Hit:3 http://192.168.1.104:81/ubuntu focal-updates InRelease
Hit:4 http://192.168.1.104:81/ubuntu focal-proposed InRelease
Hit:5 https://mirrors.aliyun.com/docker-ce/linux/ubuntu focal InRelease
Get:6 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial InRelease [9,383 B]
Ign:7 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
Get:7 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages [52.6 kB]
Fetched 62.0 kB in 1s (59.9 kB/s)   
Reading package lists... Done
root@hello:~#
```

  

# 使用apt安装kubectl工具

```
root@hello:~# apt-get install -y kubectl
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  kubectl
0 upgraded, 1 newly installed, 0 to remove and 54 not upgraded.
Need to get 8,928 kB of archives.
After this operation, 46.6 MB of additional disk space will be used.
Get:1 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 kubectl amd64 1.23.1-00 [8,928 kB]
Fetched 8,928 kB in 2s (5,599 kB/s)   
Selecting previously unselected package kubectl.
(Reading database ... 129153 files and directories currently installed.)
Preparing to unpack .../kubectl_1.23.1-00_amd64.deb ...
Unpacking kubectl (1.23.1-00) ...
Setting up kubectl (1.23.1-00) ...
root@hello:~# 
root@hello:~# 
root@hello:~# 
root@hello:~# mkdir /root/.kube/
```

  

# 配置kubectl配置文件


```
root@hello:~# vim /root/.kube/config 
root@hello:~# 
root@hello:~# 
root@hello:~# cat /root/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUR1RENDQXFDZ0F3SUJBZ0lVSlF3R05rQS9BaGxLYVpEcS9oaVpQNStteVJ3d0RRWUpLb1pJaHZjTkFRRUwKQlFBd1lURUxNQWtHQTFVRUJoTUNRMDR4RVRBUEJnTlZCQWdUQ0VoaGJtZGFhRzkxTVFzd0NRWURWUVFIRXdKWQpVekVNTUFvR0ExVUVDaE1EYXpoek1ROHdEUVlEVlFRTEV3WlRlWE4wWlcweEV6QVJCZ05WQkFNVENtdDFZbVZ5CmJtVjBaWE13SUJjTk1qRXhNakF6TURJME1EQXdXaGdQTWpFeU1URXhNRGt3TWpRd01EQmFNR0V4Q3pBSkJnTlYKQkFZVEFrTk9NUkV3RHdZRFZRUUlFd2hJWVc1bldtaHZkVEVMTUFrR0ExVUVCeE1DV0ZNeEREQUtCZ05WQkFvVApBMnM0Y3pFUE1BMEdBMVVFQ3hNR1UzbHpkR1Z0TVJNd0VRWURWUVFERXdwcmRXSmxjbTVsZEdWek1JSUJJakFOCkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXRvcisyNkVLY2VRZGE5eDZodXRoL0h1S21ZRWIKVWhadWJSWVR0VW85WTBpaFc2ME1GK1RBTndNSURFdHo0MGhkSXhrTmtJaDhITEdUcjlwek9hWGNzSVg2NzJsZwpheTdQVGlVZ3I2cVRYcmEzcnpxMjJrdVJtU05yY29ZVmpRbDVXa2ZITWR6cS9GZFpRVDVsRytZZWlLS1Q0c2tzCmJUcmFwSGFUc0VYY0lMb2VBREdCUVJrSXhvTmswWGo3RzNXbEt4enFRRXJ3cVIvbkE3b0U2MStYbHJZaTJTYUkKVFFoaUpMV0lYRTluUkRRNG9hOVNDSXhKUFp5Ukl5UTJFSVc2TG1DRDVtazNtZ2lPNFlVK3ZiMXg3amppS3ZKcQo0MExaaklFQllxY1R4RVN3K2J6cnYrQ1JaMm9UUlRaVGxveGVtYzliOWdhM2pwSjZBbWdvYjRmQkVRSURBUUFCCm8yWXdaREFPQmdOVkhROEJBZjhFQkFNQ0FRWXdFZ1lEVlIwVEFRSC9CQWd3QmdFQi93SUJBakFkQmdOVkhRNEUKRmdRVThBWGxiWis4cnRySmxxYzhpUUxIVjVHUis3TXdId1lEVlIwakJCZ3dGb0FVOEFYbGJaKzhydHJKbHFjOAppUUxIVjVHUis3TXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBRkxFMGFGclQzTnptcWRRdCtPN1c1OW04WnJVCnNtbFFzNGt2cFhET0FwdUxaNzROVUY0K3F1aVVRaFB4VEZFVnA2azBqVjlwWVVzbURMKzZmR1BaQldwdVpscisKSjRYZlcwaENITjlnZ05JelcxWUNZNEVxWGp5ZmY1dTZZQ1MyNmU2ZVB3dFA2RGhObE0xNzRNOXpKbnhGbllZdApZYmFjdDhjOTlwRDZvYlI3VGhnd3BFdE9YbW11ajM5OU5ycjR5cXBaQk95dGxQR291N2JzcFl2dkFhMnJ3QnNJCkh4NTNUT1paMXFNRjBYemNWbVk2eHQ1MklkVUtSdDV1QWsxRGRsQ2RkMHplL2RsZmN4MVBxbnV6dDNndldpL3MKRERCYXg0SnB0cXloMjgwZkVlU1pEd0hpYnY4V3AwRi8ranI1N2Q1K0p0cXgrOTlBSXZiUlM5U1JLMmc9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://192.168.1.11:6443
  name: cluster1
contexts:
- context:
    cluster: cluster1
    user: admin
  name: context-cluster1
current-context: context-cluster1
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUQxekNDQXIrZ0F3SUJBZ0lVVFprVnpuSFYxMStjdVRWSnNqWHpUVDVOZHQ4d0RRWUpLb1pJaHZjTkFRRUwKQlFBd1lURUxNQWtHQTFVRUJoTUNRMDR4RVRBUEJnTlZCQWdUQ0VoaGJtZGFhRzkxTVFzd0NRWURWUVFIRXdKWQpVekVNTUFvR0ExVUVDaE1EYXpoek1ROHdEUVlEVlFRTEV3WlRlWE4wWlcweEV6QVJCZ05WQkFNVENtdDFZbVZ5CmJtVjBaWE13SUJjTk1qRXhNakF6TURJME1EQXdXaGdQTWpBM01URXhNakV3TWpRd01EQmFNR2N4Q3pBSkJnTlYKQkFZVEFrTk9NUkV3RHdZRFZRUUlFd2hJWVc1bldtaHZkVEVMTUFrR0ExVUVCeE1DV0ZNeEZ6QVZCZ05WQkFvVApEbk41YzNSbGJUcHRZWE4wWlhKek1ROHdEUVlEVlFRTEV3WlRlWE4wWlcweERqQU1CZ05WQkFNVEJXRmtiV2x1Ck1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBbXNKUHBvdEcyNVE4bExyNC9NK3MKdVYzdWduQU14ZWRKYldFQmcxem81UGtyVW8wTUpDUEkyMTgrby9yTHh2eWJ1SlJKRm5qZlJMWlBZNmYrTGZTKwpJQmppbHJQN3J2OHdLMTh1V0EvNVdoWWNQeUZZYTZKeTVRM1RFdkZBYkdLVU5FUjBiWUhNOXdmTGJhVWNmdGkyCnI5dEd5TFVPYzBpemJ5QkFPZFU3Wkx0Z2d2OVdZb213aThLZG84bXVTTjdqSGlpd1BXTmIvQlBDUzE1WElvTXcKZDRzUW15MFFLVENTOHRuR2FzeFlPQ1pqMkhZMTV6dTdmbFJBeWZZcDNCM1pLZVZzQXdvUkhLQmVCa0NlMklwMQpYVnI3aEtkaEtkRWlaNGROcFVjd1V1U2xBRml3K1lPeUREbDZLdmVsSGVHVCs0N3E5SStjbXc0Rm1Ra1hhNGZFCkhRSURBUUFCbzM4d2ZUQU9CZ05WSFE4QkFmOEVCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUcKQ0NzR0FRVUZCd01DTUF3R0ExVWRFd0VCL3dRQ01BQXdIUVlEVlIwT0JCWUVGQTRwcGFzNUZzTGJuNVJIVGxwTQo5T1FlQTlZaE1COEdBMVVkSXdRWU1CYUFGUEFGNVcyZnZLN2F5WmFuUElrQ3gxZVJrZnV6TUEwR0NTcUdTSWIzCkRRRUJDd1VBQTRJQkFRQnRsQ21xN1pZQ2lRVVFHSGdSc2lDY2Q0UmVEZy8rcWVmbkJRT3h4SWN4TzU3UE1uNkwKWjVJNnJwUE9TSi9XaFlwUkNGUGVPTzZTUE5GS1RrUzNIQzlocytmY3dCaFBtV0gzNmJXQytDOXkrU1dXcXpkWQpWRzhpbDF1YW8wK04wWTZVdDdnZ0h5V1RscnByem43MmsrT1dKUlA4VWM5SVpBaWx5TUlHTmdZZENoMDVnbVBlCkd3Z0VyMHBLU3A5UE9SUDFZTGF5VVFsdUdCZkhtWERHM21kd3RYVmFFRmJNbEJsRU1CdCsvMW8xMWNVSFdNVWgKYXVBVWNPYy9RTGUvZUVZcFZTT25NRWpmalJZd1BwY1RybnNsYjNjblFnU2VrdE51QXJWZ1Y5UXg3WkhvZ1o3NApJZTJzSU9tRDRBUGEzNWJWb0c1SkMwYkc2NHVVM2hKOEIzNGgKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBbXNKUHBvdEcyNVE4bExyNC9NK3N1VjN1Z25BTXhlZEpiV0VCZzF6bzVQa3JVbzBNCkpDUEkyMTgrby9yTHh2eWJ1SlJKRm5qZlJMWlBZNmYrTGZTK0lCamlsclA3cnY4d0sxOHVXQS81V2hZY1B5RlkKYTZKeTVRM1RFdkZBYkdLVU5FUjBiWUhNOXdmTGJhVWNmdGkycjl0R3lMVU9jMGl6YnlCQU9kVTdaTHRnZ3Y5VwpZb213aThLZG84bXVTTjdqSGlpd1BXTmIvQlBDUzE1WElvTXdkNHNRbXkwUUtUQ1M4dG5HYXN4WU9DWmoySFkxCjV6dTdmbFJBeWZZcDNCM1pLZVZzQXdvUkhLQmVCa0NlMklwMVhWcjdoS2RoS2RFaVo0ZE5wVWN3VXVTbEFGaXcKK1lPeUREbDZLdmVsSGVHVCs0N3E5SStjbXc0Rm1Ra1hhNGZFSFFJREFRQUJBb0lCQUVEa0tUSFVSS25keG1rMgozU0JrbERCRnlyUzI5eVFrancxbUY1UlZhUEpaNkdoODdCSmJUdVZ0VW42L3NxS0ZXV1pVQnpGOURXRnFjRytCCkNYdUxuQTBwWWhsKzdwRzZQeUJ3a0tZc1RJb1JxMVp0VFA0VTU4aFR1Nlc5c3gyL1dCVnlmcjlNSmYyUEx5V1MKamhoQ0ZwZzJnYisyNjVBN2M4R3M3RUZUdjh2RWZ3S3RYVm50SDVKOVA1R3RWTnBEcTNncnM0UWNSajVzNWI0MwppVFZBTGNabkRHTktrS2JwYzdmYWVxdUc0R0VOWUZQcUJ1RnNvM3BUTzEwWlIrbmQxaFFiWm9xdW5JYlRxUDNGClV3NzJ5MTNLSkdjNkRRbnhpUWROeTFIUWlYbFVtTzk1dER4UHhzdFBmM3BpSVhkU1RpRTUzMDNkVEZpMWtFaG4KN2dWcDhxRUNnWUVBeSsvMEdrZVgzTU4ySG1qNU9iWGlnUzM0Sk90elBpUGdmMENMdzQ4K20wV2VNdkVhZmhwbApNRnl2T2V0bWpQWlpIaWNOaTErMkladDBUWnQ4Qmo4QXc1LzVnd3hXRS9pVm5uYVkyd1NaVlcwYlh4QlJqTkNLCnhYTXJJWlRCK2dwcG9tUGpKRlBFMGNnOWgzWUJFMkdBRUc3RjdnNi9yeGNkOVUrV2VMbE9OczhDZ1lFQXdrUmcKa1Y3ajRIU2llMTFHUzgvTGc1YXd4VTNFTXNMdVNSL2RVRTZ3c1hRMWxBS0x0dTlURXZyTFdydHpzYkhwU0JEYgpIUXVOQWhXandQS0RvY0lzcVNpVFdPdkdMa2NrZGphY2dPL3lYcmpTMng1cmpUWjc2NWRjaFRQUGFRVEE1VFdwCmRjbEI4S0g2Z1k2M1FwTWg1RURFZ3VaS2dRWFNCU0IwdUtnRDBWTUNnWUFkc3V3Umg2dU44c2tZMUtDMnpzNFYKa2VRNVBEQ2tOQVZWZ3NqWHlkeU1NQzlCcStyM3dsQktJclZCOGc0VktTc0JRUjZ2MVZob3ZJTExhb0U5UjUrTQozWmN3aG5OaXBTamswdENmMUtPZjFTdlBSRWtjQUtLMDduaXhnMEJjY1hmQXRsczF4eDA2ajdhbUs0RXNtVjVWCkJreTh4bGtUM29IMlg0akNPL292OFFLQmdIZlhVSzg5RjF5Rzl4a2RVRmxDUmV6V1VCUlhSZnAra0JyaUlsZ0IKUXpVbFdFd0hTZ00vSGtOdUhYYktmck9XNmk4LzNydkxQV0NVMHVFYmVpS1dzNUJpN0lzRlg4dDZyYjZUTC9iRwpqd0RxQ1lHTkFaSXFrMFdocVR5dTJudVJxQ0Y5K2gwa1c1NURmbExnSktOWU9xY2hZVmpURWhFSDh5aWdmZ0RQCi9STHJBb0dBUStWMnlJa2VRYm95b2VKSzJGZnhvUXVaNmY3NERraGFhQkV3UU4rM1NIdmIvWlE0YnpDbktvaUYKODA0bWZuN1VZN0ptN0hOTVYzRHpGRzNxYkhiWDZnSEYyWlFiWm4rb0Ywck8vbWxITnE5QzlJWXpXWS9sZERYVApwS3hMaWsxeEt1VURGUFp2Y01XTmY5Vk82NW5HZXo3R2I5UE9UMTdTQ3FmWGZBRHN2V1U9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
root@hello:~#
```

  

\*注意：配置文件中的server需要修改，并且该配置文件在原有的集权管理节点上。

  

# 配置自动补全，并测试kubectl

```
root@hello:~# apt install -y bash-completion
Reading package lists... Done
Building dependency tree       
Reading state information... Done
bash-completion is already the newest version (1:2.10-1ubuntu1).
bash-completion set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 54 not upgraded.
root@hello:~# 
root@hello:~# 
root@hello:~# 
root@hello:~# echo "source <(kubectl completion bash)" >> ~/.bashrc
root@hello:~# 
root@hello:~# 
root@hello:~# source <(kubectl completion bash)
root@hello:~# 
root@hello:~# 
root@hello:~# kubectl get deployments.apps 
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
cby                      1/1     1            1           3h53m
hello-server             2/2     2            2           30d
ingress-demo-app         2/2     2            2           30d
nfs-client-provisioner   1/1     1            1           34d
nginx-demo               2/2     2            2           30d
root@hello:~#
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