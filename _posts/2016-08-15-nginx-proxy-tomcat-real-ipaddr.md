---
layout: post
title: nginx反向代理下tomcat获取客户端真实IP
date: 2016/8/15
categories: blog
tags: [NGINX,TOMCAT,IP]
description: nginx,tomcat,ip
---



### 背景

最近帮朋友部署一个网站，一台ECS服务器，两个前台，两个后端，理所当然的使用nginx反向代理了。

一堆上传入库，网站正常访问，进入正题吧，这时发现，tomcat的日志中的ip地址为ECS的地址。

好吧，这就是今天的问题了。很简单，但是很久不碰也会遗忘。

### 反向代理的意义

为什么要使用反向代理，不外乎以下几个原因

+ 方便：统一访问入口，只需要记住一个IP，一个端口外加应用名就可以方便的访问。
+ 安全：对外只暴露nginx的端口，减少安全隐患。
+ 负载均衡：提升系统负载。

### nginx反向代理的情况下tomcat获取客户端的真实IP

为了使后端的tomcat能获取到客户端的真实IP，需要使用的模块为ngx_http_proxy_module,nginx默认编译的模块

默认编译安装nginx就可以了。

典型的配置方法为，在nginx.conf的http zone中加入如下配置

	#proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

这里只需要配置tomcat的访问日志，只用到其中的一条。

tomcat配置文件修改,这里以tomcat7为例,默认的host中有如下配置

	<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
	               prefix="localhost_access_log." suffix=".txt"
	               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

根据日志文件对比不难发现pattern中的%h对应的为客户端的IP地址，这里需要修改的也是%h,将其修改为`%{X-Real-IP}i`,对应nginx配置文件中的`proxy_set_header X-Real-IP $remote_addr;`

	<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
	               prefix="localhost_access_log." suffix=".txt"
	               pattern="%{X-Real-IP}i %l %u %t &quot;%r&quot; %s %b" />

重启tomcat，然后访问，发现客户端的真实IP地址已经可以被tomcat记录到了。

**这只是一个比较普通的nginx反向代理的场景，`这里假设在nginx前面还有一个nginx或是F5呢`**

### 多层反向代理的后端获取真实IP地的nginx配置

这里需要使用到另外一个模块，而nginx非默认编译，需要手工添加

检查方法: /path/to/sbin/nginx -V

	nginx version: nginx/1.10.1
	built by gcc 4.4.7 20120313 (Red Hat 4.4.7-17) (GCC) 
	configure arguments: --prefix=/usr/local/nginx/ --with-http_realip_module

如果没有--with-http_realip_module这个模块则需要重新编译安装，安装方法如下
	
	./configure --prefix=/usr/local/nginx/ --with-http_realip_module

[平滑升级nginx](http://nginx.org/en/docs/control.html)
	
	kill -USR2 `cat /usr/local/nginx/log/nginx.pid`    #向nginx主进程发送USR2信号，用于升级可执行文件
	kill -WINCH  `cat /usr/local/nginx/log/nginx.pid`  #此时查看nginx进程可以发现有两个nginx主进程，发送WINCH信号量用于停止nginx worker processes
	kill -QUIT `cat /usr/local/nginx/log/nginx.pid`	   #退出旧的nginx进程，平滑升级完毕

这里以F5->nginx->tomcat结构来讲解如何配置realip

**F5**

+ 通过配置irules向后端传递客户端的真实IP地址
	
		when HTTP_REQUEST {
		HTTP::header replace "X-Forwarded-For" [IP::client_addr]             #将X-Forward-For替换为客户端IP地址
		HTTP::redirect https://[getfield [HTTP::host] ":" 1][HTTP::uri]    #此条忽略吧。用于https跳转http
		}
	
	向目标VS的resource中添加iRules

+ 通过修改配置文件插入X-Forwarded-For

	Local Traffic -> Virtual Servers -> Profiles -> http,修改Insert X-Forwarded-For	的值为Enabled

	Local Traffic -> Virtual Servers -> 目标VS，修改Configuration为Standard,然后修改HTTP Profile	的值为 http

**NGINX**

配置从F5接收真实客户端IP,假设F5的IP地址为192.168.1.253
	
	set_real_ip_from 192.168.1.253;
	set_ip_header	X-Forwarded-For;   #Defines the request header field whose value will be used to replace the client address.(from nginx.org)

这样配置就完成了nginx从F5接收客户端的真实IP

**TOMCAT**

同上面的nginx+tomcat获取真实IP地址配置，这里不重复。









