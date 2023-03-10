---
title: 前后端分离的CAS单点登录
description: 如何理解单点登录的原理，并将其改造为前后端分离形式
slug: study-cas
date: 2021-09-02 14:28:00+0000
image: cas.png
math: true
comments: false
categories:
    - 技术分享
tags:
    - CAS
    - Java
---
# 前后端分离的CAS单点登录

## CAS登录原理

>官方网址：<https://apereo.github.io/cas/6.6.x/protocol/CAS-Protocol.html>

从官网上把流程图找到
![cas_flow_diagram.png](cas_flow_diagram.png)
我们简答梳理一下流程，主要分为两个步骤，未登录（包括失效的）和登录的

### 第一次登录未流程

1. 浏览器向应用发起请求，应用发现该请求未授权（无session或者过期）
2. 应用响应一个重定向给浏览器，重定向去CAS登录页面
3. 用户登录页面输入密码
4. CAS认证用户通过，写入TGC到cookie，带着Ticket重定向到应用
5. 应用向CAS验证Ticket的有效性，有效就set Session并重定向到服务（去掉Ticket）
6. 浏览器带着Session请求服务，服务校验Session


### 第二次登录流程

1. 浏览器向应用发起请求，应用校验Session
2. Session合法就正常执行并返回

### 第一次登录另一个应用

1. 浏览器向应用发起请求，应用发现请求未授权
2. 应用将浏览器携带TGT重定向去CAS登录页面
3. CAS校验TGT合法性，并重定向回应用（带着Ticket）
4. 应用向CAS确认Ticket的合法性，合法就签发Session

## CAS登录存在的问题

一个域名下的前端页面往往非常多，如果Session在某一个页面访问的时候超时，需要重新登录，那么CAS服务发送重定向的时候不知道来源所以只能重定向到首页，会给用户带来较差的体验。

## 解决办法

