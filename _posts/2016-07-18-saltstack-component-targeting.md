---
layout: post
title: saltstack基础组件之Targeting
date: 2016/7/19
categories: blog
tags: [自动化,运维,配置管理]
description: saltstack Targeting
---


### Targeting

Targeting是根据方法匹配出来需要执行命令的minion,
通过`salt --help`可得到saltTargeting的匹配方法	


#####  全名匹配，通配符

	salt 'SALT-MINION1' test.ping
	salt 'SALT-MINION*' test.ping


#####  grains -G，--grain
	
	salt -G 'os:CentOS' test.ping
	salt -G 'kernel:Linux' test.ping


#####  list匹配 -L,--list
	
	salt -L 'SALT-MINION1,SALT-MINION2' test.ping


#####  正则表达式 -E,--pcre
 
	salt -E 'SALT-MINION\d' test.ping


#####  grain正则表达式 --grain-pcre
	
	salt --grain-pcre 'os:(RedHat|CentOS)' test.ping

##### pillar -I,--pillar
使用pillar匹配需要先在master上执行`salt '*' saltutil.refresh_pillar`或`salt '*' saltutil.sync_all`缓存pillar数据才能执行成功。否则返回`Minion did not return. [No response]`

	salt -I 'from:HB' test.ping

##### pillar正则表达式 --pillar-pcre
同pillar,需要缓存pillar数据

	salt --pillar-pcre 'from:H\w' test.ping

##### 分组匹配 -N,--nodegroup
需要在master配置文件中配置分组信息使用

	salt -N 'webserver' test.ping

##### 范围表达式 -R，--range
需要使用模块seco.range,没有测试，具体使用方法见[seco range](https://docs.saltstack.com/en/latest/topics/targeting/range.html)

##### 复合匹配 -C，--compound

	salt -C 'G@os:CentOS and I@from:HB' test.ping

##### 网络位数匹配 -S，--ipcidr

	salt -S '192.168.100.0/24' test.ping


##### 在State File配置文件中使用Targeting
默认为通配符，如需要使用以上的其他各种匹配方式，需要使用 - match: MatcherName
	
	'*':
	  - test
	'web*':
	  - test
	'os:CentOS':
	  - match: grain
	  - test