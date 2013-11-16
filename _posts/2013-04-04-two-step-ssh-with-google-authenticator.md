---
title: 使用google-authenticator做两步ssh验证
author: jolestar
layout: post
permalink:  /two-step-ssh-with-google-authenticator/
tags:
  - startup
  - 运维
  - ssh
  - linux
---

一般大公司的服务器都通过 RSA SecurID 来实现 两步(two-step/two-factor) ssh 验证,但这个东西一个要几百块钱，对个人或者小公司来说不划算。但如果直接ssh开放裸奔也不放心，所以考虑使用google-authenticator基于软件实现两步验证。

<!--more-->

## 安装 Google Authenticator 应用

该应用支持android/IOS/BlackBerry 
<http://support.google.com/accounts/bin/answer.py?hl=en&answer=1066447>


## 在服务上编译安装 google-authenticator

先安装pam的devel包: apt系列的是 libpam0g-dev，yum系列的是 pam-devel

	git clone https://code.google.com/p/google-authenticator/
	cd google-authenticator/libpam
	make
	make install
	
修改 /etc/pam.d/sshd

在开始部分增加:

	auth required pam_google_authenticator.so

修改/etc/ssh/sshd_config 开启 ChallengeResponseAuthentication

	ChallengeResponseAuthentication yes
	
## 设置服务器 

切换到需要启用 google-authenticator 的用户执行google-authenticator

	$ google-authenticator 

	Do you want authentication tokens to be time-based (y/n) y
	https://www.google.com//chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/jolestar@xxxxx%3Fsecret%xxxxxxxx
	Your new secret key is: xxxxxx
	Your verification code is xxxxx
	Your emergency scratch codes are:
	  xxxxxxxx

	Do you want me to update your "/home/jolestar/.google_authenticator" file (y/n) y

	# 禁止同一个token被多个人使用
	Do you want to disallow multiple uses of the same authentication
	token? This restricts you to one login about every 30s, but it increases
	your chances to notice or even prevent man-in-the-middle attacks (y/n) y

	#是否需要延长过期时间，默认是30秒，但允许提前和延后一个段，所以过期时间是1分钟30秒。
	By default, tokens are good for 30 seconds and in order to compensate for
	possible time-skew between the client and the server, we allow an extra
	token before and after the current time. If you experience problems with poor
	time synchronization, you can increase the window from its default
	size of 1:30min to about 4min. Do you want to do so (y/n) n

	#是否启用频次限制
	If the computer that you are logging into isn't hardened against brute-force
	login attempts, you can enable rate-limiting for the authentication module.
	By default, this limits attackers to no more than 3 login attempts every 30s.
	Do you want to enable rate-limiting (y/n) y

重启ssh服务

## 设置客户端

打开google-authenticator(Google身份验证器),手动输入上面的token或者打开上面的url通过扫描二维码的方式添加账号。

可以看到token已经在定时变化。
尝试登陆一下，可以看到登陆方式已经变更了。

	ssh xxxx.com 
	Verification code: 
	Password: 

注意: 最好预先登陆两个终端上去，一个操作，另外一个预备。万一设置错误还可以救回来。


## 扩展

如果觉得软件方式不方便（有的手机不支持），可以考虑通过短信发token的方式。这儿有个发邮件的例子: <http://ben.akrin.com/?p=1068> 如果已经有发短信的接口，实现也很容易。
