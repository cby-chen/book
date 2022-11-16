---
layout: post
cid: 167
title: 部署kubernetes官网博客
slug: 167
date: 2022/05/11 16:29:00
updated: 2022/05/12 10:25:41
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


部署kubernetes官网博客
================

访问 https://kubernetes.io/ 有些时候不问题，部署离线内网使用官网以及博客， 各位尝鲜可以访问 https://doc.oiox.cn/

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/706f068caadd43369590a9291605c3fe~tplv-k3u1fbpfcp-zoom-1.image)

安装docker
========

```
root@cby:~# curl -sSL https://get.daocloud.io/docker | sh
# Executing docker install script, commit: 0221adedb4bcde0f3d18bddda023544fc56c29d1
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
+ sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" | gpg --dearmor --yes -o /usr/share/keyrings/docker-archive-keyring.gpg
+ sh -c chmod a+r /usr/share/keyrings/docker-archive-keyring.gpg
+ sh -c echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu focal stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq --no-install-recommends docker-ce docker-ce-cli docker-compose-plugin docker-scan-plugin >/dev/null
+ version_gte 20.10
+ [ -z  ]
+ return 0
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq docker-ce-rootless-extras >/dev/null
+ sh -c docker version
Client: Docker Engine - Community
 Version:           20.10.15
 API version:       1.41
 Go version:        go1.17.9
 Git commit:        fd82621
 Built:             Thu May  5 13:19:23 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.15
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.17.9
  Git commit:       4433bf6
  Built:            Thu May  5 13:17:28 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.4
  GitCommit:        212e8b6fa2f44b9c21b2798135fc6fb7c53efc16
 runc:
  Version:          1.1.1
  GitCommit:        v1.1.1-0-g52de29d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

================================================================================

root@cby:~# 

```

克隆库
===

```
root@cby:~# git clone https://github.com/kubernetes/website.git
Cloning into 'website'...
remote: Enumerating objects: 269472, done.
remote: Counting objects: 100% (354/354), done.
remote: Compressing objects: 100% (240/240), done.
remote: Total 269472 (delta 201), reused 221 (delta 112), pack-reused 269118
Receiving objects: 100% (269472/269472), 334.98 MiB | 1.92 MiB/s, done.
Resolving deltas: 100% (190520/190520), done.
Updating files: 100% (7124/7124), done.
root@cby:~# cd website
root@cby:~/website# 

```

安装依赖
====

