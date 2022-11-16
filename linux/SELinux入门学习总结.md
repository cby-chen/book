---
layout: post
cid: 28
title: SELinux入门学习总结
slug: 28
date: 2021/12/30 17:04:23
updated: 2021/12/30 17:04:23
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


**前言**

安全增强型 Linux（Security-Enhanced Linux）简称 SELinux，它是一个 Linux 内核模块，也是 Linux 的一个安全子系统。

  

SELinux 主要由美国国家安全局开发。2.6 及以上版本的 Linux 内核都已经集成了 SELinux 模块。

  

SELinux 的结构及配置非常复杂，而且有大量概念性的东西，要学精难度较大。很多 Linux 系统管理员嫌麻烦都把 SELinux 关闭了。

  

如果可以熟练掌握 SELinux 并正确运用，我觉得整个系统基本上可以到达“坚不可摧”的地步了（请永远记住没有绝对的安全）。

  

掌握 SELinux 的基本概念以及简单的配置方法是每个 Linux 系统管理员的必修课。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33a7ae230f5d409fb7f165b0010ff55e~tplv-k3u1fbpfcp-zoom-1.image)

  

**一、基本概念**

**1、TE模型的安全上下文**

所有的操作系统访问控制都基于主体、客体，以及与他们相关的访问控制属性。

  

在selinux中，**访问控制属性叫做安全上下文。**所有对象(文件、进程间通信通道、套接字、网络主机等)和主体(进程)都有一个与之关联的安全上下文。

  

**一个安全上下文包含三个元素：**用户（user）、角色（role）和类型标识符（type identifiers）

  

**安全上下文的形式如下：**user：role：type

  

**对进程来说：**分别表示用户、角色、类型标识符也被称为域

  

**对客体来说：**前两项基本没有实际用途，role通常为object_r，user通常位创建这个对象的进程的user，对访问控制没有影响

  

**总结：**

SELinux是通过MAC(Mandatory Access Control)方式来控管进程，它控制的主体是进程，而目标则是该进程能否读取的”文件资源”。

**主体**

SELinux 主要是想管理控制进程。

注\*：为了方便理解，如无特别说明，以下均把进程视为主体。

**目标**

主体进程能否访问的”目标资源”一般是文件系统。

**对象**

被主体访问的资源。可以是文件、目录、端口、设备等。

注\*：为了方便理解，如无特别说明，以下均把文件或者目录视为对象。

**策略**

因为进程和文件的数量庞大，因此 SELiunx会根据某些服务来制定基本的访问安全性策略。

这些策略内部还有详细的规则来指定不同的服务开放某些资源的访问与否。

系统中通常有大量的文件和进程，为了节省时间和开销，通常我们只是选择性地对某些进程进行管制。

而哪些进程需要管制、要怎么管制是由政策决定的。

一套政策里面有多个规则。部分规则可以按照需求启用或禁用（以下把该类型的规则称为布尔型规则）。

规则是模块化、可扩展的。在安装新的应用程序时，应用程序可通过添加新的模块来添加规则。用户也可以手动地增减规则。

  

**SELINUX参数值：**

enforcing:强制执行SELinux功能；

permissive:只显示警告信息；

disabled:停用SELinux功能。

  

SELINUXTYPE参数值：

targeted:针对网络服务限制较多，针对本机限制较少，是默认的策略；

strict:完整的保护功能，包括网络服务、一般指令、应用程序，限制方面较为严格。

  

**安全上下文**

  

安全上下文是 SELinux 的核心。

安全上下文我自己把它分为「进程安全上下文」和「文件安全上下文」。

一个「进程安全上下文」一般对应多个「文件安全上下文」。

只有两者的安全上下文对应上了，进程才能访问文件。它们的对应关系由政策中的规则决定。

文件安全上下文由文件创建的位置和创建文件的进程所决定。而且系统有一套默认值，用户也可以对默认值进行设定。

需要注意的是，单纯的移动文件操作并不会改变文件的安全上下文。

安全上下文的结构及含义

安全上下文有四个字段，分别用冒号隔开。形如：system_u:object_r:admin_home_t:s0。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fd13628198b4df29e5b5b497955a78d~tplv-k3u1fbpfcp-zoom-1.image)

  

**SELinux 的工作模式**

