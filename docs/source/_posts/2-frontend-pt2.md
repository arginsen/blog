---
title: 前端 | 练习二之使用bootstrap复刻练习一
date: 2020-4-17
tags: 
- CSS
- bootstrap
- html
- Frontend
categories: practice
photos:
    - /blog/img/bootstrap.jpg
---
<br>
<!--more-->

>本文使用 bootstrap 4.4.1 版本, 采用 bootcss 网站提供的cdn, 可自行查阅 [中文文档](https://v4.bootcss.com/docs/getting-started/introduction/)

# page

作品: [https://lixupeng.cn/E-commerce_page_layout/bootstrap](https://lixupeng.cn/E-commerce_page_layout/bootstrap)
项目地址: [https://github.com/E-commerce_page_layout/bootstrap](https://github.com/E-commerce_page_layout/bootstrap)

# link

```html
<!-- 参考的中文文档对应的连接如下 -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.4.1/dist/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">

<!-- 引用 4.5.0 也可, 对应的专有名词都相同 -->
<link rel="stylesheet" href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/4.5.0/css/bootstrap.css">
```

# code

## html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.4.1/dist/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
    <link rel="stylesheet" href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/4.5.0/css/bootstrap.css">
    <link rel="stylesheet" href="main.css">
    <title>蘑菇街 - 你的剁手街</title>
</head>
<body>
    <div class="topnav">
        <div class="navbar navbar-light">
            <div class="container">
                <div class="nav navbar ml-3">
                    <li><a href="#">首页</a></li>
                </div>
                <div class="nav navbar navbar-right">
                    <li class="nav-item">
                        <a class="nav-link" href="#">我的订单</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="#">收藏夹</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="#">登录</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="#">注册</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="#">客户服务</a>
                    </li>
                </div>
            </div>
        </div>
    </div>
    <div class="header clearfix">
        <div class="navbar navbar-light">
            <div class="container">
                <div class="col-2">
                    <div class="nav-brand">
                        <img src="../img/logo.png" alt="">
                    </div>
                </div>
                <div class="col-7">
                    <div class=" nav navbar navbar-right">
                        <form class="form-inline">
                            <input class="form-control" type="search" aria-label="Search">
                            <button class="btn btn-light my-2 my-sm-0" type="submit">Search</button>
                        </form>
                    </div>
                </div>
                <div class="col-3">
                    <a class="btn btn-light" href="#">
                        我的购物车
                    </a>
                </div>
            </div>
        </div>
    </div>
    <div class="main-promote clearfix">
        <div class="container">
            <div class="row my-2">
                <div class="col-3 px-0">
                    <ul class="list-group">
                        <li class="list-group-item">item1 / item1</li>
                        <li class="list-group-item">item2 / item2</li>
                        <li class="list-group-item">item3 / item3</li>
                        <li class="list-group-item">item4 / item4</li>
                        <li class="list-group-item">item5 / item5</li>
                        <li class="list-group-item">item6 / item6</li>
                        <li class="list-group-item">item7 / item7</li>
                        <li class="list-group-item">item8 / item8</li>
                        <li class="list-group-item">item9 / item9</li>
                        <li class="list-group-item">item10 / item10</li>
                        <li class="list-group-item">item11 / item11</li>
                        <li class="list-group-item">item12 / item12</li>
                    </ul>
                </div>
                <div class="col-6 px-0">
                    <img src="../img/slider_02.png" alt="">
                    <div class="row mx-0">
                        <div class="col px-0">
                            <img src="../img/001.png" alt="">
                        </div>
                        <div class="col px-0">
                            <img src="../img/002.png" alt="">
                        </div>
                    </div>
                </div>
                <div class="col-3 px-0">
                    <div class="info pt-3 px-2 mx-0">
                        <div class="auth clearfix">
                            <div class="avatar "></div>
                            <div class="title pt-2">Yo, 欢迎来到香菇街！</div>
                        </div>
                        <div class="title clearfix px-2 mt-3">
                            公告
                        </div>
                        <div class="content px-2">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ab debitis id illum modi necessitatibus obcaecati. Autem doloremque explicabo itaque magnam, minima nam nostrum odit qui quos rerum sed similique velit! Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ab debitis id illum modi necessitatibus obcaecati. Autem doloremque explicabo itaque magnam, minima nam nostrum odit nam nostrum odit qui quos rerum sed similique velit!</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="cat-promote clearfix my-3">
        <div class="container">
            <div class="row">
                <div class="title">男装</div>
            </div>
            <div class="row">
                <div class="col-2 px-0">
                    <div class="card bg-success"></div>
                </div>
                <div class="col-5">
                    <div class="card bg-primary"></div>
                </div>
                <div class="col-2 pl-0">
                    <div class="card bg-warning"></div>
                </div>
                <div class="col-3 px-0">
                    <div class="card bg-info"></div>
                </div>
            </div>
            <div class="row">
                <div class="title">女装</div>
            </div>
            <div class="row">
                <div class="col-3 px-0">
                    <div class="card bg-danger"></div>
                </div>
                <div class="col-4">
                    <div class="card bg-warning"></div>
                </div>
                <div class="col-2 pl-0">
                    <div class="card bg-dark"></div>
                </div>
                <div class="col-3 px-0">
                    <div class="card bg-primary"></div>
                </div>
            </div>
            <div class="row">
                <div class="title">彩装</div>
            </div>
            <div class="row">
                <div class="col-4 px-0">
                    <div class="card bg-warning"></div>
                </div>
                <div class="col-3">
                    <div class="card bg-success"></div>
                </div>
                <div class="col-3 pl-0">
                    <div class="card bg-secondary"></div>
                </div>
                <div class="col-2 px-0">
                    <div class="card bg-info"></div>
                </div>
            </div>
        </div>
    </div>
    <div class="footer clearfix">
        <div class="container">
            <div class="footer-info float-left">
                <a href="#">分类浏览</a>
                <a href="#">使用条款</a>
                <a href="#">销售条款</a>
                <a href="#">隐私政策</a>
            </div>
            <div class="footer-info float-right">
                <a href="../index.html">标准CSS布局</a>
            </div>
        </div>
    </div>
</body>
</html>
```

## css

```css
body {
    background: #f5f5f5;
    font-size: 14px;
    font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
}

.container {
    width: 1080px;
}

a {
    color: #777;
    text-decoration: none;
}

a:hover {
    color: #333;
    text-decoration: none;
}


.topnav .navbar {
    background: #eee;
    padding: 0;
    font-size: 14px;
}

.header .container {
    padding: 10px;
}
.header .nav-brand img {
    height: 60px;
}

.header .form-inline,
.header a {
    width: 100%;
}

.header .form-control,
.header .btn {
    border-radius: 0;
    border-color: #ce1829;
}

.header .btn {
    background: #ce1829;
    color: #ffffff;
}

.header .btn:hover,
.btn-outline-success:not(:disabled):not(.disabled):active {
    background: rgba(228, 18, 18, 0.87);
    box-shadow: 0;
}

.header .form-inline .form-control {
    width: 80%;
}

.header .form-inline .btn {
    width: 20%;
}

.main-promote > .row {
    margin-left: -45px;
    padding: 0 -15px;
}

.main-promote .col-3 {
    flex: 0 0 20%;
}

.main-promote .col-6 {
    flex: 0 0 80%;
    max-width: 60%;
}

.main-promote .list-group {
    border-radius: 0;
}

.main-promote .list-group-item {
    background: #6e6568;
    border: 0;
    text-align: center;
    color: #ffffff;
    padding: 12px 20px;
}

.main-promote img {
    max-width: 100%;;
}

.main-promote .info {
    margin-left: 10px;
    background: #fff;
}

.main-promote .info .avatar {
    width: 50px;
    height: 50px;
    background: #777;
    border-radius: 50%;
    float: left;
    margin: 0 10px;
}

.main-promote .content {
    line-height: 1.5rem;
}

.cat-promote .title:before {
    content: "";
    display: inline-block;
    vertical-align: middle;
    width: 10px;
    height: 22px;
    background: #dd182b;
    margin-right: 10px;
}

.cat-promote .row {
    margin-top: 10px;
    font-size: 16px;
}


.cat-promote .card {
    height: 300px;
}

.footer {
    padding: 50px 10px;
    background: #bbb;
}

.footer a {
    padding: 0 10px;
}
```