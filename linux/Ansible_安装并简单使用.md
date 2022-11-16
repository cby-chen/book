---
layout: post
cid: 34
title: Ansible 安装并简单使用
slug: 34
date: 2021/12/30 17:07:00
updated: 2022/03/25 15:48:24
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


Ansible 简介

Ansible 是一款 IT 自动化工具。主要应用场景有配置系统、软件部署、持续发布及不停服平滑滚动更新的高级任务编排。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc088adca8ce488482700e675f8717c3~tplv-k3u1fbpfcp-zoom-1.image)

  

Ansible 本身非常简单易用，同时注重安全和可靠性，以最小化变动为特色，使用 OpenSSH 实现数据传输 ( 如果有需要的话也可以使用其它传输模式或者 pull 模式 )，其语言设计非常利于人类阅读，即使是针对不刚接触 Ansible 的新手来讲亦是如此。

  

我们坚信无论什么范围的环境，简单都是必须的，所以我们的设计尽可能满足各类型的繁忙人群：开发人员、系统管理员、发布工程师、IT 管理员等所有类型的人。同时， Ansible 适用于各种环境，小到几台多到成千上万台的企业实际环境都完全满足。

  

Ansible 不使用C/S架构管理节点，即没有 Agent 。这样的架构使得 Ansible 不会存在如何升级远程 Agent 管理进程或者因为没有安装 Agent 而无法管理系统。因为 OpenSSH 是非常流行的开源组件，安全问题也非常少 。Ansible 的 去中心化 管理方式深受业内认可， 即它只依赖 OS 的 KEY 认证访问远程主机。如需， Ansible 可以便捷接入 Kerberos, LDAP 或者其它认证系统。

  

安装ansible工具

  

```
root@Ansible:~# apt update && apt install ansible
root@Ansible:~# apt install sshpass
```

创建秘钥

  

```
root@Ansible:~# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:ZlnekfYdDkp4AA2zZLysbtr8Epcp6tMgFB2TGEY/zFU root@Ansible
The key's randomart image is:
+---[RSA 3072]----+
|.++oo.oE+.       |
|.o+oo o.o.o  .   |
|  .=  .....o+. . |
| .  .  o +oo.oo..|
|.     . S ... ...|
| . . + *         |
|  . = +          |
|   oo=           |
|  .o+oo.         |
+----[SHA256]-----+
root@Ansible:~#


批量拷贝脚本


root@Ansible:~# vim copy_ssh_id.sh


root@Ansible:~# cat copy_ssh_id.sh
#!/bin/bash
rm -f ./authorized_keys; touch ./authorized_keys
sed -i '/StrictHostKeyChecking/s/^#//; /StrictHostKeyChecking/s/ask/no/' /etc/ssh/ssh_config
sed -i "/#UseDNS/ s/^#//; /UseDNS/ s/yes/no/" /etc/ssh/sshd_config


cat hostsname.txt | while read host ip pwd; do
  sshpass -p $pwd ssh-copy-id -f $ip 2>/dev/null
  ssh -nq $ip "hostnamectl set-hostname $host"
  ssh -nq $ip "echo -e 'y\n' | ssh-keygen -q -f ~/.ssh/id_rsa -t rsa -N ''"
  echo "===== Copy id_rsa.pub of $ip ====="
  scp $ip:/root/.ssh/id_rsa.pub ./$host-id_rsa.pub
  #cat ./$host-id_rsa.pub >> ./authorized_keys
  echo $ip $host >> /etc/hosts
done


root@Ansible:~#
```

  

添加主机信息

  

```
root@Ansible:~# vim hostsname.txt


root@Ansible:~# cat hostsname.txt
node  192.168.1.2    123123
node  192.168.1.3    123123
node  192.168.1.4    123123
node  192.168.1.5    123123
node  192.168.1.6    123123
node  192.168.1.7    123123
node  192.168.1.8    123123
node  192.168.1.9    123123


------
```

  

fetch模块：

copy模块：

  

```
1、从远程主机获取文件：


root@Ansible:~# ansible k8s -m fetch -a "src=/root/node.sh dest=/root/test"


2、从本地主机传到远程：
root@Ansible:~# ansible k8s -m copy -a "src=/root/node.sh dest=/root"


3、远程复制或者本地上传，加上force=yes，则会覆盖掉原来的文件，加上backup=yes，在覆盖的时候会把原来的文件做一个备份：
root@Ansible:~# ansible k8s -m copy -a "src=/root/node.sh dest=/root force=yes backup=yes"


4、复制的时候可以带参数：owner,group,mode
```

  

\---------

  

```
将本地的源拷贝到服务器上


root@Ansible:~# ansible k8s -m copy -a "src=/etc/apt/sources.list dest=/etc/apt/"


更新源


root@Ansible:~# ansible k8s  -m  command -a 'apt update'


安装ntpdate


root@Ansible:~# ansible k8s  -m  command -a 'apt install ntpdate'


同步时间


root@Ansible:~# ansible k8s  -m  command -a 'ntpdate -u ntp.aliyun.com'



修改时区


root@Ansible:~#
root@Ansible:~#
root@Ansible:~# ansible k8s  -m  command -a 'cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime'



查看是否修改


root@Ansible:~#
root@Ansible:~# ansible k8s  -m  command -a 'date -R '
192.168.1.13 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:57 +0800
192.168.1.10 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:57 +0800
192.168.1.14 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:57 +0800
192.168.1.12 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:57 +0800
192.168.1.11 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:57 +0800
192.168.1.15 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:57 +0800
192.168.1.51 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:57 +0800
192.168.1.52 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:57 +0800
192.168.1.16 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:57 +0800
192.168.1.53 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:57 +0800
192.168.1.55 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:58 +0800
192.168.1.54 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:58 +0800
192.168.1.57 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:58 +0800
192.168.1.56 | CHANGED | rc=0 >>
Thu, 11 Nov 2021 14:52:58 +0800
root@Ansible:~#
root@Ansible:~#
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