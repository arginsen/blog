---
title: 前端 | 练习三之bootstrap制作新闻站
date: 2020-4-18
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

>根据表叔所讲，使用 bootstrap.min.css 3.3.7 , 根据 bootstrap 提供的类, 编写 html , 及根据需求编写独立的 main.css 。完成页面 index.html news.html signin.html signup.html , 使用到 bootstrap 的 css 组件有栅格系统、导航栏、列表、表单、按钮、标签等。

# page

作品: [https://lixupeng.cn/News_site_by_bootstrap](https://lixupeng.cn/News_site_by_bootstrap)
项目地址: [https://github.com/arginsen/News_site_by_bootstrap](https://github.com/arginsen/News_site_by_bootstrap)

# code

## html

### index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="css/bootstrap.min.css">
    <link rel="stylesheet" href="css/main.css">
    <title>News site</title>
</head>
<body>
    <div class="navbar navbar-default">
        <div class="container">
            <div class="navbar-header">
                <a href="index.html" class="navbar-brand"></a>
            </div>
            <label id="toggle-label" class="visible-xs-inline-block" for="toggle-checkbox">MENU</label for="toggle-checkbox">
            <input id="toggle-checkbox" type="checkbox" class="hidden">
            <div class="hidden-xs">
                <ul class="nav navbar-nav">
                    <li class="active"><a href="index.html">首页</a></li>
                    <li><a href="#">国内</a></li>
                    <li><a href="#">国际</a></li>
                    <li><a href="#">数读</a></li>
                    <li><a href="#">社会</a></li>
                </ul>
                <ul class="nav navbar-nav navbar-right">
                    <li><a href="signin.html">登录</a></li>
                    <li><a href="signup.html">注册</a></li>
                </ul>
            </div>
        </div>
    </div>
    <div class="container">
        <div class="row">
            <div class="col-sm-2">
                <div class="hidden-xs side-bar list-group">
                    <a href="#" class="list-group-item active">推荐</a>
                    <a href="#" class="list-group-item">长沙</a>
                    <a href="#" class="list-group-item">娱乐</a>
                    <a href="#" class="list-group-item">电视</a>
                    <a href="#" class="list-group-item">电影</a>
                    <a href="#" class="list-group-item">音乐</a>
                    <a href="#" class="list-group-item">阅读</a>
                    <a href="#" class="list-group-item">美图</a>
                </div>
            </div>
            <div class="col-sm-7">
                <div class="news-list">
                    <div class="news-list-item clearfix">
                        <div class="col-xs-5">
                            <img src="img/002.jpg" alt="">
                        </div>
                        <div class="col-xs-7">
                            <a href="news.html" class="title">
                            2年前他为教育事业和高圆圆分手，今成这般，高圆圆：我有一句MMP如鲠在喉
                            </a>
                            <div class="info">
                                <span class="avatar">
                                    <img src="img/logo.png" alt="">
                                </span>
                                <span class="subinfo">
                                    <span>王花花</span>
                                    <span>25k阅读</span>
                                    <span>10分钟前</span>
                                </span>
                            </div>
                        </div>
                    </div>
                    <div class="news-list-item clearfix">
                        <div class="col-xs-5">
                            <img src="img/003.jpg" alt="">
                        </div>
                        <div class="col-xs-7">
                            <a href="news.html" class="title">
                            2年前他为教育事业和高圆圆分手，今成这般，高圆圆：我有一句MMP如鲠在喉
                            </a>
                            <div class="info">
                                <span class="avatar">
                                    <img src="img/logo.png" alt="">
                                </span>
                                <span class="subinfo">
                                    <span>王花花</span>
                                    <span>25k阅读</span>
                                    <span>10分钟前</span>
                                </span>
                            </div>
                        </div>
                    </div>
                    <div class="news-list-item clearfix">
                        <div class="col-xs-5">
                            <img src="img/004.jpg" alt="">
                        </div>
                        <div class="col-xs-7">
                            <a href="news.html" class="title">
                            2年前他为教育事业和高圆圆分手，今成这般，高圆圆：我有一句MMP如鲠在喉
                            </a>
                            <div class="info">
                                <span class="avatar">
                                    <img src="img/logo.png" alt="">
                                </span>
                                <span class="subinfo">
                                    <span>王花花</span>
                                    <span>25k阅读</span>
                                    <span>10分钟前</span>
                                </span>
                            </div>
                        </div>
                    </div>
                    <div class="news-list-item clearfix">
                        <div class="col-xs-5">
                            <img src="img/005.jpg" alt="">
                        </div>
                        <div class="col-xs-7">
                            <a href="news.html" class="title">
                            2年前他为教育事业和高圆圆分手，今成这般，高圆圆：我有一句MMP如鲠在喉
                            </a>
                            <div class="info">
                                <span class="avatar">
                                    <img src="img/logo.png" alt="">
                                </span>
                                <span class="subinfo">
                                    <span>王花花</span>
                                    <span>25k阅读</span>
                                    <span>10分钟前</span>
                                </span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="col-sm-3">
                <div class="search-bar">
                    <input type="search" class="form-control" placeholder="搜索资讯">
                </div>
                <div class="side-bar-card flag clearfix">
                    <div class="col-xs-5">
                        <img src="img/1-1.png" alt="">
                    </div>
                    <div class="col-xs-7">
                        <div class="text-lg">有害信息举报专区</div>
                        <div>举报电话：12377</div>
                    </div>
                </div>
                <div class="side-bar-card clearfix">
                    <div class="card-title text-lg">24小时热闻</div>
                    <div class="card-body">
                        <div class="list">
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <footer class="footer">
        © 2020 拴蛋头条 中国互联网举报中心京ICP证1401号京ICP备125439号-3京公网安备
    </footer>
</body>
</html>
```

### news.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="css/bootstrap.min.css">
    <link rel="stylesheet" href="css/main.css">
    <title>News site</title>
</head>
<body>
    <div class="navbar navbar-default">
        <div class="container">
            <div class="navbar-header">
                <a href="index.html" class="navbar-brand"></a>
            </div>
            <label id="toggle-label" class="visible-xs-inline-block" for="toggle-checkbox">MENU</label for="toggle-checkbox">
            <input id="toggle-checkbox" type="checkbox" class="hidden">
            <div class="hidden-xs">
                <ul class="nav navbar-nav">
                    <li class="active"><a href="index.html">首页</a></li>
                    <li><a href="#">国内</a></li>
                    <li><a href="#">国际</a></li>
                    <li><a href="#">数读</a></li>
                    <li><a href="#">社会</a></li>
                </ul>
                <ul class="nav navbar-nav navbar-right">
                    <li><a href="signin.html">登录</a></li>
                    <li><a href="signup.html">注册</a></li>
                </ul>
            </div>
        </div>
    </div>
    <div class="container">
        <div class="col-md-8">
            <h1 class="news-title">
                2年前他为教育事业和高圆圆分手，今成这般，高圆圆：我有一句MMP如鲠在喉
            </h1>
            <div class="news-status">
                <span class="desc">25k阅读 · 12分钟前发布</span>
                <div class="label label-info">教育</div>
                <div class="label label-info">情感</div>
            </div>
            <div class="news-content">
                <blockquote><p>Possimus numquam saepe ad, minus non voluptatum a ut necessitatibus, nostrum et. deleniti sit repudiandae.</p></blockquote>
                <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Qui quasi facilis eos earum unde blanditiis est, id accusamus molestiae recusandae fugit sapiente similique excepturi provident quos veritatis aperiam? Dolor, sequi!</p>
                <img src="img/2-4.jpg" alt="">
                <p>Possimus numquam saepe ad, minus non! Vitae, aut doloribus excepturi impedit quidem dolore, minima fugit provident, omnis deleniti voluptas rem maiores ea libero facere deserunt. Quaerat possimus, minus accusantium dolorum!</p>
                <p>Error, voluptatum a ut necessitatibus, nostrum et, natus tempore libero facilis nisi ipsum porro, velit quas quae qui officiis labore impedit aspernatur voluptate blanditiis maiores? Optio repellat expedita ad debitis!
                    Omnis perferendis distinctio ullam id at delectus deleniti sit repudiandae. Esse ducimus sapiente in quod cum earum ratione cumque. Nemo aspernatur earum perspiciatis nobis sit repellat quidem reprehenderit, nisi culpa?</p>
                <p>Sit, libero modi. Eius possimus praesentium blanditiis pariatur temporibus dolorum minus iusto, libero ducimus. Unde quia, reprehenderit corporis, eaque facere illo ut, consequatur nesciunt vero quidem commodi vel labore ipsa?</p>
                <p>Labore nam molestiae a consequatur expedita alias qui nemo officiis quas voluptates doloremque quis, maiores repellat, minima consequuntur incidunt eum laudantium omnis eos, voluptatem. Asperiores saepe enim unde explicabo, quibusdam.</p>
                <p>Dolor dolorum unde aut consectetur reiciendis eum laudantium. Aliquam obcaecati ex sapiente officia vitae, reprehenderit, accusamus qui recusandae in assumenda! Magnam, voluptate! Minus quidem nihil nobis, perferendis? Nam, repudiandae doloremque.</p>
                <p>Doloremque dicta quos cumque totam suscipit dolor esse odio nulla, atque temporibus, velit et eaque nobis alias laboriosam, eveniet minima itaque assumenda quo modi nisi libero exercitationem consequuntur! Deleniti, sequi.</p>
                <p>Tempore, cupiditate, perspiciatis. Odio cum voluptates facere ad rem laborum dolorem nisi pariatur quasi, perspiciatis at, repellendus iure eos necessitatibus molestias enim inventore, fuga! Tempore sit animi nesciunt veniam distinctio.</p>
                <p>Quam dolorem, accusamus autem quibusdam iure excepturi amet pariatur voluptates, dolores ab, praesentium ducimus esse dolore dolor! At enim, quod quam alias facere nisi aliquam incidunt necessitatibus est voluptates cupiditate.</p>
            </div>
        </div>
        <div class="col-md-4">
            <div class="side-bar-card">
                <div class="card-title">相关推荐</div>
                <div class="card-body">
                    <div class="list">
                        <div class="item clearfix">
                            <div class="col-xs-5 no-padding-x"><img src="https://dummyimage.com/1000x700/666/ccc" alt=""></div>
                            <div class="col-xs-7">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                        </div>
                        <div class="item clearfix">
                            <div class="col-xs-5 no-padding-x"><img src="https://dummyimage.com/1000x700/666/ccc" alt=""></div>
                            <div class="col-xs-7">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                        </div>
                        <div class="item clearfix">
                            <div class="col-xs-5 no-padding-x"><img src="https://dummyimage.com/1000x700/666/ccc" alt=""></div>
                            <div class="col-xs-7">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                        </div>
                        <div class="item clearfix">
                            <div class="col-xs-5 no-padding-x"><img src="https://dummyimage.com/1000x700/666/ccc" alt=""></div>
                            <div class="col-xs-7">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                        </div>
                        <div class="item clearfix">
                            <div class="col-xs-5 no-padding-x"><img src="https://dummyimage.com/1000x700/666/ccc" alt=""></div>
                            <div class="col-xs-7">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="side-bar-card">
                <div class="card-title text-lg">24小时热闻</div>
                    <div class="card-body">
                        <div class="list">
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                            <div class="item">
                                <div class="title">Lorem ipsum dolor sit amet, consectetur adipisicing</div>
                                <div class="desc">15k阅读  1k评论</div>
                            </div>
                        </div>
                    </div>
            </div>
        </div>
    </div>
    <footer class="footer">
        © 2020 拴蛋头条 中国互联网举报中心京ICP证1401号京ICP备125439号-3京公网安备
    </footer>
</body>
</html>
```

### signup.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="css/bootstrap.min.css">
    <link rel="stylesheet" href="css/main.css">
    <title>News site</title>
</head>
<body>
    <div class="navbar navbar-default">
        <div class="container">
            <div class="navbar-header">
                <a href="index.html" class="navbar-brand"></a>
            </div>
            <label id="toggle-label" class="visible-xs-inline-block" for="toggle-checkbox">MENU</label for="toggle-checkbox">
            <input id="toggle-checkbox" type="checkbox" class="hidden">
            <div class="hidden-xs">
                <ul class="nav navbar-nav">
                    <li class="active"><a href="index.html">首页</a></li>
                    <li><a href="#">国内</a></li>
                    <li><a href="#">国际</a></li>
                    <li><a href="#">数读</a></li>
                    <li><a href="#">社会</a></li>
                </ul>
                <ul class="nav navbar-nav navbar-right">
                    <li><a href="signin.html">登录</a></li>
                    <li><a href="signup.html">注册</a></li>
                </ul>
            </div>
        </div>
    </div>
    <div class="container container-small">
        <h1>注册
            <small>已有帐号？<a href="signin.html">登录</a></small>
        </h1>
        <form>
            <div class="form-group">
                <label for="">用户名</label>
                <input type="text" class="form-control">
            </div>
            <div class="form-group">
                <label for="">邮箱/手机</label>
                <input type="text" class="form-control">
            </div>
            <div class="form-group">
                <label for="">验证码</label>
                <div class="input-group">
                <!-- input-group 的 display 为 table
                    input-group 下的 form-control 和 input-group-btn 的display 为 table-cell  -->
                    <input type="text" class="form-control">
                    <div class="input-group-btn">
                        <button class="btn btn-primary">获取验证码</button>
                    </div>
                </div>
            </div>
            <div class="form-group">
                <label for="">密码</label>
                <input type="password" class="form-control">
            </div>
            <div class="form-group">
                <button class="btn btn-primary btn-block" type="submit">注册</button>
            </div>
            <div class="sup">注册拴蛋头条即代表您同意<a href="#">拴蛋头条服务条款</a></div>
        </form>
    </div>
    <footer">
        <div class="footer-bottom footer-signup">
            © 2020 拴蛋头条 中国互联网举报中心京ICP证1401号京ICP备125439号-3京公网安备
        </div>
    </footer>
</body>
</html>
```

### signin.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="css/bootstrap.min.css">
    <link rel="stylesheet" href="css/main.css">
    <title>News site</title>
</head>
<body>
    <div class="navbar navbar-default">
        <div class="container">
            <div class="navbar-header">
                <a href="index.html" class="navbar-brand"></a>
            </div>
            <label id="toggle-label" class="visible-xs-inline-block" for="toggle-checkbox">MENU</label for="toggle-checkbox">
            <input id="toggle-checkbox" type="checkbox" class="hidden">
            <div class="hidden-xs">
                <ul class="nav navbar-nav">
                    <li class="active"><a href="index.html">首页</a></li>
                    <li><a href="#">国内</a></li>
                    <li><a href="#">国际</a></li>
                    <li><a href="#">数读</a></li>
                    <li><a href="#">社会</a></li>
                </ul>
                <ul class="nav navbar-nav navbar-right">
                    <li><a href="signin.html">登录</a></li>
                    <li><a href="signup.html">注册</a></li>
                </ul>
            </div>
        </div>
    </div>
    <div class="container container-small">
        <h1>登录
            <small>没有账号？<a href="signup.html">注册</a></small>
        </h1>
        <form>
            <div class="form-group">
                <label for="">用户名/手机/邮箱</label>
                <input type="text" class="form-control">
            </div>
            <div class="form-group">
                <label for="">密码</label>
                <input type="password" class="form-control">
            </div>
            <div class="form-group">
                <button class="btn btn-primary btn-block" type="submit">登录</button>
            </div>
            <div class="form-group">
                <a href="">忘记密码？</a>
            </div>
        </form>
    </div>
    <footer">
        <div class="footer-bottom footer-signin">
            © 2020 拴蛋头条 中国互联网举报中心京ICP证1401号京ICP备125439号-3京公网安备
        </div>
    </footer>
</body>
</html>
```

## css

### main.css

```css
container {
    min-height: 100%;
}

.container-small {
    max-width: 500px;
}

img {
    display: block;
    max-width: 100%;
}

img,
.side-bar .list-group-item,
.side-bar-card {
    border-radius: 3px;
}

a:hover {
    text-decoration: none;
}

#toggle-checkbox:checked ~ div {
    display: block !important;
}

#toggle-label {
    position: absolute;
    top: 15px;
    right: 10px;
    font-size: 15px;
    color: #777;
}

#toggle-label:hover {
    color: #333;
}

.navbar-brand {
    background-image: url(../img/logo.png);
    width: 50px;
    background-size: 75%;
    background-repeat: no-repeat;
    background-position: center center;
}

.navbar-nav li:hover,
.navbar-default .navbar-nav li a:focus {
    color: #555;
    background-color: #e7e7e7;
}

.side-bar .list-group-item {
    border: 0;
    margin-bottom: 5px;
}

.side-bar .list-group-item.active,
.side-bar .list-group-item:focus,
.side-bar .list-group-item:hover {
    background-color: #ea0c3c;
    color: #ffffff;
}

.navbar-nav li:hover,
.navbar-default .navbar-nav li a:focus,
.side-bar .list-group-item.active,
.side-bar .list-group-item:focus,
.side-bar .list-group-item:hover {
    -webkit-transition: background 200ms;
    -moz-transition: background 200ms;
    -ms-transition: background 200ms;
    -o-transition: background 200ms;
    transition: background 200ms;
}

.news-list-item {
    padding: 15px 0;
    border-bottom: 1px solid #eee;
}

.news-list-item:first-child {
    padding-top: 0;
}

.news-list-item:last-child {
    border-bottom: 0;
}

.news-list-item .title {
    display: block;
    color: #444;
    font-size: 18px;
    font-weight: bold;
    line-height: 1.5;
    margin: 5px 0;
}

.news-list-item .title:hover {
    color: #337ab7;
}

.news-list-item .info {
    color: #ccc;
}

.avatar img {
    display: inline-block;
    width: 18px;
    margin-right: 5px;
}

.news-list-item .subinfo span {
    display: inline;
    vertical-align: middle;
}

.side-bar-card {
    background: rgba(0, 0, 0, .05);
    margin-top: 15px;
    padding: 15px 0;
}

.side-bar-card .flag {
    padding: 20px 0;
}

.side-bar-card .text-lg {
    font-size: 150%;
}

.side-bar-card .card-title,
.side-bar-card .card-body .list .item {
    padding: 10px 15px;
}

.side-bar-card .card-title {
    font-size: 18px;
    font-weight: bold;
    padding-bottom: 0;
    padding-top: 0;
}

.side-bar-card .card-body .list .item:hover {
    background: rgba(0, 0, 0, .05);
}

.card-body .list .item .title {
    padding-bottom: 5px;
}

.card-body .list .item .desc {
    color: #999;
}

.news-title {
    line-height: 1.3;
}

.news-status {
    color: #999;
}

.news-status .desc {
    display: inline-block;
    vertical-align: middle;
    padding: 3px 3px 0 0;
}

.news-content {
    display: block;
    margin-top: 20px;
    font-size: 15px;
    line-height: 1.5;
}

.news-content img {
    margin: 0 auto;
    padding-bottom: 5px;
}

.no-padding-x {
    padding-left: 0;
    padding-right: 0;
}

.btn-primary {
    background: #ea0c3c;
    border-color: #db0d3a;
}

.btn-block {
    margin-top: 30px;
}
.btn-primary:hover {
    background: #d01a41;
    border-color: #c9173e;
}

.btn-primary:active,
.btn-primary:focus,
.btn-primary:active:hover,
.btn-primary:active:focus {
    background: #e6244d;
    border-color: #d01a41;
}

.footer,
.footer-bottom {
    margin-top: 20px;
    padding: 50px 0;
    text-align: center;
    color: #999;
    background: #eee;
}

.footer-bottom {
    position: absolute;
    bottom: 0;
    width: 100%;
}

@media (max-width:767px) {
    .container-small {
        margin: 40px 0 60px;
    }
    .footer-bottom {
        position: relative;
    }
    .footer-signin {
        bottom: -205px;
    }
    .footer-signup {
        bottom: -70px;
    }
}
```

