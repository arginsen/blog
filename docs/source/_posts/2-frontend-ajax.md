---
title: 前端 | Ajax
date: 2021-7-4
tags: 
- Browser
- Frontend
- Ajax
- HTTP
categories: notes
photos:
    - /blog/img/ajax.jpg
---

<br>
<!--more-->

# 定义

`AJAX` 全称 `Async JavaScript and XML`
即异步的 `JavaScript` 和 `XML`，是一种创建交互式网页应用的网页开发技术，可以在不重新加载整个网页的情况下，与服务器交换数据，并且更新部分网页。

`Ajax` 的实际原理比较简单，通过 `XMLHttpRequest` 对象来向服务器发异步请求，从服务器获得数据，然后用 `JavaScript` 来操作 `DOM` 更新页面

# 实现

## 创建 XMLHttpRequest 对象

```js
const xhr = new XMLHttpRequest();
```

## 与服务器建立连接

```js
xhr.open(method, url, [async][, user][, password])

// method: 当前请求方式
// url: 当前请求地址
// async: 布尔值，是否异步执行操作，默认 true
// user: 可选的用户名用于认证用途，默认 null
// password: 可选的密码用于认证用途，默认为 null
```

## 绑定 onreadystatechange 事件

`onreadystatechange` 事件用于监听服务器端的通信状态，主要监听的属性为 `XMLHttpRequest.readyState` ，该属性有五种状态，每次状态发生变化，就会触发 `onreadystatechange` 事件。
`XMLHttpRequest.responseText` 属性用于接收服务器端的响应结果

值 | 状态 | 描述
-- | ---- | ---
0  | UNSENT(未打开)                | open() 方法还未被调用
1  | OPENED(未发送)                | send() 方法还未被调用
2  | HEADERS_RECEIVED(以获取响应头) | send() 方法已经被调用，响应头和响应状态已经返回
3  | LOADING(正在下载响应体)        | 响应体下载中，responseText 中已经获取部分数据
4  | DONE(请求完成)                | 整个请求过程已完毕

```js
var request = new XMLHttpRequest()
var response

request.onreadystatechange = function(e) {
    if (request.readyState === 4) {
        if (request.status >= 200 && request.status <= 300) {
            response = request.responseText
        } else {
            console.log('Request failed with status code ' + request.status)
        }
    }
}

```

## 给服务端发送数据

```js
xhr.send([body])

// body: XHR 请求中要发送的数据体，如不传递数据则为 null

// 如使用 get 请求发送数据：
// 1. 将请求数据添加到 open() 方法中的 url 里
// 2. 发送请求数据中的 send() 方法中参数设为 null
```

# 封装

封装一个简单的 `ajax` 请求

```js
function ajax(options) {
    //创建XMLHttpRequest对象
    const xhr = new XMLHttpRequest()

    //初始化参数的内容
    options = options || {}
    options.type = (options.type || 'GET').toUpperCase()
    options.dataType = options.dataType || 'json'
    const params = options.data

    //发送请求
    if (options.type === 'GET') {
        xhr.open('GET', options.url + '?' + params, true)
        xhr.send(null)
    } else if (options.type === 'POST') {
        xhr.open('POST', options.url, true)
        xhr.send(params)

    //接收请求
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            let status = xhr.status
            if (status >= 200 && status < 300) {
                options.success && options.success(xhr.responseText, xhr.responseXML)
            } else {
                options.fail && options.fail(status)
            }
        }
    }
}
```

# axios 里的 XHR 请求

