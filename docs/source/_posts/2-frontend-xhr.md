---
title: 前端 | XMLHttpRequest
date: 2021-7-3
tags: 
- Browser
- Frontend
- HTTP
categories: notes
photos:
    - /blog/img/xhr.jpg
---

<br>
<!--more-->

XMLHttpRequest（XHR）对象用于与服务器交互。通过 XMLHttpRequest 可以在不刷新页面的情况下请求特定 URL，获取数据。这允许网页在不影响用户操作的情况下，更新页面的局部内容。XMLHttpRequest 在 AJAX 编程中被大量使用。

# 构造函数

XMLHttpRequest()
该构造函数用于初始化一个 XMLHttpRequest 实例对象。在调用下列任何其他方法之前，必须先调用该构造函数，或通过其他方式，得到一个实例对象。

# 属性

## XMLHttpRequest.onreadystatechange

XMLHttpRequest 的 readyState 属性改变的时候，触发 readystatechange 事件, callback 函数会被调用。

```js
XMLHttpRequest.onreadystatechange = callback;
```

当一个 XMLHttpRequest 请求被 abort() 方法取消时，其对应的 readystatechange 事件不会被触发。

```js
var xhr = new XMLHttpRequest(),
    method = "GET",
    url = "https://developer.mozilla.org/";

xhr.open(method, url, true);
xhr.onreadystatechange = function () {
  if(xhr.readyState === xhr.DONE && xhr.status === 200) {
    console.log(xhr.responseText)
  }
}
xhr.send();
```

## XMLHttpRequest.readyState(read)

返回一个无符号短整型（unsigned short）数字，代表请求的状态码。

```
值	状态	             描述
0	  UNSENT	          代理被创建，但尚未调用 open() 方法。
1	  OPENED	open()    方法已经被调用。
2	  HEADERS_RECEIVED	send() 方法已经被调用，并且头部和状态已经可获得。
3	  LOADING	          下载中； responseText 属性已经包含部分数据。
4	  DONE	            下载操作已完成。
```

## XMLHttpRequest.response(read)

返回响应的正文。返回的类型为 ArrayBuffer 、 Blob 、 Document 、 JavaScript Object 或 DOMString 中的一个。 这取决于 responseType 属性。

如果请求尚未完成或未成功，则取值是 null 。例外的，读取文本数据时如果将 responseType 的值设置成"text"或空字符串（""）且当请求状态还在是 LOADING readyState (3) 时，response 包含到目前为止该请求已经取得的内容。

## XMLHttpRequest.responseType

一个枚举类型的属性，返回响应数据的类型。它允许我们手动的设置返回数据的类型。如果我们将它设置为一个空字符串，它将使用默认的"text"类型。

## XMLHttpRequest.responseText(read)

当处理一个异步request的时候，尽管当前请求并没有结束，responseText的返回值是当前从后端收到的内容。

当请求状态 readyState 变为 XMLHttpRequest.DONE (4)，且 status 值为 200（"OK"）时，responseText 是全部后端的返回数据

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', '/server', true);

// If specified, responseType must be empty string or "text"
xhr.responseType = 'text';

// 
xhr.onload = function () {
    if (xhr.readyState === xhr.DONE) {
        if (xhr.status === 200) {
            console.log(xhr.response);
            console.log(xhr.responseText);
        }
    }
};

xhr.send(null); // 请求为 get
```

## XMLHttpRequest.responseURL(read)

返回经过序列化（serialized）的响应 URL，如果该 URL 为空，则返回空字符串

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://example.com/test', true);
xhr.onload = function () {
  console.log(xhr.responseURL); // http://example.com/test
};
xhr.send(null);
```

## XMLHttpRequest.status(read)

返回了 XMLHttpRequest 响应中的数字状态码，status码是标准的 HTTP status codes。status 的值是一个无符号短整型。在请求完成前，status的值为0。值得注意的是，如果 XMLHttpRequest 出错，浏览器返回的 status 也为 0。

```js
var xhr = new XMLHttpRequest();
console.log('UNSENT', xhr.status);

xhr.open('GET', '/server', true);
console.log('OPENED', xhr.status);

xhr.onprogress = function () {
  console.log('LOADING', xhr.status);
};

xhr.onload = function () {
  console.log('DONE', xhr.status);
};

xhr.send(null);

/**
 * 输出如下：
 *
 * UNSENT（未发送） 0
 * OPENED（已打开） 0
 * LOADING（载入中） 200
 * DONE（完成） 200
 */
```

## XMLHttpRequest.timeout

请求在被自动终止前所消耗的毫秒数。默认值为 0，意味着没有超时。

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', '/server', true);

xhr.timeout = 2000; // 超时时间，单位是毫秒

xhr.onload = function () {
  // 请求完成。在此进行处理。
};

xhr.ontimeout = function (e) {
  // XMLHttpRequest 超时。在此做某事。
};

xhr.send(null);
```

## XMLHttpRequest.withCredentials

一个布尔值 (en-US)，用来指定跨域 Access-Control 请求是否应当带有授权信息，如 cookie 或授权 header 头。

# 事件

progress：检测数据发生变化，周期性地触发。
load：传输完成，所有数据保存在 response 里
abort：取消请求
error：当 request 遭遇错误时触发
loadstart：接收到响应数据时触发。
loadend：当请求结束时触发, 无论请求成功 ( load) 还是失败 (abort 或 error)。
timeout：在预设时间内没有接收到响应时触发。

## xhr 对象添加 addEventListener 

```js
var xhr = new XMLHttpRequest();

// xhr 对象直接绑定事件
xhr.addEventListener("progress", updateProgress);
xhr.addEventListener("load" , transferComplete);
xhr.addEventListener("error", transferFailed  );
xhr.addEventListener("abort", transferCanceled);

xhr.open();

// 再定义具体的事件方法
function updateProgress (Event) {
  if (Event.lengthComputable) {
    var percentComplete = Event.loaded / Event.total * 100;
    // ...
  } else {
    // 总大小未知时不能计算进程信息
  }
}
```

## 使用属性 on + ()

```js
xhr.onprogress = function () {
  // 请求进度发生变化
};

xhr.onload = function () {
  // 请求完成。在此进行处理。
};

xhr.onabort = function () {
  // 请求取消
};

xhr.onerror = function () {
  // 请求发生错误
};
```

# 方法

## XMLHttpRequest.abort()

如果请求已被发出，则立刻中止请求。在 axios 源码中，请求发送之前会检查用户是否定义 `cancelToken` ，如生成则执行 abort()

## XMLHttpRequest.getAllResponseHeaders()

以字符串的形式返回所有用 CRLF 分隔的响应头，如果没有收到响应，则返回 null。

## XMLHttpRequest.getResponseHeader()

返回包含指定响应头的字符串，如果响应尚未收到或响应中不存在该报头，则返回 null。

```js
xhr.getResponseHeader("Content-Type")
```

## XMLHttpRequest.open()

初始化一个请求

```js
xhr.open(method, url, async, user, password);

// method : 要使用的 HTTP 方法，比如「GET」、「POST」
// url : 请求的地址
// async : 可选。是否异步执行操作，默认为 true
// user : 可选。用户名用于认证用途，默认为 null。
// password : 可选。密码用于认证用途，默认为 null。
```

# 参考

XMLHttpRequest ：https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest