---
title: JMeter插件开发
author: jolestar
layout: post
permalink:  /jmeter-plugin-develop/
tags:
  - jmeter
  - 测试
  - 工具
  
---

最近在使用jmeter作为回环和压力测试工具。虽然jmeter自带的插件基本能满足大多数场景，但有时候也需要自定义一些插件来实现。网上的jmeter的插件开发文档稀少，官方的[jmeter_tutorial](http://jmeter.apache.org/extending/jmeter_tutorial.pdf)也已经过时。通过本人的一些尝试,总结了一些jmeter插件开发相关的经验。

<!--more-->

## JMeter的核心组件

JMeter的核心组件主要有:

1. **Timer** 这个比较简单,用于配置每次sampling之间的等待时间。实现例子:org.apache.jmeter.timers.RandomTimer
2. **Sampler** 取样器，这个是最主要的组件。压力测试主要是靠Sampler实现的。通常用的有HttpSampler，用于测试http接口。如果是其他的协议需要实现其他协议的Sampler。
3. **ConfigElement** 配置组件，主要用于定义前置配置。如数据库连接，csv输入数据集等。主要功能是将配置转换为变量设置到JMeter context中。
4. **Assertion** 验证Sampler的结果是否符合预期。
5. **PostProcessor** 一般用于对Sampler结果进行二次加工。
6. **Visualizer** 将sampler的结果进行可视化展示。
7. **Controller** 对sampler进行逻辑控制。
8. **SampleListener** 监听sampler,基于事件机制。一般用于保存sampler的结果等耗费时间的操作。

## JMeter的插件加载机制

JMeter加载插件的机制比较简单，扫描扩展下的的所有实现了JMeterGUIComponent和TestBean接口的类，然后进行初始化。

	ClassFinder.findClassesThatExtend(
		JMeterUtils.getSearchPaths(), 
		new Class[] {JMeterGUIComponent.class, TestBean.class }

所以只要确保插件的jar包在扩展路径下，默认路径是: JMETER_HOME/lib/ext

## JMeter的GUI机制
由于jmeter是一个基于Swing的GUI工具,所以对它的GUI框架也需要有一定了解。
JMeter内部有两种GUI框架

1. 直接继承JMeterGUIComponent接口的抽象实现类:

		 org.apache.jmeter.config.gui.AbstractConfigGui
		 org.apache.jmeter.assertions.gui.AbstractAssertionGui
		 org.apache.jmeter.control.gui.AbstractControllerGui
		 org.apache.jmeter.timers.gui.AbstractTimerGui
		 org.apache.jmeter.visualizers.gui.AbstractVisualizer
		 org.apache.jmeter.samplers.gui.AbstractSamplerGui
2. 通过Swing的Bean绑定机制

前者的好处是自由度高,可定制性强，但需要开发者关心GUI控件布局,以及从控件到Model的转换。

后者基本不需要开发者接触到GUI层的东西，定义好Bean以及BeanInfo即可。但SampleListener不支持BeanInfo方式定义。不知道没来得及改造还是其他原因，我在这个坑里浪费许多时间。

## 示例
本例子是一个jmeter mongodb 插件,主要功能是将http json接口的Sampler的结果保存到mongodb中，供查询分析。

主要扩展了JMeter的三个组件:

* MongoDBConfig 实现 ConfigElement,用于定义mongodb 数据库
* MongoDBResultSaver 实现 SampleListener，将结果保存到mongodb中
* JSONPathExtractor 实现 PostProcessor，用于通过json path抽取json sampler结果中的数据（只需要保存一部分结果的情况）
* JSONPathAssertion 实现Assertion，用于通过json path 方式校验 sampler结果。

![jmeter mongodb plugin](http://jolestar.com/images/java/jmeter-plugin.png)

XXXXBeanInfo类即Swing Bean绑定机制的定义类。注意命名规则,XXX类的BeanInfo定义类必须是同一package下的XXXBeanInfo类。

项目github地址: [https://github.com/jolestar/jmeter-mongodb-plugin](https://github.com/jolestar/jmeter-mongodb-plugin)

## 示例截图
![mongodb config](https://github.com/jolestar/jmeter-mongodb-plugin/raw/master/screenshots/config.png)

![mongodb result saver](https://github.com/jolestar/jmeter-mongodb-plugin/raw/master/screenshots/result-saver.png)

## 扩展话题

JMeter作为回环测试以及压力测试是一个非常不错的工具，但由于用例管理仅限于GUI客户端模式，输出的是xml文件，不便于团队共同维护。如果将JMeter的gui改造成web界面模式，然后用例保存到数据库中，作为自动化测试工具使用应该非常不错。
