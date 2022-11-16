---
layout: post
cid: 10
title: YUM下载全量依赖
slug: 10
date: 2021/12/30 17:00:46
updated: 2021/12/30 17:00:46
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


    在离线的内网环境下进行安装一些软件的时候会出现依赖不完整的情况，一般情况下会使用如下方式进行下载依赖包  

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/942f25d7509a404f9b6e5f7cbc194fd4~tplv-k3u1fbpfcp-zoom-1.image)

  

  

**查看依赖包可以使用 yum deplist 进行查找**

  

```
[root@localhost ~]# yum deplist nginx
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * epel: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
package: nginx.x86_64 1:1.20.1-2.el7
  dependency: /bin/sh
   provider: bash.x86_64 4.2.46-34.el7
  dependency: libc.so.6(GLIBC_2.17)(64bit)
   provider: glibc.x86_64 2.17-324.el7_9
  dependency: libcrypt.so.1()(64bit)
   provider: glibc.x86_64 2.17-324.el7_9
  dependency: libcrypt.so.1(GLIBC_2.2.5)(64bit)
   provider: glibc.x86_64 2.17-324.el7_9
  dependency: libcrypto.so.1.1()(64bit)
   provider: openssl11-libs.x86_64 1:1.1.1g-3.el7
  dependency: libcrypto.so.1.1(OPENSSL_1_1_0)(64bit)
   provider: openssl11-libs.x86_64 1:1.1.1g-3.el7
  dependency: libdl.so.2()(64bit)
   provider: glibc.x86_64 2.17-324.el7_9
  dependency: libdl.so.2(GLIBC_2.2.5)(64bit)
   provider: glibc.x86_64 2.17-324.el7_9
  dependency: libpcre.so.1()(64bit)
   provider: pcre.x86_64 8.32-17.el7
  dependency: libprofiler.so.0()(64bit)
   provider: gperftools-libs.x86_64 2.6.1-1.el7
  dependency: libpthread.so.0()(64bit)
   provider: glibc.x86_64 2.17-324.el7_9
  dependency: libpthread.so.0(GLIBC_2.2.5)(64bit)
   provider: glibc.x86_64 2.17-324.el7_9
  dependency: libpthread.so.0(GLIBC_2.3.2)(64bit)
   provider: glibc.x86_64 2.17-324.el7_9
  dependency: libssl.so.1.1()(64bit)
   provider: openssl11-libs.x86_64 1:1.1.1g-3.el7
  dependency: libssl.so.1.1(OPENSSL_1_1_0)(64bit)
   provider: openssl11-libs.x86_64 1:1.1.1g-3.el7
  dependency: libssl.so.1.1(OPENSSL_1_1_1)(64bit)
   provider: openssl11-libs.x86_64 1:1.1.1g-3.el7
  dependency: libz.so.1()(64bit)
   provider: zlib.x86_64 1.2.7-19.el7_9
  dependency: nginx-filesystem
   provider: nginx-filesystem.noarch 1:1.20.1-2.el7
  dependency: nginx-filesystem = 1:1.20.1-2.el7
   provider: nginx-filesystem.noarch 1:1.20.1-2.el7
  dependency: openssl
   provider: openssl.x86_64 1:1.0.2k-21.el7_9
  dependency: pcre
   provider: pcre.x86_64 8.32-17.el7
   provider: pcre.i686 8.32-17.el7
  dependency: redhat-indexhtml
   provider: centos-indexhtml.noarch 7-9.el7.centos
  dependency: rtld(GNU_HASH)
   provider: glibc.x86_64 2.17-324.el7_9
   provider: glibc.i686 2.17-324.el7_9
  dependency: system-logos
   provider: centos-logos.noarch 70.0.6-3.el7.centos
  dependency: systemd
   provider: systemd.x86_64 219-78.el7_9.3
[root@localhost ~]#

```

  

**使用 repotrack 命令进行下载所需依赖**

  

