---
title: python爬虫学习笔记Ⅳ：正则表达式
date: 2020-2-13 13:25:00
categories: notes
tags:
 - clawer
 - Regex
 - Python
photo: 
    - /blog/img/crawler.jpg
---

<br>
<!--more-->

# 概念

regular expression，regex，RE
正则表达式是用来简介表达一组字符串的表达式，是一种通用的字符串表达框架。

# 使用

编译：将符和正则表达式语法的字符串转换成正则表达式特征。而这个特征就可以表达这一组字符串。

# 语法

正则表达式由字符和操作符组成

## 常用操作符

```re
操作符  说明                             实例
--------------------------------------------------------------------------------
.    |  表示任何单个字符                  
[]   |  字符集，对单个字符给出取值范围     [abc] 表示a、b、c,[a-z]表示a到z单个字符
[^ ] |  非字符集，对单个字符给出排除范围   [^abc] 表示非a或b或c的单个字符
*    |  前一个字符0次或无限次扩展          abc* 表示ab、abc、abcc、abccc...
+    |  前一个字符1次或无限次扩展          abc+ 表示abc、abcc、abccc...
?    |  前一个字符0次或1次扩展             abc? 表示ab、abc
|    |  左右表达式任意一个                abc|def 表示abc、def
{m}  |  扩展前一个字符m次                 ab{2}c 表示abbc
{m,n}|  扩展前一个字符m至n次              ab{1,2} 表示abc、abbc
^    |  匹配字符串开头                    ^abc 表示abc且在一个字符串的开头
$    |  匹配字符串结尾                    abc$ 表示abc且在一个字符串的结尾
( )  |  分组标记，内部只能使用`|`操作符    (abc) 表示abc,(abc|def)表示abc、def
\d   |  数字，等价于[0-9]            
\w   |  单词字符，等价于[A-Za-zo-9_]  
```

## 经典正则表达式实例

```re
^[A‐Za‐z]+$                 由26个字母组成的字符串
^[A‐Za‐z0‐9]+$              由26个字母和数字组成的字符串
^‐?\d+$                     整数形式的字符串
^[0‐9]*[1‐9][0‐9]*$         正整数形式的字符串
[1‐9]\d{5}                  中国境内邮政编码
[\u4e00‐\u9fa5]             匹配中文字符
\d{3}‐\d{8}|\d{4}‐\d{7}     国内电话号码,010-68913536
```

# Re库

## 表示类型

1. raw string类型(原生字符串类型)
r后单引号内填写内容：`r'[1-9]\d{5}'`    不用写转义符

2. string类型
直接表示'[1-9]\\d{5}'    更繁琐，当正则表达式包含转义符时，使用raw string

## 主要功能函数

函数 | 说明
--- | ---
re.search()   | 在一个字符串中搜索匹配正则表达式的第一个位置，返回**match对象**
re.match()    | 从一个字符串的开始位置起匹配正则表达式，返回**match对象**
re.findall()  | 搜索字符串，以列表类型返回全部能匹配的子串
re.split()    | 将一个字符串按照正则表达式匹配结果进行分割，返回列表类型
re.finditer() | 搜索字符串，返回一个匹配结果的迭代类型，每个**迭代元素是match对象**
re.sub()      | 在一个字符串中替换所有匹配正则表达式的子串，返回替换后的字符串

### re.search(pattern, string, flags=0)

在一个字符串中搜索匹配正则表达式的第一个位置返回match对象
+ pattern ：正则表达式的字符串或原生字符串表示
+ string  ：待匹配字符串
+ flags   ：正则表达式使用时的控制标记

### re.split(pattern, string, maxsplit=0, flags=0)

+ maxsplit ：最大分割数，剩余部分作为最后一个元素输出

### re.sub(pattern, repl, string, count=0, flags=0)

+ repl  ：替换匹配字符串的字符串
+ count ：匹配最大替换次数

## Re库的另一种等价用法

+ 函数式用法
```python
>>> rst = re.search(r'[1-9]\d{5}', 'BIT 100081')
```

+ 面向对象用法
```python
>>> pat = re.compile(r'[1-9]\d{5}')     #对pat多次使用时
>>> rst = pat.search('BIT 100021')
```

### regex = re.compile(pattern, flags=0)

将正则表达式的字符串形式编译成正则表达式对象
+ pattern ：正则表达式的字符串或原生字符串表示
+ flags   ：正则表达式使用时的控制标记

## Re库的Match对象

Match对象是一次匹配的结果，包含匹配的很多信息
```python
>>> match = re.search(r'[1-9]\d{5}', 'BIT 100081')
>>> if match:
        print(match.group(0))

100081                          #输出
>>> type(match)
<class '_sre.SRE_Match'>        #输出
```
### Match对象的属性

属性 | 说明
---- | ----
.string | 待匹配的文本
.re     | 匹配时使用的pattern对象(正则表达式)
.pos    | 正则表达式搜索文本的开始位置
.endpos | 正则表达式搜索文本的结束位置

### Match对象的方法

方法 | 说明
---- | ----
.group(0) | 获得匹配后的字符串
.start()  | 匹配字符串在原始字符串的开始位置
.end()    | 匹配字符串在原始字符串的结束位置
.span()   | 返回(.start(), .end())

### Match对象的实例

```python
>>> import re
>>> m = re.search(r'[1-9]\d{5}', 'BIT100081 TSU100084')
>>> m.string
'BIT100081 TSU100084'
>>> m.re
re.compile('[1-9]\\d{5}')
>>> m.pos
0
>>> m.endpos
19
>>> m.group(0)
'100081'
>>> m.start()
3
>>> m.end()
9
>>> m.span()
(3, 9)
```

## Re库的贪婪匹配和最小匹配

1. 贪婪匹配

Re库默认采用贪婪匹配，即输出匹配最长的子串，例：
```python
>>> match = re.search(r'PY.*N', 'PYANBNCNDN')
>>> match.group(0)
'PYANBNCNDN'               #输出最长的符和正则表达式的子串
```

2. 最小匹配

采用最小匹配操作符：

操作符  | 说明
------- | ---
*?      | 前一个字符0次或无限次扩展，最小匹配
+?      | 前一个字符1次或无限次扩展，最小匹配
??      | 前一个字符0次或1次扩展，最小匹配
{m,n}?  | 扩展前一个字符m至n次 (含n) ，最小匹配

例：
```python
>>> match = re.search(r'PY.*?N', 'PYANBNCNDN') 
>>> match.group(0) 
'PYAN'
```