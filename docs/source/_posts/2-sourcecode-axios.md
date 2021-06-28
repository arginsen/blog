---
title: 源码 | axios
date: 2021-6-20 8:29:00
tags: 
- axios
categories: notes
photos:
    - /blog/img/sourcecode.jpg
---

<br>
<!--more-->


# 调试

`axios` 打包后有 `sourcemap` 文件

```
下载、解压，学习源码用了 0.19.0 版本
https://github.com/axios/axios/archive/refs/tags/v0.19.0.zip


进入目录、安装运行
npm install
npm start
```

可以查看 `package.json` 文件，`npm start` 运行的是 `/sandbox/server.js`，通过 `node` 发布在 3000 端口，浏览器打开 http://localhost:3000，可以浏览器打断点查看`axios`运行流程是怎样的

此外，源码目录中 `examples` 目录下有已经写好的例子，方便调试。下边做一些必要的修改，注释掉原有的启服务的代码，新添如下代码

```js
// ./examples/server.js

server = http.createServer(function (req, res) {
  var url = req.url;

  // // Process axios itself
  // if (/axios\.min\.js$/.test(url)) {
  //   pipeFileToResponse(res, '../dist/axios.min.js', 'text/javascript');
  //   return;
  // }
  // if (/axios\.min\.map$/.test(url)) {
  //   pipeFileToResponse(res, '../dist/axios.min.map', 'text/javascript');
  //   return;
  // }
  // if (/axios\.amd\.min\.js$/.test(url)) {
  //   pipeFileToResponse(res, '../dist/axios.amd.min.js', 'text/javascript');
  //   return;
  // }
  // if (/axios\.amd\.min\.map$/.test(url)) {
  //   pipeFileToResponse(res, '../dist/axios.amd.min.map', 'text/javascript');
  //   return;
  // }

  // 修改方便调试 -------------------
  if (/axios\.min\.js$/.test(url)) {
    pipeFileToResponse(res, '../dist/axios.js', 'text/javascript');
    return;
  }

  if (/axios\.map$/.test(url)) {
    pipeFileToResponse(res, '../dist/axios.map', 'text/javascript');
    return;
  }
  // -------------------------------
```

修改好后直接运行

```
npm run examples

node ./examples/server.js -p 5000 // 或者直接指定端口同理
```

# 源码

## 实例结构

在运行了 `/sandbox/server.js` 之后，浏览器打开 `http://localhost:3000`，在调试器中可以直接打印 `axios`

```js
console.log({axios: axios});
```

显示如下：

```
{
  axios: ƒ wrap()
    Axios: ƒ Axios(instanceConfig)
    Cancel: ƒ Cancel(message)
    CancelToken: ƒ CancelToken(executor)
    all: ƒ all(promises)
    create: ƒ create(instanceConfig)
    default: ƒ wrap()
    defaults: {transformRequest: Array(1), transformResponse: Array(1), timeout: 0, xsrfCookieName: "XSRF-TOKEN", adapter: ƒ, …}
    delete: ƒ wrap()
    get: ƒ wrap()
    getUri: ƒ wrap()
    head: ƒ wrap()
    interceptors: {request: InterceptorManager, response: InterceptorManager}
    isCancel: ƒ isCancel(value)
    options: ƒ wrap()
    patch: ƒ wrap()
    post: ƒ wrap()
    put: ƒ wrap()
    request: ƒ wrap()
    spread: ƒ spread(callback)
    arguments: (...)
    caller: (...)
    length: 0
    name: "wrap"
    prototype: {constructor: ƒ}
    __proto__: ƒ ()
    [[FunctionLocation]]: bind.js:5
    [[Scopes]]: Scopes[2]
}

```

可以看出，`axios` 本身作为一个函数，函数的位置在 `bind.js`，这是因为 `axios` 在创建时，是通过 `Axios` 构造函数先生成实例，接着使用 `bind` 方法将 `Axios` 原型上的 `request` 方法的 `this` 指向刚生成的新实例（`context`），并返回 `request` 函数，而`bind` 又返回 `wrap` 函数，即 `axios` 实际上就是被绑在新实例下的 `request` 函数

