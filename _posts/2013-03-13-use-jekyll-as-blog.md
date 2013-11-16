---
title: 使用jeklly搭建自己的blog
author: jolestar
layout: post
permalink:  /use-jekyll-as-blog/
tags:
  - blog
  - 爬行在网上
  - jekyll
---

博客荒废好久了，杂草丛生，都不好意思打开了。那天看到一个用git作为博客存储的博客搭建工具: jekyll。github的page服务就使用的是它。和我去年用git后产生的用git做wiki以及博客服务的思路类似，于是又激起了我捣鼓博客的兴趣。用jekyll的好处是可以使用Markdown的语法来写博客。下面介绍一下搭建过程中的一些总结。

<!--more-->

##安装

安装jekyll非常容易
安装ruby后，使用gem一行命令搞定

	gem install jekyll

具体参看:

<https://github.com/mojombo/jekyll/wiki/install>

但安装rdiscount的时候遇到了问题,rdiscount 在ruby 1.9.2版本上有个bug <https://github.com/rtomayko/rdiscount/issues/48>.我是升级到 1.9.3才解决的。 

	sudo gem install rdiscount

推荐使用rdiscount作为Markdown语法解析工具，速度快，兼容性强。

##使用

使用起来很简单。按照 jekyll的标准设置目录，修改_config.yml配置文件，然后用

	jekyll --server

启动即可。
按照Markdown的语法编写博客放置在 _post文件夹下。

如果手动编写html的页面模板太麻烦，可以使用 [jekyllbootstrap]. jekyllbootstrap是一个整合twitter的 bootstrap 和 jekyll的工具，可以用bootstrap制作博客theme。

##git仓库

git仓库可以托管到github上，也可以自己搭建。如果托管到github上作为page服务，请参考 <http://pages.github.com/>。另外也可以考虑使用国人的 [gitcafe](https://gitcafe.com/)。

##历史数据导入

如果博客以前是用wordpress搭建的，则有许多工具支持导入数据
这篇文章介绍如何通过一些ruby脚本完成这个事情
<http://vitobotta.com/how-to-migrate-from-wordpress-to-jekyll/>
也有一些现成的工具 <https://github.com/mojombo/jekyll/wiki/Blog-Migrations>

我用的是一个wordpress插件 <https://github.com/benbalter/wordpress-to-jekyll-exporter>。效果还好，只是部分链接转换的时候会坏掉。


## 扩展解决方案

### 博客摘要

默认jekyll的首页只展示文章列表。可以让显示全文，但会导致首页过大。所以希望通过摘要的方式实现。博客摘要的功能当前有几种实现方式：

1. 插件方式
	
	插件方式功能较强，但github的pages服务不支持插件。
	
	_plugins/more.rb:

		module More
		    def more(input, type)
		        if input.include? "<!--more-->"
		            if type == "excerpt"
		                input.split("<!--more-->").first
		            elsif type == "remaining"
		                input.split("<!--more-->").last
		            else
		                input
		            end
		        else
		            input
		        end
		    end
		end

		Liquid::Template.register_filter(More)
	
	博客中增加标记
	
		---
		layout: post
		title: "Your post title"
		published: true
		---
		<p>This is the excerpt.</p>
		<!--more-->
		<p>This is the remainder of the post.</p>

	获取摘要方法
	
		<summary>{{ post.content | more: "excerpt" }}</summary>
	获取全文
	
		<article>{{ post.content | more: "remaining" }}</article>		

2. liquid 过滤器方式
	
	这种模式不需要插件，但缺点是截取的字符是固定的，会把一个词切开，体验不好。

		{{ post.content | strip_html | truncatewords: 25 }} 
	
3. 通过html的注解进行hacks
	
	在文档中增加特殊的标记。

		---
		title: some post
		layout: post
		---
		Some intro, this will be visible on the index page.
		<!-- more start -->
		More content, this will not be visible on the index page.
		<!-- more end -->

	显示summary的时候通过文本替换，将后面的全文内容放到html的注释中。这样页面上就不会显示出来。虽然看起来会好一点，但白白传输了一部分不需要显示的内容。
	
		{{ post.content | replace:'more start -->','' | replace:'<!-- more end','' }}

	来源:
	<http://kaspa.rs/2011/04/jekyll-hacks-html-excerpts/>

