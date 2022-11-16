---
layout: post
cid: 129
title: 使用HTMLform表单操作腾讯云DNS控制台
slug: 129
date: 2022/03/29 15:33:27
updated: 2022/03/29 15:33:54
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


        在使用中经常需要修改DNS记录，或者查询、删除操作。每次都得登录腾讯云控制台，腾讯云比较鸡肋的一点就是需要进行微信扫码登录，每次操作太不方便。

  

        可以使用api接口进行操作腾讯云上的产品。所以使用HTML写了一个前端页面，暂时没有美化，目前只有基础功能。

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3dfd96bec70e4c8f96ee85d99088d882~tplv-k3u1fbpfcp-zoom-1.image)

  

前端代码如下，同时可以访问：http://dns.oiox.cn/ 使用

  

```HTML
<!--
 * @Author: 陈步云
 * @Date: 2022-01-07 16:52:23
 * @LastEditTime: 2022-03-29 15:09:56
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: /html/index.nginx-debian.html
-->
<!DOCTYPE html>
<html>
<head>
<title>Welcome to chenby!</title>
<meta charset="UTF-8">

<!-- <script src="http://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="http://oss.maxcdn.com/jquery.form/3.50/jquery.form.min.js"></script> -->

<style>
    body {
        width: 50em;
        margin:  auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
    h1{
        text-align:center
    }
    button{
        display:block;
        margin:0 auto
    }
</style>
</head>
<body>

    <h1>腾讯云DNS记录控制台</h1>
    <h2>查询记录</h2>
    <form action="https://dnsapi.cn/Record.List" method="POST" >
        <div>
            * 腾讯云token <input name="login_token" type="text">    <br> <br>「去控制台创建  https://console.dnspod.cn/account/token/token  <br> 比如 ID 为：13490,ToKen为：6b5976c68aba5b14a0558b77c17c3932。<br> 即完整的 Token 为：13490,6b5976c68aba5b14a0558b77c17c3932 。」 <br> <br>
        </div>
        <div>
              返回类型 <input name="format" type="text" value="json">「默认json」<br><br>
        </div>
        <div>
            * 操作域名 <input name="domain" type="text">「如 oiox.cn」<br><br>
        </div>
        <div>
            子域名 <input name="sub_domain" type="text">「www」<br><br>
        </div>
        <div>
            <button type="submit" value="提交">提交</button>
        </div>
    </form>
    

    <h2>新增记录</h2>
    <form action="https://dnsapi.cn/Record.Create" method="POST">
        <div>
            * 腾讯云token <input name="login_token" type="text">    <br> <br>「去控制台创建  https://console.dnspod.cn/account/token/token  <br> 比如 ID 为：13490,ToKen为：6b5976c68aba5b14a0558b77c17c3932。<br> 即完整的 Token 为：13490,6b5976c68aba5b14a0558b77c17c3932 。」 <br> <br>
        </div>
        <div>
              返回类型 <input name="format" type="text" value="json"> 「默认json」<br><br>
        </div>
        <div>
            * 操作域名 <input name="domain" type="text"> 「如 oiox.cn」<br><br>
        </div>
        <div>
                * 记录类型：
                <select name="record_type" type="text">
                    <option value="A">A</option>
                    <option value="AAAA">AAAA</option>
                    <option value="SPF">SPF</option>
                    <option value="CAA">CAA</option>
                    <option value="CNAME">CNAME</option>
                    <option value="MX">MX</option>
                    <option value="TXT">TXT</option>
                </select>
                <br>
                <br>
        </div>
        <div>
            * 主机记录 <input name="sub_domain" type="text"> 「如 www 」<br><br>
        </div>
        <div>
                解析线路：
                <select name="record_line" type="text">
                    <option value="默认">默认</option>
                    <option value="联通">联通</option>
                    <option value="移动">移动</option>
                    <option value="电信">电信</option>
                    <option value="铁通">铁通</option>
                    <option value="境内">境内</option>
                    <option value="境外">境外</option>
                </select>
                <br>
                <br>
        </div>
        <div>
            * 记录值 <input name="value" type="text">  <br>「如 IPv6:2620:119:35::35 IPv4:8.8.8.8, CNAME: cname.dnspod.com., MX: mail.dnspod.com. 等等」<br><br>
        </div>
        </div>
        <div>
            <button type="submit" value="提交">提交</button>
        </div>
    </form>


    <h2>修改记录</h2>
    <form action="https://dnsapi.cn/Record.Modify" method="POST">
        <div>
            * 腾讯云token <input name="login_token" type="text">    <br> <br>「去控制台创建  https://console.dnspod.cn/account/token/token  <br> 比如 ID 为：13490,ToKen为：6b5976c68aba5b14a0558b77c17c3932。<br> 即完整的 Token 为：13490,6b5976c68aba5b14a0558b77c17c3932 。」 <br> <br>
        </div>
        <div>
              返回类型 <input name="format" type="text" value="json"> 「默认json」<br><br>
        </div>
        <div>
            * 操作域名 <input name="domain" type="text"> 「如 oiox.cn」<br><br>
        </div>
        <div>
            * 记录ID <input name="record_id" type="text">  「先使用查询功能查询到record_id」<br><br>
        </div>
        <div>
                * 记录类型：
                <select name="record_type" type="text">
                    <option value="A">A</option>
                    <option value="AAAA">AAAA</option>
                    <option value="SPF">SPF</option>
                    <option value="CAA">CAA</option>
                    <option value="CNAME">CNAME</option>
                    <option value="MX">MX</option>
                    <option value="TXT">TXT</option>
                </select>
                <br>
                <br>
        </div>
        <div>
            * 主机记录 <input name="sub_domain" type="text">  「如 www 」<br><br>
        </div>
        <div>
                解析线路：
                <select name="record_line" type="text">
                    <option value="默认">默认</option>
                    <option value="联通">联通</option>
                    <option value="移动">移动</option>
                    <option value="电信">电信</option>
                    <option value="铁通">铁通</option>
                    <option value="境内">境内</option>
                    <option value="境外">境外</option>
                </select>
                <br>
                <br>
        </div>
        <div>
            * 修改记录值 <input name="value" type="text">  <br>「如 IPv6:2620:119:35::35 IPv4:8.8.8.8, CNAME: cname.dnspod.com., MX: mail.dnspod.com. 等等」<br><br>
        </div>
        </div>
        <div>
            <button type="submit" value="提交">提交</button>
        </div>
    </form>


    <h2>删除记录</h2>
    <form action="https://dnsapi.cn/Record.Remove" method="POST">
        <div>
            * 腾讯云token <input name="login_token" type="text">    <br> <br>「去控制台创建  https://console.dnspod.cn/account/token/token  <br> 比如 ID 为：13490,ToKen为：6b5976c68aba5b14a0558b77c17c3932。<br> 即完整的 Token 为：13490,6b5976c68aba5b14a0558b77c17c3932 。」 <br> <br>
        </div>
        <div>
            返回类型 <input name="format" type="text" value="json"> 「默认json」<br><br>
        </div>
        <div>
            * 操作域名 <input name="domain" type="text">  「如 oiox.cn」<br><br>
        </div>
        <div>
            * 记录ID <input name="record_id" type="text">  「先使用查询功能查询到record_id」<br><br>
        </div>
        <div>
            <button type="submit" value="提交">提交</button>
        </div>
    </form>

</body>    
</html>

```

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95dccecc7814450ca0f9a85a51edc60b~tplv-k3u1fbpfcp-zoom-1.image)

  

