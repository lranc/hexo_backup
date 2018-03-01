---
title: 1_随机UA与代理IP
date: 2018-02-17 10:53:10
tags:
---
在抓取爬虫的过程中,往往会遇到各种反爬虫措施
现记录下在网上学习到的关于UA和IP的方案
  

<!--more-->
# 1.随机UA:
部分网站会检查是否是浏览器访问,对于这种反爬虫策略加上User-Agent访问即可.
在大量爬取的情况下,使用同一个UA是不够的.
<font color="red">常用方法如下:</font>
```
# -*- coding:utf-8 -*-
filename = 'user-agents.txt'
with open('filename','r') as f:
	user-agents = f.readlines()
user-agent = random.choice(user-agents)
```

# 2.代理IP
当使用同一个IP大量快速访问某个页面时,网站服务器可能将你判定为爬虫
此时可能会出现IP封禁或者出现验证码,输入正确才能继续访问

<font color="red">应对这种反爬虫措施,有以下几种解决方案:</font>
1).使用VPN,该方法简便,安全性高,实时性高,速度快,但价格较高.
2).IP代理商,方法简便,安全性高,可用率大,速度快,稳定性好,但价格较高
3).ASDL拨号,ASDL断开重连拨号后分配的IP地址会变化,免费,但效率低,实时性不高
4).自建IP代理池,免费,方法较为简便,实时性一般,速度一般,适合个人学习使用.

自建IP代理池的原理及步骤:
1).利用正则表达式,Xpath,CSS等爬取各个ip代理网站的ip地址
2).将IP地址保存进数据库(sqlite3,mysql,mongodb,redis等)
3).间断持续检测数据库中的ip有效性和速度,当有效IP少于一定数值,进行新一轮爬取
4).设计API接口,提供从Pool中获取IP地址,及删除IP地址的接口,以及个性化方案

网上已有许多成熟的开源IP代理池项目
本人的话,使用的是七夜的<font color='blue'>IPProxyPool项目</font>([地址][])
[地址]:https://github.com/qiyeboy/IPProxyPool

需要<font color="red">注意</font>的是,随机UA和频繁更换IP有时并不一定有效,甚至是错误的行为
实际爬取时,部分网站会请求重定向,或多重定向
在这些过程中,在一个会话中更换UA或IP可能会导致出错.