4. js操作模式
	
	思路是页面渲染后通过js把过长的文章中的html节点给隐藏了。
	具体参看:
	<http://blog.evercoding.net/2013/03/09/traversing-dom-tree-with-javascript/>
	
5. 我的解决方案
	
	参考方案3，我的解决方案更简单:
	
		---	
		title: some post
		layout: post
		---
		Some intro, this will be visible on the index page.
		<!--more-->
		More content, this will not be visible on the index page.
		
	然后用split方法截取第一部分显示
	
		post.content | split:'<!--more-->' |first
		
### 代码高亮

代码高亮有两种解决方案

1. 使用pygmentize或者redcarpet在生成页面的时候自动将代码格式化成带样式的html
	参考: <https://github.com/mojombo/jekyll/wiki/install>	

2. 纯js解决方案

	有名的有,[google-code-prettify]和 [SyntaxHighlighter]。后者功能比较强大，但也比较臃肿,最后选择了 [google-code-prettify]
	
		<script type="text/javascript" src="{{ ASSET_PATH }}/js/jquery-1.9.1.min.js"></script>
		<script type="text/javascript">
		    $(function() {
		        $('pre').addClass('prettyprint').attr('style', 'overflow:auto');
		    });
		    $(function() {
		         window.prettyPrint && prettyPrint();
		    });
		</script>
		<script src="{{ ASSET_PATH }}/google-code-prettify/prettify.js" defer="defer"></script>



### 评论方案
评论可以使用[jekyllbootstrap]提供的comments-providers(disqus,facebook,intensedebate,livefyre)。
我基于微博的评论箱功能扩展了一个基于微博的comments-providers

<https://github.com/jolestar/jolestar.github.com/blob/master/_includes/JB/comments-providers/weibo>

### 和nginx的整合
部署定时任务自动从git仓库拉取源码然后生成静态页面。
如这篇博客中的介绍:<http://grepalex.com/2012/08/17/full-jekyll-VM-setup/>。
但为了节省资源，没必要每次都重新生成，所以我使用了git的post-merge hooks来触发生成静态页面。
	
post-merge
	
	#!/bin/bash
	workspace=`pwd`;
	exec sh $GIT_DIR/hooks/jekyll.sh $workspace > 	$GIT_DIR/jekyll.log

jekyll.sh

	#!/bin/bash

	workspace=$1
	
	send_email_and_exit() {
	  recipient=$1
	  message=$2
	
	  echo "Sending email and exiting due to error"
	  echo $message
	  mail -s "Blog generation failure" "${recipient}" << EOF
	${message}
	EOF
	
	  exit 1
	}
	
	echo "Running at "`date`
	
	emailto="youremail@xxx.com"
	
	echo $workspace
	cd $workspace
	
	jekyll --no-auto  --no-safe
	
	exitCode=$?
	
	if [ ${exitCode} != "0" ]; then
	  send_email_and_exit "${emailto}" "Jekyll failed with exit code ${exitCode}"
	fi

将这两个文件放置到服务器的git workspace路径下的.git/hooks中，然后设置定时任务定时从仓库pull源码，如果有更新则会重新生成页面。默认的路径在 workspace/_site 目录下，将ngnix的web路径指向到该目录即可。

至此大功告成。可以使用任意自己喜欢的文本编辑器来写博客了。除了vim，另外推荐一个 Markdown 的编辑器: Mou。

##参考资料

* [YAML头](http://wiki.github.com/mojombo/jekyll/yaml-front-matter)
* [Markdown](http://daringfireball.net/projects/markdown/syntax) [Markdown中文说明](http://wowubuntu.com/markdown)
* [Liquid](http://www.liquidmarkup.org/) 与 [Jekyll-Liquid-extensions](http://wiki.github.com/mojombo/jekyll/liquid-extensions)

[google-code-prettify]: <http://google-code-prettify.googlecode.com/> "google-code-prettify"
[SyntaxHighlighter]: <http://alexgorbatchev.com/SyntaxHighlighter/> "SyntaxHighlighter"
[jekyllbootstrap]: <http://jekyllbootstrap.com/> "jekyllbootstrap"
