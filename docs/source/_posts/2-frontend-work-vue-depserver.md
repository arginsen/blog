---
title: 前端 | 使用 vue 
date: 2021-1-15
tags: 
- work
- video.js
- Frontend
categories: notes
photos:
  - /blog/img/frontend.jpg
---

<br>
<!--more-->

如果你的前端应用和后端 API 服务器没有运行在同一个主机上，你需要在开发环境下将 API 请求代理到 API 服务器。这个问题可以通过 vue.config.js 中的 devServer.proxy 选项来配置。

devServer.proxy 可以是一个指向开发环境 API 服务器的字符串：

```js
module.exports = {
  devServer: {
    proxy: 'http://localhost:4000'
  }
}
```

通常不同主机指向不同本地地址即可

```js
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: '<url>',
        ws: true,
        changeOrigin: true
      },
      '/foo': {
        target: '<other_url>'
      }
    }
  }
}
```