---
layout: post
cid: 252
title: Git命令简单使用
slug: 252
date: 2022/06/14 09:01:00
updated: 2022/06/14 09:05:40
status: publish
author: cby
categories: 
  - 默认分类
tags: 
abstract: 
description: Git命令简单使用
keywords: 小陈运维,小陈,小陈博客,文档,
mode: default
thumb: 
video: 
---


## Git命令简单使用



#### 背景

最近经常使用Github，每次修改个文件代码都要在网页端操作，感觉效率低下，所以简答学习了解了一下Git命令。至使于可以在命令行进行管理Git仓库，这样就不需要每次都要打开网页版Github进行操作。

### 常用命令使用

```
# 拉取服务器代码，更新本地代码，避免覆盖他人代码
root@hello:~/Kubernetes# git pull 
Already up to date.
root@hello:~/Kubernetes# 

# 修改文件
root@hello:~/Kubernetes# vim README.md

# 将状态改变的代码提交至缓存
root@hello:~/Kubernetes# git add .

# 查看当前项目中有哪些文件被修改过
root@hello:~/Kubernetes# git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   README.md

root@hello:~/Kubernetes# 

# 将代码提交到本地仓库中
root@hello:~/Kubernetes# git commit -m update
[main 260d794] update
 1 file changed, 25 insertions(+), 25 deletions(-)
root@hello:~/Kubernetes# 

# 将缓存区代码推送到远程仓库
root@hello:~/Kubernetes# git push
Username for 'https://github.com': cby-chen
Password for 'https://cby-chen@github.com': 
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 434 bytes | 434.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To https://github.com/cby-chen/Kubernetes.git
   a101d22..260d794  main -> main
root@hello:~/Kubernetes# 

```



### 注意

1. 在push的时候是需要输入token，这个token需要在：https://github.com/settings/tokens 进行创建
2. GitHub的README.md文件内容换行，直接在要换行的语句最后打上2个空格。
3. 如果你觉得 git add 提交缓存的流程太过繁琐，Git 也允许你用 -a 选项跳过这一步。命令格式如下： `git commit -a`



> **关于**
>
> https://www.oiox.cn/
>
> https://www.oiox.cn/index.php/start-page.html
>
> **CSDN、GitHub、知乎、微信、开源中国、思否、掘金、简书、华为云、阿里云、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客、全网可搜《小陈运维》**
>
> **文章主要发布于微信**