```
root@cby:~/website# git submodule update --init --recursive --depth 1
Submodule 'api-ref-generator' (https://github.com/kubernetes-sigs/reference-docs) registered for path 'api-ref-generator'
Submodule 'themes/docsy' (https://github.com/google/docsy.git) registered for path 'themes/docsy'
Cloning into '/root/website/api-ref-generator'...
Cloning into '/root/website/themes/docsy'...
remote: Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Enumerating objects: 104, done.
remote: Counting objects: 100% (104/104), done.
remote: Compressing objects: 100% (53/53), done.
remote: Total 61 (delta 34), reused 23 (delta 6), pack-reused 0
Unpacking objects: 100% (61/61), 103.64 KiB | 252.00 KiB/s, done.
From https://github.com/kubernetes-sigs/reference-docs
 * branch            55bce686224caba37f93e1e1eb53c0c9fc104ed4 -> FETCH_HEAD
Submodule path 'api-ref-generator': checked out '55bce686224caba37f93e1e1eb53c0c9fc104ed4'
Submodule 'themes/docsy' (https://github.com/google/docsy.git) registered for path 'api-ref-generator/themes/docsy'
Cloning into '/root/website/api-ref-generator/themes/docsy'...
remote: Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Enumerating objects: 251, done.
remote: Counting objects: 100% (251/251), done.
remote: Compressing objects: 100% (119/119), done.
remote: Total 130 (delta 82), reused 34 (delta 3), pack-reused 0
Receiving objects: 100% (130/130), 43.96 KiB | 308.00 KiB/s, done.
Resolving deltas: 100% (82/82), completed with 77 local objects.
From https://github.com/google/docsy
 * branch            6b30513dc837c5937de351f2fb2e4fedb04365c4 -> FETCH_HEAD
Submodule path 'api-ref-generator/themes/docsy': checked out '6b30513dc837c5937de351f2fb2e4fedb04365c4'
Submodule 'assets/vendor/Font-Awesome' (https://github.com/FortAwesome/Font-Awesome.git) registered for path 'api-ref-generator/themes/docsy/assets/vendor/Font-Awesome'
Submodule 'assets/vendor/bootstrap' (https://github.com/twbs/bootstrap.git) registered for path 'api-ref-generator/themes/docsy/assets/vendor/bootstrap'
Cloning into '/root/website/api-ref-generator/themes/docsy/assets/vendor/Font-Awesome'...
Cloning into '/root/website/api-ref-generator/themes/docsy/assets/vendor/bootstrap'...
remote: Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Enumerating objects: 8924, done.
remote: Counting objects: 100% (8921/8921), done.
remote: Compressing objects: 100% (2868/2868), done.
remote: Total 4847 (delta 3027), reused 2286 (delta 1978), pack-reused 0
Receiving objects: 100% (4847/4847), 5.77 MiB | 4.38 MiB/s, done.
Resolving deltas: 100% (3027/3027), completed with 884 local objects.
From https://github.com/FortAwesome/Font-Awesome
 * branch            fcec2d1b01ff069ac10500ac42e4478d20d21f4c -> FETCH_HEAD
Submodule path 'api-ref-generator/themes/docsy/assets/vendor/Font-Awesome': checked out 'fcec2d1b01ff069ac10500ac42e4478d20d21f4c'
remote: Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Enumerating objects: 701, done.
remote: Counting objects: 100% (701/701), done.
remote: Compressing objects: 100% (511/511), done.
remote: Total 528 (delta 115), reused 186 (delta 13), pack-reused 0
Receiving objects: 100% (528/528), 2.01 MiB | 5.52 MiB/s, done.
Resolving deltas: 100% (115/115), completed with 73 local objects.
From https://github.com/twbs/bootstrap
 * branch            a716fb03f965dc0846df479e14388b1b4b93d7ce -> FETCH_HEAD
Submodule path 'api-ref-generator/themes/docsy/assets/vendor/bootstrap': checked out 'a716fb03f965dc0846df479e14388b1b4b93d7ce'
remote: Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Enumerating objects: 76, done.
remote: Counting objects: 100% (76/76), done.
remote: Compressing objects: 100% (37/37), done.
remote: Total 39 (delta 30), reused 6 (delta 0), pack-reused 0
Unpacking objects: 100% (39/39), 4.48 KiB | 654.00 KiB/s, done.
From https://github.com/google/docsy
 * branch            1c77bb24483946f11c13f882f836a940b55ad019 -> FETCH_HEAD
Submodule path 'themes/docsy': checked out '1c77bb24483946f11c13f882f836a940b55ad019'
Submodule 'assets/vendor/Font-Awesome' (https://github.com/FortAwesome/Font-Awesome.git) registered for path 'themes/docsy/assets/vendor/Font-Awesome'
Submodule 'assets/vendor/bootstrap' (https://github.com/twbs/bootstrap.git) registered for path 'themes/docsy/assets/vendor/bootstrap'
Cloning into '/root/website/themes/docsy/assets/vendor/Font-Awesome'...
Cloning into '/root/website/themes/docsy/assets/vendor/bootstrap'...
remote: Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Enumerating objects: 8925, done.
remote: Counting objects: 100% (8922/8922), done.
remote: Compressing objects: 100% (2801/2801), done.
remote: Total 4848 (delta 3031), reused 2433 (delta 2046), pack-reused 0
Receiving objects: 100% (4848/4848), 5.65 MiB | 4.21 MiB/s, done.
Resolving deltas: 100% (3031/3031), completed with 855 local objects.
From https://github.com/FortAwesome/Font-Awesome
 * branch            7d3d774145ac38663f6d1effc6def0334b68ab7e -> FETCH_HEAD
Submodule path 'themes/docsy/assets/vendor/Font-Awesome': checked out '7d3d774145ac38663f6d1effc6def0334b68ab7e'
remote: Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Enumerating objects: 770, done.
remote: Counting objects: 100% (770/770), done.
remote: Compressing objects: 100% (497/497), done.
remote: Total 524 (delta 161), reused 183 (delta 19), pack-reused 0
Receiving objects: 100% (524/524), 2.01 MiB | 2.53 MiB/s, done.
Resolving deltas: 100% (161/161), completed with 122 local objects.
From https://github.com/twbs/bootstrap
 * branch            043a03c95a2ad6738f85b65e53b9dbdfb03b8d10 -> FETCH_HEAD
Submodule path 'themes/docsy/assets/vendor/bootstrap': checked out '043a03c95a2ad6738f85b65e53b9dbdfb03b8d10'
root@cby:~/website# 

```

