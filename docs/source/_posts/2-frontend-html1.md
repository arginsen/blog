---
title: 前端 | html学习笔记Ⅰ：标签
date: 2019-4-6
categories: notes
tags:
 - html
 - Frontend
photo: 
    - /blog/img/html.jpeg
---

# 前言

* Html是网页内容的载体

* CSS样式是表现，就是用来改变内容外观的东西

* JavaScript是用来实现网页的特效（交互）

<!-- more -->

# 各标签

```html
<html> 根标签

    <head> 用于定义文档的头部，是头部元素的容器

    头部元素：

        <title>网页的标题信息-用于告诉用户和搜索引擎这个网页的主要内容
        <script>用于定义客户端脚本,如JavaScript
        <style>定义html样式，如是外部样式表需用<link>
        <link>插入链接
        <meta>描述HTML文档的元数据

 ---
 
    <body>是网页的主要内容

    网页内容标签：
        <h1>标题标签  1为一级标题，共有1-6
        <p>段落标签    
        <a>
 
    修饰标签：
        <img>插入图片
            <img src="图片地址"  alt="下载失败时替换文本"  title="鼠标滑过提示文本">

        <!--注释文字--> html里的注释语法-等同于python里的#
        <em>斜体</em>  或<i></i>
        <strong>粗体  或 <b></b>
        <sup>上标
        <sub>下标
        <ins>下划线
        <del>删除线

        <span>在段落中对个别需强调的设置单独的样式-在<head>标签里进行     
        <style>span{}设置
        </span>

        <q>引用短文本-不需加双引号
        <blockquote>引用长文本


    功能性标签：
        <hr/> 分割线
        <br/> 换行---xhtml1.0版本
        &nbsp; 空格
        <hr/> 水平横线
        <address>地址信息、电子邮件地址、签名


    代码标签：
        <code>代码语言</code>
        <pre>语言代码段-可以保留换行空格</pre>


    列表类标签：
        <ul>无顺序信息列表
            <li>信息1</li>
            <li>信息2</li>
        </ul>

        <ol>有顺序信息列表
            <li>信息1</li>
        </ol>

        <dl>自定义列表（项目+注释）
            <dt>定义一个描述列表的项目/名字</dt>
            <dd>描述每一个项目</dd>
        </dl>


    <div>独立部分的容器</div>
    <div id="板块名称">...</div>


    表格标签：
        <table>关于表格
            <tbody>优先显示部分</tbody>
            <table summary="表格简介">
            <caption>表格标题</caption>
            <tr>表格的一行</tr>
            <th>表格头部的一个单元格</th>
            <td>表格的一个单元格</td>
        </table>


    链接标签:
        <a href="目标网址"  title="鼠标滑过显示的文本">
        添加链接的文本
        </a>

        <a href="目标网址”  target="_blank">
        添加链接的文本
        </a>

        <a href="mailto:目标地址?cc=抄送地址&bbc=密件抄送地址&subject=&body=邮件内容">
        发送
        </a>


    交互表单标签:
        <form method="数据传送的方式：post/get"   action="浏览者输入数据传送到php界面：save.php">
            <label for="username">用户名:</label>
            <input type="text" name="username" />
            <label for="pass">密码:</label>
            <input type="password" name="pass" />
        </form>

    </body>

</html>
```

<br>
参考链接：
https://www.imooc.com/learn/9
https://wiki.jikexueyuan.com/project/html5