```
[root@localhost ~]# yum -y install yum-utils
[root@localhost ~]# repotrack  nginx
Downloading acl-2.2.51-15.el7.x86_64.rpm
Downloading audit-libs-2.8.5-4.el7.x86_64.rpm
Downloading audit-libs-2.8.5-4.el7.i686.rpm
Downloading basesystem-10.0-7.el7.centos.noarch.rpm
Downloading bash-4.2.46-34.el7.x86_64.rpm
Downloading binutils-2.27-44.base.el7.x86_64.rpm
Downloading bzip2-libs-1.0.6-13.el7.x86_64.rpm
Downloading bzip2-libs-1.0.6-13.el7.i686.rpm
Downloading ca-certificates-2020.2.41-70.0.el7_8.noarch.rpm
Downloading centos-indexhtml-7-9.el7.centos.noarch.rpm
Downloading centos-logos-70.0.6-3.el7.centos.noarch.rpm
Downloading centos-release-7-9.2009.1.el7.centos.x86_64.rpm
Downloading chkconfig-1.7.6-1.el7.x86_64.rpm
Downloading coreutils-8.22-24.el7_9.2.x86_64.rpm
Downloading cpio-2.11-28.el7.x86_64.rpm
Downloading cracklib-2.9.0-11.el7.x86_64.rpm
Downloading cracklib-2.9.0-11.el7.i686.rpm
Downloading cracklib-dicts-2.9.0-11.el7.x86_64.rpm
Downloading cryptsetup-libs-2.0.3-6.el7.x86_64.rpm
Downloading curl-7.29.0-59.el7_9.1.x86_64.rpm
Downloading cyrus-sasl-lib-2.1.26-23.el7.x86_64.rpm
Downloading cyrus-sasl-lib-2.1.26-23.el7.i686.rpm
Downloading dbus-1.10.24-15.el7.x86_64.rpm
Downloading dbus-libs-1.10.24-15.el7.x86_64.rpm
Downloading device-mapper-1.02.170-6.el7_9.5.x86_64.rpm
Downloading device-mapper-libs-1.02.170-6.el7_9.5.i686.rpm
Downloading device-mapper-libs-1.02.170-6.el7_9.5.x86_64.rpm
Downloading diffutils-3.3-5.el7.i686.rpm
Downloading diffutils-3.3-5.el7.x86_64.rpm
Downloading dracut-033-572.el7.x86_64.rpm
Downloading elfutils-default-yama-scope-0.176-5.el7.noarch.rpm
Downloading elfutils-libelf-0.176-5.el7.x86_64.rpm
Downloading elfutils-libelf-0.176-5.el7.i686.rpm
Downloading elfutils-libs-0.176-5.el7.x86_64.rpm
Downloading elfutils-libs-0.176-5.el7.i686.rpm
Downloading expat-2.1.0-12.el7.x86_64.rpm
Downloading filesystem-3.2-25.el7.x86_64.rpm
Downloading findutils-4.5.11-6.el7.x86_64.rpm
Downloading gawk-4.0.2-4.el7_3.1.x86_64.rpm
Downloading glib2-2.56.1-9.el7_9.i686.rpm
Downloading glib2-2.56.1-9.el7_9.x86_64.rpm
Downloading glibc-2.17-324.el7_9.i686.rpm
Downloading glibc-2.17-324.el7_9.x86_64.rpm
Downloading glibc-common-2.17-324.el7_9.x86_64.rpm
Downloading gmp-6.0.0-15.el7.i686.rpm
Downloading gmp-6.0.0-15.el7.x86_64.rpm
Downloading gperftools-libs-2.6.1-1.el7.x86_64.rpm
Downloading grep-2.20-3.el7.x86_64.rpm
Downloading gzip-1.5-10.el7.x86_64.rpm
Downloading hardlink-1.0-19.el7.x86_64.rpm
Downloading info-5.1-5.el7.x86_64.rpm
Downloading json-c-0.11-4.el7_0.x86_64.rpm
Downloading keyutils-libs-1.5.8-3.el7.i686.rpm
Downloading keyutils-libs-1.5.8-3.el7.x86_64.rpm
Downloading kmod-20-28.el7.x86_64.rpm
Downloading kmod-libs-20-28.el7.x86_64.rpm
Downloading kpartx-0.4.9-134.el7_9.x86_64.rpm
Downloading krb5-libs-1.15.1-50.el7.i686.rpm
Downloading krb5-libs-1.15.1-50.el7.x86_64.rpm
Downloading libacl-2.2.51-15.el7.x86_64.rpm
Downloading libacl-2.2.51-15.el7.i686.rpm
Downloading libattr-2.4.46-13.el7.i686.rpm
Downloading libattr-2.4.46-13.el7.x86_64.rpm
Downloading libblkid-2.23.2-65.el7_9.1.i686.rpm
Downloading libblkid-2.23.2-65.el7_9.1.x86_64.rpm
Downloading libcap-2.22-11.el7.x86_64.rpm
Downloading libcap-2.22-11.el7.i686.rpm
Downloading libcap-ng-0.7.5-4.el7.i686.rpm
Downloading libcap-ng-0.7.5-4.el7.x86_64.rpm
Downloading libcom_err-1.42.9-19.el7.x86_64.rpm
Downloading libcom_err-1.42.9-19.el7.i686.rpm
Downloading libcurl-7.29.0-59.el7_9.1.i686.rpm
Downloading libcurl-7.29.0-59.el7_9.1.x86_64.rpm
Downloading libdb-5.3.21-25.el7.i686.rpm
Downloading libdb-5.3.21-25.el7.x86_64.rpm
Downloading libdb-utils-5.3.21-25.el7.x86_64.rpm
Downloading libffi-3.0.13-19.el7.i686.rpm
Downloading libffi-3.0.13-19.el7.x86_64.rpm
Downloading libgcc-4.8.5-44.el7.x86_64.rpm
Downloading libgcc-4.8.5-44.el7.i686.rpm
Downloading libgcrypt-1.5.3-14.el7.x86_64.rpm
Downloading libgcrypt-1.5.3-14.el7.i686.rpm
Downloading libgpg-error-1.12-3.el7.i686.rpm
Downloading libgpg-error-1.12-3.el7.x86_64.rpm
Downloading libidn-1.28-4.el7.i686.rpm
Downloading libidn-1.28-4.el7.x86_64.rpm
Downloading libmount-2.23.2-65.el7_9.1.i686.rpm
Downloading libmount-2.23.2-65.el7_9.1.x86_64.rpm
Downloading libpwquality-1.2.3-5.el7.i686.rpm
Downloading libpwquality-1.2.3-5.el7.x86_64.rpm
Downloading libselinux-2.5-15.el7.x86_64.rpm
Downloading libselinux-2.5-15.el7.i686.rpm
Downloading libsemanage-2.5-14.el7.x86_64.rpm
Downloading libsepol-2.5-10.el7.i686.rpm
Downloading libsepol-2.5-10.el7.x86_64.rpm
Downloading libsmartcols-2.23.2-65.el7_9.1.i686.rpm
Downloading libsmartcols-2.23.2-65.el7_9.1.x86_64.rpm
Downloading libssh2-1.8.0-4.el7.x86_64.rpm
Downloading libssh2-1.8.0-4.el7.i686.rpm
Downloading libstdc++-4.8.5-44.el7.x86_64.rpm
Downloading libstdc++-4.8.5-44.el7.i686.rpm
Downloading libtasn1-4.10-1.el7.i686.rpm
Downloading libtasn1-4.10-1.el7.x86_64.rpm
Downloading libuser-0.60-9.el7.x86_64.rpm
Downloading libuser-0.60-9.el7.i686.rpm
Downloading libutempter-1.1.6-4.el7.x86_64.rpm
Downloading libutempter-1.1.6-4.el7.i686.rpm
Downloading libuuid-2.23.2-65.el7_9.1.x86_64.rpm
Downloading libuuid-2.23.2-65.el7_9.1.i686.rpm
Downloading libverto-0.2.5-4.el7.i686.rpm
Downloading libverto-0.2.5-4.el7.x86_64.rpm
Downloading libxml2-2.9.1-6.el7.5.x86_64.rpm
Downloading lua-5.1.4-15.el7.x86_64.rpm
Downloading lz4-1.8.3-1.el7.x86_64.rpm
Downloading lz4-1.8.3-1.el7.i686.rpm
Downloading make-3.82-24.el7.x86_64.rpm
Downloading ncurses-5.9-14.20130511.el7_4.x86_64.rpm
Downloading ncurses-base-5.9-14.20130511.el7_4.noarch.rpm
Downloading ncurses-libs-5.9-14.20130511.el7_4.x86_64.rpm
Downloading ncurses-libs-5.9-14.20130511.el7_4.i686.rpm
Downloading nginx-1.20.1-2.el7.x86_64.rpm
Downloading nginx-filesystem-1.20.1-2.el7.noarch.rpm
Downloading nspr-4.25.0-2.el7_9.x86_64.rpm
Downloading nspr-4.25.0-2.el7_9.i686.rpm
Downloading nss-3.53.1-7.el7_9.x86_64.rpm
Downloading nss-3.53.1-7.el7_9.i686.rpm
Downloading nss-pem-1.0.3-7.el7.x86_64.rpm
Downloading nss-pem-1.0.3-7.el7.i686.rpm
Downloading nss-softokn-3.53.1-6.el7_9.x86_64.rpm
Downloading nss-softokn-3.53.1-6.el7_9.i686.rpm
Downloading nss-softokn-freebl-3.53.1-6.el7_9.i686.rpm
Downloading nss-softokn-freebl-3.53.1-6.el7_9.x86_64.rpm
Downloading nss-sysinit-3.53.1-7.el7_9.x86_64.rpm
Downloading nss-tools-3.53.1-7.el7_9.x86_64.rpm
Downloading nss-util-3.53.1-1.el7_9.i686.rpm
Downloading nss-util-3.53.1-1.el7_9.x86_64.rpm
Downloading openldap-2.4.44-23.el7_9.i686.rpm
Downloading openldap-2.4.44-23.el7_9.x86_64.rpm
Downloading openssl-1.0.2k-21.el7_9.x86_64.rpm
Downloading openssl-libs-1.0.2k-21.el7_9.x86_64.rpm
Downloading openssl-libs-1.0.2k-21.el7_9.i686.rpm
Downloading openssl11-libs-1.1.1g-3.el7.x86_64.rpm
Downloading p11-kit-0.23.5-3.el7.i686.rpm
Downloading p11-kit-0.23.5-3.el7.x86_64.rpm
Downloading p11-kit-trust-0.23.5-3.el7.i686.rpm
Downloading p11-kit-trust-0.23.5-3.el7.x86_64.rpm
Downloading pam-1.1.8-23.el7.x86_64.rpm
Downloading pam-1.1.8-23.el7.i686.rpm
Downloading pcre-8.32-17.el7.i686.rpm
Downloading pcre-8.32-17.el7.x86_64.rpm
Downloading pkgconfig-0.27.1-4.el7.x86_64.rpm
Downloading pkgconfig-0.27.1-4.el7.i686.rpm
Downloading popt-1.13-16.el7.i686.rpm
Downloading popt-1.13-16.el7.x86_64.rpm
Downloading procps-ng-3.3.10-28.el7.x86_64.rpm
Downloading procps-ng-3.3.10-28.el7.i686.rpm
Downloading qrencode-libs-3.4.1-3.el7.x86_64.rpm
Downloading readline-6.2-11.el7.i686.rpm
Downloading readline-6.2-11.el7.x86_64.rpm
Downloading rpm-4.11.3-45.el7.x86_64.rpm
Downloading rpm-libs-4.11.3-45.el7.x86_64.rpm
Downloading sed-4.2.2-7.el7.x86_64.rpm
Downloading setup-2.8.71-11.el7.noarch.rpm
Downloading shadow-utils-4.6-5.el7.x86_64.rpm
Downloading shared-mime-info-1.8-5.el7.x86_64.rpm
Downloading sqlite-3.7.17-8.el7_7.1.i686.rpm
Downloading sqlite-3.7.17-8.el7_7.1.x86_64.rpm
Downloading systemd-219-78.el7_9.3.x86_64.rpm
Downloading systemd-libs-219-78.el7_9.3.x86_64.rpm
Downloading systemd-libs-219-78.el7_9.3.i686.rpm
Downloading tar-1.26-35.el7.x86_64.rpm
Downloading tzdata-2021a-1.el7.noarch.rpm
Downloading ustr-1.0.4-16.el7.x86_64.rpm
Downloading util-linux-2.23.2-65.el7_9.1.x86_64.rpm
Downloading util-linux-2.23.2-65.el7_9.1.i686.rpm
Downloading xz-5.2.2-1.el7.x86_64.rpm
Downloading xz-libs-5.2.2-1.el7.x86_64.rpm
Downloading xz-libs-5.2.2-1.el7.i686.rpm
Downloading zlib-1.2.7-19.el7_9.x86_64.rpm
Downloading zlib-1.2.7-19.el7_9.i686.rpm
[root@localhost ~]# ll
total 114432
-rw-r--r--. 1 root root    83408 Apr  4  2020 acl-2.2.51-15.el7.x86_64.rpm
-rw-------. 1 root root     1201 Jun 25 09:37 anaconda-ks.cfg
-rw-r--r--. 1 root root   104824 Aug 23  2019 audit-libs-2.8.5-4.el7.i686.rpm
-rw-r--r--. 1 root root   104408 Aug 23  2019 audit-libs-2.8.5-4.el7.x86_64.rpm
-rw-r--r--. 1 root root     5124 Jul  4  2014 basesystem-10.0-7.el7.centos.noarch.rpm
-rw-r--r--. 1 root root  1037976 Apr  4  2020 bash-4.2.46-34.el7.x86_64.rpm
-rw-r--r--. 1 root root  6196400 Oct 15  2020 binutils-2.27-44.base.el7.x86_64.rpm
-rw-r--r--. 1 root root    40620 Nov 25  2015 bzip2-libs-1.0.6-13.el7.i686.rpm
-rw-r--r--. 1 root root    40740 Nov 25  2015 bzip2-libs-1.0.6-13.el7.x86_64.rpm
-rw-r--r--. 1 root root   391340 Jun 24  2020 ca-certificates-2020.2.41-70.0.el7_8.noarch.rpm
-rw-r--r--. 1 root root    93872 Jul  4  2014 centos-indexhtml-7-9.el7.centos.noarch.rpm
-rw-r--r--. 1 root root 22354804 Oct  1  2015 centos-logos-70.0.6-3.el7.centos.noarch.rpm
-rw-r--r--. 1 root root    27288 Dec  3  2020 centos-release-7-9.2009.1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root   186016 Oct 15  2020 chkconfig-1.7.6-1.el7.x86_64.rpm
-rw-r--r--. 1 root root  3417472 Nov 18  2020 coreutils-8.22-24.el7_9.2.x86_64.rpm
-rw-r--r--. 1 root root   216500 Oct 15  2020 cpio-2.11-28.el7.x86_64.rpm
-rw-r--r--. 1 root root    80952 Jul  4  2014 cracklib-2.9.0-11.el7.i686.rpm
-rw-r--r--. 1 root root    81964 Jul  4  2014 cracklib-2.9.0-11.el7.x86_64.rpm
-rw-r--r--. 1 root root  3751124 Jul  4  2014 cracklib-dicts-2.9.0-11.el7.x86_64.rpm
-rw-r--r--. 1 root root   346748 Apr  4  2020 cryptsetup-libs-2.0.3-6.el7.x86_64.rpm
-rw-r--r--. 1 root root   277288 Nov 18  2020 curl-7.29.0-59.el7_9.1.x86_64.rpm
-rw-r--r--. 1 root root   158124 Apr 25  2018 cyrus-sasl-lib-2.1.26-23.el7.i686.rpm
-rw-r--r--. 1 root root   159156 Apr 25  2018 cyrus-sasl-lib-2.1.26-23.el7.x86_64.rpm
-rw-r--r--. 1 root root   251300 Oct 15  2020 dbus-1.10.24-15.el7.x86_64.rpm
-rw-r--r--. 1 root root   173428 Oct 15  2020 dbus-libs-1.10.24-15.el7.x86_64.rpm
-rw-r--r--. 1 root root   304544 Apr 29 23:03 device-mapper-1.02.170-6.el7_9.5.x86_64.rpm
-rw-r--r--. 1 root root   335628 Apr 29 23:04 device-mapper-libs-1.02.170-6.el7_9.5.i686.rpm
-rw-r--r--. 1 root root   333248 Apr 29 23:03 device-mapper-libs-1.02.170-6.el7_9.5.x86_64.rpm
-rw-r--r--. 1 root root   327892 Aug 23  2019 diffutils-3.3-5.el7.i686.rpm
-rw-r--r--. 1 root root   329696 Aug 23  2019 diffutils-3.3-5.el7.x86_64.rpm
-rw-r--r--. 1 root root   337240 Oct 15  2020 dracut-033-572.el7.x86_64.rpm
-rw-r--r--. 1 root root    33680 Oct 15  2020 elfutils-default-yama-scope-0.176-5.el7.noarch.rpm
-rw-r--r--. 1 root root   204904 Oct 15  2020 elfutils-libelf-0.176-5.el7.i686.rpm
-rw-r--r--. 1 root root   199352 Oct 15  2020 elfutils-libelf-0.176-5.el7.x86_64.rpm
-rw-r--r--. 1 root root   325648 Oct 15  2020 elfutils-libs-0.176-5.el7.i686.rpm
-rw-r--r--. 1 root root   297844 Oct 15  2020 elfutils-libs-0.176-5.el7.x86_64.rpm
-rw-r--r--. 1 root root    82628 Oct 15  2020 expat-2.1.0-12.el7.x86_64.rpm
-rw-r--r--. 1 root root  1067124 Apr 25  2018 filesystem-3.2-25.el7.x86_64.rpm
-rw-r--r--. 1 root root   572216 Nov 12  2018 findutils-4.5.11-6.el7.x86_64.rpm
-rw-r--r--. 1 root root   894476 Jun 29  2017 gawk-4.0.2-4.el7_3.1.x86_64.rpm
-rw-r--r--. 1 root root  2554540 Jun 11 23:07 glib2-2.56.1-9.el7_9.i686.rpm
-rw-r--r--. 1 root root  2571788 Jun 11 23:04 glib2-2.56.1-9.el7_9.x86_64.rpm
-rw-r--r--. 1 root root  4465908 Apr 29 23:05 glibc-2.17-324.el7_9.i686.rpm
-rw-r--r--. 1 root root  3817176 Apr 29 23:03 glibc-2.17-324.el7_9.x86_64.rpm
-rw-r--r--. 1 root root 12059644 Apr 29 23:03 glibc-common-2.17-324.el7_9.x86_64.rpm
-rw-r--r--. 1 root root   425588 Aug 11  2017 gmp-6.0.0-15.el7.i686.rpm
-rw-r--r--. 1 root root   287768 Aug 11  2017 gmp-6.0.0-15.el7.x86_64.rpm
-rw-r--r--. 1 root root   278636 Apr 25  2018 gperftools-libs-2.6.1-1.el7.x86_64.rpm
-rw-r--r--. 1 root root   352624 Aug 11  2017 grep-2.20-3.el7.x86_64.rpm
-rw-r--r--. 1 root root   132636 Apr 25  2018 gzip-1.5-10.el7.x86_64.rpm
-rw-r--r--. 1 root root    14640 Jul  4  2014 hardlink-1.0-19.el7.x86_64.rpm
-rw-r--r--. 1 root root   238564 Apr 25  2018 info-5.1-5.el7.x86_64.rpm
-rw-r--r--. 1 root root    31312 Jul  5  2014 json-c-0.11-4.el7_0.x86_64.rpm
-rw-r--r--. 1 root root    25852 Jul  4  2014 keyutils-libs-1.5.8-3.el7.i686.rpm
-rw-r--r--. 1 root root    25920 Jul  4  2014 keyutils-libs-1.5.8-3.el7.x86_64.rpm
-rw-r--r--. 1 root root   125760 Apr  4  2020 kmod-20-28.el7.x86_64.rpm
-rw-r--r--. 1 root root    52412 Apr  4  2020 kmod-libs-20-28.el7.x86_64.rpm
-rw-r--r--. 1 root root    82532 Nov 18  2020 kpartx-0.4.9-134.el7_9.x86_64.rpm
-rw-r--r--. 1 root root   830508 Oct 15  2020 krb5-libs-1.15.1-50.el7.i686.rpm
-rw-r--r--. 1 root root   828540 Oct 15  2020 krb5-libs-1.15.1-50.el7.x86_64.rpm
-rw-r--r--. 1 root root    28280 Apr  4  2020 libacl-2.2.51-15.el7.i686.rpm
-rw-r--r--. 1 root root    27976 Apr  4  2020 libacl-2.2.51-15.el7.x86_64.rpm
-rw-r--r--. 1 root root    18632 Apr 25  2018 libattr-2.4.46-13.el7.i686.rpm
-rw-r--r--. 1 root root    18656 Apr 25  2018 libattr-2.4.46-13.el7.x86_64.rpm
-rw-r--r--. 1 root root   191636 Feb  4  2021 libblkid-2.23.2-65.el7_9.1.i686.rpm
-rw-r--r--. 1 root root   187272 Feb  4  2021 libblkid-2.23.2-65.el7_9.1.x86_64.rpm
-rw-r--r--. 1 root root    48904 Apr  4  2020 libcap-2.22-11.el7.i686.rpm
-rw-r--r--. 1 root root    48548 Apr  4  2020 libcap-2.22-11.el7.x86_64.rpm
-rw-r--r--. 1 root root    24976 Nov 25  2015 libcap-ng-0.7.5-4.el7.i686.rpm
-rw-r--r--. 1 root root    25244 Nov 25  2015 libcap-ng-0.7.5-4.el7.x86_64.rpm
-rw-r--r--. 1 root root    43132 Oct 15  2020 libcom_err-1.42.9-19.el7.i686.rpm
-rw-r--r--. 1 root root    43092 Oct 15  2020 libcom_err-1.42.9-19.el7.x86_64.rpm
-rw-r--r--. 1 root root   231516 Nov 18  2020 libcurl-7.29.0-59.el7_9.1.i686.rpm
-rw-r--r--. 1 root root   228648 Nov 18  2020 libcurl-7.29.0-59.el7_9.1.x86_64.rpm
-rw-r--r--. 1 root root   748832 Aug 23  2019 libdb-5.3.21-25.el7.i686.rpm
-rw-r--r--. 1 root root   737156 Aug 23  2019 libdb-5.3.21-25.el7.x86_64.rpm
-rw-r--r--. 1 root root   135576 Aug 23  2019 libdb-utils-5.3.21-25.el7.x86_64.rpm
-rw-r--r--. 1 root root    28144 Apr  4  2020 libffi-3.0.13-19.el7.i686.rpm
-rw-r--r--. 1 root root    30960 Apr  4  2020 libffi-3.0.13-19.el7.x86_64.rpm
-rw-r--r--. 1 root root   113236 Oct 15  2020 libgcc-4.8.5-44.el7.i686.rpm
-rw-r--r--. 1 root root   105308 Oct 15  2020 libgcc-4.8.5-44.el7.x86_64.rpm
-rw-r--r--. 1 root root   272044 Aug 11  2017 libgcrypt-1.5.3-14.el7.i686.rpm
-rw-r--r--. 1 root root   269660 Aug 11  2017 libgcrypt-1.5.3-14.el7.x86_64.rpm
-rw-r--r--. 1 root root    89068 Jul  4  2014 libgpg-error-1.12-3.el7.i686.rpm
-rw-r--r--. 1 root root    89332 Jul  4  2014 libgpg-error-1.12-3.el7.x86_64.rpm
-rw-r--r--. 1 root root   213888 Nov 25  2015 libidn-1.28-4.el7.i686.rpm
-rw-r--r--. 1 root root   213816 Nov 25  2015 libidn-1.28-4.el7.x86_64.rpm
-rw-r--r--. 1 root root   188712 Feb  4  2021 libmount-2.23.2-65.el7_9.1.i686.rpm
-rw-r--r--. 1 root root   189228 Feb  4  2021 libmount-2.23.2-65.el7_9.1.x86_64.rpm
-rw-r--r--. 1 root root    87100 Apr 25  2018 libpwquality-1.2.3-5.el7.i686.rpm
-rw-r--r--. 1 root root    86540 Apr 25  2018 libpwquality-1.2.3-5.el7.x86_64.rpm
-rw-r--r--. 1 root root   169856 Apr  4  2020 libselinux-2.5-15.el7.i686.rpm
-rw-r--r--. 1 root root   166012 Apr  4  2020 libselinux-2.5-15.el7.x86_64.rpm
-rw-r--r--. 1 root root   154244 Nov 12  2018 libsemanage-2.5-14.el7.x86_64.rpm
-rw-r--r--. 1 root root   301460 Nov 12  2018 libsepol-2.5-10.el7.i686.rpm
-rw-r--r--. 1 root root   304196 Nov 12  2018 libsepol-2.5-10.el7.x86_64.rpm
-rw-r--r--. 1 root root   147288 Feb  4  2021 libsmartcols-2.23.2-65.el7_9.1.i686.rpm
-rw-r--r--. 1 root root   146164 Feb  4  2021 libsmartcols-2.23.2-65.el7_9.1.x86_64.rpm
-rw-r--r--. 1 root root    90480 Oct 15  2020 libssh2-1.8.0-4.el7.i686.rpm
-rw-r--r--. 1 root root    89984 Oct 15  2020 libssh2-1.8.0-4.el7.x86_64.rpm
-rw-r--r--. 1 root root   326568 Oct 15  2020 libstdc++-4.8.5-44.el7.i686.rpm
-rw-r--r--. 1 root root   313196 Oct 15  2020 libstdc++-4.8.5-44.el7.x86_64.rpm
-rw-r--r--. 1 root root   328116 Aug 11  2017 libtasn1-4.10-1.el7.i686.rpm
-rw-r--r--. 1 root root   328028 Aug 11  2017 libtasn1-4.10-1.el7.x86_64.rpm
-rw-r--r--. 1 root root   405604 Apr 25  2018 libuser-0.60-9.el7.i686.rpm
-rw-r--r--. 1 root root   409732 Apr 25  2018 libuser-0.60-9.el7.x86_64.rpm
-rw-r--r--. 1 root root    25576 Jul  4  2014 libutempter-1.1.6-4.el7.i686.rpm
-rw-r--r--. 1 root root    25428 Jul  4  2014 libutempter-1.1.6-4.el7.x86_64.rpm
-rw-r--r--. 1 root root    87036 Feb  4  2021 libuuid-2.23.2-65.el7_9.1.i686.rpm
-rw-r--r--. 1 root root    86332 Feb  4  2021 libuuid-2.23.2-65.el7_9.1.x86_64.rpm
-rw-r--r--. 1 root root    16728 Jul  4  2014 libverto-0.2.5-4.el7.i686.rpm
-rw-r--r--. 1 root root    16820 Jul  4  2014 libverto-0.2.5-4.el7.x86_64.rpm
-rw-r--r--. 1 root root   684200 Oct 15  2020 libxml2-2.9.1-6.el7.5.x86_64.rpm
-rw-r--r--. 1 root root   205412 Nov 21  2016 lua-5.1.4-15.el7.x86_64.rpm
-rw-r--r--. 1 root root    98452 Oct 15  2020 lz4-1.8.3-1.el7.i686.rpm
-rw-r--r--. 1 root root    86572 Oct 15  2020 lz4-1.8.3-1.el7.x86_64.rpm
-rw-r--r--. 1 root root   430712 Aug 23  2019 make-3.82-24.el7.x86_64.rpm
-rw-r--r--. 1 root root   310928 Sep  7  2017 ncurses-5.9-14.20130511.el7_4.x86_64.rpm
-rw-r--r--. 1 root root    69900 Sep  7  2017 ncurses-base-5.9-14.20130511.el7_4.noarch.rpm
-rw-r--r--. 1 root root   323976 Sep  7  2017 ncurses-libs-5.9-14.20130511.el7_4.i686.rpm
-rw-r--r--. 1 root root   323192 Sep  7  2017 ncurses-libs-5.9-14.20130511.el7_4.x86_64.rpm
-rw-r--r--. 1 root root   600265 Jun  2 08:27 nginx-1.20.1-2.el7.x86_64.rpm
-rw-r--r--. 1 root root    23333 Jun  2 08:27 nginx-filesystem-1.20.1-2.el7.noarch.rpm
-rw-r--r--. 1 root root   131612 Oct 15  2020 nspr-4.25.0-2.el7_9.i686.rpm
-rw-r--r--. 1 root root   129900 Oct 15  2020 nspr-4.25.0-2.el7_9.x86_64.rpm
-rw-r--r--. 1 root root   890208 Apr 29 23:05 nss-3.53.1-7.el7_9.i686.rpm
-rw-r--r--. 1 root root   889384 Apr 29 23:04 nss-3.53.1-7.el7_9.x86_64.rpm
-rw-r--r--. 1 root root    74872 Aug 23  2019 nss-pem-1.0.3-7.el7.i686.rpm
-rw-r--r--. 1 root root    75584 Aug 23  2019 nss-pem-1.0.3-7.el7.x86_64.rpm
-rw-r--r--. 1 root root   370104 Nov  7  2020 nss-softokn-3.53.1-6.el7_9.i686.rpm
-rw-r--r--. 1 root root   362588 Nov  7  2020 nss-softokn-3.53.1-6.el7_9.x86_64.rpm
-rw-r--r--. 1 root root   329232 Nov  7  2020 nss-softokn-freebl-3.53.1-6.el7_9.i686.rpm
-rw-r--r--. 1 root root   329744 Nov  7  2020 nss-softokn-freebl-3.53.1-6.el7_9.x86_64.rpm
-rw-r--r--. 1 root root    67104 Apr 29 23:04 nss-sysinit-3.53.1-7.el7_9.x86_64.rpm
-rw-r--r--. 1 root root   547892 Apr 29 23:04 nss-tools-3.53.1-7.el7_9.x86_64.rpm
-rw-r--r--. 1 root root    79396 Oct 15  2020 nss-util-3.53.1-1.el7_9.i686.rpm
-rw-r--r--. 1 root root    80948 Oct 15  2020 nss-util-3.53.1-1.el7_9.x86_64.rpm
-rw-r--r--. 1 root root   363460 Apr 29 23:05 openldap-2.4.44-23.el7_9.i686.rpm
-rw-r--r--. 1 root root   364488 Apr 29 23:04 openldap-2.4.44-23.el7_9.x86_64.rpm
-rw-r--r--. 1 root root   505208 Dec 18  2020 openssl-1.0.2k-21.el7_9.x86_64.rpm
-rw-r--r--. 1 root root  1521105 Mar 30 07:56 openssl11-libs-1.1.1g-3.el7.x86_64.rpm
-rw-r--r--. 1 root root  1020608 Dec 18  2020 openssl-libs-1.0.2k-21.el7_9.i686.rpm
-rw-r--r--. 1 root root  1254644 Dec 18  2020 openssl-libs-1.0.2k-21.el7_9.x86_64.rpm
-rw-r--r--. 1 root root   247076 Aug 11  2017 p11-kit-0.23.5-3.el7.i686.rpm
-rw-r--r--. 1 root root   257620 Aug 11  2017 p11-kit-0.23.5-3.el7.x86_64.rpm
-rw-r--r--. 1 root root   131504 Aug 11  2017 p11-kit-trust-0.23.5-3.el7.i686.rpm
-rw-r--r--. 1 root root   131984 Aug 11  2017 p11-kit-trust-0.23.5-3.el7.x86_64.rpm
-rw-r--r--. 1 root root   736976 Apr  4  2020 pam-1.1.8-23.el7.i686.rpm
-rw-r--r--. 1 root root   737960 Apr  4  2020 pam-1.1.8-23.el7.x86_64.rpm
-rw-r--r--. 1 root root   430428 Aug 11  2017 pcre-8.32-17.el7.i686.rpm
-rw-r--r--. 1 root root   432020 Aug 11  2017 pcre-8.32-17.el7.x86_64.rpm
-rw-r--r--. 1 root root    54276 Jul  4  2014 pkgconfig-0.27.1-4.el7.i686.rpm
-rw-r--r--. 1 root root    54928 Jul  4  2014 pkgconfig-0.27.1-4.el7.x86_64.rpm
-rw-r--r--. 1 root root    42436 Jul  4  2014 popt-1.13-16.el7.i686.rpm
-rw-r--r--. 1 root root    42740 Jul  4  2014 popt-1.13-16.el7.x86_64.rpm
-rw-r--r--. 1 root root   292948 Oct 15  2020 procps-ng-3.3.10-28.el7.i686.rpm
-rw-r--r--. 1 root root   298092 Oct 15  2020 procps-ng-3.3.10-28.el7.x86_64.rpm
-rw-r--r--. 1 root root    51112 Jul  4  2014 qrencode-libs-3.4.1-3.el7.x86_64.rpm
-rw-r--r--. 1 root root   194000 Aug 23  2019 readline-6.2-11.el7.i686.rpm
-rw-r--r--. 1 root root   197696 Aug 23  2019 readline-6.2-11.el7.x86_64.rpm
-rw-r--r--. 1 root root  1219860 Oct 15  2020 rpm-4.11.3-45.el7.x86_64.rpm
-rw-r--r--. 1 root root   285088 Oct 15  2020 rpm-libs-4.11.3-45.el7.x86_64.rpm
-rw-r--r--. 1 root root   236688 Oct 15  2020 sed-4.2.2-7.el7.x86_64.rpm
-rw-r--r--. 1 root root   170000 Apr  4  2020 setup-2.8.71-11.el7.noarch.rpm
-rw-r--r--. 1 root root  1250180 Aug 23  2019 shadow-utils-4.6-5.el7.x86_64.rpm
-rw-r--r--. 1 root root   319576 Apr  4  2020 shared-mime-info-1.8-5.el7.x86_64.rpm
-rw-r--r--. 1 root root   406104 Jan 29  2020 sqlite-3.7.17-8.el7_7.1.i686.rpm
-rw-r--r--. 1 root root   403100 Jan 29  2020 sqlite-3.7.17-8.el7_7.1.x86_64.rpm
-rw-r--r--. 1 root root  5324844 Feb  4  2021 systemd-219-78.el7_9.3.x86_64.rpm
-rw-r--r--. 1 root root   435292 Feb  4  2021 systemd-libs-219-78.el7_9.3.i686.rpm
-rw-r--r--. 1 root root   428468 Feb  4  2021 systemd-libs-219-78.el7_9.3.x86_64.rpm
-rw-r--r--. 1 root root   865848 Nov 12  2018 tar-1.26-35.el7.x86_64.rpm
-rw-r--r--. 1 root root   513088 Jan 27  2021 tzdata-2021a-1.el7.noarch.rpm
-rw-r--r--. 1 root root    94456 Jul  4  2014 ustr-1.0.4-16.el7.x86_64.rpm
-rw-r--r--. 1 root root  2104080 Feb  4  2021 util-linux-2.23.2-65.el7_9.1.i686.rpm
-rw-r--r--. 1 root root  2076012 Feb  4  2021 util-linux-2.23.2-65.el7_9.1.x86_64.rpm
-rw-r--r--. 1 root root   234160 Nov 21  2016 xz-5.2.2-1.el7.x86_64.rpm
-rw-r--r--. 1 root root   111716 Nov 21  2016 xz-libs-5.2.2-1.el7.i686.rpm
-rw-r--r--. 1 root root   105728 Nov 21  2016 xz-libs-5.2.2-1.el7.x86_64.rpm
-rw-r--r--. 1 root root    92968 Feb  4  2021 zlib-1.2.7-19.el7_9.i686.rpm
-rw-r--r--. 1 root root    92068 Feb  4  2021 zlib-1.2.7-19.el7_9.x86_64.rpm
[root@localhost ~]#

```

  

