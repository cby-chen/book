---
layout: post
cid: 23
title: MySQL8.0允许外部访问
slug: 23
date: 2021/12/30 17:03:22
updated: 2021/12/30 17:03:22
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


MySQL8.0允许外部访问
==============

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ea2bc3ed27549908286720b6a9f1215~tplv-k3u1fbpfcp-zoom-1.image)

### 一、前置条件：

按照https://blog.csdn.net/h996666/article/details/80917268安装完MySQL之后。

### 二、开始修改配置：

1，登进MySQL之后，

2，输入以下语句，进入mysql库：

```
use mysql
```

3，更新域属性，’%’表示允许外部访问：

```
update user set host='%' where user ='root';
```

4，执行以上语句之后再执行：

```
FLUSH PRIVILEGES;
```

5，再执行授权语句：

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;
```

然后外部就可以通过账户密码访问了。

6，其它说明：

FLUSH PRIVILEGES; 命令本质上的作用是：

将当前user和privilige表中的用户信息/权限设置从mysql库(MySQL数据库的内置库)中提取到内存里。

MySQL用户数据和权限有修改后，希望在”不重启MySQL服务”的情况下直接生效，那么就需要执行这个命令。

通常是在修改ROOT帐号的设置后，怕重启后无法再登录进来，那么直接flush之后就可以看权限设置是否生效。

而不必冒太大风险。

### 三、可能存在的其它问题：

执行完之后，再用Navicat连接mysql，报错如下：

```
Client does not support authentication protocol requested by server；
```

报错原因：

mysql8.0 引入了新特性 caching_sha2_password；这种密码加密方式Navicat 12以下客户端不支持；

Navicat 12以下客户端支持的是mysql_native_password 这种加密方式；

解决方案：

1，用如下语句查看MySQL当前加密方式

```
select host,user,plugin from user;
```

查询结果

```
+-----------+------------------+-----------------------+
| host      | user             | plugin                |
+-----------+------------------+-----------------------+
| %         | root             | caching_sha2_password |
| localhost | mysql.infoschema | mysql_native_password |
| localhost | mysql.session    | mysql_native_password |
| localhost | mysql.sys        | mysql_native_password |
+-----------+------------------+-----------------------+
```

看第一行，root加密方式为caching_sha2_password。  

2，使用命令将他修改成mysql_native_password加密模式：

```
update user set plugin='mysql_native_password' where user='root';
```

再次连接的时候，就成功了。

### 四、如果还连接不上

通过以上操作后，依然无法连接上，问题可能出在了防火墙上。

**1，MySQL部署在实体服务器上解决方案如下：**  
a.开放MySQL的端口号，默认端口号是3306。  
b.直接关闭防火墙（慎重操作，不建议。当然测试玩的话就随意了。。。。）

**2，MySQL部署在云计算机上的方案如下：**  
a.以阿里云为例，找到实例，设置安全组，开放端口号即可。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a99b8c6d78e64b13b6f8626c8c2b1fad~tplv-k3u1fbpfcp-zoom-1.image)