SELinux 有三种工作模式，分别是：

  

enforcing：强制模式。违反 SELinux 规则的行为将被阻止并记录到日志中。

permissive：宽容模式。违反 SELinux 规则的行为只会记录到日志中。一般为调试用。

disabled：关闭 SELinux。

SELinux 工作模式可以在 /etc/selinux/config 中设定。

  

如果想从 disabled 切换到 enforcing 或者 permissive 的话，需要重启系统。反过来也一样。

  

enforcing 和 permissive 模式可以通过 setenforce 1|0 命令快速切换。

  

需要注意的是，如果系统已经在关闭 SELinux 的状态下运行了一段时间，在打开 SELinux 之后的第一次重启速度可能会比较慢。因为系统必须为磁盘中的文件创建安全上下文。

  

SELinux 日志的记录需要借助 auditd.service 这个服务，请不要禁用它。

  

**SELinux 工作流程**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5281d13148334d958546b13cbbd26914~tplv-k3u1fbpfcp-zoom-1.image)

  

  

**显示安全上下文**

  

加上-Z能显示主体、客体的上下文

  

ls -Z能显示文件系统的安全上下文

  

ps -Z能显示进程的安全上下文

  

id -Z能显示shell的安全上下文：joe：usr_r：usr_t

  

  

**2、TE访问控制**

在SELinux中，默认时没有允许规则的，也没有超级用户。被允许的访问必须由规则给出。

  

一条规则如下：

  

allow Source type(s) Target type(s): Object class(es) Permission(s)

  

比如这样的访问规则：

  

allow user_t bin_t : file {read execute getattr};

  

表示允许域为user_t的进程对type为bin_t的文件具有读、执行、得到属性的操作

  

**3、角色的作用**

SELinux也提供基于角色的访问控制

  

通过以下语句指定role的type：

  

role user_r type passwd_t;

  

如果没有以上这条语句，则：

  

安全上下文joe：user_r：passwd_t则不能被创建

exec调用则失败，即便策略允许

  

**二、架构**

**1、内核架构**

基于LSM（linux security module），为所有的内核的资源提供强制访问控制

  

**注\*：LSM（linux security module）一种轻量级的安全访问控制框架，主要利用Hook函数对权限进行访问控制，并在部分对象中内置了透明的安全属性。**

  

LSM提供了一系列的钩子函数

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f19f42d70854b0d852dae925d4a2fbc~tplv-k3u1fbpfcp-zoom-1.image)

如果访问被DAC拒绝，则会影响审计结果

  

  

**SELinux的架构如下所示：**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f42877ac2734297b0a6910d6b3f07cb~tplv-k3u1fbpfcp-zoom-1.image)

  

  

  

策略决定包含在安全服务器中，与具体架构无关，便于移植

  

对象管理者时各对象的管理者，在LSM架构中，是一系列的LSM钩子，遍布在内核的子系统中。

  

**注\*：Linux安全模块（LSM）提供的接口就是钩子，其初始化时所指向的虚拟函数实现了缺省的传统UNIX超级用户机制，模块编写者必须重新实现这些钩子函数来满足自己的安全策略。**

  

  

**2、用户空间的对象管理器**

SELinux支持将对象管理器放到用户态，使用内核的对象管理策略服务器来管理用户态的对象

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b3a83bda020406ca8995d7478e2528a~tplv-k3u1fbpfcp-zoom-1.image)

  

  

然而，支持用户空间的对象管理器有一些弱点：

  

对于TE模型，还需要定义class

对于对象管理器的管理策略不再内核之中

**策略服务架构如下：**

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78acf33ec8ff4ff7ab7324697bfce65d~tplv-k3u1fbpfcp-zoom-1.image)

  

  

AVC表示各种缓存

  

**三、SELinux 的作用及权限管理机制**

**1 SELinux 的作用**

SELinux 主要作用就是最大限度地减小系统中服务进程可访问的资源（最小权限原则）。

  

设想一下，如果一个以 root 身份运行的网络服务存在 0day 漏洞，黑客就可以利用这个漏洞，以 root 的身份在您的服务器上为所欲为了。是不是很可怕？

  

SELinux 就是来解决这个问题的。

  

**2 DAC**

在没有使用 SELinux 的操作系统中，决定一个资源是否能被访问的因素是：某个资源是否拥有对应用户的权限（读、写、执行）。

  

