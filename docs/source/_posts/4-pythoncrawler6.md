---
title: python爬虫学习笔记Ⅵ：爬取淘宝物品信息
date: 2020-2-25 11:25:00
categories: practice
tags:
 - clawer
 - Python
photo: 
    - /blog/img/crawler.jpg
---

<br>
<!--more-->

# 爬虫框架

```python
import requests
import re

def getHTMLText(url):
    print("")

def parsePage(list, html):
    print("")

def printFoodsList(list):
    print("")

def main():
    goods = '口红'
    depth = 2
    start_url = 'https://s.taobao.com/search?q=' + goods
    infoList = []
    for i in range(depth):
        try:
            url = start_url + '&s=' + str(44*i)
            html = getHTMLText(url)
            parsePage(infoList, html)
        except:
            continue
    printGoodsList(infoList)

main()
```

# 函数填充

```python
import requests
import re

def getHTMLText(url, header):
    try:
        r = requests.get(url, timeout=30, headers = header)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return ""


def parsePage(ilt, html):
    try:
        plt = re.findall(r'\"view_price\"\:\"[\d\.]*\"', html)
        tlt = re.findall(r'\"raw_title\"\:\".*?\"', html)
        for i in range(len(plt)):
            price = eval(plt[i].split(':')[1])
            title = eval(tlt[i].split(':')[1])
            ilt.append([price, title])
    except:
        print("")


def printGoodsList(ilt):
    tplt = "{:4}\t{:8}\t{:16}"
    print(tplt.format("序号", "价格", "商品名称"))
    count = 0
    for g in ilt:
        count = count + 1
        print(tplt.format(count, g[0], g[1]))


if __name__ == '__main__':
    goods = '口红'
    header = {
          'cookie' : '_samesite_flag_=true; cookie2=1a373cd6416e792db100c45aba4e09bc; t=b4fedc3a591217d49f5a99397136932f; _tb_token_=5e1e53531ae65; sgcookie=DZ6%2FLQpGvGYIIeqkGRcZ2; enc=VOV4YnQ%2FmGdtuKcljRFVEeq3iv4JRcXNAt2ohVXodoH0DDKPTQVM7%2FeF4uMoOzLOxg505fKFIn3jqmI%2BpYYKeg%3D%3D; hng=CN%7Czh-CN%7CCNY%7C156; thw=cn; mt=ci=0_0; cna=Kjc7FitxIjoCAa8IpCr0dnC+; tfstk=cTocB7gdFqzX9B3NLiZjCxhBxRrGZB84Gcor4mc1a6zbKDmPiT7P8jXJEShmXN1..; v=0; JSESSIONID=CD31AF9B1B2FDB1D4A69B180B1355F77; alitrackid=www.taobao.com; lastalitrackid=www.taobao.com; l=dBNRER34QiOrAguDBOCNIAyCEc7OSIRx5u8ZR8qvi_5Zl6L1n1QOo8LwFFp62jWfTBTB4Tn8Nrv9-etkiKy06Pt-g3fPaxDc.; isg=BEhIJqPWgj-QHe67Kq0A9-I1GbZa8az7Z-WyHwL5lEO23ehHqgF8i97fULWtEGTT',
          'user-agent' : 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36'
    }
    depth = 3
    start_url = 'https://s.taobao.com/search?q=' + goods
    infoList = []
    for i in range(depth):
        try:
            url = start_url + '&s=' + str(44 * i)
            html = getHTMLText(url)
            parsePage(infoList, html)
        except:
            continue
    printGoodsList(infoList)
```

# 参考链接

https://www.icourse163.org/learn/BIT-1001870001?tid=1003245012#/learn/content?type=detail&id=1004574449
https://blog.csdn.net/pursuingparadise/article/details/104345676