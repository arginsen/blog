---
title: 前端 | Fetch API
date: 2021-7-6
tags: 
- Browser
- Frontend
- HTTP
categories: notes
photos:
    - /blog/img/fetch.jpg
---

<br>
<!--more-->

`fetch()` 是 XMLHttpRequest 的升级版，用于在 JavaScript 脚本里面发出 HTTP 请求。浏览器原生提供这个对象。

# 基本用法

`fetch()` 的功能与 XMLHttpRequest 基本相同，但有三个主要的差异。

（1）`fetch()` 使用 Promise，不使用回调函数，因此大大简化了写法，写起来更简洁。

（2）`fetch()` 采用模块化设计，API 分散在多个对象上（Response 对象、Request 对象、Headers 对象），更合理一些；相比之下，XMLHttpRequest 的 API 设计并不是很好，输入、输出、状态都在同一个接口管理，容易写出非常混乱的代码。

（3）`fetch()` 通过数据流（Stream 对象）处理数据，可以分块读取，有利于提高网站性能表现，减少内存占用，对于请求大文件或者网速慢的场景相当有用。XMLHTTPRequest 对象不支持数据流，所有的数据必须放在缓存里，不支持分块读取，必须等待全部拿到后，再一次性吐出来。

```js
fetch(url)
  .then(...)
  .catch(...)
```

如下例书写：

```js
fetch('https://v1.hitokoto.cn')
  .then(response => response.json())
  .then(data => {
    const hitokoto = document.getElementById('hitokoto');
    hitokoto.innerText = data.hitokoto
  })
  .catch(console.error);
```

`fetch()` 接收到的 `response` 是一个 Stream 对象，`response.json()` 是一个异步操作，取出所有内容，并将其转为 JSON 对象。

`Promise` 可以使用 `await` 语法改写，使得语义更清晰。`await` 语句必须放在 `try...catch` 里面，这样才能捕捉异步操作中可能发生的错误。

```js
// 上例可以写为
async function getOneword() {
  try {
    let response = await fetch('https://v1.hitokoto.cn')
    return await response.json()
  } catch (error) {
    console.log('Request Failed', error)
  }
}
```

# 请求

`fetch(url, optionObj)` 方法接受两个参数，第一个为参数，第二个为自定义的配置。

- `method`：HTTP 请求的方法，POST、DELETE、PUT都在这个属性设置。
- `headers`：一个对象，用来定制 HTTP 请求的标头。
- `body`：POST 请求的数据体。
- `cache`：指定如何处理缓存。可能的取值如下：
  - `default`：默认值，先在缓存里面寻找匹配的请求。
  - `no-store`：直接请求远程服务器，并且不更新缓存。
  - `reload`：直接请求远程服务器，并且更新缓存。
  - `no-cache`：将服务器资源跟本地缓存进行比较，有新版本才使用服务器资源，否则使用缓存。
  - `force-cache`：缓存优先，只有不存在缓存的情况下，才请求远程服务器。
  - `only-if-cached`：只检查缓存，如果缓存里面不存在，将返回504错误。
- `mode`：指定请求的模式。可能的取值如下：
  - `cors`：默认值，允许跨域请求。
  - `same-origin`：只允许同源请求。
  - `no-cors`：请求方法只限于 GET、POST 和 HEAD，并且只能使用有限的几个简单标头，不能添加跨域的复杂标头，相当于提交表单所能发出的请求。
- `credentials`：指定是否发送 Cookie。可能的取值如下：
  - `same-origin`：默认值，同源请求时发送 Cookie，跨域请求时不发送。
  - `include`：不管同源请求，还是跨域请求，一律发送 Cookie。（跨域需设定）
  - `omit`：一律不发送。
