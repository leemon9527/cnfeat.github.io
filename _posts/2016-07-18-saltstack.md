---
layout: post
title: saltstack使用小记
date: 2016/7/14
categories: blog
tags: [自动化,运维,配置管理]
description: saltstack
---

### 简介
官网的介绍如下

>Salt is a new approach to infrastructure management built on a dynamic communication bus. Salt can be used for data-driven orchestration, remote execution for any infrastructure, configuration management for any app stack, and much more.

翻译过来就是

>SaltStack是一个服务器基础架构集中化管理平台，具备远程执行、配置管理等功能

通过saltstack可以实现服务器的命令远程执行，服务状态的监控与自动恢复，配置文件的自定义配置、分发、监控，服务器基础环境的定义管理，可以有效提供运维效率。

### 最初接触
最初接触saltstack是因为需要给公司内部进行一次运维相关的技术分享。

目前成熟的自动化运维工具主要有puppet,saltstack,ansible,它们各有特色，在进行选型的时候有些纠结，最终选择了可以远程执行命令的saltstack，至于为什么不选择ansible是因为其特色不明显。

选型完成之后就动刀了，找了几台虚拟机安装，安装配置步骤就不再累述了。

初次使用有一种被惊艳到的感觉，以往批量修改服务器的口令是一件比较头疼的事情，需要登陆服务器上面修改或是远程执行ssh命令，使用saltstack一条命令搞定

	salt -G 'kernel:Linux' cmd.run 'echo password |passwd --stdin username'

这个是不是很方便？

