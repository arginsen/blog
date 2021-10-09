---
title: 前端 | JS基础 | JS学习笔记Ⅸ：JQuery
date: 2020-4-30 18:32:11
tags: 
- JavaScript
- JQuery
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

> - jQuery 是一个JavaScript函数库。
> - jQuery 是一个轻量级的"写的少，做的多"的JavaScript库。
> - jQuery 库包含以下功能：
> - HTML 元素选取、HTML 元素操作、CSS 操作、HTML 事件函数、JavaScript 特效和动画、HTML DOM 遍历和修改、AJAX、Utilities


# 选择器

```js
// id 选择器
$('#a')
  .css('background', 'blue')  // jquery 的 css 方法

// 等同于
document
  .getElementById('a')
  .style
  .background = 'blue' 


// 类选择器
$('.a')
  .css('border', '2px solid black')


// 选取 a 下边的 p 元素
$('.a p')


// 选择指定属性值的 input 元素
$("input[type='text']")
  .css('background', 'yellow')


// 选择子元素
$('div:first')
  .css('color', 'grey')

```

# 过滤器

```js
<div class="grandpa">
    <div class="pa">
        <div id="child1" class="child"></div>
        <div id="child2" class="child"></div>
    </div>
</div>


// 选中 child 类
$('div')
  .find('.child')   // 使用 find 方法过滤
  .css('border', '2px solid #999');

// 更精准的过滤
$('grandpa')
  .find('.child')   // 使用 find 方法过滤
  .css('border', '2px solid #999');

// 从下向上过滤
$('#child1')
  .parent()   // 只能选择父级
  .css('border', '2px solid #999');

// 从下向上精细过滤
$('#child1')
  .parents('.grandpa')  // 上级链中指定级别
  .css('border', '2px solid #999');

// 在同级中过滤
$('.child')
  .filter('.not-gay')  // 选择 filter 过滤出的元素
  .css('backgroud', 'red');
```

# 操作样式

```js
// 使用对象统一添加样式
// 有连字符的样式名称采用字符串或驼峰式写法，值都用字符串形式
$('.a')
  .css({
      color: 'red',
      border: '15px solid grey',
      'background-color': 'black'
  })


// 可以预先定义样式类名，再进行添加
<style>
.black {
    background: #000;
    color: #fff;
}
</style>

$('.a')
  .addClass('black')  // 添加上边定义的类选择器
  .removeClass('b')   // 移除一个类

$('.a').hasClass('b') // false  返回一个布尔值，所在节点是否存在指定类


// 隐藏，显示元素节点
var a = $('.a')
  .css('display', 'none')

// jQuery 有直接定义方法对类进行操作
var a = $('.a')
a.hide()     // 该元素隐藏
a.show()     // 该元素显示

a.fadeOut()  // 该元素淡出，参数可指定时间
a.fadeIn()   // 该元素淡入，参数可指定时间

a.slideUp()  // 该元素向上滑出
a.slideDown()// 该元素向下滑入
```

## 制作一个简易广告牌

```js
// html 部分
// <div id="board" class="active">按摩保健 洗浴推拿</div>

// js 部分

var laststyle = $("<style>.active {color: red}</style>");
laststyle.insertAfter('style');

var board = $('#board');

board.css({
    'text-align': 'center',
    'font-size': '5em',
    background: '#000',
    margin: '5px auto'
});


function toggle() {
    if(board.hasClass('active')) {
        board.removeClass('active')
    } else {
        board.addClass('active')
    }
}

setInterval(toggle, 200);
```

<p id="board" class="active">按摩保健 洗浴推拿</p>

<script>
$('head').append("<style>.active {color: red}</style>")
var board = $('#board');

board.css({
    'text-align': 'center',
    'font-size': '5em',
    background: '#000',
    margin: '5px auto'
});


function toggle() {
    if(board.hasClass('active')) {
        board.removeClass('active')
    } else {
        board.addClass('active')
    }
}

setInterval(toggle, 200);
</script>


# 操作 DOM

```js
// 获取 文本值
$('#a').text()
// 添加 文本值
$('#a').text('Yo.')

// 获取 元素所有内容  包含标签
$('#a').html()
// 添加 标签  将当前元素内全部替换指定内容
$('#a').html('<p> Yo. </p>')


// 添加内容至指定元素的子元素下最后一个
$('#a').append('<div>后</div>')
// 添加内容至指定元素的子元素下第一个
$('#a').prepend('<div>前</div>')

// 添加内容至指定元素的后边
$('#a').appendTo('<div>后</div>')
// 添加内容至指定元素的前边
$('#a').prependTo('<div>前</div>')
// 等同于
// 在被选元素后插入 HTML 元素
$('#a').insertAfter()	
// 在被选元素前插入 HTML 元素
$('#a').insertBefore()	


// 删除该元素
$('#a').remove()

```

# 事件

```html
<!-- click 事件 -->
<!-- mouse 事件 -->
<button id="card-trigger"> 显示/隐藏快讯 </button>
<div id="card">
    <b>快讯</b>
    <span>2020年7月6日，沪深两市成交额突破1.5万亿，北向资金净流入136.5亿元，A股总市值突破10万亿。A股40只ETF涨停，另38只涨幅超7%；70只分级B基金涨停，共85只涨幅超9%。截至收盘，上证指数上涨5.71%，深证指数上涨4.09%，创业板指上涨2.72%,恒生指数上涨3.81%。</span>
</div>

<script>
$('head').prepend('<style>#card {display: none;background: #eee;padding: 0 20px 10px;border-radius: 10px;box-shadow: 1px 2px rgba(0, 0, 0, .1);} #card.cc {color: #fff; background-color: #555;}</style>');

var cardt = $('#card-trigger')
var card = $('#card')


// 添加 click 事件
cardt.on('click', function() {
        card.toggle()
    })

// toggle() 等同于
// cardt.on('click', function() {
//         if (card.is(':visible')) {
//             card.hide()
//         } else {
//             card.show()
//         }
//     })

// 添加 mouse 事件
card.on('mouseenter', function() {
    card.addClass('cc')
})

card.on('mouseleave', function() {
    card.removeClass('cc')
})
</script>

```

