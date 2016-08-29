---
layout: post
title: saltstack基础组件之Pillar
date: 2016/7/19
categories: blog
tags: [自动化,运维,配置管理]
description: saltstack Pillar
---


### Pillar 

Pillar是salt用来给minions提供全局变量的接口，Pillar是用来存储敏感数据的。相对于grains的静态属性，Pillar属性是动态可变的，根据匹配条件的不同，Pillar可设置相应的属性。

同Grains一样，设置Pillar也有多种不同的方式

#### 配置文件自定义pillar
master中默认注释掉的pillar的目录为`/srv/pillar`,默认的pillar配置文件入口为`/srv/pillar/top.sls`

##### 启用pillar--`/etc/salt/master`

	pillar_roots:
	  base:
	    - /srv/pillar

top.sls的结构与匹配方式

	base:
	  '*':
	    - pkgs

此配置表示默认所有的minion全部引用pkgs.sls中定义的KV

##### Pillar的命名空间
所有的Pillar值共享同一个命名空间，相同name的pillar值后定义值会覆盖之前的定义，即`last one wins`，同一个KEY下面如果有多个值，则合并,如果同一个key下面的值也有相同，则同样执行`last one wins`
	
	top.sls
	
	base:
	  '*':
	    - users
	    - users1
	
	users.sls
	
	from: NJ
	userinfo:
	  name: Test
	  age: 20
	  isWorking: False
	
	users1.sls
	
	from: HB
	userinfo:
	  isWorking: True
	

执行`salt '*' pillar.items`返回结果
![pillar](http://7xweaf.com1.z0.glb.clouddn.com/t_pillar.jpg)

##### 命令行自定义pillar
	
	salt '*' state.apply pillar='{"cheese": "spam"}'

`使用命令行自定义pillar发送pillar数据，是直接发送给全部的minions,而不是直接发给目标minion`

##### pillar定义的变量在State File中的引用
直接引用

	apache:
	  pkg.installed:
	    - name: {% raw %}{{ pillar['pkgs']['apache']}}{% endraw%}

使用pillar的GET方法引用 

	apache:
	  pkg.installed:
	    - name: {% raw %}{{ salt['pillar.get']('pkgs','apache')}}{% endraw%}
