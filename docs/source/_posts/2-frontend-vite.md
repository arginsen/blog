---
title: 前端 | Vite 使用
date: 2021-10-3 18:45:00
tags: 
- Vite
- Frontend
categories: notes
photos:
    - /blog/img/vite.jpg
---

<br>

<!--more-->

# 初始化项目

通过 npm 直接调用 vite 初始化一个 vue 模板的项目
模板可选: vue, vue-ts, react, react-ts 等

```bash
# npm 6.x
npm init vite@latest my-vue-app --template vue-ts

# npm 7+, 需要额外的双横线：
npm init vite@latest my-vue-app -- --template vue

# yarn
yarn create vite my-vue-app --template vue
```

初始化后可以发现 index.html 在项目最外层而不是在 public , 在开发期间 Vite 是一个服务器，index.html 是该 Vite 项目的入口文件

vite 将 index.html 视为源码和模块图的一部分。vite 解析 `<script type="module" src="">`, script 标签的 module 模式可以直接加载 js 模块源码

vite 的根目录用 `<root>` 来代称，一般以当前工作目录作为根目录启动开发服务器，也可以通过 `vite serve som/sub/dir` 来指定一个替代的根目录

# vite 的特点

vite 通过原生 ESM 导入提供了许多主要用于打包场景的增强功能

## npm 依赖解析和预构建

预构建 vite 会将 CMD/UMD 转换为 ESM 格式，预构建由 esbuild 执行，使得 vite 冷启动时间比任何 js 打包器都快

依赖是强缓存

## 模块热重载

vite 有一套原生 ESM 的 HMR API，具有 HMR 功能的框架可以利用该 API 提供即时、准确的更新，无需重新加载页面或清除应用程序状态

## TypeScript

vite 天然支持 ts 文件

vite 仅执行 ts 文件的转译工作，并不执行任何类型检查。
可以安装 `vue-tsc` 运行 `vue-tsc --noEmit` 来对 vue 文件做类型检查

vite 使用 esbuild 将 typescript 转译到 javascript，约是 tsc 速度的 20-30 倍，HMR 更新反应到浏览器小于 50ms

## JSX

jsx 和 tsx 文件同样开箱即用
jsx 的转译同样是通过 esbuild，默认为 react16 的风格

## JSON

JSON 可以直接导入，也支持具名导入

## Glob 导入

vite 支持使用 import.meta.glob 函数从文件系统导入多个模块

```js
const modules = importmeta.glob('./dir/*.js')

// 被转译为：
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
  './dir/bar.js': () => import('./dir/bar.js')
}
```

## WebAssembly

预编译的 .wasm 文件可以直接被导入 —— 默认导出一个函数，返回值为所导出 wasm 实例对象的 Promise：

```js
import init from './example.wasm'

init().then((exports) => {
  exports.test()
})
```

## 构建优化

css 代码分割

vite 会自动将一个异步 chunk 模块中使用到的 css 代码抽取出来并为其生成一个单独的文件。这个css 文件将在该异步 chunk 加载完成时自动通过一个 `<link>` 标签载入，该异步 chunk 会保证只在 css 加载完毕后再执行

预加载指令生成 `<link rel="modulepreload">`

异步 chunk 加载优化

# 配置项一览

> 介绍: https://cn.vitejs.dev/config/

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import ElementPlus from "unplugin-element-plus/vite"

export default defineConfig({
  base: '/',
  root: process.cwd(),
  mode: 'development',
  resolve: {
    alias: {
      '@/': new URL('./src/', import.meta.url).pathname,
      '#/': new URL('./types/', import.meta.url).pathname
    },
  },
  server: {
    https: false,
    port: 3000,
    host: '127.0.0.1',
    proxy: {
      '/api': ''
    }
  },
  plugins: [
    vue(),
    ElementPlus({}),
  ],
  build: {
    // @ts-ignore
    sourcemap: false,
    brotliSize: false,
    // 消除打包大小超过500kb警告
    chunkSizeWarningLimit: 2000
  },
  define: {
    __INTLIFY_PROD_DEVTOOLS__: false
  },
})
```