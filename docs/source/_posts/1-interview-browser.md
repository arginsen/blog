---
title: 面试 | 浏览器
date: 2020-8-29
tags: 
  - interview
  - Frontend
categories: notes
photos:
    - /blog/img/interview.jpg
---


<br>
<!--more-->

# cookie、localStorage、sessionStorage

cookie 是网站用来标识用户身份而存储在用户本地终端上的数据，cookie 数据始终在同源的http请求中携带，会在浏览器和服务器间传递
localStorage、sessionStorage 仅在本地保存，不参与服务器通信


相同点：都保存在客户端。
不同点：有效期、大小、服务器端通信方面
cookie：可设置有效期，默认浏览器关闭失效。大小4K。参与服务器端通信，携带在请求头里。
localStorage：永久有效，除非手动删除。大小5M。不参与服务器端通信。
sessionStorage：当前会话期有效。大小5M。不参与服务器端通信。


# 输入 url 到页面加载发生了些什么

1. 解析域名：搜索本地浏览器 DNS 服务器，如没用则搜索系统的，再是路由器，运营商
2. TCP 连接：三次握手、四次挥手
3. 发送请求报文
4. 接受响应
5. 渲染页面：解析 HTML，生成 DOM 树；解析 CSS，生成 CSSOM 树；结合 DOM 和 CSSOM，渲染树；回流（layout），得到各元素几何信息；重绘（painting），得到绝对像素；展示（display），将像素发给 GPU，渲染至页面

# get 和 post 有哪些区别

区别 | GET | POST
---- | --- | ---
大小 | GET 传输一般 2K-8K，IE 限制 2K | 无限制
安全 | url 明文传输，不安全 | 通过 body 传输，不安全
编码方式 | url 编码 | 支持多种编码
TCP 数据包 | 1个数据包 | 2个数据包
浏览器记录 | 浏览器会记录 | 不会记录
浏览器后退 | 无影响 | 再次提交
浏览器收藏 | 可收藏 | 不可
浏览器缓存 | 可缓存 | 不可
使用方式 | 获取数据 | 提交保存数据

