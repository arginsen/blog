---
title: 前端 | Cors
date: 2021-7-5
tags: 
- Browser
- Frontend
- Cors
- HTTP
categories: notes
photos:
    - /blog/img/cors.jpg
---

<br>
<!--more-->

Cors 是一个 W3C 标准，全称是跨域资源共享（Cross-origin resource sharing）

Cors 允许浏览器向跨源服务器，发出 `XMLHttpRequest` 请求。

# 前提

Cors 需要浏览器和服务器同时支持。

整个 Cors 通信是浏览器自动完成，不需要用户参与。对于开发者来说，Cors 通信与同源的 Ajax 通信没有差别，代码完全一样。浏览器识别出 Ajax 请求跨源，就会自动添加一些附加的头信息。

# Cors 请求

Cors 请求可分为两类：简单请求和非简单请求

只要同时满足以下两大条件，就属于简单请求：

（1) 请求方法是以下三种方法之一：
- HEAD
- GET
- POST

（2）HTTP的头信息不超出以下几种字段：
- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：只限于三个值`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

## 简单请求

对于简单请求，浏览器直接发出 Cors 请求。浏览器会在**请求头**信息之中，增加一个 `Origin` 字段，用来说明，本次请求的源（协议+域名+端口）

```json
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

1. 如果 `Origin` 指定的源不在许可范围内，服务器会返回一个正常的 HTTP 响应，但响应的头信息没有包含 `Access-Control-Allow-Origin` 字段，浏览器就会抛出一个错误，被 `XMLHttpRequest` 的 `onerror` 回调函数捕获（这种错误无法通过状态码识别，有可能是200） 

2. 如果 Origin 指定的域名在许可范围内，服务器返回的**响应头**信息里，就会多出几个字段：

```json
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```

- Access-Control-Allow-Origin
该字段是必须的。其值为请求时 Origin 字段的值，要么是一个 * （表示接受任意域名的请求）

- Access-Control-Allow-Credentials
该字段可选。为一个布尔值，表示是否允许发送 Cookie。默认 Cookie 不在 Cors 请求中，设为 `true` 时表示服务器允许 Cookie 置于请求中发给服务器

- Access-Control-Expose-Headers
该字段可选。CORS 请求时，`XMLHttpRequest` 对象的 `getResponseHeader()` 方法只能拿到6个基本字段：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`。如果想拿到其他字段，就必须在 Access-Control-Expose-Headers 里面指定。如上，通过 `getResponseHeader('FooBar')`，就可返回 `Foobar` 字段

### withCredentials

Cors 请求默认不发送 Cookie 和 HTTP 认证信息。如果要把 Cookie 发到服务器，一方面要服务器同意；

```json
Access-Control-Allow-Credentials: true
```

另一方面，必须在 Ajax 请求中打开 withCredentials 属性。

```js
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;
```

需要注意的是，如果要发送 `Cookie`，`Access-Control-Allow-Origin` 就**不能**设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie 依然遵循同源政策，只有用服务器域名设置的 Cookie 才会上传，其他域名的 Cookie 并不会上传，且（跨源）原网页代码中的 `document.cookie` 也无法读取服务器域名下的 Cookie。


## 非简单请求

非简单请求是对服务器有特殊要求的请求，比如请求方法是 `PUT` 或 `DELETE`，或者 `Content-Type` 字段的类型是 `application/json`。

### 预检请求

非简单请求的 Cors 请求，会在正是通信之前，增加一次 HTTP 查询请求，称预检请求。
浏览器会先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些 HTTP 动词和头信息字段。在得到肯定答复后，浏览器才发出正式的 `XMLHttpRequest` 请求，否则就报错。

```js
var url = 'http://api.alice.com/cors';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
```

上面代码中，HTTP 请求的方法是 PUT，并且发送一个自定义头信息 `X-Custom-Header`。

浏览器发现，这是一个非简单请求，就自动发出一个"预检"请求，要求服务器确认可以这样请求。下面是这个"预检"请求的 HTTP 头信息：

```json
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

"预检"请求用的请求方法是 `OPTIONS`，表示这个请求是用来询问的。头信息里面，关键字段是Origin，表示请求来自哪个源。

除了 Origin 字段，"预检"**请求**的头信息包括两个特殊字段。

- Access-Control-Request-Method
该字段是必须的，用来列出浏览器的 CORS 请求会用到哪些 HTTP 方法，上例是PUT。

- Access-Control-Request-Headers
该字段是一个逗号分隔的字符串，指定浏览器 CORS 请求会额外发送的头信息字段，上例是`X-Custom-Header`。

### 预检请求的响应

服务器收到"预检"请求以后，检查了 `Origin`、`Access-Control-Request-Method` 和 `Access-Control-Request-Headers` 字段以后，确认允许跨源请求，就可以做出回应。

```json
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

上面的 HTTP 回应中，关键的是 `Access-Control-Allow-Origin` 字段，表示 `http://api.bob.com` 可以请求数据。该字段也可以设为 `*`，表示同意任意跨源请求。

如果服务器否定了"预检"请求，会返回一个正常的 HTTP 回应，但是没有任何 CORS 相关的头信息字段。这时，浏览器就会认定，服务器不同意预检请求，因此触发一个错误，被 `XMLHttpRequest` 对象的 `onerror` 回调函数捕获。控制台会打印出如下的报错信息：

```
XMLHttpRequest cannot load http://api.alice.com.
Origin http://api.bob.com is not allowed by Access-Control-Allow-Origin.
```

同时服务器响应的其他字段：

```json
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```

### 浏览器正常请求和响应

服务器通过预检请求，**之后**每次浏览器正常的 Cors 请求，都跟简单请求一样，会有一个 Origin 请求头信息字段。服务器的回应，也都会有一个 `Access-Control-Allow-Origin` 响应头信息字段。

Cors 请求：

```json
PUT /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
X-Custom-Header: value
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

服务器响应：

```json
Access-Control-Allow-Origin: http://api.bob.com
Content-Type: text/html; charset=utf-8
```

# 与JSONP的比较

CORS与JSONP的使用目的相同，但是比JSONP更强大。

JSONP只支持GET请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。

# 参考

跨域资源共享 CORS 详解：http://www.ruanyifeng.com/blog/2016/04/cors.html