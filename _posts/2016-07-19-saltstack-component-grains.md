---
layout: post
title: saltstack基础组件之grains
date: 2016/7/19
categories: blog
tags: [自动化,运维,配置管理]
description: saltstack grains
---


###  Grains

grains是salt获取信息的接口，它能获取包括操作系统、域名、IP地址、操作系统类型、内存等一系列的`系统静态属性`

#### 获取系统可用grains条目
	salt '*' grains.ls

#### 获取系统可用的所有grains数据
	salt '*'  grains.items

#### 获取指定grains条目的值
	salt '*' grains.item os

同样，grains值也可以通过配置文件或脚本自定义

#### 通过minion.conf自定义grains

`以下配置文件片段引用自minion config`

	grains:
      roles:
        - webserver
        - memcache
      deployment: datacenter4
      cabinet: 13
      cab_u: 14-15

此时执行`salt 'SALT-MINION1' grains.item roles`则返回如下结果
![return img](http://7xweaf.com1.z0.glb.clouddn.com/grains-roles.jpg)

#### 通过/etc/salt/grains on minion自定义grains

如果不希望修改minion的配置文件，可以通过单独的grains配置文件来自定义grain,配置的方法如在minion config中一样，只是少了一个顶层的grains标识

	roles:
	   - webserver
	   - memcache
	 deployment: datacenter4
	 cabinet: 13
	 cab_u: 14-15

`通过minion config或是grains配置文件自定义grains，需要重启minion使之生效`

#### 通过脚本自定义grains

自定义grains的脚本需要放在master的`file_root`的`_grains`目录下，默认的目录为`/srv/salt/_grains`。

`需要注意的是在master端定义的grains需要执行state.highstate,saltutil.sync_grains或saltutil.sync_all方法才能同步到minion。`

+ `salt 'minionid' state.highstate`是同步minion节点的所有状态，

+ `salt 'minionid' saltutil.sync_grains`是同步minion节点的grains信息，返回结果为新增的grains的方法名，如无新增返回为空

+ `salt 'minionid' saltutil.sync_all`是同步minion节点所有的自定义配置信息

---
	#!/usr/bin/env python
	def test_grains():
		#初始化一个空的grains字典
	    grains = {}
	    #设置grains的KV
	    grains['max_open_file'] = '1024'
	    return grains

为了避免同步自定义信息需要手工同步的问题，[saltstack官网给出了解决办法](https://docs.saltstack.com/en/latest/topics/reactor/index.html#syncing-custom-types-on-minion-start)，使用reactor监控minion_start事件，当minion被master命中时，首先会执行reactor中的minion start定义的内容。
新建配置文件 `/etc/reactor/sync_grains.sls`

	sync_grains:
	  local.saltutil.sync_grains:
	    - tgt: {{ data['id' ]}}

修改master的配置文件

	reactor:
	  - 'minion_start':
	    - /etc/reactor/sync_grains.sls

同样的，可以把sync_grains换成任意其他的sync_modules或sync_all

#### 关于自定义grains的优先级
由以上可知grains信息的获取来源于四个地方，分别为

>1. 系统自带的grains(core grains)
>2. 在/etc/salt/grains(on minion)中自定义的grains
>3. 在/etc/salt/minion(on minions)中自定义的grains
>4. 在master的file_root下_grains目录中配置的grains

 **其优先级顺序为4>3>2>1**
