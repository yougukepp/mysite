---
title: 搭建内部的yum仓库以及通过rpmbuild创建rpm包
author: jolestar
layout: post
permalink:  /create-custom-yum-repository-and-create-rpm-by-rpmbuild/
tags:
  - startup
  - 运维
  - yum
  - linux
---

部署服务器，由于操作系统版本较老，许多软件都需要从源码编译安装,编译时还要解决依赖问题。为了避免重复劳动，最好的方式是搭建一个内部的yum仓库，然后将源码编译打包成rpm包,使用的时候直接通过yum安装即可。

<!--more-->

## 创建yum仓库

这个比较简单 

1. 安装createrepo
	
		yum -y install createrepo

2. 创建目录

		mkdir -p /yum/centos/5/{i386,x86_64}
		
3. 初始化repodata

		createrepo -d  /yum/centos/5/x86_64
		
4. 配置http服务以nginx为例

		server {
  		  
  		  listen 80;
  		  server_name yum.example.com
  		  access_log  /var/log/nginx/yum_access.log;
		  error_log   /var/log/nginx/yum_error.log;
		  
		  location / {
		        root /yum;
		        autoindex  on;
		   }

		}

5. 配置服务器的yum源
		
		[testrepo]
		name=test repo
		baseurl=http://yum.example.com/centos/5/$basearch/
		gpgcheck=0
		enabled=1

	注意，由于没有配置gpg，所以要将gpgcheck设置为0.

仓库搭建好了，但还没有东西。下一步就需要通过rpmbuild创建软件包。

## 创建rpm包

1. 安装rpmbuild
	
		yum -y install rpmbuild

2. 创建用户

		adduser rpmbuild
		
	创建rpm包千万别使用root。否则如果脚本编写错误，可能导致破坏文件。
			
3. 创建目录

		su rpmbuild
		mkdir -p ~/rpmbuild/{BUILD,RPMS,S{OURCE,PEC,RPM}S}

	SOURCES 目录是放置源码包的位置，打包后的rpm文件会在RPMS下,SRPMS用于保存源码rpm包。spec文件会在SPEC目录下。

4. 下载源码	
	源码放置在SOURCES目录下。以git为例来说明如何制作rpm。由于git在centos仓库中的版本太老，所以需要制作一个新的。
	
		cd ~/rpmbuild/SOURCES
		wget -c https://git-core.googlecode.com/files/git-1.8.2.1.tar.gz
		
5. 编写spec文件

	spec文件是用于描述如果编译以及打包的描述文件
	
		cd ~/rpmbuild/SPEC
		vim git.spec

	下面是示例spec文件
	
		Summary: Git Scm 
		Name: git
		Version: 1.8.2.1 
		Release: 1
		License: GPLv2
		Group: Development/Tools
		URL: https://code.google.com/p/git-core/
		
		Source: %{name}-%{version}.tar.gz
		BuildRoot: %{_tmppath}/%{name}-%{version}-%{release}-root
		%define prefix /usr/local/
		
		BuildRequires: curl-devel,expat-devel,gettext-devel,openssl-devel,zlib-devel 
		Packager: jolestar 
		
		%description
		Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.
		
		%prep
		%setup -q
		
		%build
		%{__make} prefix=$RPM_BUILD_ROOT%{prefix} all 
		
		%install 
		%{__rm} -rf %{buildroot}
		%{__make} prefix=$RPM_BUILD_ROOT%{prefix} install 
		
		%clean
		%{__rm} -rf %{buildroot}
		
		%files 
		%defattr(-, root, root, 0755)
		%{prefix}/bin
		%{prefix}/share
		%{prefix}/lib
		%{prefix}/libexec
		%{prefix}/lib64


	说明:
		
	* Summary  摘要
	* Name    package名字
	* Version    用来 build 的 source code 版本。
	* Release    发行号。同于区分同一个源码版本的多次build。
	* Group      分组。cat /usr/share/doc/rpm-4.4.2.3/GROUPS 可查看所有标准分组
	* License    软件的版权。
	* URL        软件地址。
	* BuildRequires   编译代码需要的软件。这个只影响编译命令，不影响打包后的rpm包的依赖关系。
	* %prep     打包前置操作
	* %build    定义该源码包如何build
	* %install  定义该源码包如何安装（此处相当于copy到rpm包的虚拟系统路径下）
	* %files  定义哪些文件需要打包到rpm包中
	* %changelog    列出更改记录。
	
	总的概括一下，rpmbuild做的事情是以临时目录作为虚拟的系统根目录，然后build，install（需要制定参数，安装到临时虚拟根目录下）,然后打包 file块中制定的目录结构。注意: file目录是相对虚拟根目录的路径。

6. 执行

		rpmbuild -bb git.spec
	
	如果打包成功，RPMS目录下会有对应的rpm包。以我的为例:
		
		RPMS/x86_64/git-1.8.2.1-1.x86_64.rpm 

	-bb参数是指定rpmbuild只生成二进制rpm，不需要生成源码rpm。

7. 更新仓库

		cp ~/rpmbuild/RPMS/x86_64/git-1.8.2.1-1.x86_64.rpm /yum/centos/5/x86_64/RPMS/
		createrepo --update /yum/centos/5/x86_64/
		
	在其他服务器上使用yum list git看看git的新版本是否检测到了。如果未检测到可能需要yum clean all清空一下缓存。

		
