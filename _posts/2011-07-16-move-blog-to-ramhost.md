---
title: 博客迁移到ramhost
author: jolestar
layout: post
permalink:  /move-blog-to-ramhost/
sina_t:
  - 'true'
tags:
  - android
  - 翻墙
  - 爬行在网上
  - vpn
---


鉴于天朝的墙越来越高，所以想入手一个vps 搭梯子翻墙。

网上搜索一下，国外vps服务商中，ramhost的评价似乎不错。ramhost的vps从 $3.99 到 $29.99 不等. 看的那天正好便宜的都没了，就买了个$19.9 的.

订单付款第二天,ramhost发邮件说服务器好了。ramhost的web面板很简单，基本就只能重装系统和重启系统.不如dreamhost那么强大,所以不适合对linux不熟悉的用户。

登陆上去看了下,默认安装的是Debian5.我从web面板上换成Debian6. ramhost的系统优化的不错。刚装的系统，只占用3m内存。主要也是因为ramhost刚开始的系统及其精简,连ssh都没有。装了ssh-server，openvpn,又装了mysql,apache,php. 把wordpress搭建起来,就用了678M内存了。

<!--more-->

下面是 UNIX Benchmarks 的测评结果:

    BYTE UNIX Benchmarks (Version 4.1-wht.2)
    System -- Linux vps18922587.vm.ramnoc.net 2.6.27 #1 SMP Mon Jun 13 17:16:12 UTC 2011 x86_64 GNU/Linux
    /dev/simfs            62914560    812072  62102488   2% /
    
    Start Benchmark Run: Sat Jul 16 10:21:35 UTC 2011
     10:21:35 up 17:30,  1 user,  load average: 0.00, 0.03, 0.01
    
    End Benchmark Run: Sat Jul 16 10:34:26 UTC 2011
     10:34:26 up 17:43,  1 user,  load average: 13.25, 5.34, 2.38
    
                         INDEX VALUES
    TEST                                        BASELINE     RESULT      INDEX
    
    Dhrystone 2 using register variables        376783.7 21991555.2      583.7
    Double-Precision Whetstone                      83.1     1835.0      220.8
    Execl Throughput                               188.3     9166.8      486.8
    File Copy 1024 bufsize 2000 maxblocks         2672.0   361678.0     1353.6
    File Copy 256 bufsize 500 maxblocks           1077.0   135449.0     1257.7
    File Read 4096 bufsize 8000 maxblocks        15382.0  1844144.0     1198.9
    Pipe-based Context Switching                 15448.6   339692.6      219.9
    Pipe Throughput                             111814.6  3554180.7      317.9
    Process Creation                               569.3        1.0        0.0
    Shell Scripts (8 concurrent)                    44.8     2146.6      479.2
    System Call Overhead                        114433.5 10195707.5      891.0
                                                                     =========
         FINAL SCORE                                                     222.8

性价比还可以。  
ramhost上只能搭建openvpn[搭建教程](https://forum.ramhost.us/bbs/viewtopic.php?id=4)。

android默认不支持openvpn,于是又在手机上折腾openvpn。需要有root权限，先安装 BusyBox(有的改造版的ROM里会带着) ,再安装OpenVPN Installer，OpenVPN Settings。将openvpn client 的配置文件，证书以及key拷贝到手机的/sdcard/openvpn/文件夹里,运行OpenVPN Settings进行设置就可以了。[详细安装教程](http://vpnblog.info/android-openvpn-strongvpn.html).  
需要注意的是要启用Open VPN Settings的 Fix DNS功能，否则被功夫网dns污染的网站(如 twitter,youtube)也是无法访问的。  
linux下使用Openvpen很简单，将openvpn的client配置文件在NetworkManager中import一下就行了。  
[windows下openvpn配置说明](https://forum.ramhost.us/bbs/viewtopic.php?id=165)  
感谢 @[蓝槐locust][1] 以前提供的dreamhost空间。  
另外感觉每月$20 翻墙有点奢侈,如果有熟悉的朋友需要vpn或者ssh代理，请联系我。我给开帐号。

 [1]: http://weibo.com/lanhuai "蓝槐locust"