- `signal`：指定一个 AbortSignal 实例，用于取消 fetch() 请求。
- `keepalive`：用于页面卸载时，告诉浏览器在后台保持连接，继续发送数据。
- `redirect`：指定 HTTP 跳转的处理方法。可能的取值如下：
  - `follow`：默认值，fetch() 跟随 HTTP 跳转。
  - `error`：如果发生跳转，fetch() 就报错。
  - `manual`：fetch() 不跟随 HTTP 跳转，但是 response.url 属性会指向新的 URL，response.redirected 属性会变为 true，由开发者自己决定后续如何处理跳转。
- `integrity`：指定一个哈希值，用于检查 HTTP 回应传回的数据是否等于这个预先设定的哈希值。
- `referrer`：用于设定fetch()请求的referer标头。
- `referrerPolicy`：用于设定Referer标头的规则。可能的取值如下：
  - `no-referrer-when-downgrade`：默认值，总是发送Referer标头，除非从 HTTPS 页面请求 HTTP 资源时不发送。
  - `no-referrer`：不发送Referer标头。
  - `origin`：Referer标头只包含域名，不包含完整的路径。
  - `origin-when-cross-origin`：同源请求Referer标头包含完整的路径，跨域请求只包含域名。
  - `same-origin`：跨域请求不发送 Referer，同源请求发送。
  - `strict-origin`：Referer 标头只包含域名，HTTPS 页面请求 HTTP 资源时不发送Referer标头。
  - `strict-origin-when-cross-origin`：同源请求时 Referer 标头包含完整路径，跨域请求时只包含域名，HTTPS 页面请求 HTTP 资源时不发送该标头。
  - `unsafe-url`：不管什么情况，总是发送 Referer 标头。


```js
const response = fetch(url, {
  method: "GET",
  headers: {
    "Content-Type": "text/plain;charset=UTF-8"
  },
  body: undefined,
  referrer: "about:client",
  referrerPolicy: "no-referrer-when-downgrade",
  mode: "cors", 
  credentials: "same-origin",
  cache: "default",
  redirect: "follow",
  integrity: "",
  keepalive: false,
  signal: undefined
});
```

## 取消请求

`fetch()` 请求发送以后，如果中途想要取消，需要使用 `AbortController` 对象。

```js
// 一秒后取消请求
let controller = new AbortController();
setTimeout(() => controller.abort(), 1000);

try {
  let response = await fetch('/long-operation', {
    signal: controller.signal
  });
} catch(err) {
  if (err.name == 'AbortError') {
    console.log('Aborted!');
  } else {
    throw err;
  }
}
```

实际同 axios 的取消模式差不多：

```js
const CancelToken = axios.CancelToken;
const source = CancelToken.source();
source.cancel(message);

axios.get('/get/server', {
  cancelToken: source.token
})
```

# 响应

fetch() 请求后返回一个 promise 对象，也是 Stream 对象。该 response 对象上定义了完整的属性、方法供使用

## 属性

### response.ok

返回一个布尔值，表示请求是否成功，`true` 对应 HTTP 请求的状态码 200 到 299，false对应其他的状态码。

### response.status

返回一个数字，表示 HTTP 回应的状态码（例如200，表示成功请求）。fetch 请求发出后只有网络错误或无法连接时才会报错，其他请求均认为请求成功，所以需要加以判断

```js
// 通过 response.status 判断请求是否成功
async function fetchText() {
  let response = await fetch('/readme.txt');
  if (response.status >= 200 && response.status < 300) {
    return await response.text();
  } else {
    throw new Error(response.statusText);
  }
}

// 或者也可以用上一个属性 ok
if (response.ok) {
  // success
} else {
  // error
}
```

### response.statusText

返回一个字符串，表示 HTTP 回应的状态信息（例如请求成功以后，服务器返回"OK"）。

### response.url

返回请求的 URL。如果 URL 存在跳转，该属性返回的是最终 URL。

### response.type

返回请求的类型。可能的值如下：

basic：普通请求，即同源请求。
cors：跨域请求。
error：网络错误，主要用于 Service Worker。
opaque：如果 `fetch()` 请求的 type 属性设为 `no-cors`，就会返回这个值
opaqueredirect：如果 `fetch()` 请求的 `redirect` 属性设为 `manual`，就会返回这个值

