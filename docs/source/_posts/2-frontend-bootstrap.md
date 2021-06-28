---
title: 前端 | bootstrap常用样式
date: 2020-4-16
tags: 
- html
- bootstrap
- Frontend
categories: notes
photos:
    - /blog/img/bootstrap.jpg
---

<br>
<!--more-->

>Bootstrap4 目前是 Bootstrap 的最新版本，是一套用于 HTML、CSS 和 JS 开发的开源工具集。利用我们提供的 Sass 变量和大量 mixin、响应式栅格系统、可扩展的预制组件、基于 jQuery 的强大的插件系统，能够快速为你的想法开发出原型或者构建整个 app 。



# 引用

- 本文引文 bootstrap 版本为 3.3.7 , 采用 cdn 为中文 BootCDN 网站所提供的加速服务

```html
<link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.3.7/css/bootstrap.css" rel="stylesheet">
```

# 页面容器 container

Bootstrap 需要为页面内容和栅格系统包裹一个 `.container` 容器
- `.container` 类用于固定宽度并支持响应式布局的容器。
- `.container-fluid` 类用于 100% 宽度，占据全部视口（viewport）的容器。

# 栅格系统 col

- Bootstrap 提供了一套响应式、移动设备优先的流式网格系统，随着屏幕或视口（viewport）尺寸的增加，系统会自动分为 12 列。
- 使用行 (.row) 来创建水平的列组, 使用列 (.col) 放置具体块的内容
- 通过为“列（col）”设置 padding 属性，从而创建列与列之间的间隔（gutter）。通过为 `.row` 元素设置负值 margin 从而抵消掉为 `.container` 元素设置的 padding，也就间接为“行（row）”所包含的“列（column）”抵消掉了padding
- Bootstrap 4 栅格系统有以下 5 个类:
    - `.col-*`    针对所有设备
    - `.col-sm-*` 平板 - 屏幕宽度等于或大于 576px
    - `.col-md-*` 桌面显示器 - 屏幕宽度等于或大于 768px)
    - `.col-lg-*` 大桌面显示器 - 屏幕宽度等于或大于 992px)
    - `.col-xl-*` 超大桌面显示器 - 屏幕宽度等于或大于 1200px)

```html
<div class="container">
    <div class="row">
        <!-- 四六分 -->
        <div class="col-md-8">.col-md-8</div>
        <div class="col-md-4">.col-md-4</div>
    </div>
    <div class="row">
        <!-- 三等分 -->
        <div class="col-md-4">.col-md-4</div>
        <div class="col-md-4">.col-md-4</div>
        <div class="col-md-4">.col-md-4</div>
    </div>
    <div class="row">
        <!-- 对半分 -->
        <div class="col-md-6">.col-md-6</div>
        <div class="col-md-6">.col-md-6</div>
    </div>
</div>
```

# 表单 form

```html
<!-- 基本样式, 使用 .form-group , 会设定相应的边距等样式 -->
<!-- 使用 .form-inline 将两个显示在一行 -->
<div class="form-inline">
    <div class="form-group">
        <label>姓</label>
        <input type="text" class="form-control">
    </div>
    <div class="form-group">
        <label>名</label>
        <input type="text" class="form-control">
    </div>
</div>
<!-- .form-control 在选择表单的使用，也拥有相应样式 -->
<div class="form-group">
    <label>选择</label>
    <select class="form-control">
        <option value="1">A</option>
        <option value="1">B</option>
        <option value="1">C</option>
    </select>
</div>
<!-- 上传文件的表单 -->
<div class="form-group">
    <label for="exampleInputFile">上传照片</label>
    <input type="file" id="exampleInputFile">
    <p class="help-block">请选择不低于500x500的图片</p>
</div>



<!-- 对于表单输入的提示反馈，lable 与 input 均形成反馈 -->
<!-- 状态：错误、成功、警示 -->
<div class="form-group has-error">
    <label class="control-label">XX</label>
    <input type="text" class="form-control">
</div>
<div class="form-group has-success">
    <label class="control-label">XX</label>
    <input type="text" class="form-control">
</div>
<div class="form-group has-warning">
    <label class="control-label">XX</label>
    <input type="text" class="form-control">
</div>

<!-- 给表单一端 (.input-group) 加入提示图标 -->
<div class="form-group">
    <label class="control-label">XX</label>
    <div class="input-group">
        <div class="input-group-addon">￥</div>
        <input type="text" class="form-control">
    </div>
</div>

<!-- 输入框大小: 大 小 -->
<div class="form-group has-warning">
    <label class="control-label">XX</label>
    <input type="text" class="form-control input-lg">
</div>
<div class="form-group has-warning">
    <label class="control-label">XX</label>
    <input type="text" class="form-control input-sm">
</div>

<!-- 与栅格功能相结合，形成密钥输入样式 -->
<!-- row 会使内容与列头对齐，通过负的margin抵消掉container的padding -->
<div class="row">
    <div class="col-sm-4">
        <input type="text" class="form-control" placeholder="xxx">
    </div>
    <div class="col-sm-4">
        <input type="text" class="form-control" placeholder="xxx">
    </div>
    <div class="col-sm-4">
        <input type="text" class="form-control" placeholder="xxx">
    </div>
</div>
```