只要访问这个资源的进程符合以上的条件就可以被访问。

  

而最致命问题是，root 用户不受任何管制，系统上任何资源都可以无限制地访问。

  

这种权限管理机制的主体是用户，也称为自主访问控制（DAC）。

  

**3 MAC**

在使用了 SELinux 的操作系统中，决定一个资源是否能被访问的因素除了上述因素之外，还需要判断每一类进程是否拥有对某一类资源的访问权限。

  

这样一来，即使进程是以 root 身份运行的，也需要判断这个进程的类型以及允许访问的资源类型才能决定是否允许访问某个资源。进程的活动空间也可以被压缩到最小。

  

即使是以 root 身份运行的服务进程，一般也只能访问到它所需要的资源。即使程序出了漏洞，影响范围也只有在其允许访问的资源范围内。安全性大大增加。

  

这种权限管理机制的主体是进程，也称为强制访问控制（MAC）。

  

而 MAC 又细分为了两种方式，一种叫类别安全（MCS）模式，另一种叫多级安全（MLS）模式。

  

下文中的操作均为 MCS 模式。

  

**4 DAC 和 MAC 的对比**

这里引用一张图片来说明。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36cc4e3257814439ba62c13bb69d0379~tplv-k3u1fbpfcp-zoom-1.image)

  

  

  

可以看到，在 DAC 模式下，只要相应目录有相应用户的权限，就可以被访问。而在 MAC 模式下，还要受进程允许访问目录范围的限制。

  

**四、SELinux 基本操作**

**1 查询文件或目录的安全上下文**

命令基本用法

  

ls -Z能显示文件系统的安全上下文

ps -Z能显示进程的安全上下文

id -Z能显示shell的安全上下文：joe：usr_r：usr_t

  

用法举例

  

查询 /etc/hosts 的安全上下文。

  

ls -Z /etc/hosts

  

执行结果

  

```
[root@localhost ~]# ls -Z /etc/hosts
-rw-r--r--. root root system_u:object_r:net_conf_t:s0  /etc/hosts
[root@localhost ~]# 
```

  

  

**2 查询进程的安全上下文**

命令基本用法

  

ps auxZ | grep -v grep | grep <进程名>

用法举例

  

查询 Nginx 相关进程的安全上下文。

  

ps auxZ | grep -v grep | grep sshd

执行结果

  

  

```
[root@localhost ~]# 
[root@localhost ~]# ps auxZ | grep -v grep | grep sshd
system_u:system_r:sshd_t:s0-s0:c0.c1023 root 1454 0.0  0.0 112940 4324 ?        Ss   Sep03   0:00 /usr/sbin/sshd -D
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 root 11664 0.0  0.0 158944 5596 ? Ss 10:34   0:00 sshd: root@pts/0
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 root 11668 0.0  0.0 156812 5444 ? Ss 10:34   0:00 sshd: root@notty
[root@localhost ~]#


```

  

**3 手动修改文件或目录的安全上下文**

命令基本用法

  

chcon <选项> <文件或目录 1> \[<文件或目录 2>...\]

  

用法举例

  

修改 test 的安全上下文为 system_u:object_r:httpd_sys_content_t:s0。

  

```
chcon -u system_u -r object_r -t httpd_sys_content_t html2/*



[root@localhost nginx]# ls -Z
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 html
drwxr-xr-x. root root unconfined_u:object_r:usr_t:s0   html2
[root@localhost nginx]# ls -Z html2
-rw-r--r--. root root unconfined_u:object_r:usr_t:s0   404.html
-rw-r--r--. root root unconfined_u:object_r:usr_t:s0   50x.html
-rwxr-xr-x. root root unconfined_u:object_r:usr_t:s0   index.html
-rw-r--r--. root root unconfined_u:object_r:usr_t:s0   nginx-logo.png
-rw-r--r--. root root unconfined_u:object_r:usr_t:s0   poweredby.png

[root@localhost nginx]# chcon -u system_u -r object_r -t httpd_sys_content_t html2/*
[root@localhost nginx]# ls -Z html2
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 404.html
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 50x.html
-rwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 index.html
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 nginx-logo.png
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 poweredby.png
[root@localhost nginx]# 


```

  

**4 把文件或目录的安全上下文恢复到默认值**

