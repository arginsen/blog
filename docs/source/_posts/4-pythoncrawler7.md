---
title: python爬虫学习笔记Ⅶ：Scrapy框架
date: 2020-3-2 11:25:00
categories: notes
tags:
 - clawer
 - Scrapy
 - Python
photo: 
    - /blog/img/crawler.jpg
---

<br>
<!--more-->

# Scrapy爬虫框架结构(5+2)

## 5

+ Engine
    - 控制所有木块之间的数据流
    - 根据条件触发事件

+ Downloader    
    - 根据请求下载网页

+ Scheduler
    - 对所有爬取请求进行调度管理

+ Spider    需用户编辑
    - 解析Downloader返回的响应(Response)
    - 产生爬取项(scraped item)
    - 产生额外的爬取请求(Request)

+ Item Pipelines    需用户编写
    - 以流水线方式处理Spider产生的爬取项
    - 由一组操作顺序组成，类似流水线，每个操作是一个Item Pipeline类型
    - 可能操作包括：清理、检查和查重爬取项中的HTML数据、将数据存储到数据库

## 2 - 中间件

+ Download MIddleware   用户可编辑
    - 目的：实施Engine、Scheduler和Downloader之间进行用户可配置的控制
    - 功能：修改、丢弃、新增请求或响应

+ Spider Middleware     用户可编辑
    - 目的：对请求和爬取项的再处理
    - 功能：修改、丢弃、新增请求或爬取项

# Scrapy常用命令

命令 | 说明 | 格式
---- | ---- | ----
startproject | 创建一个新工程 | `scrapy startproject <name> [dir]`
genspider | 创建一个爬虫 | `scrapy genspider [options] <name> <domain> `
settings | 获得爬虫配置信息 | `scrapy settings [option]`
crawl | 运行一个爬虫 | `scrapy crawl <spider>`
list | 列出工程中所有爬虫 | `scrapy list` 
shell | 启动URL调试命令行 | `scrapyshell [url]`

# 实例-建立股票爬虫

1. 建立工程和Spider模板
```
> scrapy startproject BaiduStocks
> cd BaiduStocks
> scrapy genspider stocks baidu.com
> vim spider/stocks.py
```

2. 编写Spider
```python
def parse_stock(self, response):
    indoDict = {}
    stockIndo = response.css('.stock-bets')
    name = stockInfo.css('dt').extract()[0]
    keyList = stockInfo.css('dd').extract()
    for i in range(len(keyList)):
        key = re.findall(r'\d+\.?.*</dd?>', keyList[i])[0][0:-5])
        try:
            val = re.findall(r'\d+\.?.*</dd?>', valueList[i])[0][0:-5])
        except:
            val = '--'
        infoDict[key] = val
    
    infoDict.update(
        {'股票名称': re.findall('\s.*\(', name)[0].split()[0] + \
        re.findall('\>/*\<', name)[0][1:-1]})
    
    yield infoDict
```

3. 编写Item Pipelines
```python
# 配置pipelines.py
class BaidustocksPipeline(object):
    def process_item(self, item, spider):
        return item

class BaidustocksInfoPipeline(object):
    def open_spider(self, spider):
        self.f = open('BaiduStockInfo.txt', 'w')

    def close_spider(self, spider):
        try:
            line = str(dict(item)) + "\n"
            self.f.write(line)
        except:
            pass
        return item

# 配置settings.py中pipelines选项
ITEM_PIPELINES = {
    'BaiduStocks.pipelines.BaidustocksInfoPipeline': 300,
}
```

4. 执行爬虫
```
> scrapy crawl stocks
```
