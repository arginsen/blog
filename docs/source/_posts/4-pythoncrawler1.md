---
title: python爬虫学习笔记Ⅰ：requests库
date: 2019-12-10 14:41:00
categories: notes
tags:
 - clawer
 - requests
 - Python
photo: 
    - /blog/img/crawler.jpg
---
<br>
<!--more-->

# Requests库入门

## 2个对象

r = requests.get(url)

1. r 表示Response对象，包含爬虫返回的内容。

属性               | 说明 
-------- | ---------
r.status_code      | HTTP请求的返回状态，200表示连接成功，404表示失败 r.textHTTP         | 响应内容的字符串形式，即，url对应的页面内容 
r.encoding         | 从HTTP header中猜测的响应内容编码方式 
r.apparent_encoding| 从内容中分析出的响应内容编码方式（备选编码方式） r.contentHTTP      | 响应内容的二进制形式

2. .get(url) 表示Request对象，构造一个向服务器请求资源的Request。

## 7个方法

方法               | 说明 
-------- | ---------
requests.request() | 构造一个请求，支撑以下各方法的基础方法 
requests.get()     | 获取HTML网页的主要方法，对应于HTTP的GET 
requests.head()    | 获取HTML网页头信息的方法，对应于HTTP的HEAD 
requests.post()    | 向HTML网页提交POST请求的方法，对应于HTTP的POST 
requests.put()     | 向HTML网页提交PUT请求的方法，对应于HTTP的PUT 
requests.patch()   | 向HTML网页提交局部修改请求，对应于HTTP的PATCH 
requests.delete()  | 向HTML页面提交删除请求，对应于HTTP的DELETE

### requests.request(method, url, **kwargs)

- method : 请求方式，对应get/put/post等
- url: 拟获取页面的url链接
- **kwargs: 控制访问的参数，共13个
    - params: 字典或字节序列，作为参数增加到url中 
    - data  : 字典、字节序列或文件对象，作为Request的内容
    - json: JSON格式的数据，作为Request的内容
    - headers : 字典，HTTP定制头
    - cookies : 字典或CookieJar，Request中的cookie 
    - auth: 元组，支持HTTP认证功能
    - files   : 字典类型，传输文件
    - timeout : 设定超时时间，秒为单位
    - proxies : 字典类型，设定访问代理服务器，可以增加登录认证
    - allow_redirects: True/False，默认为True，重定向开关 
    - stream  : True/False，默认为True，获取内容立即下载开关 
    - verify  : True/False，默认为True，认证SSL证书开关 
    - cert    : 本地SSL证书路径

```
各控制访问参数的使用(均为可选项)

--params--字典或字节序列，作为参数增加到url中
>>> kv={'key1':'value1','key2':'value2'} 
>>> r=requests.request('GET','http://python123.io/ws',params=kv) 
>>> print(r.url) 

--data--字典、字节序列或文件对象，作为Request的内容
>>> kv={'key1':'value1','key2':'value2'} 
>>> r=requests.request('POST','http://python123.io/ws',data=kv) 
>>> body='主体内容' 
>>> r=requests.request('POST','http://python123.io/ws',data=body)

--json--JSON格式的数据，作为Request的内容
>>> kv={'key1':'value1'} 
>>> r=requests.request('POST','http://python123.io/ws',json=kv)

--headers--字典，HTTP定制头
>>> hd={'user‐agent':'Chrome/10'} 
>>> r=requests.request('POST','http://python123.io/ws',headers=hd)

--file--字典类型，传输文件
>>> fs={'file':open('data.xls','rb')} 
>>> r=requests.request('POST','http://python123.io/ws',files=fs)

--timeout--设定超时时间，秒为单位
>>> r=requests.request('GET','http://www.baidu.com',timeout=10)

--proxies--字典类型，设定访问代理服务器，可以增加登录认证
>>> pxs={'http':'http://user:pass@10.10.10.1:1234' 
         'https':'https://10.10.10.1:4321'} 
>>> r=requests.request('GET','http://www.baidu.com',proxies=pxs)

```

### requests.get(url, params=None, **kwargs)

.get其实也是.request的封装  实际为.request('get',url,params=None,**kwargs)

+ url: 拟获取页面的url链接,链接需双引号
+ params: url中的额外参数，字典或字节流格式，可选
+ **kwargs: 12个控制访问的参数

```
>>> r=requests.get('http://httpbin.org/get') 
>>> print(r.status_code)
>>> r.text
```

### requests.head(url, )

- url: 拟获取页面的url链接 
- **kwargs: 12个控制访问的参数

```
>>> r=requests.head('http://httpbin.org/get') 
>>> r.headers 
```

### r=requests.post(url, data=None, json=None, **kwargs)

- url: 拟更新页面的url链接 
- data : 字典、字节序列或文件，Request的内容
- json: JSON格式的数据，Request的内容 
- **kwargs: 12个控制访问的参数

```
>>> payload={'key1': 'value1','key2':'value2'} 
>>> r=requests.post('http://httpbin.org/post', data= payload) 
>>> print(r.text)
```
### requests.put(url, data=None, **kwargs)

- url: 拟更新页面的url链接 
- data : 字典、字节序列或文件，Request的内容 
- **kwargs: 12个控制访问的参数

```
>>> payload={'key1': 'value1','key2':'value2'} 
>>> r=requests.put('http://httpbin.org/put',data= payload) 
>>> print(r.text) 
```

### requests.patch(url,data=None, **kwargs)

- url: 拟更新页面的url链接 
- data : 字典、字节序列或文件，Request的内容 
- **kwargs: 12个控制访问的参数

### requests.delete(url,**kwargs)

- url: 拟删除页面的url链接 
- **kwargs: 12个控制访问的参数

## 通用代码框架

```
import requests
def getHTMLText(url)
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apprent_encoding
        return r.rext
    except:
        return "产生异常"

-- 对于有些安全高的网站，需使爬虫模拟浏览器
import requests
def getHTMLText(url)
    try:
        kv = {'user-agent':'Mozilla/5.0'}       #使头部访问端为Mozilla标准浏览器身份标识
        r = requests.get(url, timeout=30, headers=kv)
        r.raise_for_status()
        r.encoding = r.apprent_encoding
        return r.rext
    except:
        return "产生异常"
```


整理自 [Python网络爬虫与信息提取-嵩天](https://www.icourse163.org/learn/BIT-1001870001#/learn/content)
参考链接：
http://www.imooc.com/article/36467
https://developer.mozilla.org/zh-CN/docs/Learn/Server-side/First_steps/Client-Server_overview(HTTP入门)