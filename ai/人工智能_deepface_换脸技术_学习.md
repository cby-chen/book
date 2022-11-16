---
layout: post
cid: 35
title: 人工智能 deepface 换脸技术 学习
slug: 35
date: 2021/12/30 17:07:56
updated: 2021/12/30 17:07:56
status: publish
author: cby
categories: 
  - 默认分类
tags: 
---


![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5551722f4b254661b22663128f7e538a~tplv-k3u1fbpfcp-zoom-1.image)

  

介绍  

  

Deepface是一个轻量级的python人脸识别和人脸属性分析（年龄、性别、情感和种族）框架。它是一种混合人脸识别框架缠绕状态的最先进的模型：VGG-Face，Google FaceNet，OpenFace，Facebook DeepFace，DeepID，ArcFace和Dlib。那些模型已经达到并通过了人类水平的准确性。该库主要基于 TensorFlow 和 Keras。

  

环境准备与安装

  

项目地址：

https://github.com/serengil/deepface

  

pycharm环境下载：

https://www.jetbrains.com/pycharm/download/#section=windows

  

conda虚拟环境：

https://www.anaconda.com/products/individual

  

数据集：

https://github.com/serengil/deepface_models/releases/download/v1.0/vgg_face_weights.h5

https://github.com/serengil/deepface_models/releases/download/v1.0/facial_expression_model_weights.h5

https://github.com/serengil/deepface_models/releases/download/v1.0/age_model_weights.h5

https://github.com/serengil/deepface_models/releases/download/v1.0/gender_model_weights.h5

https://github.com/serengil/deepface_models/releases/download/v1.0/race_model_single_batch.h5

  

创建项目

使用打开项目目录后，创建时使用conda的Python 3.9虚拟环境

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c75238b7236542c3b67dc3f0999438a1~tplv-k3u1fbpfcp-zoom-1.image)

  

安装pip依赖

创建完成后，在cmd中查看现有的虚拟环境，并进入刚刚创建的虚拟环境

conda env list

activate pythonProject

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6acad22ae9b64ed99174838b5daefced~tplv-k3u1fbpfcp-zoom-1.image)

  

进入环境后在进行安装pip所需依赖，并使用国内源进行安装实现下载加速

pip install deepface -i https://pypi.tuna.tsinghua.edu.cn/simple

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58f5096a70834443a804991aaf14f527~tplv-k3u1fbpfcp-zoom-1.image)

  

使用

  

面部验证

  

此功能验证同一人或不同人员的面部对。它期望精确的图像路径作为输入。也欢迎通过笨重或基于 64 编码的图像。

  

```
cd C:\Users\Administrator\PycharmProjects\pythonProject\tests\dataset

from deepface import DeepFace
result = DeepFace.verify(img1_path = "img1.jpg", img2_path = "img2.jpg")
```

  

会自动下载数据集，若无法下载数据集

可以提前下载好数据集，放入到 C:\\Users\\Administrator.deepface\\weights\\ 目录下

  
  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a60acdc50a9463081d7aad9159d6955~tplv-k3u1fbpfcp-zoom-1.image)

  

面部属性分析

  

Deepface还配备了一个强大的面部属性分析模块，包括年龄，性别，面部表情（包括愤怒，恐惧，中性，悲伤，厌恶，快乐和惊喜）和种族（包括亚洲，白人，中东，印度，拉丁和黑色）预测。

  

```
from deepface import DeepFace
obj = DeepFace.analyze(img_path = "img4.jpg", actions = ['age', 'gender', 'race', 'emotion'])
```

  

会自动下载数据集，若无法下载数据集

可以提前下载好数据集，放入到 C:\\Users\\Administrator.deepface\\weights\\ 目录下

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de61bf44979c40f58c8d53fb760fd63e~tplv-k3u1fbpfcp-zoom-1.image)

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