**使用 yumdownloader 命令下软件依赖**

  

```
[root@localhost ~]# yumdownloader --resolve --destdir=/tmp ansible
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * epel: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
--> Running transaction check
---> Package ansible.noarch 0:2.9.24-2.el7 will be installed
--> Processing Dependency: PyYAML for package: ansible-2.9.24-2.el7.noarch
--> Processing Dependency: python-httplib2 for package: ansible-2.9.24-2.el7.noarch
--> Processing Dependency: python-jinja2 for package: ansible-2.9.24-2.el7.noarch
--> Processing Dependency: python-paramiko for package: ansible-2.9.24-2.el7.noarch
--> Processing Dependency: python-setuptools for package: ansible-2.9.24-2.el7.noarch
--> Processing Dependency: python-six for package: ansible-2.9.24-2.el7.noarch
--> Processing Dependency: python2-cryptography for package: ansible-2.9.24-2.el7.noarch
--> Processing Dependency: python2-jmespath for package: ansible-2.9.24-2.el7.noarch
--> Processing Dependency: sshpass for package: ansible-2.9.24-2.el7.noarch
--> Running transaction check
---> Package PyYAML.x86_64 0:3.10-11.el7 will be installed
--> Processing Dependency: libyaml-0.so.2()(64bit) for package: PyYAML-3.10-11.el7.x86_64
---> Package python-jinja2.noarch 0:2.7.2-4.el7 will be installed
--> Processing Dependency: python-babel >= 0.8 for package: python-jinja2-2.7.2-4.el7.noarch
--> Processing Dependency: python-markupsafe for package: python-jinja2-2.7.2-4.el7.noarch
---> Package python-paramiko.noarch 0:2.1.1-9.el7 will be installed
--> Processing Dependency: python2-pyasn1 for package: python-paramiko-2.1.1-9.el7.noarch
---> Package python-setuptools.noarch 0:0.9.8-7.el7 will be installed
--> Processing Dependency: python-backports-ssl_match_hostname for package: python-setuptools-0.9.8-7.el7.noarch
---> Package python-six.noarch 0:1.9.0-2.el7 will be installed
---> Package python2-cryptography.x86_64 0:1.7.2-2.el7 will be installed
--> Processing Dependency: python-idna >= 2.0 for package: python2-cryptography-1.7.2-2.el7.x86_64
--> Processing Dependency: python-cffi >= 1.4.1 for package: python2-cryptography-1.7.2-2.el7.x86_64
--> Processing Dependency: python-ipaddress for package: python2-cryptography-1.7.2-2.el7.x86_64
--> Processing Dependency: python-enum34 for package: python2-cryptography-1.7.2-2.el7.x86_64
---> Package python2-httplib2.noarch 0:0.18.1-3.el7 will be installed
---> Package python2-jmespath.noarch 0:0.9.4-2.el7 will be installed
---> Package sshpass.x86_64 0:1.06-2.el7 will be installed
--> Running transaction check
---> Package libyaml.x86_64 0:0.1.4-11.el7_0 will be installed
---> Package python-babel.noarch 0:0.9.6-8.el7 will be installed
---> Package python-backports-ssl_match_hostname.noarch 0:3.5.0.1-1.el7 will be installed
--> Processing Dependency: python-backports for package: python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch
---> Package python-cffi.x86_64 0:1.6.0-5.el7 will be installed
--> Processing Dependency: python-pycparser for package: python-cffi-1.6.0-5.el7.x86_64
---> Package python-enum34.noarch 0:1.0.4-1.el7 will be installed
---> Package python-idna.noarch 0:2.4-1.el7 will be installed
---> Package python-ipaddress.noarch 0:1.0.16-2.el7 will be installed
---> Package python-markupsafe.x86_64 0:0.11-10.el7 will be installed
---> Package python2-pyasn1.noarch 0:0.1.9-7.el7 will be installed
--> Running transaction check
---> Package python-backports.x86_64 0:1.0-8.el7 will be installed
---> Package python-pycparser.noarch 0:2.14-1.el7 will be installed
--> Processing Dependency: python-ply for package: python-pycparser-2.14-1.el7.noarch
--> Running transaction check
---> Package python-ply.noarch 0:3.4-11.el7 will be installed
--> Finished Dependency Resolution
(1/22): PyYAML-3.10-11.el7.x86_64.rpm                                               | 153 kB  00:00:00     
(2/22): python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch.rpm                |  13 kB  00:00:00     
(3/22): libyaml-0.1.4-11.el7_0.x86_64.rpm                                           |  55 kB  00:00:00     
(4/22): python-cffi-1.6.0-5.el7.x86_64.rpm                                          | 218 kB  00:00:00     
(5/22): python-backports-1.0-8.el7.x86_64.rpm                                       | 5.8 kB  00:00:00     
(6/22): python-idna-2.4-1.el7.noarch.rpm                                            |  94 kB  00:00:00     
(7/22): python-enum34-1.0.4-1.el7.noarch.rpm                                        |  52 kB  00:00:00     
(8/22): python-jinja2-2.7.2-4.el7.noarch.rpm                                        | 519 kB  00:00:00     
(9/22): python-babel-0.9.6-8.el7.noarch.rpm                                         | 1.4 MB  00:00:00     
(10/22): python-paramiko-2.1.1-9.el7.noarch.rpm                                     | 269 kB  00:00:00     
(11/22): python-ply-3.4-11.el7.noarch.rpm                                           | 123 kB  00:00:00     
(12/22): python-pycparser-2.14-1.el7.noarch.rpm                                     | 104 kB  00:00:00     
(13/22): python-six-1.9.0-2.el7.noarch.rpm                                          |  29 kB  00:00:00     
(14/22): python-setuptools-0.9.8-7.el7.noarch.rpm                                   | 397 kB  00:00:00     
(15/22): python2-cryptography-1.7.2-2.el7.x86_64.rpm                                | 502 kB  00:00:00     
(16/22): python-markupsafe-0.11-10.el7.x86_64.rpm                                   |  25 kB  00:00:00     
(17/22): python-ipaddress-1.0.16-2.el7.noarch.rpm                                   |  34 kB  00:00:00     
(18/22): ansible-2.9.24-2.el7.noarch.rpm                                            |  17 MB  00:00:01     
(19/22): python2-httplib2-0.18.1-3.el7.noarch.rpm                                   | 125 kB  00:00:00     
(20/22): python2-jmespath-0.9.4-2.el7.noarch.rpm                                    |  41 kB  00:00:00     
(21/22): sshpass-1.06-2.el7.x86_64.rpm                                              |  21 kB  00:00:00     
(22/22): python2-pyasn1-0.1.9-7.el7.noarch.rpm                                      | 100 kB  00:00:00     
[root@localhost ~]# 
[root@localhost ~]# 
[root@localhost ~]# ll /tmp/*
-rw-r--r--. 1 root root 17829351 Jul 29 02:38 /tmp/ansible-2.9.24-2.el7.noarch.rpm
-rw-r--r--. 1 root root    56068 Jan 30  2015 /tmp/libyaml-0.1.4-11.el7_0.x86_64.rpm
-rw-r--r--. 1 root root   514504 Apr 25  2018 /tmp/python2-cryptography-1.7.2-2.el7.x86_64.rpm
-rw-r--r--. 1 root root   128003 Jun 20  2020 /tmp/python2-httplib2-0.18.1-3.el7.noarch.rpm
-rw-r--r--. 1 root root    42303 Apr 23  2020 /tmp/python2-jmespath-0.9.4-2.el7.noarch.rpm
-rw-r--r--. 1 root root   102132 Nov 21  2016 /tmp/python2-pyasn1-0.1.9-7.el7.noarch.rpm
-rw-r--r--. 1 root root  1426348 Jul  4  2014 /tmp/python-babel-0.9.6-8.el7.noarch.rpm
-rw-r--r--. 1 root root     5932 Mar 14  2015 /tmp/python-backports-1.0-8.el7.x86_64.rpm
-rw-r--r--. 1 root root    12896 Apr 25  2018 /tmp/python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch.rpm
-rw-r--r--. 1 root root   223012 Nov 21  2016 /tmp/python-cffi-1.6.0-5.el7.x86_64.rpm
-rw-r--r--. 1 root root    53496 Nov 25  2015 /tmp/python-enum34-1.0.4-1.el7.noarch.rpm
-rw-r--r--. 1 root root    95952 Aug 11  2017 /tmp/python-idna-2.4-1.el7.noarch.rpm
-rw-r--r--. 1 root root    35176 Nov 21  2016 /tmp/python-ipaddress-1.0.16-2.el7.noarch.rpm
-rw-r--r--. 1 root root   531040 Aug 23  2019 /tmp/python-jinja2-2.7.2-4.el7.noarch.rpm
-rw-r--r--. 1 root root    25792 Jul  4  2014 /tmp/python-markupsafe-0.11-10.el7.x86_64.rpm
-rw-r--r--. 1 root root   275112 Nov 21  2018 /tmp/python-paramiko-2.1.1-9.el7.noarch.rpm
-rw-r--r--. 1 root root   125732 Aug 11  2017 /tmp/python-ply-3.4-11.el7.noarch.rpm
-rw-r--r--. 1 root root   106984 Nov 25  2015 /tmp/python-pycparser-2.14-1.el7.noarch.rpm
-rw-r--r--. 1 root root   406404 Aug 11  2017 /tmp/python-setuptools-0.9.8-7.el7.noarch.rpm
-rw-r--r--. 1 root root    29404 Nov 25  2015 /tmp/python-six-1.9.0-2.el7.noarch.rpm
-rw-r--r--. 1 root root   156952 Jul  4  2014 /tmp/PyYAML-3.10-11.el7.x86_64.rpm
-rw-r--r--. 1 root root    21896 Sep  8  2017 /tmp/sshpass-1.06-2.el7.x86_64.rpm
[root@localhost ~]# 


destdir：指定 rpm 包下载目录（不指定时，默认为当前目录）
resolve：下载依赖的 rpm 包。
```

  

  

