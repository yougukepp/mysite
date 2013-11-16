---
title: 创业公司的数据收集以及监控系统方案
author: jolestar
layout: post
permalink:  /monitor-and-data-collect/
tags:
  - startup
  - 运维
  - monitor
  
---

精益的Build-Measure-Learn环中的一个要点是Measure。只有足够的数据,才能进行测量评估，增加认知。所以数据监控和收集对创业公司来说非常重要。
需要监控和收集的数据主要有**系统数据**和**业务数据**。

<!--more-->

##系统数据

系统数据主要指的是应用和服务的性能，可用性，频次，等数据

看了一些开源方案，基本业界的流行解决方案:

nagios+collected/ganglia/munin/cacti

* nagios用于设置阀值，报警。
* 整合数据收集与展示系统（collected/ganglia/munin/cacti）

	1. collected 收集数据 https://collectd.org/   只负责收集，不负责展示，如果展示需要整合一个展示系统。比较优秀的是graphite.
	2. ganglia http://ganglia.info/ 是一个完整的监控方案。
	3. munin http://munin-monitoring.org/ 的优势是比较简单直接定时生成静态图片。
	4. cacti  http://www.cacti.net/ 相当于 munin的升级版，动态绘制。
	5. graphite 用于绘图，一般和以上工具整合。优点是实时绘制，便于多个监控指标进行比较。

以上的监控系统的数据都是RRD格式。RRD格式的好处是随着时间的推移降低精度，缺点是不会保存完整的数据。

以上工具主要用于收集服务器状况：

1. cpu,硬盘,网络,load 等
2. 通用的服务器状况 mc,mysql,nginx,jvm 等各种指标变化

以上开源的服务都有现成的插件可用，使用的是拉模式。

当前选择的方案是 collected + graphite。

应用层的性能,监控指标(如 接口响应时间 实事调用频次等）需要自行设计方案

1. 依然是拉模式，应用层先收集数据缓存，然后通过接口方式输出。监控系统定时拉取。简单的可以直接暴露http接口。如果是java应用也可以通过java的MBean模式，通过JMX输出。
2. 推模式，应用层将数据推送到一个中间层，由中间层汇总然后输出到监控系统。如: [statsd](https://github.com/etsy/statsd/)。

当前选择的方案是 statsd.

接口层的可用性监控当前没找到非常合适的开源工具，有第三方服务可供选择，但一般都只支持简单的接口调用，无法做回环功能监控。想到的办法是通过自动化测试工具来定时监控可用性。

## 业务数据

和业务相关的数据，一般无法通过程序直接记录。需要通过日志等方式汇总并通过一定的计算公式得出。

一般的解决方案是 收集日志（scribe等），写入 hadoop集群，然后通过日志分析工具每天出报表。

但对创业公司来说，hadoop的成本较高，小成本的解决方案是写日志到mongodb中，通过mongodb的MapReduce和集合方法来汇总统计。
收集日志最终没有选择 scribe,而选择了[fluentd](http://fluentd.org/). 

fluentd 优点是和mongodb整合容易，并且支持多种方式收集日志。

1. 应用层通过本地端口直接写入。
2. 应用层写入本地文件，fluentd 进行读取该文件并且转换格式记录日志