构建镜像
====

```
root@cby:~/website# make container-image
docker build . \
    --network=host \
    --tag gcr.io/k8s-staging-sig-docs/k8s-website-hugo:v0.87.0-c8ffb2b5979c \
    --build-arg HUGO_VERSION=0.87.0
Sending build context to Docker daemon  4.096kB
Step 1/12 : FROM golang:1.16-alpine
1.16-alpine: Pulling from library/golang
59bf1c3509f3: Pull complete 
666ba61612fd: Pull complete 
8ed8ca486205: Pull complete 
ca4bf87e467a: Pull complete 
0435e0963794: Pull complete 
Digest: sha256:5616dca835fa90ef13a843824ba58394dad356b7d56198fb7c93cbe76d7d67fe
Status: Downloaded newer image for golang:1.16-alpine
 ---> 7642119cd161
Step 2/12 : LABEL maintainer="Luc Perkins <lperkins@linuxfoundation.org>"
 ---> Running in f6a8d1fa0c42
Removing intermediate container f6a8d1fa0c42
 ---> 291fd45ae748
Step 3/12 : RUN apk add --no-cache     curl     gcc     g++     musl-dev     build-base     libc6-compat
 ---> Running in 209e30a852d3
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/community/x86_64/APKINDEX.tar.gz
(1/25) Installing libgcc (10.3.1_git20211027-r0)
(2/25) Installing libstdc++ (10.3.1_git20211027-r0)
(3/25) Installing binutils (2.37-r3)
(4/25) Installing libmagic (5.41-r0)
(5/25) Installing file (5.41-r0)
(6/25) Installing libgomp (10.3.1_git20211027-r0)
(7/25) Installing libatomic (10.3.1_git20211027-r0)
(8/25) Installing libgphobos (10.3.1_git20211027-r0)
(9/25) Installing gmp (6.2.1-r1)
(10/25) Installing isl22 (0.22-r0)
(11/25) Installing mpfr4 (4.1.0-r0)
(12/25) Installing mpc1 (1.2.1-r0)
(13/25) Installing gcc (10.3.1_git20211027-r0)
(14/25) Installing musl-dev (1.2.2-r7)
(15/25) Installing libc-dev (0.7.2-r3)
(16/25) Installing g++ (10.3.1_git20211027-r0)
(17/25) Installing make (4.3-r0)
(18/25) Installing fortify-headers (1.1-r1)
(19/25) Installing patch (2.7.6-r7)
(20/25) Installing build-base (0.5-r2)
(21/25) Installing brotli-libs (1.0.9-r5)
(22/25) Installing nghttp2-libs (1.46.0-r0)
(23/25) Installing libcurl (7.80.0-r1)
(24/25) Installing curl (7.80.0-r1)
(25/25) Installing libc6-compat (1.2.2-r7)
Executing busybox-1.34.1-r3.trigger
OK: 198 MiB in 40 packages
Removing intermediate container 209e30a852d3
 ---> 83dfeba4ff34
Step 4/12 : ARG HUGO_VERSION
 ---> Running in fdbe162165c2
Removing intermediate container fdbe162165c2
 ---> d6219e970f50
Step 5/12 : RUN mkdir $HOME/src &&     cd $HOME/src &&     curl -L https://github.com/gohugoio/hugo/archive/refs/tags/v${HUGO_VERSION}.tar.gz | tar -xz &&     cd "hugo-${HUGO_VERSION}" &&     go install --tags extended
 ---> Running in fe0b26ed3841
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 35.2M    0 35.2M    0     0  2216k      0 --:--:--  0:00:16 --:--:-- 3037k
go: downloading github.com/alecthomas/chroma v0.9.2
go: downloading github.com/bep/debounce v1.2.0
go: downloading github.com/fsnotify/fsnotify v1.4.9
go: downloading github.com/pkg/errors v0.9.1
go: downloading github.com/spf13/afero v1.6.0
go: downloading github.com/spf13/cobra v1.2.1
go: downloading github.com/spf13/fsync v0.9.0
go: downloading github.com/spf13/jwalterweatherman v1.1.0
go: downloading github.com/spf13/pflag v1.0.5
go: downloading golang.org/x/sync v0.0.0-20210220032951-036812b2e83c
go: downloading github.com/pelletier/go-toml v1.9.3
go: downloading github.com/spf13/cast v1.4.0
go: downloading github.com/PuerkitoBio/purell v1.1.1
go: downloading github.com/gobwas/glob v0.2.3
go: downloading github.com/mattn/go-isatty v0.0.13
go: downloading github.com/mitchellh/mapstructure v1.4.1
go: downloading github.com/aws/aws-sdk-go v1.40.8
go: downloading github.com/dustin/go-humanize v1.0.0
go: downloading gocloud.dev v0.20.0
go: downloading github.com/pelletier/go-toml/v2 v2.0.0-beta.3.0.20210727221244-fa0796069526
go: downloading golang.org/x/text v0.3.6
go: downloading google.golang.org/api v0.51.0
go: downloading github.com/jdkato/prose v1.2.1
go: downloading github.com/kyokomi/emoji/v2 v2.2.8
go: downloading github.com/mitchellh/hashstructure v1.1.0
go: downloading github.com/olekukonko/tablewriter v0.0.5
go: downloading github.com/armon/go-radix v1.0.0
go: downloading github.com/gohugoio/locales v0.14.0
go: downloading github.com/gohugoio/localescompressed v0.14.0
go: downloading github.com/gorilla/websocket v1.4.2
go: downloading github.com/rogpeppe/go-internal v1.8.0
go: downloading gopkg.in/yaml.v2 v2.4.0
go: downloading github.com/niklasfasching/go-org v1.5.0
go: downloading github.com/bep/gitmap v1.1.2
go: downloading github.com/gobuffalo/flect v0.2.3
go: downloading golang.org/x/sys v0.0.0-20210630005230-0f9fa26af87c
go: downloading github.com/cpuguy83/go-md2man/v2 v2.0.0
go: downloading github.com/cli/safeexec v1.0.0
go: downloading github.com/dlclark/regexp2 v1.4.0
go: downloading github.com/BurntSushi/locker v0.0.0-20171006230638-a6e239ea1c69
go: downloading github.com/disintegration/gift v1.2.1
go: downloading golang.org/x/image v0.0.0-20210220032944-ac19c3e999fb
go: downloading github.com/PuerkitoBio/urlesc v0.0.0-20170810143723-de5bf2ad4578
go: downloading golang.org/x/net v0.0.0-20210614182718-04defd469f4e
go: downloading go.opencensus.io v0.23.0
go: downloading golang.org/x/xerrors v0.0.0-20200804184101-5ec99f83aff1
go: downloading github.com/Azure/azure-pipeline-go v0.2.2
go: downloading github.com/Azure/azure-storage-blob-go v0.9.0
go: downloading github.com/google/uuid v1.1.2
go: downloading github.com/google/wire v0.4.0
go: downloading cloud.google.com/go v0.87.0
go: downloading github.com/googleapis/gax-go v2.0.2+incompatible
go: downloading github.com/googleapis/gax-go/v2 v2.0.5
go: downloading cloud.google.com/go/storage v1.10.0
go: downloading golang.org/x/oauth2 v0.0.0-20210628180205-a41e5a781914
go: downloading google.golang.org/genproto v0.0.0-20210716133855-ce7ef5c701ea
go: downloading github.com/mattn/go-runewidth v0.0.9
go: downloading github.com/bep/tmc v0.5.1
go: downloading github.com/rwcarlsen/goexif v0.0.0-20190401172101-9e8deecbddbd
go: downloading github.com/gohugoio/go-i18n/v2 v2.1.3-0.20210430103248-4c28c89f8013
go: downloading github.com/russross/blackfriday v1.5.3-0.20200218234912-41c5fccfd6f6
go: downloading github.com/bep/gowebp v0.1.0
go: downloading github.com/muesli/smartcrop v0.3.0
go: downloading google.golang.org/grpc v1.39.0
go: downloading github.com/mattn/go-ieproxy v0.0.1
go: downloading github.com/russross/blackfriday/v2 v2.0.1
go: downloading google.golang.org/protobuf v1.27.1
go: downloading github.com/danwakefield/fnmatch v0.0.0-20160403171240-cbb64ac3d964
go: downloading github.com/yuin/goldmark v1.4.0
go: downloading github.com/yuin/goldmark-highlighting v0.0.0-20200307114337-60d527fdb691
go: downloading github.com/miekg/mmark v1.3.6
go: downloading github.com/tdewolff/minify/v2 v2.9.21
go: downloading github.com/sanity-io/litter v1.5.1
go: downloading github.com/getkin/kin-openapi v0.68.0
go: downloading github.com/ghodss/yaml v1.0.0
go: downloading github.com/golang/groupcache v0.0.0-20200121045136-8c9f03a8e57e
go: downloading github.com/shurcooL/sanitized_anchor_name v1.0.0
go: downloading github.com/jmespath/go-jmespath v0.4.0
go: downloading github.com/BurntSushi/toml v0.3.1
go: downloading github.com/evanw/esbuild v0.12.17
go: downloading github.com/tdewolff/parse/v2 v2.5.19
go: downloading github.com/bep/godartsass v0.12.0
go: downloading github.com/bep/golibsass v1.0.0
go: downloading github.com/golang/protobuf v1.5.2
go: downloading github.com/google/go-cmp v0.5.6
go: downloading github.com/go-openapi/jsonpointer v0.19.5
go: downloading github.com/go-openapi/swag v0.19.5
go: downloading github.com/mailru/easyjson v0.0.0-20190626092158-b2ccc519800e
Removing intermediate container fe0b26ed3841
 ---> 034cde1adc00
Step 6/12 : FROM golang:1.16-alpine
 ---> 7642119cd161
Step 7/12 : RUN apk add --no-cache     runuser     git     openssh-client     rsync     npm &&     npm install -D autoprefixer postcss-cli
 ---> Running in 2af5902e5287
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/community/x86_64/APKINDEX.tar.gz
(1/27) Installing brotli-libs (1.0.9-r5)
(2/27) Installing nghttp2-libs (1.46.0-r0)
(3/27) Installing libcurl (7.80.0-r1)
(4/27) Installing expat (2.4.7-r0)
(5/27) Installing pcre2 (10.39-r0)
(6/27) Installing git (2.34.2-r0)
(7/27) Installing c-ares (1.18.1-r0)
(8/27) Installing libgcc (10.3.1_git20211027-r0)
(9/27) Installing libstdc++ (10.3.1_git20211027-r0)
(10/27) Installing icu-libs (69.1-r1)
(11/27) Installing libuv (1.42.0-r0)
(12/27) Installing nodejs-current (17.9.0-r0)
(13/27) Installing npm (8.1.3-r0)
(14/27) Installing openssh-keygen (8.8_p1-r1)
(15/27) Installing ncurses-terminfo-base (6.3_p20211120-r0)
(16/27) Installing ncurses-libs (6.3_p20211120-r0)
(17/27) Installing libedit (20210910.3.1-r0)
(18/27) Installing openssh-client-common (8.8_p1-r1)
(19/27) Installing openssh-client-default (8.8_p1-r1)
(20/27) Installing libacl (2.2.53-r0)
(21/27) Installing lz4-libs (1.9.3-r1)
(22/27) Installing popt (1.18-r0)
(23/27) Installing zstd-libs (1.5.0-r0)
(24/27) Installing rsync (3.2.3-r5)
(25/27) Installing libeconf (0.4.2-r0)
(26/27) Installing linux-pam (1.5.2-r0)
(27/27) Installing runuser (2.37.4-r0)
Executing busybox-1.34.1-r3.trigger
OK: 106 MiB in 42 packages

added 73 packages, and audited 74 packages in 15s

17 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
Removing intermediate container 2af5902e5287
 ---> 620ef2580a98
Step 8/12 : RUN mkdir -p /var/hugo &&     addgroup -Sg 1000 hugo &&     adduser -Sg hugo -u 1000 -h /var/hugo hugo &&     chown -R hugo: /var/hugo &&     runuser -u hugo -- git config --global --add safe.directory /src
 ---> Running in dc169979de70
Removing intermediate container dc169979de70
 ---> 1006a4277115
Step 9/12 : COPY --from=0 /go/bin/hugo /usr/local/bin/hugo
 ---> 9bd8581cf0c3
Step 10/12 : WORKDIR /src
 ---> Running in 89fb367fe208
Removing intermediate container 89fb367fe208
 ---> b299d26f87a7
Step 11/12 : USER hugo:hugo
 ---> Running in 353a5aec3b6e
Removing intermediate container 353a5aec3b6e
 ---> ec88a8ce29a5
Step 12/12 : EXPOSE 1313
 ---> Running in 2649b06d597f
Removing intermediate container 2649b06d597f
 ---> 20b483234fde
Successfully built 20b483234fde
Successfully tagged gcr.io/k8s-staging-sig-docs/k8s-website-hugo:v0.87.0-c8ffb2b5979c
root@cby:~/website# 

```

