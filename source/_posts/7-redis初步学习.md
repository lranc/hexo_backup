---
title: 7_redis初步学习
date: 2018-03-03 15:49:21
tags:
---
...前几天回了学校,整理了下生活物品并玩了好久游戏...着实荒废了一阵,今天重拾学习,完成了初步学习redis的计划...真是惭愧.
  
以下是学习redis局域网部署的详细过程
  
<!--more-->
# 1.绑定ip
在尝试连接到局域网下的另一台电脑时,产生了无法连接的错误,仔细查阅了网上各种资料,终于在这个[网页] [1]找到了对redis的conf文件中bind ip的较为详细的解释,解决了问题

即修改redis安装文件夹下的redis.windows.conf文件:
```
#修改bind 127.0.0.1 
bind 192.168.0.* 127.0.0.1     (*为局域网IP) 
```

# 2.测试连接另一台电脑
连接另一台电脑
```
$ redis-cli -h 192.168.1.101 -p 6373
```

出现错误(error) NOAUTH Authentication required.
则输入在conf文件中设置的密码
```
auth xxxxxx
```

测试是否通讯成功
```
192.168.1.101:6379> set name lranc
ok
#另一台
127.0.0.1:6379> get name
'lranc'
```

# 3.简单的分布式爬虫案例
此案例根据TTyb的博客爬虫教程的第14章,之前由于虚拟机的问题,放弃了学习,今天苦苦搜寻了有关redis相关教程,发现还是该教程简单易懂,故重温.[网址] [2]
由于对redis的掌握不熟练,故将原blog的redis集群改为了redis单机
思路与其blog一致,共有四个文件,insert.py与check.py用于验证redis通讯是否连接成功,并添加需要抓取的url.spider1.py与spider2.py的内容基本一致,只在将html写入文件时,保存的文件名处作出了区分.
代码如下
<code class='highlighter-rouge'>insert.py</code>:
```
# -*- coding: utf-8 -*-
#插入数据脚本
import redis

#连接redis
redis_client = redis.Redis(host='192.168.1.101',port=6379,password='mima123427')
redis_client.flushdb()  #清空管道

#增加url到redis
def pushToRedis(name,valuelist):
	for i in range(50):
		for item in valuelist:
			redis_client.lpush(name,item)

name = 'url'
urlList = ['https://www.baidu.com','http://www.tybai.com/']
pushToRedis(name,urlList)
```

<code class='highlighter-rouge'>check.py</code>:
```
# -*- coding: utf-8 -*-
#检查数据脚本
import redis 

#连接redis
redis_client = redis.Redis(host='192.168.1.101',port=6379,password='mima123427')
name = 'url'

length = redis_client.llen(name)
print(length)
print(redis_client.lrange(name,0,-1))
```

<code class='highlighter-rouge'>spider1.py,spider2.py</code>:
```
# -*- coding: utf-8 -*-
import requests
import socket
import time
import redis

session = requests.session()
headers = {"User-Agent": "Mozilla/5.0 (Windows NT 6.3; WOW64; rv:32.0) Gecko/20100101 Firefox/32.0"}

redis_client = redis.Redis(host='192.168.1.101',port=6379,password='mima123427')

#从redis中取出url
def pop_from_redis(name):
	return redis_client.rpop(name).decode()

def get_html(url):
	#修饰头部
	headers.update(dict(Referer=url))
	#抓取页面
	response = session.get(url=url,headers=headers)
	return response.content.decode('utf-8','ignore')

#保持到本地
def save_to_local(html):
	hostname = socket.gethostname()
	ip_name = ("2" + socket.gethostbyname(hostname) + "#" + str(time.time())).replace(".", "_")  #与spider1的不同点
	with open(ip_name+'.html','w',encoding='utf-8',errors='ignore') as f:
		f.write(html)

def main():
	name = 'url'
	length = redis_client.llen(name)
	for i in range(length):
		url = pop_from_redis(name)
		print(url)
		html = get_html(url)
		save_to_local(html)
		time.sleep(1)

if __name__ == '__main__':
	time.sleep(5)
	main()
```

当出现<code class='highlighter-rouge'>AttributeError: 'NoneType' object has no attribute 'decode'</code>则说明redis消息集合中的url已经爬取完毕.
可以在两台机子存放的文件夹中看到爬取的html文件,一台爬取了56页面,一台爬取了44个页面.

# 总结
关于redis的初步学习就到此,下一步是继续学习redis在scrapy的应用,以及Django部署骗子查询


[1]: https://segmentfault.com/q/1010000007547672?_ea=1380149
[2]: http://www.tybai.com/crawler/14_%E5%88%86%E5%B8%83%E5%BC%8F%E6%8A%93%E5%8F%96.html