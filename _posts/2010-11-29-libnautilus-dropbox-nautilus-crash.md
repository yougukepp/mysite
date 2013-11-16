---
title: Dropbox的nautilus插件导致 nautilus 不能启动的问题
author: jolestar
layout: post
permalink:  /libnautilus-dropbox-nautilus-crash/
sina_t:
  - 'true'
tags:
  - 全部
  - dropbox
  - gnome
  - linux
---
# 

早上打开电脑，突然发现 nautilus 无法启动。  
刚启动就立刻关闭，也不包错。Terminal 下启动也没异常提示, 日志里也没错误记录。  
很是郁闷，本人刚从kde下转到gnome下就遇到这个问题。

最后用trace发现这个我问题是  Dropbox的nautilus插件导致的  
libnautilus-dropbox.so 报错 undefined symbol: g\_malloc\_n

删除了  libnautilus-dropbox , nautilus就正常了。

ps: nautilus的插件系统也太脆弱了吧，一个插件就能搞坏整个nautilus。

![][1]

 [1]: http://img.zemanta.com/pixy.gif?x-id=4114e3bd-f419-8d14-9419-e2b87f13348f