除作为一个函数本身所有的 `arguments、caller、length、name、__proto__` 属性，随后在绑定完 `request` 后，又对实例（指向新实例的`request`函数）进行扩展，将 `Axios` 原型上的属性都复制到 `instance` 上；并且 `Axios` 原型上的方法 `this` 均指向 `context`（同之前的bind）；接着又扩展 `instance`，将 `Axios` 传递了默认设置的新实例的属性都复制到 `instance` 上，最终返回 `instance`。

所以 `axios` 对象本身其实是一个函数，一个拥有很多属性的函数。

## 源码结构

```
-examples
-lib
  -adapters
    -http.js node环境下http请求适配器
    -README.md
    -xhr.js 浏览器环境XMLHttpReq对象适配器
  -cancel 取消请求
    -Cancel.js 构造函数Cancel，cancel对象是请求被取消时抛出的对象
    -CancelToken.js 构造函数CancelToken，用于取消请求的对象
    -isCancel.js 判断请求失败时的错误对象是否是Cancel对象
  -core
    -Axios 构造函数Axios，Axios.prototype.request方法
    -createError.js 创建一个error对象
    -dispatchRequest.js 使用对应环境的适配器向服务器发起请求
    -enhanceError.js 使用指定信息升级error对象
    -InterceptorManager.js 构造函数InterceptorManager，拦截器
    -mergeConfig.js 合并两个配置项为一个配置项
    -README.md
    -settle.js 根据响应状态码决定promise的状态
    -transformData.js 请求之前或响应之后处理data或者response
  -helpers 协助模块
    -bind.js 绑定方法到指定this上
    -buildURL.js 将params查询字符串连接到url之后
    -combineURLs.js 连接指定的ur1创建出新的url
    -cookies.js 读写、移除浏览器cookie
    -deprecatedMethod.js 向不赞成的方法发出警告
    -isAbsoluteURL.js 检查url是否是绝对路径
    -isURLSameOrigin.js 请求url和当前页面地址是否处于同一域
    -normalizeHeaderName.js 标准化header名
    -parseHeaders.js 将请求头或响应头转换成一个对象
    -README.md
    -spread.js 将一个函数包装成接收一个数组参数并将其展开成参数列表的函数
  axios.js 入口，创建axios对象
  defaults.js 默认配置
  utils.js 工具方法
-sandbox
-test
```

## 源码解析

### axios.js

如上介绍的，该文件主要用于创建 `axios` 实例，实际上做了如下事：

1. 通过 `Axios` 构造函数先生成实例
2. 使用 `bind` 方法将 `Axios` 原型上的 `request` 方法的 `this` 指向刚生成的新实例（`context`），并返回 `request` 函数，赋给 `instance`
3. `bind` 又返回 `wrap` 函数，即 `axios` 实际上就是被绑在新实例下的 `request` 函数                                                        

```js
// ./axios.js

'use strict';

var utils = require('./utils');
var bind = require('./helpers/bind');
var Axios = require('./core/Axios');
var mergeConfig = require('./core/mergeConfig');
var defaults = require('./defaults');

/**
 * Create an instance of Axios
 *
 * @param {Object} defaultConfig The default config for the instance
 * @return {Axios} A new instance of Axios
 */
function createInstance(defaultConfig) {
  var context = new Axios(defaultConfig);
  var instance = bind(Axios.prototype.request, context);

  // Copy axios.prototype to instance
  utils.extend(instance, Axios.prototype, context);

  // Copy context to instance
  utils.extend(instance, context);

  return instance;
}

// Create the default instance to be exported
var axios = createInstance(defaults);

// Expose Axios class to allow class inheritance
axios.Axios = Axios;

// Factory for creating new instances
axios.create = function create(instanceConfig) {
  return createInstance(mergeConfig(axios.defaults, instanceConfig));
};

// Expose Cancel & CancelToken
axios.Cancel = require('./cancel/Cancel');
axios.CancelToken = require('./cancel/CancelToken');
axios.isCancel = require('./cancel/isCancel');

// Expose all/spread
axios.all = function all(promises) {
  return Promise.all(promises);
};
axios.spread = require('./helpers/spread');

module.exports = axios;

// Allow use of default import syntax in TypeScript
module.exports.default = axios;

```







# 参考

学习 axios 源码整体架构-若川 ：https://mp.weixin.qq.com/s/GNYpgHo97xml0NxT93dHxQ

创建 axios 对象 ：https://www.cnblogs.com/hahazexia/p/9994209.html