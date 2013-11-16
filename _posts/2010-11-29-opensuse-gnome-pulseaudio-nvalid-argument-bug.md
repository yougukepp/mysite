---
title: opensuse 下 pulseaudio 的一个bug
author: jolestar
layout: post
permalink:  /opensuse-gnome-pulseaudio-nvalid-argument-bug/
sina_t:
  - 'true'
tags:
  - 全部
  - bug
  - linux
  - pulseaudio
---
# 

查看了下系统messages日志（/var/log/messages）

结果发现每几秒钟就报错：

    pulseaudio[3046]: sap.c: sendmsg() failed: Invalid argument

搜索了一下，应该是一个bug

<https://bugs.launchpad.net/ubuntu/source/pulseaudio/bug/187963>

修改一下 pulseaudio Local Sound Server 的设置，Multicast/RTP 选项，把下面两个复选框(Enable Multicast/RTP  sender,Enable Multicast/RTP  receiver)都给禁用了,就没这个错误了。