命令基本用法

  

restorecon \[选项\] <文件或目录 1> \[<文件或目录 2>...\]

  

用法举例

  

添加一些网页文件到 Nginx 服务器的目录之后，为这些新文件设置正确的安全上下文。

  

```
[root@localhost ~]# 
[root@localhost ~]# restorecon -R /root/test/
[root@localhost ~]# 
[root@localhost ~]# ls -Z 
-rw-------. root root system_u:object_r:admin_home_t:s0 anaconda-ks.cfg
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 test
[root@localhost ~]# 


```

  

  

**5 查询系统中的布尔型规则及其状态**

命令基本用法

  

getsebool -a

由于该命令要么查询所有规则，要么只查询一个规则，所以一般都是先查询所有规则然后用 grep 筛选。

  

用法举例

  

查询与 httpd 有关的布尔型规则。

  

getsebool -a | grep ssh

执行结果

  

```
[root@localhost ~]# 
[root@localhost ~]# getsebool -a | grep ssh
fenced_can_ssh --> off
selinuxuser_use_ssh_chroot --> off
ssh_chroot_rw_homedirs --> off
ssh_keysign --> off
ssh_sysadm_login --> off
[root@localhost ~]#


```

  

**6 开关一个布尔型规则**  

命令基本用法

  

setsebool \[选项\] <规则名称> <on|off>

  

用法举例

  

开启 httpd_anon_write 规则。

  

setsebool -P httpd_anon_write on

  

执行结果

  

```
[root@localhost ~]#  getsebool -a | grep ssh
fenced_can_ssh --> off
selinuxuser_use_ssh_chroot --> off
ssh_chroot_rw_homedirs --> off
ssh_keysign --> off
ssh_sysadm_login --> off
[root@localhost ~]# 
[root@localhost ~]# 
```

  

  

修改布尔型规则

  

```
[root@localhost ~]# setsebool -P ssh_sysadm_login on

[root@localhost ~]# 
[root@localhost ~]#  getsebool -a | grep ssh
fenced_can_ssh --> off
selinuxuser_use_ssh_chroot --> off
ssh_chroot_rw_homedirs --> off
ssh_keysign --> off
ssh_sysadm_login --> on
[root@localhost ~]# 
[root@localhost ~]# setsebool -P ssh_sysadm_login off
[root@localhost ~]# 
[root@localhost ~]#  getsebool -a | grep ssh
fenced_can_ssh --> off
selinuxuser_use_ssh_chroot --> off
ssh_chroot_rw_homedirs --> off
ssh_keysign --> off
ssh_sysadm_login --> off
[root@localhost ~]# 


```

  

配置文件目录即文件内容

  

```
[root@localhost booleans]# pwd
/sys/fs/selinux/booleans
[root@localhost booleans]# cat mpd_use_cifs
0 0
```

  

**7 添加目录的默认安全上下文**

命令基本用法

  

（如果提示找不到命令的话请安装 `policycoreutils-python` 软件包，下同。）

  

semanage fcontext -a -t <文件安全上下文中的类型字段> "<目录（后面不加斜杠）>(/.\*)?"

注：目录或文件的默认安全上下文可以通过 semanage fcontext -l 命令配合 grep 过滤查看。

用法举例

  

为 Nginx 新增一个网站目录 /usr/share/nginx/html2 之后，需要为其设置与原目录相同的默认安全上下文。

  

semanage fcontext -a -t httpd_sys_content_t " /usr/share/nginx/html2(/.\*)?"

  

  

**8 添加某类进程允许访问的端口**

命令基本用法

  

semanage port -a -t <服务类型> -p <协议> <端口号>

注：各种服务类型所允许的端口号可以通过 semanage port -l 命令配合 grep 过滤查看。

用法举例

  

为 Nginx 需要使用 10080 的端口用于 HTTP 服务。

  

semanage port -a -t http_port_t -p tcp 10080

  

**9 参考其它进行修改**

命令基本用法

  

chcon --reference=<源文件>  <要修改的文件>

  

修改 1.txt文件

  