### salt的基本组件
[saltstack doc](https://docs.saltstack.com/en/latest/ "doc")
在官方文档中列出使用salt你必须了解的几大基本组件

+ [Grains](/blog/2016/07/19/saltstack-component-pillar/)
+ [Pillar](/blog/2016/07/19/saltstack-component-pillar/)
+ [Targeting](blog/2016/07/19/saltstack-component-pillar/)
+ [Runners](blog/2016/07/19/saltstack-component-pillar/)
+ [YAML](blog/2016/07/19/saltstack-component-pillar/)


### salt的基本功能
在saltstack文档的首页列出了saltstack的几大功能

#### **REMOTE EXECUTION** 远程执行

>Running commands on remote systems is the core function of Salt.Salt can execute commands across thousands of systems in seconds.

salt提供了很多远程执行的模块，远程执行的基本命令格式为
	
	salt 'TargetMinions' module_name.module_function ['args'] 

由于salt提供的远程执行模块超级多，这里用一些比较常用的模块来举例

+ test
  - test.ping
		
		#用于测试minions的联通性
    	salt '*' test.ping
		
+ cmd
  - cmd.run

  		#用于执行一些自定义的命令或批处理
  		salt '*' cmd.run 'ls -l /root/source; echo "hello world"'
+ mine
  - mine.get

    	#用于获取一些周期性从客户端生成的信息(需要配置自定义的脚本)
    	#官方提供了cron模块，但是cron模块需要提供用户，而每个客户端的用户对于不主动获取的master来说是未知的，所以mine function可以周期性的收集minion上面所有的计划任务
    	salt '*' mine.get '*' get_cron
+ pip
  - pip.install
		
		#安装python模块
    	salt '*' pip.install salt

+ salt执行组合命令
	
		#模块命令与对应参数均以','分隔并严格对应
		salt '*' cmd.run,test.ping,test.echo 'ls -l /root',,'hello'

`salt远程执行命令的功能是平时操作中用得比较多的功能，但是在使用过程中需要小心，对于targeting中的通配符'*'要小心使用，避免由于命令的错误导致不可恢复的问题`

同时，在平时进行SLS文件调试使用的也是salt的远程执行功能，比如
	
	#检索所有minion的状态数据并执行，即同步所有minion
	salt '*' state.highstate
	#同步执行指定的SLS文件中描述的状态
	salt '*' state.sls python

#### **CONFIGURATION MANAGEMENT** 配置管理

配置管理的基本语法，引用官网的介绍图片
![](http://7xweaf.com1.z0.glb.clouddn.com/config_syntax.jpg)

在开始使用配置管理之前，需要进行简单的配置

新建`file_roots`目录

	touch /srv/salt

修改master配置文件并重启master

	file_roots:
	  base:
	    - /srv/salt

	service salt-master restart

##### 关于top.sls
`top.sls`是salt进行配置管理的入口文件，标识了minions对应的sls文件。

`file_roots`可配置多个，但`file_roots`目录中必须要包含一个入口的`top.sls`文件,当master执行`state.highstate`的命令时，首先会去file_roots中`base`的目录中寻找`top.sls`,如果没有找到，则会根据file_roots中配置的目录按配置的先后顺序找，找到任意一个top.sls则结束寻找top.sls，并开始依据top.sls标识的sls文件编译数据并同步指定的minion

这里以一个base目录为例来进行讲解配置管理，以下为一个典型的top.sls文件

	base:
      '*':
        - webserver
        - python
      'salt-minion1':
        - users
        - javaapp

+ top.sls中的base对应file_roots中设置的base目录/srv/salt
+ '*' 为通配符target，更多的用法可以参见基本组件中的[Grains](#salt)，表示所有minions， - webserver表示/srv/salt目录下的webserver.sls文件或是目录webserver中的init.sls文件，以此类推其他

下面以一个例子来说明下标准sls文件的一些基本写法

`python/init.sls`  在python默认版本为2.4的服务器上源码编译安装python2.7

	src_copy:
	  file.recurse:
	    - name: /tmp/python/2.7
	    - source: salt://source/python/2.7
	    - makedirs: True
	    - include_empty: True
	extract_python:
	  cmd.run:
	    - cwd: /tmp/python/2.7
	    - names:
	      - tar zxvf Python-2.7.12.tgz
	    - unless: test -d /tmp/python/2.7/Python-2.7.12
	    - require:
	      - file: src_copy
	python_pkg:
	  pkg.installed:
	    - pkgs:
	      - gcc
	compile_python:
	  cmd.run:
	    - cwd: /tmp/python/2.7/Python-2.7.12
	    - names:
	      - ./configure --prefix=/usr/local/python27 && make && make install
	    - require:
	      - cmd: extract_python
	      - pkg: python_pkg
	    - unless: test -d /usr/local/python27
	soft_link:
	  cmd.run:
	    - names:
	      - rm -f /usr/bin/python && ln -s /usr/local/python27/bin/python2.7 /usr/bin/python
	    - require:
	      - cmd: compile_python
	    - unless: python -V 2>&1 | grep "^Python 2.7.12$"
	shebang_yum:
	  cmd.run:
	    - names:
	      - sed -i '1i\#!/usr/bin/python2.4' /usr/bin/yum
	    - require:
	      - cmd: soft_link
	    - unless: sed -n '1p' /usr/bin/yum | grep -e "^#\!/usr/bin/python2.4$"
	shebang_repoquery:
	  cmd.run:
	    - names:
	      - sed -i '1i\#!/usr/bin/python2.4 -tt' /usr/bin/repoquery
	    - require:
	      - cmd: soft_link
	    - unless: sed -n '1p' /usr/bin/repoquery  | grep -e "^#\!/usr/bin/python2.4 -tt$"

在写sls文件的时候，根据最上面介绍的sls文件的基本语法，ID字段是`全局唯一`的，在所有的SLS文件中都不能重复

+ `require` 仅当依赖的模块执行成功才会执行当前模块
+ `unless` 仅当条件判断不成功时才会执行当前模块
+ `source` 资源文件链接以salt://打头，其根目录为file_roots中配置的对应目录

上面这个python的安装模块主要使用的是文件分发、包安装以及命令行。其他具体模块的使用及参数可参见**[StateModule](https://docs.saltstack.com/en/latest/ref/states/all/)**

上面介绍了一个python的整体的安装模块，使用saltstack另外一个亮点就是配置文件模板

比如现在有10台nginx服务器，每台nginx的配置都不一样，那在配置nginx.conf的时候就存在问题了，难道需要给每一台nginx服务器都单独保存一份nginx.conf在master上面吗？

当然不用，配置文件模板可以很轻松的解决这个问题。

saltstack默认模板为jinja,下面介绍一个nginx.conf的模板
{% raw %}

	#nginx_config.sls
	{% set nginx_user = 'nginx' + ' ' + 'nginx' %}
	{% set pid = '/var/run/nginx.pid' %}
	nginx_config:
	  file.managed:
	    - name: /usr/local/nginx/conf/nginx.conf
	    - source: salt://webserver/nginx/conf/nginx.conf
	    - template: jinja
	    - default:
	      nginx_user: {{ nginx_user }}
	      num_cpus: {{ grains['num_cpus'] }}
	      pid: {{ pid }}
	#nginx.conf
	user  {{ nginx_user }};
	worker_processes  {{ num_cpus }};
	pid        {{ pid }};
	...
{% endraw%}

这样配置之后就可以根据不同服务器的CPU个数来定义worker_processes的数量，不需要针对每台服务器配置不同的配置文件



### 小结

saltstack能够满足日常自动化运维的要求，能很方便进行基础环境搭建，文件分发，配置文件管理，服务状态监控与自动恢复，还能通过远程执行模块进行一些部署操作。结合一些python的web框架，能很方便的构建基础运维平台。





