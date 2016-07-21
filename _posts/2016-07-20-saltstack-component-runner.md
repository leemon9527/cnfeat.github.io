---
layout: post
title: saltstack基本组件之runners
date: 2016/7/20
categories: blog
tags: [自动化,运维,配置管理]
description: saltstack runners
---


### Runners
runners是salt用于master上的执行模块。通过salt-run命令执行，根据在runner目录中py脚本中定义的函数，执行不同的功能。

#### 简单配置方法

##### 配置runners目录
	
	#/etc/salt/master
	runner_dirs: [ /srv/runners]

##### 定义runner
一个runner就是一个py脚本中的一个公共方法，由salt-run命令运行，如在runners目录中有一个test.py脚本，其中定义一个test方法，通过salt-run执行就是`salt-run test.test`

官网提供的一个打印所有minions 名字的一个例子

	#touch /srv/runners/get_minions.py

	import salt.client

	def up():
	  client = salt.client.LocalClient(__opts__['conf_file'])
	  minions = client.cmd('*','test.ping',timeout=1)
	  for minion in sorted(minions):
	    print minion

执行 `salt-run get_minions.up`,得到如下结果
![get_minions](http://7xweaf.com1.z0.glb.clouddn.com/get_minions.jpg)
执行salt-run命令，默认会在返回结果的最后一行打印方法的返回值，如没有返回值则会显示None,可以加上--quiet选项屏蔽掉

#### 用途
runners是仅在master上运行的一系列脚本的集合，可以将一些日常任务写到runners中，通过salt-run命令方便快捷的执行。














