---
layout: post
cid: 76
title: kubectl管理多个集群配置
slug: 76
date: 2022/01/07 09:32:02
updated: 2022/01/07 09:32:18
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


 # **需求描述：**
在一台机器上通过kubectl管理多个Kubernetes集群。

  

操作过程：将各集群的kubectl config文件中的证书内容转换，通过命令创建config文件；通过上下文切换使用不同集群。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb87c3ee85654b439b1bbeca5fb8de01~tplv-k3u1fbpfcp-zoom-1.image)

```
root@hello:~/.kube# ll
total 44
drwxr-xr-x 3 root root 4096 Jan 6 16:23 ./
drwx------ 21 root root 4096 Jan 6 16:22 ../
drwxr-x--- 4 root root 4096 Jan 6 14:50 cache/
-rw-r--r-- 1 root root 6252 Jan 6 16:21 config1
-rw-r--r-- 1 root root 6254 Jan 6 16:22 config2
root@hello:~/.kube#
root@hello:~/.kube#
```

  

 # **准备配置文件**

```
root@hello:~/.kube# cat config1
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUR1RENDQXFDZ0F3SUJBZ0lVSlF3R05rQS9BaGxLYVpEcS9oaVpQNStteVJ3d0RRWUpLb1pJaHZjTkFRRUwKQlFBd1lURUxNQWtHQTFVRUJoTUNRMDR4RVRBUEJnTlZCQWdUQ0VoaGJtZGFhRzkxTVFzd0NRWURWUVFIRXdKWQpVekVNTUFvR0ExVUVDaE1EYXpoek1ROHdEUVlEVlFRTEV3WlRlWE4wWlcweEV6QVJCZ05WQkFNVENtdDFZbVZ5CmJtVjBaWE13SUJjTk1qRXhNakF6TURJME1EQXdXaGdQTWpFeU1URXhNRGt3TWpRd01EQmFNR0V4Q3pBSkJnTlYKQkFZVEFrTk9NUkV3RHdZRFZRUUlFd2hJWVc1bldtaHZkVEVMTUFrR0ExVUVCeE1DV0ZNeEREQUtCZ05WQkFvVApBMnM0Y3pFUE1BMEdBMVVFQ3hNR1UzbHpkR1Z0TVJNd0VRWURWUVFERXdwcmRXSmxjbTVsZEdWek1JSUJJakFOCkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXRvcisyNkVLY2VRZGE5eDZodXRoL0h1S21ZRWIKVWhadWJSWVR0VW85WTBpaFc2ME1GK1RBTndNSURFdHo0MGhkSXhrTmtJaDhITEdUcjlwek9hWGNzSVg2NzJsZwpheTdQVGlVZ3I2cVRYcmEzcnpxMjJrdVJtU05yY29ZVmpRbDVXa2ZITWR6cS9GZFpRVDVsRytZZWlLS1Q0c2tzCmJUcmFwSGFUc0VYY0lMb2VBREdCUVJrSXhvTmswWGo3RzNXbEt4enFRRXJ3cVIvbkE3b0U2MStYbHJZaTJTYUkKVFFoaUpMV0lYRTluUkRRNG9hOVNDSXhKUFp5Ukl5UTJFSVc2TG1DRDVtazNtZ2lPNFlVK3ZiMXg3amppS3ZKcQo0MExaaklFQllxY1R4RVN3K2J6cnYrQ1JaMm9UUlRaVGxveGVtYzliOWdhM2pwSjZBbWdvYjRmQkVRSURBUUFCCm8yWXdaREFPQmdOVkhROEJBZjhFQkFNQ0FRWXdFZ1lEVlIwVEFRSC9CQWd3QmdFQi93SUJBakFkQmdOVkhRNEUKRmdRVThBWGxiWis4cnRySmxxYzhpUUxIVjVHUis3TXdId1lEVlIwakJCZ3dGb0FVOEFYbGJaKzhydHJKbHFjOAppUUxIVjVHUis3TXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBRkxFMGFGclQzTnptcWRRdCtPN1c1OW04WnJVCnNtbFFzNGt2cFhET0FwdUxaNzROVUY0K3F1aVVRaFB4VEZFVnA2azBqVjlwWVVzbURMKzZmR1BaQldwdVpscisKSjRYZlcwaENITjlnZ05JelcxWUNZNEVxWGp5ZmY1dTZZQ1MyNmU2ZVB3dFA2RGhObE0xNzRNOXpKbnhGbllZdApZYmFjdDhjOTlwRDZvYlI3VGhnd3BFdE9YbW11ajM5OU5ycjR5cXBaQk95dGxQR291N2JzcFl2dkFhMnJ3QnNJCkh4NTNUT1paMXFNRjBYemNWbVk2eHQ1MklkVUtSdDV1QWsxRGRsQ2RkMHplL2RsZmN4MVBxbnV6dDNndldpL3MKRERCYXg0SnB0cXloMjgwZkVlU1pEd0hpYnY4V3AwRi8ranI1N2Q1K0p0cXgrOTlBSXZiUlM5U1JLMmc9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://192.168.1.11:6443
  name: cluster1
contexts:
- context:
    cluster: cluster1
    user: dev-admin
  name: context-cluster1
current-context: context-cluster1
kind: Config
preferences: {}
users:
- name: dev-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUQxekNDQXIrZ0F3SUJBZ0lVVFprVnpuSFYxMStjdVRWSnNqWHpUVDVOZHQ4d0RRWUpLb1pJaHZjTkFRRUwKQlFBd1lURUxNQWtHQTFVRUJoTUNRMDR4RVRBUEJnTlZCQWdUQ0VoaGJtZGFhRzkxTVFzd0NRWURWUVFIRXdKWQpVekVNTUFvR0ExVUVDaE1EYXpoek1ROHdEUVlEVlFRTEV3WlRlWE4wWlcweEV6QVJCZ05WQkFNVENtdDFZbVZ5CmJtVjBaWE13SUJjTk1qRXhNakF6TURJME1EQXdXaGdQTWpBM01URXhNakV3TWpRd01EQmFNR2N4Q3pBSkJnTlYKQkFZVEFrTk9NUkV3RHdZRFZRUUlFd2hJWVc1bldtaHZkVEVMTUFrR0ExVUVCeE1DV0ZNeEZ6QVZCZ05WQkFvVApEbk41YzNSbGJUcHRZWE4wWlhKek1ROHdEUVlEVlFRTEV3WlRlWE4wWlcweERqQU1CZ05WQkFNVEJXRmtiV2x1Ck1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBbXNKUHBvdEcyNVE4bExyNC9NK3MKdVYzdWduQU14ZWRKYldFQmcxem81UGtyVW8wTUpDUEkyMTgrby9yTHh2eWJ1SlJKRm5qZlJMWlBZNmYrTGZTKwpJQmppbHJQN3J2OHdLMTh1V0EvNVdoWWNQeUZZYTZKeTVRM1RFdkZBYkdLVU5FUjBiWUhNOXdmTGJhVWNmdGkyCnI5dEd5TFVPYzBpemJ5QkFPZFU3Wkx0Z2d2OVdZb213aThLZG84bXVTTjdqSGlpd1BXTmIvQlBDUzE1WElvTXcKZDRzUW15MFFLVENTOHRuR2FzeFlPQ1pqMkhZMTV6dTdmbFJBeWZZcDNCM1pLZVZzQXdvUkhLQmVCa0NlMklwMQpYVnI3aEtkaEtkRWlaNGROcFVjd1V1U2xBRml3K1lPeUREbDZLdmVsSGVHVCs0N3E5SStjbXc0Rm1Ra1hhNGZFCkhRSURBUUFCbzM4d2ZUQU9CZ05WSFE4QkFmOEVCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUcKQ0NzR0FRVUZCd01DTUF3R0ExVWRFd0VCL3dRQ01BQXdIUVlEVlIwT0JCWUVGQTRwcGFzNUZzTGJuNVJIVGxwTQo5T1FlQTlZaE1COEdBMVVkSXdRWU1CYUFGUEFGNVcyZnZLN2F5WmFuUElrQ3gxZVJrZnV6TUEwR0NTcUdTSWIzCkRRRUJDd1VBQTRJQkFRQnRsQ21xN1pZQ2lRVVFHSGdSc2lDY2Q0UmVEZy8rcWVmbkJRT3h4SWN4TzU3UE1uNkwKWjVJNnJwUE9TSi9XaFlwUkNGUGVPTzZTUE5GS1RrUzNIQzlocytmY3dCaFBtV0gzNmJXQytDOXkrU1dXcXpkWQpWRzhpbDF1YW8wK04wWTZVdDdnZ0h5V1RscnByem43MmsrT1dKUlA4VWM5SVpBaWx5TUlHTmdZZENoMDVnbVBlCkd3Z0VyMHBLU3A5UE9SUDFZTGF5VVFsdUdCZkhtWERHM21kd3RYVmFFRmJNbEJsRU1CdCsvMW8xMWNVSFdNVWgKYXVBVWNPYy9RTGUvZUVZcFZTT25NRWpmalJZd1BwY1RybnNsYjNjblFnU2VrdE51QXJWZ1Y5UXg3WkhvZ1o3NApJZTJzSU9tRDRBUGEzNWJWb0c1SkMwYkc2NHVVM2hKOEIzNGgKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBbXNKUHBvdEcyNVE4bExyNC9NK3N1VjN1Z25BTXhlZEpiV0VCZzF6bzVQa3JVbzBNCkpDUEkyMTgrby9yTHh2eWJ1SlJKRm5qZlJMWlBZNmYrTGZTK0lCamlsclA3cnY4d0sxOHVXQS81V2hZY1B5RlkKYTZKeTVRM1RFdkZBYkdLVU5FUjBiWUhNOXdmTGJhVWNmdGkycjl0R3lMVU9jMGl6YnlCQU9kVTdaTHRnZ3Y5VwpZb213aThLZG84bXVTTjdqSGlpd1BXTmIvQlBDUzE1WElvTXdkNHNRbXkwUUtUQ1M4dG5HYXN4WU9DWmoySFkxCjV6dTdmbFJBeWZZcDNCM1pLZVZzQXdvUkhLQmVCa0NlMklwMVhWcjdoS2RoS2RFaVo0ZE5wVWN3VXVTbEFGaXcKK1lPeUREbDZLdmVsSGVHVCs0N3E5SStjbXc0Rm1Ra1hhNGZFSFFJREFRQUJBb0lCQUVEa0tUSFVSS25keG1rMgozU0JrbERCRnlyUzI5eVFrancxbUY1UlZhUEpaNkdoODdCSmJUdVZ0VW42L3NxS0ZXV1pVQnpGOURXRnFjRytCCkNYdUxuQTBwWWhsKzdwRzZQeUJ3a0tZc1RJb1JxMVp0VFA0VTU4aFR1Nlc5c3gyL1dCVnlmcjlNSmYyUEx5V1MKamhoQ0ZwZzJnYisyNjVBN2M4R3M3RUZUdjh2RWZ3S3RYVm50SDVKOVA1R3RWTnBEcTNncnM0UWNSajVzNWI0MwppVFZBTGNabkRHTktrS2JwYzdmYWVxdUc0R0VOWUZQcUJ1RnNvM3BUTzEwWlIrbmQxaFFiWm9xdW5JYlRxUDNGClV3NzJ5MTNLSkdjNkRRbnhpUWROeTFIUWlYbFVtTzk1dER4UHhzdFBmM3BpSVhkU1RpRTUzMDNkVEZpMWtFaG4KN2dWcDhxRUNnWUVBeSsvMEdrZVgzTU4ySG1qNU9iWGlnUzM0Sk90elBpUGdmMENMdzQ4K20wV2VNdkVhZmhwbApNRnl2T2V0bWpQWlpIaWNOaTErMkladDBUWnQ4Qmo4QXc1LzVnd3hXRS9pVm5uYVkyd1NaVlcwYlh4QlJqTkNLCnhYTXJJWlRCK2dwcG9tUGpKRlBFMGNnOWgzWUJFMkdBRUc3RjdnNi9yeGNkOVUrV2VMbE9OczhDZ1lFQXdrUmcKa1Y3ajRIU2llMTFHUzgvTGc1YXd4VTNFTXNMdVNSL2RVRTZ3c1hRMWxBS0x0dTlURXZyTFdydHpzYkhwU0JEYgpIUXVOQWhXandQS0RvY0lzcVNpVFdPdkdMa2NrZGphY2dPL3lYcmpTMng1cmpUWjc2NWRjaFRQUGFRVEE1VFdwCmRjbEI4S0g2Z1k2M1FwTWg1RURFZ3VaS2dRWFNCU0IwdUtnRDBWTUNnWUFkc3V3Umg2dU44c2tZMUtDMnpzNFYKa2VRNVBEQ2tOQVZWZ3NqWHlkeU1NQzlCcStyM3dsQktJclZCOGc0VktTc0JRUjZ2MVZob3ZJTExhb0U5UjUrTQozWmN3aG5OaXBTamswdENmMUtPZjFTdlBSRWtjQUtLMDduaXhnMEJjY1hmQXRsczF4eDA2ajdhbUs0RXNtVjVWCkJreTh4bGtUM29IMlg0akNPL292OFFLQmdIZlhVSzg5RjF5Rzl4a2RVRmxDUmV6V1VCUlhSZnAra0JyaUlsZ0IKUXpVbFdFd0hTZ00vSGtOdUhYYktmck9XNmk4LzNydkxQV0NVMHVFYmVpS1dzNUJpN0lzRlg4dDZyYjZUTC9iRwpqd0RxQ1lHTkFaSXFrMFdocVR5dTJudVJxQ0Y5K2gwa1c1NURmbExnSktOWU9xY2hZVmpURWhFSDh5aWdmZ0RQCi9STHJBb0dBUStWMnlJa2VRYm95b2VKSzJGZnhvUXVaNmY3NERraGFhQkV3UU4rM1NIdmIvWlE0YnpDbktvaUYKODA0bWZuN1VZN0ptN0hOTVYzRHpGRzNxYkhiWDZnSEYyWlFiWm4rb0Ywck8vbWxITnE5QzlJWXpXWS9sZERYVApwS3hMaWsxeEt1VURGUFp2Y01XTmY5Vk82NW5HZXo3R2I5UE9UMTdTQ3FmWGZBRHN2V1U9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
root@hello:~/.kube# 
root@hello:~/.kube# 
root@hello:~/.kube# cat config2
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUR1RENDQXFDZ0F3SUJBZ0lVUzVMSE5FQ0lOMGxhVGRNK3pFZ0Y0SlZXNmp3d0RRWUpLb1pJaHZjTkFRRUwKQlFBd1lURUxNQWtHQTFVRUJoTUNRMDR4RVRBUEJnTlZCQWdUQ0VoaGJtZGFhRzkxTVFzd0NRWURWUVFIRXdKWQpVekVNTUFvR0ExVUVDaE1EYXpoek1ROHdEUVlEVlFRTEV3WlRlWE4wWlcweEV6QVJCZ05WQkFNVENtdDFZbVZ5CmJtVjBaWE13SUJjTk1qRXhNakF6TURJME1qQXdXaGdQTWpFeU1URXhNRGt3TWpReU1EQmFNR0V4Q3pBSkJnTlYKQkFZVEFrTk9NUkV3RHdZRFZRUUlFd2hJWVc1bldtaHZkVEVMTUFrR0ExVUVCeE1DV0ZNeEREQUtCZ05WQkFvVApBMnM0Y3pFUE1BMEdBMVVFQ3hNR1UzbHpkR1Z0TVJNd0VRWURWUVFERXdwcmRXSmxjbTVsZEdWek1JSUJJakFOCkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXZwdVJYVXd1TlQ5TGl4VDFCNHdEYTBZRHdKMFkKdWlaSGNsUk1rZjJTS3BWSHByazBNamx1R2g0WmR1ZEloUkk3YUpZbVZ3c3RQendpRlRPa2J0WEtzaVp5N0g4dApVVC9WRHQya0NnTDlvc0tKUE12OEI0aGp5R3h0bjFISk9aQ2NMSWEwTUFBaUtNVjhiRXFrT0hOK2tmVjhwR1lJCmZiVjRWYmlTUzRNMGlYdnhBT1hRSFFHU2lqV3c4d0h2aWNGWUxtME50bFlUM3pUZjVjVC9kRGJSdWhSRFF2clkKaVpnUHo0ZHg4YTZibFA3SkRmeTZMWTVXZmtBMFAxdWVtS05wR29pK1BHRDRFbGluRWd1aW9tbUtsWEowZ1pZTQpiNHNBbzJlWGY1ZGxQdkZxK0lJbzlYeWVzSGR1RDFWQ3dpRitudUZ4QmdlNnI2elQ4ZGFaL2NLMGZRSURBUUFCCm8yWXdaREFPQmdOVkhROEJBZjhFQkFNQ0FRWXdFZ1lEVlIwVEFRSC9CQWd3QmdFQi93SUJBakFkQmdOVkhRNEUKRmdRVTRhZlRpTE9MNVV1TUcrOS95ZXZGUVMxWkJEMHdId1lEVlIwakJCZ3dGb0FVNGFmVGlMT0w1VXVNRys5Lwp5ZXZGUVMxWkJEMHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBRHQ1SlpqZjRoRS9VQVRCc3ZMUXphOFY5bkR4ClVZdStpaFVVTWRDenJQSkV0WXlMYWFQZXppenBicWFpU3YrblBoR2UxSndXMThIZmlsS0dsOTlCVENkc3VHSjUKVzhVTCtHbHdvVHZSRjNaK0F6M3NQL3dBelB1Vk5ORzZwVkJkanNSbDhhN29UMWV0RjM0UUovWWtOVFR4M1JrQQpRZjhqYXdZams3Wi9pL0VqM3hmd0FxRkhzT1Q2MjlXMnc0VU9SaVZBeHZuc2czWUozZ3RyVFRBK2hkVGhUWDViCkxpQjN5ZFFPUWNRTUE0SU9CeG0vdkRrR3lGMVBBL1BjMTFWTDBjZktrK25WY3J4RHAxS2JtVXhmNkhua3JqWlkKaGpNWEFqRDJIL0MzT2JjQUFMNXF6aUNKU1pDM0xDMVNwNEtKUGdMZVJPc1haZWpzUXRTRWN6YWhTRE09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://192.168.1.12:6443
  name: cluster2
contexts:
- context:
    cluster: cluster2
    user: test-admin
  name: context-cluster2
current-context: context-cluster2
kind: Config
preferences: {}
users:
- name: test-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUQxekNDQXIrZ0F3SUJBZ0lVRXF4OTlBOUFMem5GcTg5RDZLYzBjYk5GTGNVd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1lURUxNQWtHQTFVRUJoTUNRMDR4RVRBUEJnTlZCQWdUQ0VoaGJtZGFhRzkxTVFzd0NRWURWUVFIRXdKWQpVekVNTUFvR0ExVUVDaE1EYXpoek1ROHdEUVlEVlFRTEV3WlRlWE4wWlcweEV6QVJCZ05WQkFNVENtdDFZbVZ5CmJtVjBaWE13SUJjTk1qRXhNakF6TURJMU1EQXdXaGdQTWpBM01URXhNakV3TWpVd01EQmFNR2N4Q3pBSkJnTlYKQkFZVEFrTk9NUkV3RHdZRFZRUUlFd2hJWVc1bldtaHZkVEVMTUFrR0ExVUVCeE1DV0ZNeEZ6QVZCZ05WQkFvVApEbk41YzNSbGJUcHRZWE4wWlhKek1ROHdEUVlEVlFRTEV3WlRlWE4wWlcweERqQU1CZ05WQkFNVEJXRmtiV2x1Ck1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBdkJ4aDlITEVNR3g0Y2VXWTNlYnAKbERseTcvQmo3aDF0Szlod21IV09iLy9sR0I2WjB0Sy93cDhUQUxwNThRV25mcE4xNkpWdFhsNXBXMDgwQVh0TgpyTkVpVnhsQXk0RUZSVVpNVFFtWTJZVDZlYVM1ZXFpQmZVR0dRRDM1OFdiOGtOS0R4a0REeEdHek1yYXRiZE5NCng0WTF6ZGNIUGh4Qy9jU3E1amNXN2RqTEp4ZnkzS29iZFIxTjNBSW5jSnZRYjNnZVdEN0FlNk9KZkJBTFJGY2sKOHVxS29MNFdVWGZuZVBqalN1ZzZLbytOL2IyS0hXU3gzaEdDbzJLLyszVkNvQXJETjdIeFJYSm4vakd6djRuQgo0aVlIYkZ4TU5MWDJ3TUk2NjJ0SEQ2TzBMbkdPUm5ETXFQTStLeFljbWZFc01jMWcyMGxsUWFUYXd3YXVUL1hPCk1RSURBUUFCbzM4d2ZUQU9CZ05WSFE4QkFmOEVCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUcKQ0NzR0FRVUZCd01DTUF3R0ExVWRFd0VCL3dRQ01BQXdIUVlEVlIwT0JCWUVGR3VJMEZodUJPY0xJTWM0WCtGTQpIMEJPdlhKME1COEdBMVVkSXdRWU1CYUFGT0duMDRpemkrVkxqQnZ2ZjhucnhVRXRXUVE5TUEwR0NTcUdTSWIzCkRRRUJDd1VBQTRJQkFRQnRzUXg5UGhXaXg4WVBMYU05OG14dGkxNHpZK3Q3bE0yWjJPSWE2L1NQZXZFd1plMGwKWngyWUdKQ3NnaDIvSGZwMGRHZG5MYXB1OVd5ODlxaVdQdnVxUm5PVVN4cmpnVEo4TmluYjBYUG4xNU96MGJ5MQpJemova3JjU09CTzVDSEFBTzBYeXVETThxblJqRCtXN3lPMDZhNG1XRUsyWWhqWE1wa1lORTZEeFZkZ0xEaUdBCnlDM3pQczVyd28yR2JkQnNPei9qYzcvazJBRjYrbnYzTnJCRnJMcjF0NmFHUE9zby96SlZLNHR5UHBkb3Zmc1cKQkgwS1lNbGdvK3h3dlRIZ0ZLNlcwQ2JZNlVzM3VSZERGUTZXSVpPaitCME9od0QvT3JONUxGZzdydDdQbHgxRwp1MjZBc3M0OE8wbjlIdDY5d2NnSTB4WGJPWGRUWXV4MFZVUUgKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBdkJ4aDlITEVNR3g0Y2VXWTNlYnBsRGx5Ny9CajdoMXRLOWh3bUhXT2IvL2xHQjZaCjB0Sy93cDhUQUxwNThRV25mcE4xNkpWdFhsNXBXMDgwQVh0TnJORWlWeGxBeTRFRlJVWk1UUW1ZMllUNmVhUzUKZXFpQmZVR0dRRDM1OFdiOGtOS0R4a0REeEdHek1yYXRiZE5NeDRZMXpkY0hQaHhDL2NTcTVqY1c3ZGpMSnhmeQozS29iZFIxTjNBSW5jSnZRYjNnZVdEN0FlNk9KZkJBTFJGY2s4dXFLb0w0V1VYZm5lUGpqU3VnNktvK04vYjJLCkhXU3gzaEdDbzJLLyszVkNvQXJETjdIeFJYSm4vakd6djRuQjRpWUhiRnhNTkxYMndNSTY2MnRIRDZPMExuR08KUm5ETXFQTStLeFljbWZFc01jMWcyMGxsUWFUYXd3YXVUL1hPTVFJREFRQUJBb0lCQUJnblVOQ1JkKzE3MEE5WAoyc1FMWlV5Wi84OGRQOGVRVWJkQ2lGcWJKWm50OHAyaE9FRWd2R3loL2srbW9nZTNvU1VZakJnOEw1bmhaNGZJCjZMV1Qvb3BGSkRLbzFIQU05ZjlLSW52MTBvR0RtS0hMNitEN0IvMXNUMitxUlpDZ2w2ZUUwRlRCZGlHZUplTksKSDRTdGovdENtVi8venpkRGE3cW42UVc4Wng1TThuNnM0dUp3WXNGTXN6UlBwYnU4eDFjdTBuU0NkeXBvZ2RWRQo1SWlIN1ZoL3hEQVF0U2VZdUtubDhmYmlwS0pMS1hrcFNQdjAyK0FWWmtFL05Ua0M1SVFMVUdMQWZqYXdzTkdpCjI3clViT2piY2NTRjlPOHYvR1RVNEpzRnMvRGYxLzdZSnBUeFhxMU5oaUtZdW9sWGlncG5WZE9kK3dqLzRXZ1QKeCtRdjJ3MENnWUVBNkUyMFF3clk4UXNXVEs2Tlp0dlNUODVjS1dVYkNDTWJ1SVkyYXVnaDVIdU15eERySkZXZwpLUXg2TmQ3Z1h2eSt3R1V3ZHBXMmE3NGNOeTdjc0tNbjJPaEhLdEd0RDJwUit4TXUxMlIwZ0JvZFFjYVIyT1lSCllBY3RuQTNmMXFadXFYYjNuZHBOMFdJMDAvTGtwUGJqSE5GU1d1ZGd6d1RReXRpdmxzS2xiWnNDZ1lFQXoweWwKY1J3NGdMc0c1ZU9KRXA0RkVPdmhscG41dDZaajJhcjF0amg2am54Q3ZrSkFYYkxGOEdzalE0OGVlV2V2S2VMUgpkZERSMTVxY1hJQ09nTUJBR2tJQmxQMC85bXBpMmZDbUozb1VCSURPVUFta3pmMkthSVVJb3VKWGlsK3BTYXRGCjBFTitsK1hucDFWWGJnVi9ia093cG1ZdHpxUHVQamdBWkFETmxpTUNnWUJ3dGpMK1RHY1NIU1VHczdLYjg1QkoKZElDMi9QMXVwMG90NzhDN2drSGZrQ3F4NUZXUzNaREdHZTI1OFplL3ZyWDJ0NklhQjIzcFBPYUh4ODhBVFVscQpMdGxJNTA4bXFabDVUc2R0YnFvdjlYdTRqRlg3ZlRWMCtFYWk3d0JxTDNxRjh0a1YxL1BsNGRacjkvQUVNbDNqCmY1U0wwclBmL2lBb0s1YVdlWDYyZlFLQmdHT3NEN1FtQklqbzVEVXV4UjU5ZWlRYnRuanFDZWFpaTBvQ2FHZzQKR2IxZXc5eWxFRHU5RkcwM3BsbjZlNFdXTStPbzJsdVNqd0xpcFNIWThpdTN4RnFidUJVQiszb294dVRSVDZLVgprUUJsU2syemhWbEIrZ1d0U1d5LzlhVmp2NHJiWGhMNEVPdEtNS3NGWHFkWTMxK09EbWJEcEd6QjUzQmxEdE1HCmk5TVBBb0dCQUl4dndic2t6dUFhWHJNbTJoaXdwTVNscElLWjhZY2lUUjhtWHFHMTV6NGt2Wi9Hcmd4SzR4a3YKRjdmRlRtQzlqSFJzS0lqZ28rNkoxVG5QalVKeUlJdTVPWWVJL2RvelBPakllczE2NkZRbThSY3hTdGExOU91dAp1N3BOSjhlRHdVYmJKbG01WTdOOFc5cmJyRzh3VkZvRDhmSU5yZXBSTVo2elFLeEVmQUdHCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
root@hello:~/.kube# 
root@hello:~/.kube# 
root@hello:~/.kube#
```

  

 # **修改配置中：**