### response.redirected

返回一个布尔值，表示请求是否发生过跳转。

### response.headers

一个对象，为 HTTP 回应的所有标头。提供如下方法操作标头

Headers.get()：根据指定的键名，返回键值。
Headers.has()： 返回一个布尔值，表示是否包含某个标头。
Headers.set()：将指定的键名设置为新的键值，如果该键名不存在则会添加。
Headers.append()：添加标头。
Headers.delete()：删除标头。
Headers.keys()：返回一个遍历器，可以依次遍历所有键名。
Headers.values()：返回一个遍历器，可以依次遍历所有键值。
Headers.entries()：返回一个遍历器，可以依次遍历所有键值对（[key, value]）。
Headers.forEach()：依次遍历标头，每个标头都会执行一次参数函数。

```js
// 读取标头
let response =  await fetch(url);  
let headers = response.headers
headers.get('Content-Type') // application/json; charset=utf-8

// 遍历键名
for(let key of headers.keys()) {
  console.log(key);
}

// 遍历键值
for(let value of headers.values()) {
  console.log(value);
}

// 遍历键名与值
headers.forEach(
  (value, key) => console.log(key, ':', value)
);
```

### response.body

Response 对象暴露出的底层接口，返回一个 ReadableStream 对象，可以用来分块读取内容，应用之一就是显示下载的进度。

```js
const response = await fetch('flower.jpg');
const reader = response.body.getReader(); // getReader 方法返回一个遍历器

while(true) {
  const {done, value} = await reader.read(); // read 方法每次返回一个对象，表示本次读取的内容块

  // done 是一个布尔值，判断有没有读完，若读完跳出循环
  if (done) {
    break;
  }

  // value 是一个 arrayBuffer 数组，表示内容块的内容
  // value.length 表示当前块的大小
  console.log(`Received ${value.length} bytes`) 
}
```

## 方法

reponse 对象根据服务器返回的不同类型的数据，提供了不同的读取方法。以下方法均是异步，返回 promise 对象

### response.text()

用于获取文本数据，如 html 文件

```js
const response = await fetch('/users.html');
const body = await response.text();
document.body.innerHTML = body
```

### response.json()

获取服务器返回的 JSON 数据

### response.formData()

主要用在 Service Worker 里面，拦截用户提交的表单，修改某些数据以后，再提交给服务器。

### response.blob()

用于获取二进制文件。

```js
// 根据获取的照片的二进制，生成 url
const response = await fetch('flower.jpg');
const myBlob = await response.blob();
const objectURL = URL.createObjectURL(myBlob);

const myImage = document.querySelector('img');
myImage.src = objectURL;
```

### response.arrayBuffer()

主要用于获取流媒体文件。

```js
// 浏览器创建音频控件，再获取音频文件，将读取的缓存阵列连接到浏览器的音频播放控件
const audioCtx = new window.AudioContext();
const source = audioCtx.createBufferSource();

const response = await fetch('song.ogg');
const buffer = await response.arrayBuffer();

const decodeData = await audioCtx.decodeAudioData(buffer);
source.buffer = buffer;
source.connect(audioCtx.destination);
source.loop = true;
```

### Response.clone()

Stream 对象只能读取一次，读取完就没了。因此之前五个读取方法，只能使用一个，否则会报错。

```js
let text =  await response.text();
let json =  await response.json();  // 报错
```

而 clone() 方法可以创建 response 副本，供多次读取

```js
const response1 = await fetch('flowers.jpg');
const response2 = response1.clone();

const myBlob1 = await response1.blob();
const myBlob2 = await response2.blob();

image1.src = URL.createObjectURL(myBlob1);
image2.src = URL.createObjectURL(myBlob2);
```

# 参考

fetch 教程：https://www.ruanyifeng.com/blog/2020/12/fetch-tutorial.html