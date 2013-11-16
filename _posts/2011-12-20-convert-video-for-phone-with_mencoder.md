---
title: 用Mencoder和FFmpeg为Android/IPhone转换视频
author: jolestar
layout: post
permalink:  /convert-video-for-phone-with_mencoder/
tags:
  - 全部
  - linux
  
---


想转换点视频放手机上在地铁里看。第一个想到的就是mencoder。

	mencoder -noskip -vfm ffmpeg -vf scale=800:480,harddup -oac faac -faacopts br=128:mpeg=4:object=2:raw -ovc lavc -lavcopts vcodec=mpeg4:vbitrate=200:autoaspect -of lavf -lavfopts format=mp4 -font 'simhei' -subfont-text-scale 3 -sub "test.cn.srt" -o test-m.mp4 -subcp cp936  test.avi

scale 根据你手机的分辨率进行设置

更多参看:

* <http://www.mplayerhq.hu/DOCS/HTML/en/fonts-osd.html>
* <http://www.ffmpegx.com>
