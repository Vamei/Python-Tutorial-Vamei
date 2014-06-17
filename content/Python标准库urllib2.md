##Python标准库获取HTML资源之urllib2
<!--more-->

我们平时通过浏览器可以从URL中获取相应的资源并展示出来，但是很多时候我们只是需要对获取的html资源进行特定的处理，就用到了python的urliib，urllib2和httplib等标准库


urllib2用于编写需要和http服务器,ftp服务器和本地文件交互的客户端;典型的应用程序有：抓取网页数据，代理，web爬虫等

###1.使用`urlopen()`发生请求，获取HTTP资源

`urlopen(url[, data[, timeout]])`

参数解释：

- `url`：可以是包括URL的字符串，也可以是Request类的实例（下面会讲到）;
- `data`：是使用`urlencode()`方法进行编码后的查询数据，常见的如填写一网页的表单数据，然后进行编码赋值给data,传入urlopen()函数;
- `timeout`：顾名思义就是超时时间的设置

`urlopen()`函数返回类文件对象，先暂时记为u，支持下列方法：

- `u.read([nbytes])`：以字节字符串形式读取nbytes个数据
- `u.readline()`： 以字节字符串形式读取但行文本
- `u.readlines()`： 读取所有输入行并返回列表
- `u.close()`
- `u.geturl()`： 返回实际的url，因为有可能发生重定向问题
- `u.getcode()`： 获取HTTP响应代码
- `u.info()`: 获取网页Header信息，可以看到网页编码

举例：
```python
import urllib2
url = "http://www.baidu.com"
u = urllib2.urlopen(url)
print u.geturl()
print u.getcode()
print u.info()
```
运行一下上面的代码，看看效果如何

###2.urllib2的Request请求类

上述使用`urlopen()`只是一种简单的请求，`urlopen`中的`url`参数仅仅是一个字符串而已。要想进行一些更复杂的操作，如修改HTTP请求包头，可创建Request实例，来作为`url`参数

`request(url[, data[, headers[, origin_req_host[, unverifiable]]]])`

参数解释：

- `headers`：这是一个字典类型的数据，包含可表示HTTP报头内容的键值映射
- `origin_req_host`：事务请求主机，通常是发出请求的主机名称
- `unverifiable`：如果请求是无法验证的url，则设为True，默认为False

记上述request实例为r，他有以下方法：

- `r.add_data(data)`：向请求添加查询数据，这里同样需要`urlencode()`编码。这里不会将data追加到之前设置的任何数据上，而是替换
- `r.add_header(key, value)`： 向请求添加报头信息
- `r.add_unverifiable_header`：向请求添加报头信息，但不会添加到重定向请求中去
- `r.get_data()`：返回请求的数据
- `r.get_full_url()`：返回请求的完整url
- `r.get_host()`：返回接受该请求的主机
- `r.get_method()`：返回HTTP方法，可能是GET或者POST
- `r.get_orgin_req_host()`：返回发起事务的请求主机
- `r.get_selector()`：返回url的选择器部分
- `r.get_tyte()`：返回url类型（如：“http”）
- `r.has_data()`：如果数据是请求的一部分，返回True
- `r.is_unverifiable()`：如果请求是无法验证的，返回True
- `r.has_header()`：是否有报头信息
- `r.set_proxy(host, type)`：准备请求连接代理服务器，这将使用host替代原来主机，使用type请求替换原来的请求类型

举例：
```python
import urllib2, urllib
url = "http://www.baidu.com"
r = urllib2.Request(url)
u = urllib2.urlopen(r)
print u.getcode()
print u.geturl()
```

###3.自定义获取资源方式---自定义opener

基本的`urlopen()`不支持验证，cookie或者其他http高级功能，我们可以使用`build_opener()`创建自己的opener对象

`build_opener([handler1[, handler2, ...]])`handler1这些都是特殊处理程序对象的实例，目的是向opner对象添加各种功能

`build_opner()`函数返回的对象具有`opene(url[, data[, timeout]])`方法

1）HTTP Cookie

添加`HTTPCookieProcessor`处理程序
```python
cookieHander = HTTPCookieProcessor()
openr = build_opener(cookieHander)
u = opner.open(...)
```
将不同类型的`CookieJar`对象作为`HTTPCookieProcessor()`的参数，可以支持不同类型的cookie处理方法。默认情况下，使用`ttp.cookiejar`模块中的`CookieJar`对象

当然也可以使用其余的`cookiejar`对象，如
```python
cookieHander = HTTPCookieProcessor(http.cookiejar.MozillaCookieJar("cookie.txt"))
```

2）代理,添加代理处理程序

proxy = ProxyHandler({'http': 'http://someproxy.com:8080/'})

根据你的需求还可以添加很多特殊的处理程序对象实例

默认始终使用的处理程序有：`ProxyHandler(通过代理重定向请求)`，`UnknownHandler(处理未知url)`，`HTTPHandler(基本HTTP验证处理)`，`HTTPDefaultErrorHandler(异常处理)`，`HTTPSHandler(通过安全HTTP打开url)`


###4.实战：模拟登录人人网（直接贴代码）
```python
#!usr/bin/env/python
#coding: utf-8
import urllib
import urllib2
import cookielib
import hashlib
from bs4 import BeautifulSoup

import sys
reload(sys)
sys.setdefaultencoding('utf-8')

class renren(object):

	def __init__(self, email, passwd):
		self.email = email
		self.passwd = passwd
		self.cookie = cookielib.CookieJar()
		self.opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(self.cookie))
		self.headers = {
			'User-Agent':
				'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/28.0.1500.52 Chrome/28.0.1500.52 Safari/537.36'
		}
		self.login_url = 'http://www.renren.com/PLogin.do'
		self.response = None
		self.content = None
		self.soup = None

	def login(self):
		data = {
			'email': self.email,
			'password': self.passwd
		}
		post_data = urllib.urlencode(data)
		req = urllib2.Request(url=self.login_url, data=post_data, headers=self.headers)
		self.response = self.opener.open(req)
		self.content = self.response.read()
		self.soup = BeautifulSoup(self.content)

	def recent_visit(self):
		visits = []
		all_rencent_visit = self.soup.findAll('span', {'class': 'tip-content'})
		for one_visit in all_rencent_visit:
			t = one_visit.get_text()
			visits.append(t)
		visits[0] = '最近来访:'
		return visits

	def today_birthday(self):
		today_birthdays = []
		all_today_birth = self.soup.findAll('a', {'class': 'uname'})
		for one_birth in all_today_birth:
			t = one_birth.get_text()
			today_birthdays.append(t)
		return today_birthdays

	def close(self):
		self.opener.close()

def main():
	sk_renren = renren('在这填写你的人人登录邮箱', '在这填写你的人人登录密码')
	sk_renren.login()

	visits = sk_renren.recent_visit()
	for visit in visits:
		print visit

	birthdays = sk_renren.today_birthday()
	print '\n今天过生日的有：'
	for birth in birthdays:
		print birth 

	sk_renren.close()


if __name__ == '__main__':
	main()
```
