---
layout: post
cid: 49
title: Ubuntu 通过 Netplan 配置网络教程
slug: 49
date: 2021/12/30 17:10:00
updated: 2022/03/25 15:33:28
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


**Ubuntu 通过 Netplan 配置网络教程**

Ubuntu through Netplan configuration network tutorial

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f57883d164c4b79a4210075234a7ee8~tplv-k3u1fbpfcp-zoom-1.image)

  

**一、Netplan 配置流程**

1. Netplan configuration process

  

**1、Netplan默认配置文件在/etc/netplan目录下。您可以使用以下命令找到：**

1. The default configuration file of Netplan is in the /etc/netplan directory. You can find it with the following command:

  

```
ls /etc/netplan/
```

  

**就可以看到配置文件名称。**

You can see the configuration file name.

  

**2、查看Netplan网络配置文件的内容，执行以下命令：**

2. View the contents of the Netplan network configuration file and execute the following command:

  

```
cat /etc/netplan/*.yaml
```

  
  

**3、现在你需要在任何编辑器中打开配置文件： 由于我使用 vim 编辑器来编辑配置文件，所以我将运行：**

3. Now you need to open the configuration file in any editor: Since I use the vim editor to edit the configuration file, I will run:

  
  

```
vim /etc/netplan/*.yaml
```

  
  

**根据您的网络需要更新配置文件。对于静态 IP 寻址，添加 IP 地址、网关、DNS 信息，而对于动态 IP 寻址，无需添加此信息，因为它将从 DHCP 服务器获取此信息。使用以下语法编辑配置文件。**

Update the configuration file according to your network needs. For static IP addressing, add IP address, gateway, DNS information, and for dynamic IP addressing, there is no need to add this information because it will get this information from the DHCP server. Use the following syntax to edit the configuration file.

  
  

**4、在应用任何更改之前，我们将测试配置文件。**

4. We will test the configuration file before applying any changes.

  

```
sudo netplan try
```

  

**如果没有问题，它将返回配置接受消息。如果配置文件未通过测试，它将恢复为以前的工作配置。**

If there is no problem, it will return a configuration acceptance message. If the configuration file fails the test, it will revert to the previous working configuration.

  

**5、运行以下命令来应用新配置：**

5. Run the following command to apply the new configuration:

  

```
sudo netplan apply
```

  

**6、成功应用所有配置后，通过运行以下命令重新启动 Network-Manager 服务：**

6. After successfully applying all the configurations, restart the Network-Manager service by running the following command:

  

**如果是桌面版：**

If it is the desktop version:

  
  

```
sudo systemctl restart system-networkd
```

**如果您使用的是 Ubuntu 服务器，请改用以下命令：**

If you are using an Ubuntu server, use the following command instead:

  

```
sudo systemctl restart network-manager
```

  

**7、验证 IP 地址**

7. Verify the IP address

  

```
ip a
```

  
  

**二、Netplan 配置文件详解**

2. Detailed explanation of Netplan configuration file 

  

**1、使用 DHCP：**

1. Use DHCP:

  

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: true
```

  

  

**2、使用静态 IP：**

2. Use static IP:

  

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      addresses:
        - 10.0.0.10/8
      gateway4: 10.0.0.1
      nameservers:
          search: [mydomain, otherdomain]
          addresses: [10.0.0.5, 1.1.1.1]
```

  

**3、多个网口 DHCP：**

3. Multiple network ports DHCP:

  

```
network:
  version: 2
  ethernets:
    enred:
      dhcp4: yes
      dhcp4-overrides:
        route-metric: 100
    engreen:
      dhcp4: yes
      dhcp4-overrides:
        route-metric: 200
```

  

**4、连接开放的 WiFi（无密码）：**

4. Connect to open WiFi (without password):

  

```
network:
  version: 2
  wifis:
    wl0:
      access-points:
        opennetwork: {}
      dhcp4: yes
```

  

**5、连接 WPA 加密的 WiFi：**

5. Connect to WPA encrypted WiFi:

  

```
network:
  version: 2
  renderer: networkd
  wifis:
    wlp2s0b1:
      dhcp4: no
      dhcp6: no
      addresses: [10.0.0.10/8]
      gateway4: 10.0.0.1
      nameservers:
        addresses: [10.0.0.5, 8.8.8.8]
      access-points:
        "network_ssid_name":
          password: "**********"
```

  

  

**6、在单网卡上使用多个 IP 地址（同一网段）：**

6. Use multiple IP addresses on a single network card (same network segment):

  

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
     addresses:
       - 10.0.0.10/8
       - 10.0.0.10/8
     gateway4: 10.0.0.1
```

**7、在单网卡使用多个不同网段的 IP 地址：**

7. Use multiple IP addresses of different network segments on a single network card:

  

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
     addresses:
       - 9.0.0.9/24
       - 10.0.0.10/24
       - 11.0.0.11/24
     #gateway4:    # unset, since we configure routes below
     routes:
       - to: 0.0.0.0/0
         via: 9.0.0.1
         metric: 100
       - to: 0.0.0.0/0
         via: 10.0.0.1
         metric: 100
       - to: 0.0.0.0/0
         via: 11.0.0.1
         metric: 100
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