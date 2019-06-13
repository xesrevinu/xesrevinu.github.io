---
title: csrf
tags: []
date: 2017-9-22 21:08
---

# 为什么需要 csrf

首先我们需要知道 cookie 和 session, cookie 是在浏览器内存在的，服务端可以通过 set-header 来改变客户端的 cookie， 每次对服务端发  请求的时候 cookie 都会被写到 http request header 里面被提交到服务端， 而 session 的功能则是记录 cookie 和用户的对应关系。
我们知道了 cookie 会在发起请求的时候发送到服务端，服务端通过解析接受到的 cookie 拿到其中的用户名做一些操作，那么问题就来了。

A 网站 是正常用户网站提供一个接口叫做注销账户 POST 请求

B 网站 通过在页面插入一个 iframe，在浏览到该页面的时候会自动 POST 注销账号请求

此时问题就出现了，A 网站用户一旦访问 B 网站，A 网站用户就再也上不去 A 网站了因为账号已经被注销掉了

# 解决问题

在接受 POST 请求的时候我们需要接受一个 CSRF TOKEN 参数，这个参数是访问页面的时候被告知到前端的，并且存在生成一个 secret 存在 session 里面，在接受 POST 请求的时候我们就能判断提交的时候有没有这个 CSRF TOKEN 值，并且需要验证这个值是能否通过 secret 验证

这样就解决了问题，别人获取不到你的 csrf token，就没有办法用你的身份去发请求
