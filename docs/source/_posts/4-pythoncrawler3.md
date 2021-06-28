---
title: python爬虫学习笔记Ⅲ：信息标记
date: 2019-12-12 23:25:00
categories: notes
tags:
 - clawer
 - Python
photo: 
    - /blog/img/crawler.jpg
---
<br>
<!--more-->

# 三种信息标记

## XML   

+ 定义：eXtensible Markup Lauguage 扩展标记语言
+ 表达形式：
```
有内容双标签:
<name>...</name>
无内容单标签:
<name />
注释:       
<!-- 注释 -->
```
+ 特点：最早的通用标记语言，可扩展性好，繁琐；internet上的信息交互与传递

## JSON    

+ 定义：JavaScript Object Notation  JavaScript语言中对面向对象的一种表达形式
+ 表达形式：
```
有类型键值对:
"key" : "value" 
多值:
"key" : ["value1", "value2"] 
键值对嵌套:            
"key" : {                      
         "subkey1" : "subvalue1", 
         "subkey2" : "subvalue2"
        }   
```
+ 特点：信息有类型，适合程序处理(js)，较XML简洁；移动应用云端和节点的信息通信，无注释

## TAML

+ 定义：YAML Ain't Markup Language  
+ 表达形式：
```
无类型键值对:        
key : value
缩进表所属关系:
key :
    subkey : subvalue
并列关系:            
key :
- value1
- value2
表示下方整块数据:
key : |
...
注释:
key = value  #注释
```
+ 特点：信息误类型，文本信息比例最高，可读性好；各类系统的配置文件，有注释易读

# 基于bs4库的内容查找方法

<>.find_all(name, attrs, recursive, string, **kwargs)

name: 对标签名称的检索字符串
attrs: 对标签属性值的检索字符串，可标注属性检索
recursive: 是否对子孙全部检索，默认True
string: <>...</>中字符串区域的检索字符串

扩展：
方法 | 说明
---- | ----
<>.find() | 搜索且只返回-个结果，字符串类型，同.find all()参数
<>.find_parents() | 在先辈节点中搜索，返回列表类型，同.find_all()参数
<>.find_parent() | 在先辈节点中返回一个结果，字符串类型，同.find()参数
<>.find_next_siblings() | 在后续平行节点中搜索，返回列表类型，同.find_all()参数
<>.find_next_sibling() | 在后续平行节点中返回一个结果，字符串类型，同.find()参数
<>.find_previous_siblings() | 在前序平行节点中搜索，返回列表类型，同.find_all()参数
<>.find_previous_sibling() | 在前序平行节点中返回一个结果，字符串类型，同.find()参数

# 实践：爬取中国大学排名

1. 从网络上获取大学排名网页内容 getHTMLText()
2. 提取网页内容中信息到合适的数据结构 fillUnivList()
3. 利用数据结构展示并输出结果 printUnivList()

## 构建函数框架

```python
import requests
from bs4 import BeautifulSoup

def getHTMLText(url):
    return ""

def fillUnivList(ulist, html):
    pass

def printUnivList(ulist, num):
    print("Suc" + str(num))

def main():
    uinfo = []
    url = 'http://www.zuihaodaxue.cn/zuihaodaxuepaiming2019.html'
    html = getHTMLText(url)
    fillUnivList(uinfo, html)
    printUnivList(unifo, 20)
main()
```

## 填充具体函数

```python
import requests
# from bs4 import BeautifulSoup
import bs4

def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return ""

def fillUnivList(ulist, html):
   soup = BeautifulSoup(html, "html.parser")
   for tr in soup.find('tbody').children:
        if isinstance(tr, bs4.element.Tag):
            tds = tr('td')
            ulist.append([tds[0].string, tds[1].string, tds[3].string])

def printUnivList(ulist, num):
    print("{:^5}\t{:<8}\t{:^5}".format("排名","学校名称","总分"))
    for i in range(num):
        u=ulist[i]
        print("{:^7}\t{:<8}\t{:^6}".format(u[0],u[1],u[2]))

def main():
    uinfo = []
    url = 'http://www.zuihaodaxue.cn/zuihaodaxuepaiming2019.html'
    html = getHTMLText(url)
    fillUnivList(uinfo, html)
    printUnivList(uinfo, 20)      # list 20 univs
main()
```

其中，可以对printUnivList()进行优化，使输出间距更整齐
采用中文字符空格填充chr(12288)
```python
def printUnivList(ulist, num):
    tplt = "{0:^10}\t{1:{3}^10}\t{2:^10}"           # 定义变量tplt控制表头及数值的间距
    print(tplt.format("排名","学校名称","总分",chr(12288)))
    for i in range(num):
        u=ulist[i]
        print(tplt.format(u[0],u[1],u[2],chr(12288)))
```