只会下载当前系统环境下所需的依赖包

  

**yum 自带的 downloadonly 插件**

```
[root@localhost ~]# yum -y install nginx --downloadonly --downloaddir=/tmp/
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * epel: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package nginx.x86_64 1:1.20.1-2.el7 will be installed
--> Processing Dependency: nginx-filesystem = 1:1.20.1-2.el7 for package: 1:nginx-1.20.1-2.el7.x86_64
--> Processing Dependency: libcrypto.so.1.1(OPENSSL_1_1_0)(64bit) for package: 1:nginx-1.20.1-2.el7.x86_64
--> Processing Dependency: libssl.so.1.1(OPENSSL_1_1_0)(64bit) for package: 1:nginx-1.20.1-2.el7.x86_64
--> Processing Dependency: libssl.so.1.1(OPENSSL_1_1_1)(64bit) for package: 1:nginx-1.20.1-2.el7.x86_64
--> Processing Dependency: nginx-filesystem for package: 1:nginx-1.20.1-2.el7.x86_64
--> Processing Dependency: redhat-indexhtml for package: 1:nginx-1.20.1-2.el7.x86_64
--> Processing Dependency: libcrypto.so.1.1()(64bit) for package: 1:nginx-1.20.1-2.el7.x86_64
--> Processing Dependency: libprofiler.so.0()(64bit) for package: 1:nginx-1.20.1-2.el7.x86_64
--> Processing Dependency: libssl.so.1.1()(64bit) for package: 1:nginx-1.20.1-2.el7.x86_64
--> Running transaction check
---> Package centos-indexhtml.noarch 0:7-9.el7.centos will be installed
---> Package gperftools-libs.x86_64 0:2.6.1-1.el7 will be installed
---> Package nginx-filesystem.noarch 1:1.20.1-2.el7 will be installed
---> Package openssl11-libs.x86_64 1:1.1.1g-3.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                              Arch                       Version                             Repository                Size
====================================================================================================================================
Installing:
 nginx                                x86_64                     1:1.20.1-2.el7                      epel                     586 k
Installing for dependencies:
 centos-indexhtml                     noarch                     7-9.el7.centos                      base                      92 k
 gperftools-libs                      x86_64                     2.6.1-1.el7                         base                     272 k
 nginx-filesystem                     noarch                     1:1.20.1-2.el7                      epel                      23 k
 openssl11-libs                       x86_64                     1:1.1.1g-3.el7                      epel                     1.5 M

Transaction Summary
====================================================================================================================================
Install  1 Package (+4 Dependent packages)

Total download size: 2.4 M
Installed size: 6.7 M
Background downloading packages, then exiting:
(1/5): centos-indexhtml-7-9.el7.centos.noarch.rpm                                                            |  92 kB  00:00:00     
(2/5): gperftools-libs-2.6.1-1.el7.x86_64.rpm                                                                | 272 kB  00:00:00     
(3/5): nginx-1.20.1-2.el7.x86_64.rpm                                                                         | 586 kB  00:00:00     
(4/5): nginx-filesystem-1.20.1-2.el7.noarch.rpm                                                              |  23 kB  00:00:00     
(5/5): openssl11-libs-1.1.1g-3.el7.x86_64.rpm                                                                | 1.5 MB  00:00:00     
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                               8.0 MB/s | 2.4 MB  00:00:00     
exiting because "Download Only" specified
[root@localhost ~]# ll /tmp/*
-rw-r--r--. 1 root root   93872 Jul  4  2014 /tmp/centos-indexhtml-7-9.el7.centos.noarch.rpm
-rw-r--r--. 1 root root  278636 Apr 25  2018 /tmp/gperftools-libs-2.6.1-1.el7.x86_64.rpm
-rw-r--r--. 1 root root  600265 Jun  2 08:27 /tmp/nginx-1.20.1-2.el7.x86_64.rpm
-rw-r--r--. 1 root root   23333 Jun  2 08:27 /tmp/nginx-filesystem-1.20.1-2.el7.noarch.rpm
-rw-r--r--. 1 root root 1521105 Mar 30 07:56 /tmp/openssl11-libs-1.1.1g-3.el7.x86_64.rpm
-rw-------. 1 root root    1183 Aug 16 11:24 /tmp/yum_save_tx.2021-08-16.11-24.riZifR.yumtx
[root@localhost ~]#

```

  

  

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b6d26bce8904f0b931e11b19bc0cd89~tplv-k3u1fbpfcp-zoom-1.image)

  

  

  

  

```