# 按钮 btn

```html
<!-- 用 .well 的块边距 -->
<div class="well">
    <!-- 不同功能会呈现不同颜色 -->
    <button class="btn btn-default"></button>
    <button class="btn btn-primary"></button>
    <button class="btn btn-danger"></button>
    <button class="btn btn-warning"></button>
    <button class="btn btn-info"></button>

    <!-- a、input也可使用 .btn 的样式 -->
    <a class="btn btn-danger" href=""></a>
    <input class="btn btn-default" type="submit">

    <!-- 预设的按钮大小：大 默认 小 极小 占满全行 -->
    <button class="btn btn-primary btn-lg"></button>
    <button class="btn btn-primary btn-md"></button>
    <button class="btn btn-primary btn-sm"></button>
    <button class="btn btn-primary btn-xs"></button>
    <button class="btn btn-primary btn-block"></button>

    <!-- 按钮状态：活动、禁用 -->
    <button class="btn btn-default active"></button>
    <button disabled="disabled" class="btn btn-default"></button>
</div>
```

# 按钮组 btn-group

```html
<!-- 默认样式，中间连接，两端圆角 -->
<div class="well">
    <div class="btn-group">
        <button class="btn btn-default"></button>
        <button class="btn btn-default"></button>
        <button class="btn btn-default"></button>
    </div>
</div>

<!-- 按钮组尺寸: 大 中 小 极小 -->
<div class="btn-group btn-group-lg">
    <button class="btn btn-default"></button>
    <button class="btn btn-default"></button>
    <button class="btn btn-default"></button>
</div>
<div class="btn-group btn-group-md">
    <button class="btn btn-default"></button>
    <button class="btn btn-default"></button>
    <button class="btn btn-default"></button>
</div>
<div class="btn-group btn-group-sm">
    <button class="btn btn-default"></button>
    <button class="btn btn-default"></button>
    <button class="btn btn-default"></button>
</div>
<div class="btn-group btn-group-xs">
    <button class="btn btn-default"></button>
    <button class="btn btn-default"></button>
    <button class="btn btn-default"></button>
</div>

<!-- 按钮组纵向排列 .btn-group-vertical -->
<div class="btn-group-vertical">
    <button class="btn btn-default"></button>
    <button class="btn btn-default"></button>
    <button class="btn btn-default"></button>
</div>

<!-- 按钮组实现工具栏 .btn-toolbar -->
<div class="well">
    <div class="btn-toolbar>
        <div class="btn-group">
            <button class="btn btn-default"></button>
            <button class="btn btn-default"></button>
            <button class="btn btn-default"></button>
        </div>
        <div class="btn-group">
            <button class="btn btn-default"></button>
            <button class="btn btn-default"></button>
            <button class="btn btn-default"></button>
        </div>
    </div>
</div>
```

# 导航 nav

```html
<!-- nav 用于局部导航 -->
<!-- 默认样式，横排显示, nav-tabs -->
<ul class="nav nav-tabs">
    <li class="active"><a href="#">登录</a></li>
    <li><a href="#">注册</a></li>
    <li><a href="#">忘记密码</a></li>
</ul>
<!-- 显示当先活动导航页的内容 -->
<div>
    <h1>登录</h1>
    请输入账号密码
</div>

<!-- 胶囊样式, nav-pills -->
<ul class="nav nav-pills">
    <li class="active"><a href="#">登录</a></li>
    <li><a href="#">注册</a></li>
    <li><a href="#">忘记密码</a></li>
</ul>
<div>
    <h1>登录</h1>
    请输入账号密码
</div>

<!-- 侧栏纵向排列, nav-stacked -->
<ul class="nav nav-pills nav-stacked">
    <li class="active"><a href="#">登录</a></li>
    <li><a href="#">注册</a></li>
    <li><a href="#">忘记密码</a></li>
</ul>
<div>
    <h1>登录</h1>
    请输入账号密码
</div>
```

# 导航栏 navbar

```html
<!-- navbar 用于全站导航 -->
<div class="navbar navbar-default">
    <!-- 整站的logo -->
    <div class="navbar-header">
        <a href="#" class="navbar-brand">
    </div>
    <ul class="nav navbar-nav">
        <li><a href="#">首页</a></li>
        <li><a href="#">分类</a></li>
        <li><a href="#">热门</a></li>
    </ul>
</div>

<!-- 默认从左排列，可以改为右排列 -->
<div class="navbar navbar-default">
<ul class="nav navbar-nav navbar-right">
    <li><a href="#">登录</a></li>
    <li><a href="#">注册</a></li>
</ul>
</div>

<!-- 在导航栏插入搜索框 -->
<div class="navbar navbar-default">
    <!-- 创建表单，可通过左浮动调整显示到一行内 -->
    <form class="narbar-form navbar-left">
        <div class="form-group">
            <input type="text" class="form-contorl">
        </div>
        <button type="submit" class="btn btn-default">搜索</button>
    </form>
</div>
```

# 面板 panel

```html
<!-- 用于展示统计、公告、看板内容 -->
<!-- 默认样式 -->
<div class="panel panel-default">
    
    <div class="panel-heading">
        <!--  (可选) heading 部分可增加容器 .panel-title 使标题放大, 也可直接写入标题 -->
        <div class="panel-title">用户统计</div>
        <!-- 添加注释 -->
        <div class="small text">每日用户情况统计</div>
    </div>
    <div class="panel-body">
        Dkmf lkfdmlk alsmfl 
    </div>
    <!-- 添加脚注 -->
    <div class="panel-footer">
        <div class="small text-muted">
            数据更新于5秒前
        </div>
    </div>
</div>

<!-- 其他样式: 危险 成功 警示 一般信息 -->
<div class="panel panel-danger">
    <div class="panel-heading">错误</div>
    <div class="panel-body">
        Dkmf lkfdmlk alsmfl 
    </div>
</div>
<div class="panel panel-success">
    <div class="panel-heading">成功</div>
    <div class="panel-body">
        Dkmf lkfdmlk alsmfl 
    </div>
</div>
<div class="panel panel-warning">
    <div class="panel-heading">提示</div>
    <div class="panel-body">
        Dkmf lkfdmlk alsmfl 
    </div>
</div>
<div class="panel panel-info">
    <div class="panel-heading">用户统计</div>
    <div class="panel-body">
        Dkmf lkfdmlk alsmfl 
    </div>
</div>
```

# 表格 table

```html
<!-- 默认样式表格, 可通过 .table-border 增加表格边框 -->
<table class="table">
    <thead>
        <tr>
            <th></th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td></td>
            <td></td>
        </tr>
</table>

<!-- 每行颜色深浅不一样式 -->
<table class="table striped">
    <thead>
        <tr>
            <th></th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td></td>
            <td></td>
        </tr>
</table>

<!-- 鼠标悬浮反馈效果样式 -->
<table class="table hover">
    <thead>
        <tr>
            <th></th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td></td>
            <td></td>
        </tr>
</table>

<!-- 针对其中一行特殊高亮, 给 tr 标签加入状态样式: success danger warning -->
<table class="table">
    <thead>
        <tr>
            <th></th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        <tr class="danger">
            <td></td>
            <td></td>
        </tr>
</table>
```

# 页码 page

```html
<!-- 位于文章区域的底部, 加入页码显示 -->
<!-- 一般样式 上下页 -->
<ul class="pagination">
    <!-- 给 li 添加 .active 表示当前页 -->
    <li><a href="#">上一页</a></li>
    <li><a href="#">1</a></li>
    <li><a href="#">2</a></li>
    <li><a href="#">3</a></li>
    <li><a href="#">下一页</a></li>
</ul>

<!-- 简易样式 仅保留上下页 -->
<ul class="pager">
    <!-- 可在 首页、末页 添加 .disabled 禁点击 -->
    <li><a href="#">上一页</a></li>
    <li><a href="#">下一页</a></li>
</ul>

<!-- 页面路径, 展示当前所在位置 样式: /categories/practice/bootstrap -->
<div class="breadcrumb">
    <li><a href="#">categories</a></li>
    <li><a href="#">practice</a></li>
    <li class="active">bootstrap</li>
</div>
```

# 标签 label

```html
<!-- 标签样式, 可当作行元素应用到页面任何位置 -->
<p>
    <!-- 表示一篇博文涉及到的领域, 用标签展示 -->
    <span class="label label-default">python</span>
    <span class="label label-success">python</span>
    <span class="label label-info">crawler</span>
    <span class="label label-warning">re</span>
</p>

<!-- 小徽章样式 标注 可根据所给样式动态调整背景色 -->
<!-- 阅读量 -->
<span class="badge">50k</span>
```

# 提醒 alert

```html
<!-- 用于页面提醒、如错误提示, 登录成功提醒 -->
<div class="alert alert-danger">危险操作</div>
<div class="alert alert-success">登录成功</div>
```

# 列表 list

```html
<!-- 列表样式 可用于投票列表、左边侧边栏 -->
<div class="list-group">
    <a class="list-group-item" href="#"></a>
    <a class="list-group-item" href="#"></a>
    <a class="list-group-item" href="#"></a>
    <a class="list-group-item" href="#"></a>
</div>
```

# More

https://www.runoob.com/bootstrap/bootstrap-tutorial.html
https://www.runoob.com/bootstrap4/bootstrap4-tutorial.html
https://v3.bootcss.com/css/
https://v4.bootcss.com/docs/getting-started/introduction/