---
layout: post
title: Linux小记之chkconfig
date: 2016/8/15
categories: blog
tags: [linux,chkconfig]
description: linux,chkconfig
---



### 作用

引用自`man chkconfig	`

	chkconfig - updates and queries runlevel information for system services

chkconfig用于更新或查询系统服务的启动级别信息

### 语法

	chkconfig --list [name]
	chkconfig --add name
	chkconfig --del name
	chkconfig [--level levels] name <on|off|reset|resetpriorities>
	chkconfig [--level levels] name

### 用法及使用注意事项

chkconfig命令使用比较简单，只有简单的罗列，添加，删除及更新

##### 注意事项

使用chkconfig管理服务，需要在自定义脚本中添加至少两行注释

示例(man page)

	# chkconfig: 2345 20 80
    # description: Saves and restores system entropy pool for \
	#              higher quality random number generation.

其中

+ chkconfig指定服务的启动级别启动优先级及关闭优先级。第一列2345代表启动级别，如果不想指定启动级别，可以使用'-'代替，第二列20代表启动优先级，第三列80代表关闭优先级。
+ description服务的一些简单描述，如写成多行可以使用反斜杠

##### 查询指定服务的运行级别
	
	chkconfig --list tomcat

##### 添加自定义服务

	chkconfig --add tomcat
	此处添加服务的运行级别取自脚本注释行chkconfig中的第一列的值
	
通过chkconfig --add 添加服务，会在目录  /etc/rc.d/rcN.d (其中N代表0-6这7个启动级别)中添加S/K的文件，这些文件均链接到/etc/init.d的启动脚本
以 chkconfig: 2345 20 80,服务名为tomcat为例，使用chkconfig --add tomcat之后，会在/etc/rc.d/rc0.d,rc1.d,rc6.d中添加K80tomcat链接文件，在rc[2-5].d中添加S20tomcat链接文件。

为什么会是这样的结果呢，这里涉及到linux启动的过程，如果系统正常启动的话，只需要执行inittab文件中指定运行级别rcN.d中以S开头的脚本文件，如果是从其他运行级另切换过来的，则需要首先执行K开头的脚本文件，然后再执行以S开头的脚本文件。

##### 删除chkconfig管理的服务

	chkconfig --del tomcat

##### 更新指定服务的运行级别信息
	
	chkconfig --level 5 tomcat off
	chkconfig --level 235 tomcat on
	chkconfig tomcat reset

### 添加自定义服务

添加自定义服务除了服务脚本需要遵循以上的注意事项，还需要将服务脚本拷贝至目录`/etc/init.d`,然后才能通过chkconfig --add 添加自定义服务

	cp /root/scripts/tomcat /etc/init.d
	chmod +x /etc/init.d/tomcat
	chkconfig --add tomcatll