构建容器
====

```
root@cby:~/website# make container-serve
docker run --rm --interactive --tty --volume /root/website:/src --cap-drop=ALL --cap-add=AUDIT_WRITE --read-only --mount type=tmpfs,destination=/tmp,tmpfs-mode=01777 -p 1313:1313 gcr.io/k8s-staging-sig-docs/k8s-website-hugo:v0.87.0-c8ffb2b5979c hugo server --buildFuture --environment development --bind 0.0.0.0 --destination /tmp/hugo --cleanDestinationDir
Start building sites … 
hugo v0.87.0+extended linux/amd64 BuildDate=unknown


----

                   |  EN  |  ZH  | KO  | JA  | FR  | IT  | DE  | ES  | PT-BR | ID  | RU  | VI  | PL  | UK   
-------------------+------+------+-----+-----+-----+-----+-----+-----+-------+-----+-----+-----+-----+------
  Pages            | 1453 | 1015 | 539 | 450 | 338 |  71 | 164 | 292 |   186 | 335 | 155 |  77 |  69 |  92  
  Paginator pages  |   43 |    9 |   0 |   0 |   0 |   0 |   0 |   0 |     0 |   0 |   0 |   0 |   0 |   0  
  Non-page files   |  509 |  386 | 200 | 266 |  73 |  20 |  17 |  33 |    30 | 105 |  24 |   8 |   6 |  20  
  Static files     |  838 |  838 | 838 | 838 | 838 | 838 | 838 | 838 |   838 | 838 | 838 | 838 | 838 | 838  
  Processed images |    1 |    1 |   0 |   0 |   0 |   0 |   0 |   0 |     0 |   0 |   0 |   0 |   0 |   0  
  Aliases          |    8 |    2 |   3 |   1 |   0 |   1 |   0 |   0 |     1 |   1 |   1 |   0 |   0 |   0  
  Sitemaps         |    2 |    1 |   1 |   1 |   1 |   1 |   1 |   1 |     1 |   1 |   1 |   1 |   1 |   1  
  Cleaned          |    0 |    0 |   0 |   0 |   0 |   0 |   0 |   0 |     0 |   0 |   0 |   0 |   0 |   0  

Built in 15926 ms
Watching for changes in /src/{archetypes,assets,content,data,i18n,layouts,package.json,postcss.config.js,static,themes}
Watching for config changes in /src/config.toml, /src/themes/docsy/config.toml, /src/go.mod
Environment: "development"
Serving pages from /tmp/hugo
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 0.0.0.0)
Press Ctrl+C to stop

```

后台启动
====

```
root@cby:~# docker images
REPOSITORY                                     TAG                    IMAGE ID       CREATED         SIZE
gcr.io/k8s-staging-sig-docs/k8s-website-hugo   v0.87.0-c8ffb2b5979c   20b483234fde   4 minutes ago   501MB
<none>                                         <none>                 034cde1adc00   4 minutes ago   1.8GB
golang                                         1.16-alpine            7642119cd161   2 months ago    302MB
root@cby:~#


root@cby:~/website# docker run --rm --interactive -d --volume /root/website:/src --cap-drop=ALL --cap-add=AUDIT_WRITE --read-only --mount type=tmpfs,destination=/tmp,tmpfs-mode=01777 -p 1313:1313 gcr.io/k8s-staging-sig-docs/k8s-website-hugo:v0.87.0-c8ffb2b5979c hugo server --buildFuture --environment development --bind 0.0.0.0 --destination /tmp/hugo --cleanDestinationDir


docker run --rm --interactive -d --volume /root/website:/src --cap-drop=ALL --cap-add=AUDIT_WRITE --read-only --mount type=tmpfs,destination=/tmp,tmpfs-mode=01777 -p 1313:1313 gcr.io/k8s-staging-sig-docs/k8s-website-hugo:v0.87.0-c8ffb2b5979c hugo server --buildFuture --environment development --bind 0.0.0.0 --destination /tmp/hugo --cleanDestinationDir

root@cby:~/website# docker ps
CONTAINER ID   IMAGE                                                               COMMAND                  CREATED         STATUS         PORTS                                       NAMES
06f34ad73c67   gcr.io/k8s-staging-sig-docs/k8s-website-hugo:v0.87.0-c8ffb2b5979c   "hugo server --build…"   5 seconds ago   Up 4 seconds   0.0.0.0:1313->1313/tcp, :::1313->1313/tcp   nervous_kilby
root@cby:~/website# 

```