```
[root@localhost html]# ll -Z
-rw-r--r--. root root unconfined_u:object_r:usr_t:s0   1.txt
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 404.html
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 50x.html
lrwxrwxrwx. root root system_u:object_r:httpd_sys_content_t:s0 en-US -> ../../doc/HTML/en-US
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 icons
lrwxrwxrwx. root root system_u:object_r:httpd_sys_content_t:s0 img -> ../../doc/HTML/img
lrwxrwxrwx. root root system_u:object_r:httpd_sys_content_t:s0 index.html -> ../../doc/HTML/index.h
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 nginx-logo.png
lrwxrwxrwx. root root system_u:object_r:httpd_sys_content_t:s0 poweredby.png -> nginx-logo.png
```

  

执行结果

  

```
[root@localhost html]# chcon --reference=404.html 1.txt 
[root@localhost html]# 
[root@localhost html]# 
[root@localhost html]# ll -Z
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 1.txt
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 404.html
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 50x.html
lrwxrwxrwx. root root system_u:object_r:httpd_sys_content_t:s0 en-US -> ../../doc/HTML/en-US
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 icons
lrwxrwxrwx. root root system_u:object_r:httpd_sys_content_t:s0 img -> ../../doc/HTML/img
lrwxrwxrwx. root root system_u:object_r:httpd_sys_content_t:s0 index.html -> ../../doc/HTML/index.html
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 nginx-logo.png
lrwxrwxrwx. root root system_u:object_r:httpd_sys_content_t:s0 poweredby.png -> nginx-logo.png
```

  

  

**10 修改后的权限，恢复到原始的**

  

命令基本用法

  

restorecon -v <文件>

  

```
[root@localhost files]# ls -Z /etc/yum.conf 
-rw-r--r--. root root system_u:object_r:etc_t:s0       /etc/yum.conf
[root@localhost files]# 
[root@localhost files]# 
[root@localhost files]# chcon -t httpd_config_t /etc/yum.conf
[root@localhost files]# 
[root@localhost files]# ls -Z /etc/yum.conf 
-rw-r--r--. root root system_u:object_r:httpd_config_t:s0 /etc/yum.conf
[root@localhost files]# 
```

  

  

执行结果

  

```
[root@localhost files]# restorecon -v /etc/yum.conf 
restorecon reset /etc/yum.conf context system_u:object_r:httpd_config_t:s0->system_u:object_r:etc_t:s0
[root@localhost files]# ls -Z /etc/yum.conf 
-rw-r--r--. root root system_u:object_r:etc_t:s0       /etc/yum.conf
[root@localhost files]# 
```

  

  

  

  

**11 查看身份角色类似**

  

```
[root@localhost ~]# yum install setools-console



[root@localhost ~]# seinfo [选项]
选项：
-u： 列出SELinux中所有的身份（user）；
-r： 列出SELinux中所有的角色（role）；
-t： 列出SELinux中所有的类型（type）；
-b： 列出所有的布尔值（也就是策略中的具体规则名称）；
-x： 显示更多的信息；

```

  

  

**五、SELinux 错误分析和解决**

**1 认识 SELinux 日志**

当开启了 SELinux 之后，很多服务的一些正常行为都会被视为违规行为（标题及下文中的错误均指违规行为）。

  

这时候我们就需要借助 SELinux 违规日志来分析解决。

  

SELinux 违规日志保存在 /var/log/audit/audit.log 中。

  

/var/log/audit/audit.log 的内容大概是这样的。

  

