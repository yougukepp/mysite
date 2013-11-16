---
title: 丫丫微博通开发日志
author: jolestar
layout: post
permalink:  /yayaweibotong/
sina_t:
  - 'true'
tags:
  - appengine
  - 丫丫微博通
  - 微博
  - java
  - twitter
  - yayaweibotong
  
---

**更新，该应用已经停止**

<!--more-->

### 丫丫微博通介绍

丫丫微博通 (<http://yayaweibotong.appspot.com>) 是一个在线自动同步微博的工具。现在微博服务越来越多，如果你同时使用多个微博服务，如何保持同步更新？手动复制？太累了吧。丫丫微博通就是为您这样的微博达人打造的工具。只要注册登录丫丫微博通并绑定您的微博帐号，该工具就会自动同步您的多个微博。

当前支持同步方式：

** 主从同步** 您需要设置一个主帐号，其他的为次帐号。你在主帐号上的更新会在很短时间内同步到其他的多个次帐号中去。  
**互相同步** 不分主次，您在任何一个帐号上的更新都会自动同步到其他的帐号上去。

### 开发计划

支持豆瓣  
美化界面  现在的界面看起来有点山寨 并且没有用任何javascript。操作不太友好。

### 更新日志

#### 2010年10月21日

增加互相同步方式 修改发布微博的调度策略

#### 2010年10月15日

修改任务调度策略

#### 2010年10月14日

测试版上线  
支持微博服务  twitter, 新浪微博  
基于 google appengine  
注册方式 邀请注册 如需要邀请码 请在twitter或者 新浪微博上发私信给我  ([jolestar@twitter.com][1] [joletar@t.sina.com.cn][2])

 [1]: http://twitter.com/jolestar
 [2]: http://t.sina.com.cn/jolestar