会返回一个json解析，建议安装FeHelper工具，可以美化json，方便阅读。  

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e043926ff2904b85a0e2740858772c17~tplv-k3u1fbpfcp-zoom-1.image)

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab5e67a604bd45eb83285836cb5a37fe~tplv-k3u1fbpfcp-zoom-1.image)

  

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa94fe2b79134af7a59e0d1543901f7e~tplv-k3u1fbpfcp-zoom-1.image)

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6cb4848ba2c42c1bf79505948b5f623~tplv-k3u1fbpfcp-zoom-1.image)



https://www.oiox.cn/

https://www.chenby.cn/

https://cby-chen.github.io/

https://weibo.com/u/5982474121

https://blog.csdn.net/qq_33921750

https://my.oschina.net/u/3981543

https://www.zhihu.com/people/chen-bu-yun-2

https://segmentfault.com/u/hppyvyv6/articles

https://juejin.cn/user/3315782802482007

https://space.bilibili.com/352476552/article

https://cloud.tencent.com/developer/column/93230

https://www.jianshu.com/u/0f894314ae2c

https://www.toutiao.com/c/user/token/MS4wLjABAAAAeqOrhjsoRZSj7iBJbjLJyMwYT5D0mLOgCoo4pEmpr4A/

CSDN、GitHub、知乎、开源中国、思否、掘金、简书、腾讯云、哔哩哔哩、今日头条、新浪微博、个人博客、全网可搜《小陈运维》
