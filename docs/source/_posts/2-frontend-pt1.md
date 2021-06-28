---
title: 前端 | 练习一之电商页布局
date: 2020-4-13
tags: 
- CSS
- html
- Frontend
categories: practice
photos:
    - /blog/img/frontend.jpg
---
<br>
<!--more-->

>根据表叔所讲，先定大的框架，页面主要展示顶部导航栏，头部功能区域，主展示区，各分区，底部区域此五大板块；再先给大的框架调整初步样式；再依次给每个区域进行详细的填充。

# page

作品: [https://lixupeng.cn/E-commerce_page_layout](https://lixupeng.cn/E-commerce_page_layout)
项目地址: [https://github.com/E-commerce_page_layout](https://github.com/E-commerce_page_layout)

# code

## html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdn.bootcdn.net/ajax/libs/normalize/8.0.1/normalize.css">
    <link rel="stylesheet" href="http://cdn.bootcss.com/normalize/5.0.0/normalize.min.css">
    <link rel="stylesheet" href="main.css">
    <title>蘑菇街 - 你的剁手街 </title>
</head>
<body>
    <!-- 顶部导航栏 -->
    <div class="top-nav">
        <div class="container">
            <div class="fl">
                <a href="#" class="item">首页</a>
            </div>
            <div class="fr">
                <a href="#" class="item">我的订单</a>
                <a href="#" class="item">收藏夹</a>
                <a href="#" class="item">登录</a>
                <a href="#" class="item">注册</a>
                <a href="#" class="item">客户服务</a>
            </div>
        </div>
    </div>

    <!-- 头部功能区 -->
    <div class="header clear-float">
        <div class="container">
            <div class="logo col-2"></div>
            <div class="search-bar col-5">
                <input type="text" class="col-8">
                <button class="col-2">搜索</button>
            </div>
            <div class="cart col-3">
                <div class="trigger">
                    <a href="#cart">我的购物车</a>
                </div>
            </div>
        </div>
    </div>

    <!-- 主展示区 -->
    <div class="main-promote clear-float">
        <div class="container">
            <div class="cat col-2">
                <div class="row">item1 / item1</div>
                <div class="row">item2 / item2</div>
                <div class="row">item3 / item3</div>
                <div class="row">item4 / item4</div>
                <div class="row">item5 / item5</div>
                <div class="row">item6 / item6</div>
                <div class="row">item7 / item7</div>
                <div class="row">item8 / item8</div>
                <div class="row">item9 / item9</div>
                <div class="row">item10 / item10</div>
                <div class="row">item10 / item10</div>
            </div>
            <div class="col-6">
                <div class="slide">
                    <img src="./img/slider_02.png" alt="">
                </div>
                <div>
                    <div class="col-5">
                        <img src="./img/001.png" alt="">
                    </div>
                    <div class="col-5">
                        <img src="./img/002.png" alt="">
                    </div>
                </div>
            </div>
            <div class="col-2 info">
                <div class="auth clear-float">
                        <div class="avatar"></div>
                        Yo, 欢迎来到香菇街！
                </div>
                <div class="anno">
                    <div class="title">公告</div>
                    <div class="content">
                        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ab debitis id illum modi necessitatibus obcaecati. Autem doloremque explicabo itaque magnam, minima nam nostrum odit qui quos rerum sed similique velit! Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ab debitis id illum modi necessitatibus obcaecati. Autem doloremque explicabo itaque magnam, minima nam nostrum odit nam nostrum odit qui quos rerum sed similique velit!
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- 小分区 -->
    <div class="container">
        <div class="cat-promote clear-float">
            <div class="title">女装</div>
            <div class="content">
                <div class="col-2 item">
                    <div class="card c1"></div>
                </div>
                <div class="col-3 item">
                    <div class="card c2"></div>
                </div>
                <div class="col-2 item">
                    <div class="card c3"></div>
                </div>
                <div class="col-3 item">
                    <div class="card c4"></div>
                </div>
            </div>
            <div class="title">男装</div>
            <div class="content">
                <div class="col-3 item">
                    <div class="card c5"></div>
                </div>
                <div class="col-4 item">
                    <div class="card c6"></div>
                </div>
                <div class="col-3 item">
                    <div class="card c7"></div>
                </div>
            </div>
        </div>
    </div>

    <!-- 底部标签 -->
    <div class="footer clear-float">
        <div class="container">
            <div class="fl footer-info">
                <a href="#">分类浏览</a>
                <a href="#">使用条款</a>
                <a href="#">销售条款</a>
                <a href="#">隐私政策</a>
            </div>
            <div class="fr footer-id">&copy 2020 Arginsen</div>
        </div>
    </div>
</body>
</html>
```

## css

```css
* {
    /*background: rgba(0, 0, 0, .1);*/
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
    -webkit-transition: background 200ms;
    -moz-transition: background 200ms;
    -ms-transition: background 200ms;
    -o-transition: background 200ms;
    transition: background 200ms;
}

/* 栅栏 */
.col-1,
.col-2,
.col-3,
.col-4,
.col-5,
.col-6,
.col-7,
.col-8,
.col-9,
.col-10 {
    position: relative;
    min-width: 1px;
    float: left;
}

.col-1 {
    width: 10%;
}
.col-2 {
    width: 20%;
}
.col-3 {
    width: 30%;
}
.col-4 {
    width: 40%;
}
.col-5 {
    width: 50%;
}
.col-6 {
    width: 60%;
}
.col-7 {
    width: 70%;
}
.col-8 {
    width: 80%;
}
.col-9 {
    width: 90%;
}
.col-10 {
    width: 100%;
}

.container:before,
.container:after,
.clear-float:before,
.clear-float:after {
    content: "";
    display: block;
    clear: both;
}


.fl {
    float: left;
}

.fr {
    float: right;
}

a {
    text-decoration: none;
    color: #555;
}

a:hover {
    color: #333;
}

img {
    display: block;
    max-width: 100%;
}
/* 主页面 */
body {
    margin: 0 auto;
    background: #f5f5f5;
    font-size: 14px;
    font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
    color: #555;
    line-height: 1.7;
}

/* top-nav区 */
.top-nav {
    background: #eee;
}

.top-nav .item {
    display: inline-block;
    padding: 6px 20px;
    color: #666;
}

.top-nav .item:hover {
    color: #999;
    text-decoration: none;
}

.container {
    max-width: 1080px;
    margin: 0 auto;
    display: block;
}

/* header区 */
.header {
    padding: 20px 0;
}

.header .logo {
    background-image: url(./img/logo.png);
    background-size: contain;
    background-repeat:no-repeat;
    background-position: center center;
    height: 60px;
}

.header .search-bar {
    border: 2px solid #dd182b;
    margin-top: 7px;
}

.header .search-bar input,
.header .search-bar button {
    padding: 8px 4px;
    border: 0;
    outline: 0;
}

.header .search-bar input:focus {
    box-shadow: inset 0 0 1px 1px rgb(0,0,0,0.2);
}

.header .search-bar button {
    background: #dd182b;
    color: #fff;
}

.header .search-bar button:hover {
    background: #ce1829;
}

.header .cart .trigger {
    padding: 6px 20px;
    background: #fff;
    color: #dd182b;
    text-align: center;
    margin-left: 10px;
    border: 1px solid #ddd;
    cursor: pointer;
    margin-top: 6px;
}

.header .cart .trigger a {
    color: #dd182b;
}

.header .cart .trigger:hover {
    background: rgb(0,0,0,0.1);
}

/* main-promote区 */
.main-promote .cat {
    background: #6e6568;
    color: #fff;
}

.main-promote .cat .row {
    padding: 12px 20px;
    text-align: center;
}

.main-promote .cat .row:hover {
    background: rgba(0, 0, 0, .2);
}

.main-promote .info {
    padding: 10px;
    background: #fff;
    color: #888;
    -webkit-box-shadow: 0, 1px, 2px rgba(0, 0, 0, .1);
    box-shadow: 0, 1px, 2px rgba(0, 0, 0, .2);
}

.main-promote .info > * {
    margin-bottom: 10px;
}

.main-promote .info .avatar {
    width: 50px;
    height: 50px;
    background: #777;
    border-radius: 50%;
    float: left;
    margin: 0 10px;
}

/* cat-promote区 */
.cat-promote .title:before {
    content: "";
    display: inline-block;
    vertical-align: middle;
    width: 10px;
    height: 22px;
    background: #dd182b;
    margin-right: 10px;
}

.cat-promote .title {
    font-size: 22px;
    margin: 10px 0 0 0;
}

.cat-promote .item {
    padding: 5px;
}

.cat-promote .card {
    height: 300px;
    background: #ccc;
    border-radius: 6px;
    box-shadow: 0 2px 2px rgba(0, 0, 0, .2);
}

.cat-promote .c1 {
    background: #d3b3e2;
}

.cat-promote .c2 {
    background: #afc4e4;
}

.cat-promote .c3 {
    background: #f9e279;
}

.cat-promote .c4 {
    background: #eca78f;
}

.cat-promote .c5 {
    background: #95e89b;
}

.cat-promote .c6 {
    background: #8ab5f7;
}

.cat-promote .c7 {
    background: #b8a08d;
}


/* footer区 */
.footer {
    text-align: center;
    margin-top: 30px;
    font-family: 'Courier New', Courier, monospace;
    font-size: 15px;
    padding: 50px 0;
    background: #ccc;
}

.footer .footer-id {
    margin-right: 40px;
}
```