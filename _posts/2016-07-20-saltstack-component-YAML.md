---
layout: post
title: saltstack基本组件之YAML
date: 2016/7/20
categories: blog
tags: [自动化,运维,配置管理]
description: saltstack YAML
---


### YAML
salt的配置文件SaLtStae(SLS)的默认渲染器为YAML，要写配置文件则需要了解YAML的一些语法。[官方文档](https://docs.saltstack.com/en/latest/topics/yaml/)给出了写SLS文件所需要掌握的三条YAML简要规则.

#### 缩进
YAML使用缩进表示配置文件的层级并且严格使用两个空格作为缩进

	src_copy:
	  - name: /tmp/python

#### 冒号
冒号是YMAL表示KV键值对符号，类似于python中的字典。当使用时冒号后面需要跟一个空格后再写值

	keyname: value

或使用缩进

	keyname:
	  value

以上表示键值对的方法在SLS配置文件中基本不用，因为在配置SLS中，一个KEY下面可能不止一个值，这就需要用到下面提到的中划线

#### 破折号
YAML使用'-'来标识列表，使用一个'-'加一个空格,同一个列表内的项目使用相同的缩进

以下为一个典型的`top.sls`中配置的内容

	base:
	  '*':
	    - webserver
	    - appserver
