更新文档
====

```
root@hello:~/website# git pull
remote: Enumerating objects: 187, done.
remote: Counting objects: 100% (181/181), done.
remote: Compressing objects: 100% (112/112), done.
remote: Total 187 (delta 107), reused 126 (delta 69), pack-reused 6
Receiving objects: 100% (187/187), 154.37 KiB | 403.00 KiB/s, done.
Resolving deltas: 100% (107/107), completed with 35 local objects.
From https://github.com/kubernetes/website
   f559e15074..07e1929b49  main          -> origin/main
   8c980f042b..68e621e794  dev-1.24-ko.1 -> origin/dev-1.24-ko.1
Updating f559e15074..07e1929b49
Fast-forward
 content/en/docs/concepts/cluster-administration/manage-deployment.md                             |   2 +-
 content/en/docs/concepts/containers/runtime-class.md                                             |   2 +-
 content/en/docs/concepts/workloads/pods/init-containers.md                                       |   1 -
 content/en/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_generate-csr.md            |   3 ---
 content/en/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_preflight.md          |   3 ---
 content/en/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_upload-certs.md       |   3 ---
 content/en/docs/reference/setup-tools/kubeadm/generated/kubeadm_join_phase_control-plane-join.md |   3 ---
 content/en/docs/reference/setup-tools/kubeadm/generated/kubeadm_token.md                         |   3 ---
 content/en/docs/reference/setup-tools/kubeadm/generated/kubeadm_token_create.md                  |   1 -
 content/en/docs/reference/setup-tools/kubeadm/generated/kubeadm_token_delete.md                  |   3 ---
 content/en/docs/reference/setup-tools/kubeadm/generated/kubeadm_version.md                       |   3 ---
 content/en/docs/setup/production-environment/windows/intro-windows-in-kubernetes.md              |   2 +-
 content/en/docs/tasks/administer-cluster/kubeadm/adding-windows-nodes.md                         |   2 +-
 content/en/docs/tasks/configure-pod-container/configure-pod-initialization.md                    |   1 -
 content/en/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume.md              |   2 +-
 content/pt-br/blog/_posts/2022-02-17-updated-dockershim-faq.md                                   |   2 +-
 content/zh/docs/concepts/architecture/nodes.md                                                   | 134 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++---------------
 content/zh/docs/concepts/cluster-administration/system-logs.md                                   | 117 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++---------------------------------------
 content/zh/docs/concepts/containers/runtime-class.md                                             |  62 +++++++++++++++++++++-----------------------------------------
 content/zh/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins.md                | 111 +++++++++++++++++++++------------------------------------------------------------------------------------------
 content/zh/docs/concepts/overview/kubernetes-api.md                                              |  71 +++++++++++++++++++++++++++++++++++++++++++++++++----------------------
 content/zh/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_generate-csr.md            |  26 ++++++++++++++++++++------
 content/zh/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_preflight.md          |  28 +++++++++++++++++++++++++---
 content/zh/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_upload-certs.md       |  30 +++++++++++++++++++++++++++++-
 content/zh/docs/reference/setup-tools/kubeadm/generated/kubeadm_join_phase_control-plane-join.md |  20 +++++++++++++++++++-
 content/zh/docs/reference/setup-tools/kubeadm/generated/kubeadm_token.md                         |  24 +++++++++++++++++++++++-
 content/zh/docs/reference/setup-tools/kubeadm/generated/kubeadm_token_create.md                  |  51 ++++++++++++++++++++++++++++++++++++++++++++++++++-
 content/zh/docs/reference/setup-tools/kubeadm/generated/kubeadm_token_delete.md                  |  24 +++++++++++++++++++++++-
 content/zh/docs/reference/setup-tools/kubeadm/generated/kubeadm_version.md                       |  24 +++++++++++++++++++++++-
 static/_redirects                                                                                |  48 +++++++++++++++++++++++++++++++++---------------
 30 files changed, 539 insertions(+), 267 deletions(-)
root@hello:~/website# 

```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06e685ec4b12465782a405c360dd1348~tplv-k3u1fbpfcp-zoom-1.image)

> https://www.oiox.cn/
> 
> https://www.chenby.cn/
> 
> https://blog.oiox.cn/
> 
> https://cby-chen.github.io/
> 
> https://blog.csdn.net/qq_33921750
> 
> https://my.oschina.net/u/3981543
> 
> https://www.zhihu.com/people/chen-bu-yun-2
> 
> https://segmentfault.com/u/hppyvyv6/articles
> 
> https://juejin.cn/user/3315782802482007
> 
> https://cloud.tencent.com/developer/column/93230
> 
> https://www.jianshu.com/u/0f894314ae2c
> 
> https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/
> 
> CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、今日头条、个人博客、全网可搜《小陈运维》
> 
> 文章主要发布于微信：《Linux运维交流社区》