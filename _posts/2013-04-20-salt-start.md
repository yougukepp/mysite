---
title: salt系列之初体验
author: jolestar
layout: post
permalink:  /salt-start/
published: false
tags:
  - salt
  - startup
  - 运维
  - 配置管理
  
---

创业公司一般都没有专门的运维人员，
salt按照官方的说法，是一种新的基础设施管理(infrastructure management)工具。
这一领域更有名的是puppet和chef.

## 基础设施管理工具的进化
切入主题前先说个故事。

假设这样一个场景，小明是创业公司的开发人员，需要搭建一个系统。这个系统虽然不复杂，麻雀虽小，五脏俱全。创业公司木有专门的运维人员，只能开发人员兼任。公司买了十多台裸机，安排小明要在这些服务器上安装好基础环境,配置好相关的参数，修改好各种服务的配置文件。由于时间比较紧，并且服务器不多，于是开始闷头一台一台设置。小明是个聪明的程序员，设置了几台后发现基础的软件包是通用的，于是写了一些shell进行安装。然后是修改配置，为了更新方便还便于维护，小明将配置文件都放到版本管理系统中（svn or git）上。
以mysql配置为例:
	
	/mysql/userdb.cnf
	/mysql/mydb1.cnf
	/mysql/mydb2.cnf
	...
	/mysql/mydbN.cnf
	
后来发现这样导致配置文件的数量急剧膨胀，因为不同的服务的略微区别（比如mysql的配置,差异主要是数据文件的路径，监听的端口，可能还有内存的大小）就需要重新copy一份配置。于是你脑筋一转使用了一种模板语言，将不同的变量抽取到单独的配置文件，将共性的配置改写为模板，部署的时候动态替换一下变量。这样一来又缩减了不少配置。配置变化为:
	
	/mysql/template.cnf
	
	cat /mysql/userdb.var
	port=3306
	datadir=/var/mysql/user/data

增加一套配置只需要增加一个简单的var配置。

再后来小明发现每次需要登陆到服务器上执行命令也很麻烦，于是又通过ssh信任的模式写了一个脚本，在一台中心服务器上即可远程执行命令。

至此，小明公司的简易版的基础设施管理工具算是完成了。
但是接下来又有了许多问题：

* shell的执行返回结果各异，需要人肉检查。
* 服务器越来越多，顺序执行的时间越来越长。
* 等等等等

至此，这个题外的故事就到此位置。基础设施管理工具主要是解决以上问题的。

## Why Salt?

为什么选择salt？这一领域更有名的还有CFEngine,Puppet和Chef。说实话,我这几个有名的工具的的使用经验不多，没有太多的比较。

选择salt的主要原因有:

1. salt比较简单单纯 这里的单纯的意思是依赖比较少。只要python环境就搞定。
2. salt集成了配置管理和远程执行。而puppet的远程执行是通过MCollective实现的。
3. salt的Yaml配置比较容易维护（这个不是个人经验，参看 http://serverfault.com/questions/415713/what-advantages-features-does-puppet-or-chef-offer-over-salt-or-vice-versa）

## Salt的架构
salt的主要功能有两块: 配置管理和远程执行.