```
...

[root@localhost ~]# tailf /var/log/audit/audit.log
type=GRP_MGMT msg=audit(1630901844.207:878): pid=11979 uid=0 auid=0 ses=76 subj=unconfined_u:unconfined_r:groupadd_t:s0-s0:c0.c1023 msg='op=add-shadow-group id=994 exe="/usr/sbin/groupadd" hostname=? addr=? terminal=? res=success'
type=ADD_USER msg=audit(1630901844.247:879): pid=11984 uid=0 auid=0 ses=76 subj=unconfined_u:unconfined_r:useradd_t:s0-s0:c0.c1023 msg='op=add-user id=997 exe="/usr/sbin/useradd" hostname=? addr=? terminal=? res=success'
type=USER_MGMT msg=audit(1630901844.288:880): pid=11989 uid=0 auid=0 ses=76 subj=unconfined_u:unconfined_r:useradd_t:s0-s0:c0.c1023 msg='op=pam_tally2 reset=0 id=997 exe="/usr/sbin/pam_tally2" hostname=? addr=? terminal=? res=success'
type=SOFTWARE_UPDATE msg=audit(1630901844.314:881): pid=11968 uid=0 auid=0 ses=76 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='sw="nginx-filesystem-1:1.20.1-2.el7.noarch" sw_type=rpm key_enforce=0 gpg_res=1 root_dir="/" comm="yum" exe="/usr/bin/python2.7" hostname=localhost.localdomain addr=? terminal=pts/0 res=success'
type=SOFTWARE_UPDATE msg=audit(1630901844.581:882): pid=11968 uid=0 auid=0 ses=76 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='sw="nginx-1:1.20.1-2.el7.x86_64" sw_type=rpm key_enforce=0 gpg_res=1 root_dir="/" comm="yum" exe="/usr/bin/python2.7" hostname=localhost.localdomain addr=? terminal=pts/0 res=success'
type=USER_AVC msg=audit(1630904068.840:883): pid=1053 uid=81 auid=4294967295 ses=4294967295 subj=system_u:system_r:system_dbusd_t:s0-s0:c0.c1023 msg='avc:  received policyload notice (seqno=7)  exe="/usr/bin/dbus-daemon" sauid=81 hostname=? addr=? terminal=?'
type=MAC_POLICY_LOAD msg=audit(1630904065.495:884): policy loaded auid=0 ses=76
type=SYSCALL msg=audit(1630904065.495:884): arch=c000003e syscall=1 success=yes exit=3881672 a0=4 a1=7f3cf9ca2000 a2=3b3ac8 a3=7ffd6fe646a0 items=0 ppid=12014 pid=12019 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=76 comm="load_policy" exe="/usr/sbin/load_policy" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=(null)
type=PROCTITLE msg=audit(1630904065.495:884): proctitle="/sbin/load_policy"
type=USER_MAC_CONFIG_CHANGE msg=audit(1630904068.901:885): pid=12014 uid=0 auid=0 ses=76 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='resrc=port op=add lport=10080 proto=6 tcontext=system_u:object_r:http_port_t:s0 comm="semanage" exe="/usr/bin/python2.7" hostname=? addr=? terminal=? res=success'
```

  

该文件的内容很多，而且混有很多与 SELinux 错误无关的系统审计日志。我们要借助 sealert 这个实用工具来帮忙分析（如果提示找不到命令的话请安装 setroubleshoot 软件包）。

  

**2 使用 sealert 分析错误**

命令基本用法

  

sealert -a /var/log/audit/audit.log

执行完命令之后，系统需要花一段时间去分析日志中的违规行为并给出分析报告。分析报告的结构讲解请看下图：

  

```
[root@localhost ~]# yum install setroubleshoot-server python3-pydbus
```

  

**3  SELinux 错误的思路**

  

当发现一个服务出现错误的时候，请先检查一下 sealert 的分析报告里面是否有该服务进程名称的关键字。如果没有，说明不是 SELinux 造成的错误，请往其他方面排查。

  

文件目录默认的权限在这个文件下，具体文件是file_contexts

/etc/selinux/targeted/contexts/files/

  

  

接下来就是阅读 sealert 的分析报告了。

  

首先需要了解的就是违规原因。如果违规原因里面出现了该服务不该访问的文件或资源，那就要小心了。有可能是服务的配置有问题或者服务本身存在漏洞，请优先排查服务的配置文件。

  

在分析报告中，对于一个违规行为，通常会有 2~3 个解决方案。请优先选择修改布尔值、设置默认安全上下文之类操作简单、容易理解的解决方案。

  

如果没有这种操作简单、容易理解的解决方案，请先去搜索引擎搜索一下有没有其他更好的解决方案。

  

需要注意的是，可信度只是一个参考值，并不代表使用可信度最高的解决方案就一定可以解决问题。我个人感觉使用可信度越高的解决方案对系统的改动就越小。

  

不过请记住，在执行解决方案之前，一定要先搞清楚解决方案中的命令是干什么的！

  

最后，请谨慎使用 audit2allow 这个命令。这个命令的作用非常简单粗暴，就是强制允许所遇到的错误然后封装成一个 SELinux 模块，接着让 SELinux 加载这个模块来达到消除错误的目的。不是万不得已建议不要随便使用 audit2allow。

  

```