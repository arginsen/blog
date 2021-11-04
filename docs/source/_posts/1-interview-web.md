---
title: 面试 | web 性能与安全
date: 2020-8-28
tags: 
  - interview
  - Frontend
categories: notes
hide: true
photos:
    - /blog/img/interview.jpg
---


<br>
<!--more-->

# web 性能

## web 性能优化常用手段

1. 启用压缩：对静态资源采用压缩，服务端可用 nginx 开启 gzip
2. 使用 http 缓存：分为强缓存和弱缓存（协商缓存），强缓存即直接对本地静态资源进行复用，一般设入口文件为弱缓存；弱缓存是本地通过请求头部标记，询问服务端资源是否过期，若服务端确认未过期，则返回 304 状态码，复用本地缓存资源，若服务端确认资源有更新，则返回 200 状态码，从服务端获取资源
3. 减少 DNS 查询：使用域名/子域名越少，开销越小
4. 减少重定向：重定向可能会引起新的 DNS 查询、TCP 连接、HTTP 请求

## 首屏加载优化

1. VueRouter 路由懒加载
2. 使用 CDN 加速
3. Nginx 的 gzip 压缩
4. webpack 开启 gzip 压缩
5. UI 库按需加载
6. link 标签的 rel 属性设置 prefetch 和 preload

## 何时触发回流和重绘

回流：回流一定发生重绘
1. 添加或删除可见的 DOM 元素
2. 元素的位置发生变化
3. 元素的尺寸发生变化
4. 内容发生变化
5. 页面初始渲染
6. 浏览器窗口尺寸变化

重绘：
1. 元素的样式改变，不影响其位置变化

## 如何避免频繁回流和重绘

css：
1. 避免使用 table 布局
2. 尽可能在 DOM 树最末端改变 class
3. 避免设置多层内联样式
4. 将动画效果应用到 position 属性为 absolute 或 fixed 的元素上
5. 避免使用 css 表达式
6. 使用 css3 硬件加速

js：
1. 避免频繁操作 DOM
2. 避免频繁操作样式
3. 可以先给元素设置 `display: none;`，操作结束后再把它显示出来


## 防抖和节流

防抖：指任务频繁触发的情况下，只有任务触发的间隔超过指定的时间间隔的时候，任务才会被执行

```js
// 设置防抖按钮  
window.onload = function () {
  var myDebounce = document.getElementById("debounce");
  myDebounce.addEventListener("click", debounce(sayDebounce));
}

function debounce (fn, time) {
  let timeout
  return () => {
    timeout ? clearTimeout(timeout) : null
    timeout = setTimeout(fn, time)
  }
}

function sayDebouce () {
  // 防抖工作
  console.log("防抖成功")
}
```

节流：指定时间间隔内只会执行一次任务

```js
window.onload = function () {
  var myThrottle = document.getElementById("throttle")
  myThrottle.addEventListener("click", throttle(sayThrottle))
}

function throttle (fn) {
  let run = true
  return function () {
    if (!run) {
      return
    }
    run = false
    setTimeout(() => {
      fn.call(this, arguments)
      run = true
    }, 1000)
  }
}

function sayThrottle () {
  // 节流操作
  console.log("节流成功")
}
```

# 安全

## CSRF 和 XSS

- CSRF：Cross-site request forgery 跨站请求伪造，利用 cookie 进行攻击；一般添加 token 来防御
原理：登录受信人网站 A，并在本地生成 cookie，在不登出 A 的情况下，访问危险网站 B
防御：
1.Token 验证：服务器发送给客户端一个 token，客户端提交的表单中带着这个 token，如果这个 token 不合法，那么服务器拒绝这个请求
2.隐藏令牌：把 token 隐藏在 http 的 head 头中。
3.Referer 验证：Referer 指的是页面请求来源，意思是，只接受本站的请求，服务器才做响应；如果不是，就拦截

- XSS：跨站脚本攻击，一般在输入区通过插入 js 代码来攻击；一般通过过滤 `script` 标签等防止 js 代码插入
原理：Web 程序对用户的输入过滤不严格，导致恶意攻击者可向 Web 页面中嵌入恶意的脚本代码。当用户浏览该页面时，嵌入其中的代码被浏览器正常的解析执行，从而达到恶意攻击者的特殊目的
分类：
1.反射型 XSS：反射型跨站脚本攻击，主要特性：非持久性，通常出现在 URL 参数中，是最常见的 XSS 类型
2.存储型 XSS：存储型跨站脚本攻击，主要特性：持久性，存储在服务端，通常出现在用户与服务器做交互的页面，是最理想的 XSS 类型
3.DOM 型 XSS：Dom 型 xss 与以上两种方式都不同，它不需要与服务器做交互，主要特性为：非持久性，一般出现在 Web 页面的 Dom 解析中