```
#配置1
- cluster:
    server: https://192.168.1.11:6443
  name: cluster1
  
contexts:
- context:
    cluster: cluster1
    user: dev-admin
  name: context-cluster1

users:
- name: dev-admin


#配置2
- cluster:
    server: https://192.168.1.12:6443
  name: cluster2
  
contexts:
- context:
    cluster: cluster2
    user: test-admin
  name: context-cluster2

users:
- name: test-admin
```

  

  

 # **写入配置文件**

```
root@hello:~/.kube# KUBECONFIG=config1:config2 kubectl config view --flatten > $HOME/.kube/config
root@hello:~/.kube# 
root@hello:~/.kube#
```

  

 # **测试配置**

```
root@hello:~/.kube# kubectl config get-contexts
CURRENT   NAME               CLUSTER    AUTHINFO     NAMESPACE
*         context-cluster1   cluster1   dev-admin    
          context-cluster2   cluster2   test-admin   
root@hello:~/.kube# 
root@hello:~/.kube# kubectl config current-context
context-cluster1
root@hello:~/.kube# 
root@hello:~/.kube# kubectl get node
NAME           STATUS   ROLES    AGE   VERSION
192.168.1.11   Ready    master   34d   v1.22.2
root@hello:~/.kube# 
root@hello:~/.kube# kubectl config use-context context-cluster2
Switched to context "context-cluster2".
root@hello:~/.kube# 
root@hello:~/.kube# kubectl get node
NAME           STATUS                     ROLES    AGE   VERSION
192.168.1.12   Ready,SchedulingDisabled   master   34d   v1.22.2
192.168.1.13   Ready                      node     34d   v1.22.2
192.168.1.14   Ready                      node     34d   v1.22.2
root@hello:~/.kube#
```

  

 # **附录**

```
current-context 显示 current_context
delete-cluster  删除 kubeconfig 文件中指定的集群
delete-context  删除 kubeconfig 文件中指定的 context
get-clusters    显示 kubeconfig 文件中定义的集群
get-contexts    描述一个或多个 contexts
rename-context  Renames a context from the kubeconfig file.
set             设置 kubeconfig 文件中的一个单个值
set-cluster     设置 kubeconfig 文件中的一个集群条目
set-context     设置 kubeconfig 文件中的一个 context 条目
set-credentials 设置 kubeconfig 文件中的一个用户条目
unset           取消设置 kubeconfig 文件中的一个单个值
use-context     设置 kubeconfig 文件中的当前上下文
view            显示合并的 kubeconfig 配置或一个指定的 kubeconfig 文件
```

  

  

  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1dbe669de694b7382bc287aca01323c~tplv-k3u1fbpfcp-zoom-1.image)  

  

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

https://www.jianshu.com/u/0f894314ae2c

https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/

知乎、CSDN、开源中国、思否、掘金、哔哩哔哩、腾讯云、简书、今日头条

  