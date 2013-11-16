---
title: varnish的timeout设置
author: jolestar
layout: post
permalink:  /varnish-timeout/
sina_t:
  - 'true'
tags:
  - server
  - varnish
---


今天调试一个bug。手机通过2g网络上传文件的时候,常遇到服务器报503错误,并且概率比较大。  
先从程序查起,再到认证代理服务器,再到varnish。发现是varnish在文件还没有传完的时候就断开了连接，给客户端返回503。  
开始怀疑是varnish的 between\_bytes\_timeout 和 first\_byte\_timeout 的设置问题,但把这两个参数调大,问题依旧。  
最后发现是varnish sess_timeout 的问题。

<!--more-->

> first\_byte\_timeout  是varnish等待从后端接受第一个字节的超时时间。默认60秒。如果60秒内第一个字节都没收到，则给客户端返回503。一般情况下默认值都过于大，因为varnish的后端一般都在内网。
> 
> between\_bytes\_timeout 可以认为是varnish等待后端响应的 idel timeout。默认60秒。如果后端输出比较慢,传了一部分数据后，连接又空闲了60秒，varnish会放弃本次请求，给客户端发送503错误。
> 
> sess\_timeout  官方文档的说明是 “Idle timeout for persistent sessions.If a HTTP request has not been received in this many seconds, the session is closed.”。开始把这个参数和between\_bytes\_timeout 弄混了，主要也是没理解varnish的session概念。其实sess\_timeout 和between\_bytes\_timeout 的意义相似，只不过 between\_bytes\_timeout 作用在varnish到后端的连接上，而 sess_timeout 做用在 客户端到 varnish的连接上。默认5秒。这个值在恶劣的网络环境下明显不够。如前文描述的案例，就是这个参数导致的问题。
> 
> pipe_timeout 应该是http 1.1 的 keepalive长连接的timeout设置。两次请求之间等待多长时间就断开连接。
