---
title: python爬虫学习笔记Ⅴ：爬取中国大学mooc课件
date: 2020-2-22 13:25:00
categories: practice
tags:
 - clawer
 - Python
photo: 
    - /blog/img/crawler.jpg
---

<br>
<!--more-->

# Fiddler4 捕捉网页

1. 在请求慕课课程的 **课件页** 时，得到url:

/dwr/call/plaincall/CourseBean.getMocTermDto.dwr 
返回信息:
```
callCount=1
scriptSessionId=${scriptSessionId}190
httpSessionId=dd4e460b2323448891d89323c39ba8e8
c0-scriptName=CourseBean
c0-methodName=getMocTermDto
c0-id=0
c0-param0=number:1003243006
c0-param1=number:0
c0-param2=boolean:true
batchId=1582376585294
```

2. 在请求慕课课程的 **文档页** 时，得到url:

/dwr/call/plaincall/CourseBean.getLessonUnitLearnVo.dwr 
返回信息：

```
callCount=1
scriptSessionId=${scriptSessionId}190
httpSessionId=dd4e460b2323448891d89323c39ba8e8
c0-scriptName=CourseBean
c0-methodName=getLessonUnitLearnVo
c0-id=0
c0-param0=number:1008595152
c0-param1=number:3
c0-param2=number:0
c0-param3=number:1005742655
batchId=1582376585380
```

# 编写爬虫

```python
#coding=utf-8
'''
需安装certifi, chaedet, idna, requests, urllib3
'''
import requests
from bs4 import BeautifulSoup
import re
import os

# 网站课程资源
# 网站课件资源
# 课程链接
header = {'User-Agent': 'Mozilla/5.0'}
source_url = 'http://www.icourse163.org/dwr/call/plaincall/CourseBean.getMocTermDto.dwr'
resource_url = 'http://www.icourse163.org/dwr/call/plaincall/CourseBean.getLessonUnitLearnVo.dwr'
page_url = 'https://www.icourse163.org/learn/'

def get_course_info():
    course_id = input("\n请输入课程id（例如BIT-268001）:") 
    course_url = page_url + course_id
    course_info = requests.get(course_url, headers=header)
    # 匹配meta标签中的课程信息  .*?表示前一个字符0次或无限次扩展，最小匹配
    info_compile = re.compile(r'<meta name="description" .*?content=".*?,(.*?),(.*?),.*?/>')
    info_content = re.search(info_compile, course_info.text)
    course_name = info_content.group(1)
    course_college = info_content.group(2)
    print("\n")
    print('您选择的是：')
    print(course_name + '\n' + course_college)

def get_course_source():
    course_tid = input("\n请输入当前课程tid（例如1003243006）:")
    # 获取课件数据
    courseware_data = {
        'callCount': '1',
        'scriptSessionId': '${scriptSessionId}190',
        'httpSessionId': 'dd4e460b2323448891d89323c39ba8e8',
        'c0-scriptName': 'CourseBean',
        'c0-methodName': 'getMocTermDto',
        'c0-id':'0',
        'c0-param0': 'number:' + course_tid,
        'c0-param1': 'number:0',
        'c0-param2': 'boolean:true',
        'batchId': '1582376585294'
    }
    # 向source_url提交post请求，返回data数据
    # 对内容进行解码
    source_info = requests.post(source_url, data=courseware_data, headers=header)
    source_info_transcoding = source_info.text.encode('utf-8').decode('unicode_escape')
    # 正则匹配符合data的内容
    courseware_compile = re.compile(r'homeworks=.*?;.+id=(\d+).*?name="(.*?)";')
    courseware_content = re.findall(courseware_compile, source_info_transcoding)
    with open('TOC.txt', 'w', encoding='utf-8') as file:
        # 遍历所有一级目录(章)id和name并写入目录
        for index, chapter in enumerate(courseware_content):
            file.write('%s    \n' % (chapter[1]))
            # 这里id为二级目录id
            lesson_compile = re.compile(r'chapterId=' + chapter[0] + r'.*?contentType=1.*?id=(\d+).+name="(.*?)".*?test')
            # 查找所有二级目录id和name
            lesson_content = re.findall(lesson_compile, source_info_transcoding)
            # 遍历所有二级目录(节)id和name并写入目录
            for sub_index, section in enumerate(lesson_content):
                file.write('　%s    \n' % (section[1]))
                # 查找二级目录下文档，并返回 [contentid,contenttype,id,name]
                pdf_compile = re.compile(r'contentId=(\d+).+contentType=(3).+id=(\d+).+lessonId=' + section[0] + r'.+name="(.+)"')
                pdf_content = re.findall(pdf_compile, source_info_transcoding)
                pdfname_compile = re.compile(r'^[第一二三四五六七八九十\d]+[\s\d\._章课节讲]*[\.\s、]\s*\d*')
                # 遍历二级目录下pdf集合，写入目录并下载
                for pdf_index, pdf in enumerate(pdf_content):
                    rename = re.sub(pdfname_compile, '', pdf[3])
                    file.write('　　[文档] %s \n' % (rename))
                    get_course_content(pdf, '%d.%d.%d [文档] %s' % (index + 1, sub_index + 1, pdf_index + 1, rename))

def get_course_content(num, name, *args):
    # 检查文件命名，防止网站资源有特殊字符本地无法保存
    document_compile = re.compile(r'[\\/:\*\?"<>\|]')
    name = re.sub(document_compile, '', name)
    # 检查文件是否下载
    if os.path.exists('PDFs\\' + name + '.pdf'):
        print(name + "------------->已下载")
        return
    # 获取具体文件数据
    document_data = {
        'callCount': '1',
        'scriptSessionId': '${scriptSessionId}190',
        'httpSessionId': 'dd4e460b2323448891d89323c39ba8e8',
        'c0-scriptName': 'CourseBean',
        'c0-methodName': 'getLessonUnitLearnVo',
        'c0-id': '0',
        'c0-param0': 'number:' + num[0],	# 二级目录id
        'c0-param1': 'number:' + num[1],	# 判定资源类型，3为pdf
        'c0-param2': 'number:0',
        'c0-param3': 'number:' + num[2],	# 具体资源id
        'batchId': '1582376585380'
    }
    # 向resource_url提交post请求，返回data数据
    resource_info = requests.post(resource_url, headers=header, data=document_data).text
    # 对于pdf文档，c0-param1的数字为3，pdf下载地址为textOrigUrl
    if num[1] == '3':
        pdf_info = re.search(r'textOrigUrl:"(.*?)"', resource_info).group(1)
        print('正在下载:' + name + '.pdf')
        pdf_file = requests.get(pdf_info, headers=header)
        # 创建PDFs文件夹，将下载pdf存放其中
        if not os.path.isdir('PDFs'):
            os.mkdir(r'PDFs')
        with open('PDFs\\' + name + '.pdf', 'wb') as file:
                file.write(pdf_file.content)

def main():
    get_course_info()
    get_course_source()
main()
```
<br>