```js
function xhrAdapter(config) {
  return new Promise(function dispatchXhrRequest(resolve, reject) {
    var requestData = config.data;
    var requestHeaders = config.headers;

    if (utils.isFormData(requestData)) {
      delete requestHeaders['Content-Type']; // Let the browser set it
    }

    var request = new XMLHttpRequest();

    // HTTP basic authentication
    if (config.auth) {
      var username = config.auth.username || '';
      var password = config.auth.password || '';
      requestHeaders.Authorization = 'Basic ' + btoa(username + ':' + password);
    }

    // 与服务器建立连接
    request.open(config.method.toUpperCase(), buildURL(config.url, config.params, config.paramsSerializer), true);

    // Set the request timeout in MS
    request.timeout = config.timeout;

    // Listen for ready state
    request.onreadystatechange = function handleLoad() {
      if (!request || request.readyState !== 4) {
        return;
      }

      // The request errored out and we didn't get a response, this will be
      // handled by onerror instead
      // With one exception: request that using file: protocol, most browsers
      // will return status as 0 even though it's a successful request
      // 检测 http 状态码
      if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0)) {
        return;
      }

      // Prepare the response
      var responseHeaders = 'getAllResponseHeaders' in request ? parseHeaders(request.getAllResponseHeaders()) : null;
      var responseData = !config.responseType || config.responseType === 'text' ? request.responseText : request.response;
      var response = {
        data: responseData,
        status: request.status,
        statusText: request.statusText,
        headers: responseHeaders,
        config: config,
        request: request
      };

      settle(resolve, reject, response); // 用 http 状态码来处理数据的去向

      // Clean up request
      request = null;
    };

    // Handle browser request cancellation (as opposed to a manual cancellation)
    request.onabort = function handleAbort() {
      if (!request) {
        return;
      }

      reject(createError('Request aborted', config, 'ECONNABORTED', request));

      // Clean up request
      request = null;
    };

    // Handle low level network errors
    request.onerror = function handleError() {
      // Real errors are hidden from us by the browser
      // onerror should only fire if it's a network error
      reject(createError('Network Error', config, null, request));

      // Clean up request
      request = null;
    };

    // Handle timeout
    request.ontimeout = function handleTimeout() {
      reject(createError('timeout of ' + config.timeout + 'ms exceeded', config, 'ECONNABORTED',
        request));

      // Clean up request
      request = null;
    };

    // Add xsrf header
    // This is only done if running in a standard browser environment.
    // Specifically not if we're in a web worker, or react-native.
    if (utils.isStandardBrowserEnv()) {
      var cookies = require('./../helpers/cookies');

      // Add xsrf header
      var xsrfValue = (config.withCredentials || isURLSameOrigin(config.url)) && config.xsrfCookieName ?
        cookies.read(config.xsrfCookieName) :
        undefined;

      if (xsrfValue) {
        requestHeaders[config.xsrfHeaderName] = xsrfValue;
      }
    }

    // Add headers to the request
    if ('setRequestHeader' in request) {
      utils.forEach(requestHeaders, function setRequestHeader(val, key) {
        if (typeof requestData === 'undefined' && key.toLowerCase() === 'content-type') {
          // Remove Content-Type if data is undefined
          delete requestHeaders[key];
        } else {
          // Otherwise add header to the request
          request.setRequestHeader(key, val);
        }
      });
    }

    // Add withCredentials to request if needed
    // withCredentials 是否允许开启cookie ，指定跨域 Access-Control 请求是否应带有授权信息
    if (config.withCredentials) {
      request.withCredentials = true;
    }

    // Add responseType to request if needed
    if (config.responseType) {
      try {
        request.responseType = config.responseType;
      } catch (e) {
        // Expected DOMException thrown by browsers not compatible XMLHttpRequest Level 2.
        // But, this can be suppressed for 'json' type as it can be parsed by default 'transformResponse' function.
        if (config.responseType !== 'json') {
          throw e;
        }
      }
    }

    // Handle progress if needed
    if (typeof config.onDownloadProgress === 'function') {
      request.addEventListener('progress', config.onDownloadProgress);
    }

    // Not all browsers support upload events
    if (typeof config.onUploadProgress === 'function' && request.upload) {
      request.upload.addEventListener('progress', config.onUploadProgress);
    }

    if (config.cancelToken) { // 处理取消
      // Handle cancellation
      config.cancelToken.promise.then(function onCanceled(cancel) {
        if (!request) {
          return;
        }

        request.abort(); // 取消请求
        reject(cancel); // 抛出取消信息
        // Clean up request
        request = null;
      });
    }

    if (requestData === undefined) {
      requestData = null;
    }

    // Send the request
    request.send(requestData);
  });
}
```