## 样板

<button id="card-trigger"> 显示/隐藏快讯 </button>
<div id="card">
    <b>快讯</b>
    <span>2020年7月6日，沪深两市成交额突破1.5万亿，北向资金净流入136.5亿元，A股总市值突破10万亿。A股40只ETF涨停，另38只涨幅超7%；70只分级B基金涨停，共85只涨幅超9%。截至收盘，上证指数上涨5.71%，深证指数上涨4.09%，创业板指上涨2.72%,恒生指数上涨3.81%。</span>
</div>

<script>
$('head').prepend('<style>#card {display: none;background: #eee;padding: 0 20px 10px;border-radius: 10px;box-shadow: 1px 2px rgba(0, 0, 0, .1)} #card.cc {color: #fff; background-color: #555;}</style>');

var cardt = $('#card-trigger')
var card = $('#card')


// 添加 click 事件
cardt.on('click', function() {
        card.toggle()
    })

// 添加 mouse 事件
card.on('mouseenter', function() {
    card.addClass('cc')
})

card.on('mouseleave', function() {
    card.removeClass('cc')
})
</script>


# 操作属性

```js
// attribute 是显性属性，会展示在 html 标签
// 读取
$('#a')
  .attr('href')

// 写入
$('#a')
  .attr('href', 'https://lixupeng.cn')

// property 是隐性属性
$('#a')
  .prop('href')

$('#a')
  .prop('href', 'https://lixupeng.cn')

// 移除属性
$('#a')
  .removeAttr('href')
```

# 表单

```js
// 表单获取用户输入的值
// <input id="nickname" value="" type="text">

// 使用 .val() 获取
$('#nickname').val()

// 传入值
$('#nickname').val('Yo')

// 聚焦输入框   参数传入函数就是监听作用 聚焦时触发该函数
$('#nickname').focus()

// 全选输入框内容
$('#nickname').focus()

// 禁止聚焦/激活输入框
$('#nickname').blur()


// 指定按钮提交表单  指定跳转
<form id="login-trigger" action="result.html">
  用户名：<input type="text">
  密码：<input type="password">
</form>
<button id="login">提交</button>

// js 指定按钮进行提交表单
$('#login').click(function() {
    $('#loginTrigger').submit()
})
```

# Ajax

## load 方法

```html
<!-- 异步请求加载其他文件 -->

<!-- index.html -->
<button id="trigger">今日消息报送</button>
<div id="card" style="display:none;">

<script>
var trigger = $('trigger')
var card = $('card')

trigger.on('click', function() {
    card.load('card.html')
    card.toggle()
})
</script>

<!-- 再定义给指定地方加载其他文件内容 -->
<!-- card.html -->
<div>
2020.7.7，今日截至收盘，上证指数上涨0.37%，深证指数上涨1.72%，创业指板上涨2.44%。两市合计成交17300亿元，北向资金净流入118.88亿元。
</div>


<!-- 优化 js 请求 -->
<script>
var trigger = $('trigger')
var card = $('card')
var loaded = false

trigger.on('click', function() {
    if (card.is(':visible')) {
        card.slideUp()
    } else {
        if (!loaded) {
            card.load('card.html')
            loaded = true
        } else {
            card.slideDown()         
        }
    }
})
</script>
```

## ajax() 应用

```html
<div>
  <form id="search">
    <input id="username" type="search">
    <button type="submit">搜索github用户</button>
  </form>
  <div></div>
</div>


<script>
var form = $('#search')
var input = $('#username')
var result = $('#result')
var username

form.on('submit', function(e) {
    e.preventDefault()
    username = input.val()
    $.ajax('https://api.github.com/users/' + username).done(function(data) {
        var html = 
            '<div>用户名：' + data.login + '</div>' + 
            '<div>介绍：' + (data.bio || '无') + '</div>' + 
            '<div>地区：' + data.location + '</div>' +
            '<div>博客：' + data.blog + '</div>'
        result.html(html)
    })
})
</script>

```

<div style="background: #eee;padding: 0 10px;">
    <form id="search" style="">
        <input id="username" style="margin: 0 40px 0 30px;width: 20em;line-height: 2.5em" type="search"><button type="submit">搜索github用户</button>
    </form>
    <div id="result" style="display: inline-block;margin: 0 40px 10px 30px;"></div>
</div>


<script>
var form = $('#search')
var input = $('#username')
var result = $('#result')
var username

form.on('submit', function(e) {
    e.preventDefault()
    username = input.val()
    $.ajax('https://api.github.com/users/' + username).done(function(data) {
        var html = 
            '<div>用户名：' + data.login + '</div>' + 
            '<div>介绍：' + (data.bio || '无') + '</div>' + 
            '<div>地区：' + data.location + '</div>' +
            '<div>博客：' + data.blog + '</div>'
        result.html(html)
    })
})
</script>

## ajax() 介绍

```js
// JQuery 对 Ajax 的封装
$.ajax({
    url: '/signup'
    method: 'get',  // 默认 get
    data: {
        username: '',
        password: ''
    }
    success: function(data) {
        //
    }
    error: function() {
        //
    }
})

// 快捷方法
$.post('url', {
    username: '',
    password: ''
})

$.get('url')

$.getJson('url')
$.getScript('url')  // 获取代码
```

# jquery 应用












