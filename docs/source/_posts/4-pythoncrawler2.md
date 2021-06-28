---
title: python爬虫学习笔记Ⅱ：BeautifulSoup库
date: 2019-12-11 14:41:00
categories: notes
tags:
 - clawer
 - BeautifulSoup
 - Python
photo: 
    - /blog/img/crawler.jpg
---
<br>
<!--more-->

# Beautiful Soup的介绍

Beautiful Soup是一个Python库，旨在用于快速的周转项目，例如屏幕抓取。三个功能使其强大：

- Beautiful Soup提供了一些用于导航，搜索和修改解析树的简单方法和Pythonic习惯用法：用于剖析文档并提取所需内容的工具箱。编写应用程序不需要太多代码
- Beautiful Soup会自动将传入文档转换为Unicode，将传出文档转换为UTF-8。您不必考虑编码，除非文档未指定编码并且Beautiful Soup无法检测到编码。然后，您只需要指定原始编码即可。
- Beautiful Soup位于流行的Python解析器（如lxml和html5lib）的顶部，使您可以尝试不同的解析策略或提高灵活性。

# Beautiful Soup库解析器

```python
import requests
from bs4 import BeautifulSoup
r.requests.get("http://python123.io/ws/demo.html")  #请求网页
demo = r.text
soup = BeautifulSoup(demo，'html.parser')              #指定变量soup代替解析的demo
```

解析器 | 使用方法 | 条件 
------|------|------
bs4的HTML解析器 | BeautifulSoup(mk,'html.parser') | 安装bs4库 
lxml的HTML解析器| BeautifulSoup(mk,'lxml')        | pip install lxml 
lxml的XML解析器 | BeautifulSoup(mk,'xml')         | pip install lxml 
html5lib的解析器| BeautifulSoup(mk,'html5lib')    | pip install html5lib


# BeautifulSoup库的基本元素

基本元素 | 说明 
--------|--------
Tag             |  标签，最基本的信息组织单元，分别用<>和</>标明开头和结尾 
Name            |  标签的名字，`<p>…</p>`的名字是'p'，格式：`<tag>.name` 
Attributes      |  标签的属性，字典形式组织，格式：`<tag>.attrs` 
NavigableString |  标签内非属性字符串，`<>…</>`中字符串，格式：`<tag>.string` 
Comment         |  标签内字符串的注释部分，一种特殊的Comment类型

## Tag标签

```python
soup = BeautifulSoup(demo, "html.parser")
soup.title                     #解析得到demo网页的<title>标签
soup.a                         #解析得到demo网页的<a>标签
soup.p                         #解析得到demo网页的<p>标签
```

## Tag的name(名字)

```python
soup = BeautifulSoup(demo, "html.parser")
soup.a.name                    #<a>标签的名字
soup.a.parent.name             #<a>标签的父级标签
soup.a.parent.parent.name      #<a>标签父级标签的父级标签
```

## Tag的attrs(属性)

```python
<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link">Basic Python</a>

soup = BeautifulSoup(demo, "html.parser")
soup.a.attrs                   #<a>标签的属性
soup.a.attrs['class']
type(soup.a.attrs)             #一个标签可以有多个属性，字典类型
type(soup.a)
```

## Tag的NavigableString(非属性字符串)

```python
soup.a
soup.a.string
soup.p
soup.p.string
type(soup.p.string)
```

## Tag的Comment(字符串注释部分)

```python
newsoup = BeautifulSoup("<b><!--This is a comment--></b><p>This is not a comment</p>", "html.parser")
newsoup.b.string
type(newsoup.b.string)        #<class 'bs4.element.Comment'> b标签内的注释是一种特殊类型
newsoup.p.string
type(newsoup.p.string)        #<class 'bs4.element.NavigableString'>
```

# bs4库的HTML内容遍历

html的的遍历有三种：
- head标签下行遍历，html→head→title
- body标签上行遍历，b/a→p→body→html
- 相同标签的平行遍历

## 下行遍历

属性 | 说明 
-----|-----
.contents | 子节点的列表，将<tag>所有儿子节点存入列表 
.children | 子节点的迭代类型，与.contents类似，用于循环遍历儿子节点 
.descendants | 子孙节点的迭代类型，包含所有子孙节点，用于循环遍历

```python
soup = BeautifulSoup(demo, "html.parser")
soup.head               #head标签的所有内容
soup.head.contents      #head标签子标签内容，可有多个
soup.body.contents      #body标签子标签内容，可有多个
len(soup.body.contents) #body子标签有多少个，'\n'表示回车
```

## 上行遍历

属性 | 说明 
-----|-----
.parent  | 节点的父亲标签 
.parents | 节点先辈标签的迭代类型，用于循环遍历先辈节点

## 平行遍历

属性 | 说明 
-----|-----
.next_sibling     | 返回按照HTML文本顺序的下一个平行节点标签 .previous_sibling | 返回按照HTML文本顺序的上一个平行节点标签 
.next_siblings    | 迭代类型，返回按照HTML文本顺序的后续所有平行节点标签 .previous_siblings | 迭代类型，返回按照HTML文本顺序的前续所有平行节点标签

备注：平行遍历发生在同一个父节点下的各节点间，而head下的标签到body下的标签不能成为平行遍历。

# bs4库的HTML格式输出

## prettify()

```python
soup = BeautifulSoup(demo, "html.parser")
soup.prettify()            #利用\n进行分词
print(soup.prettify())     #通过print，更加工整的输出html结构
print(soup.a.prettify())   #对<a>标签美化输出
```

# BeautifulSoup库的实现
```python
import requests
from bs4 import BeautifulSoup

r.requests.get("http://python123.io/ws/demo.html")
r.status_code
r.text

soup = BeautifulSoup(r.text, 'html.parser')    #html.parser是bs4的html解析器，这里用一个变量soup代替
soup.title                  #title标签，最基本的信息组织单元
soup.title.name             #标签的名字
soup.title.parent.name      #父级标签的名字
soup.title.attrs            #标签的属性，字典形式组织
soup.title.string           #标签内非属性字符串
soup.p.string
type(soup.a)                #a里边的属性，可能含有多个，如a里的class、href
```



















