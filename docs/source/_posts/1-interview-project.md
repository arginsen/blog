---
title: 面试 | 前端工程化
date: 2020-9-1
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

# webpack

## webpack 是什么，有哪些功能

webpack 是一个用于现代 JavaScript 应用程序的静态模块打包工具，静态模块指开发阶段可以被 webpack 直接引用的资源

webpack 拥有编译代码的能力，可以提高效率，解决浏览器兼容问题；
有模块整合能力，提高性能，可维护性，解决浏览器频繁请求文件的问题；
万物皆可模块能力，项目维护性强，支持不同种类的前端模块类型，统一的模块化方案，所有资源加载都可通过代码控制

## webpack 于 grunt、gulp 的不同？

Grunt、Gulp是基于任务运⾏的⼯具： 它们会⾃动执⾏指定的任务，就像流⽔线，把资源放上去然后通过不同插件进⾏加⼯，它们包含活跃的社区，丰富的插件，能⽅便的打造各种⼯作流

Webpack是基于模块化打包的⼯具: ⾃动化处理模块，webpack把⼀切当成模块，当 webpack 处理应⽤程序时，它会递归地构建⼀个依赖关系图 (dependency graph)，其中包含应⽤程序需要的每个模块，然后将所有这些模块打包成⼀个或多个 bundle。 

## webpack 和 rollup 的优劣

webpack适⽤于⼤型复杂的前端站点构建: webpack有强⼤的loader和插件⽣态,打包后的⽂件实际上就是⼀个⽴即执⾏函数，这个⽴即执⾏函数接收⼀个参数，这个参数是模块对象，键为各个模块的路径，值为模块内容。⽴即执⾏函数内部则处理模块之间的引⽤，执⾏模块等,这种情况更适合⽂件依赖复杂的应⽤开发。 

rollup适⽤于基础库的打包，如vue、d3等: Rollup 就是将各个模块打包进⼀个⽂件中，并且通过 Tree-shaking 来删除⽆⽤的代码,可以最⼤程度上降低代码体积,但是rollup没有webpack如此多的的如代码分割、按需加载等⾼级功能，其更聚焦于库的打包，因此更适合库的开发

## 有哪些常见的 loader

loader的执行顺序是从右向左执行的

- file-loader：把⽂件输出到⼀个⽂件夹中，在代码中通过相对 URL 去引⽤输出的⽂件 
- url-loader：和 file-loader 类似，但是能在⽂件很⼩的情况下以 base64 的⽅式把⽂件内容注⼊到代码中去 
- source-map-loader：加载额外的 Source Map ⽂件，以⽅便断点调试 
- image-loader：加载并且压缩图⽚⽂件 
- babel-loader：把 ES6 转换成 ES5 
- css-loader：加载 CSS，⽀持模块化、压缩、⽂件导⼊等特性 
- style-loader：把 CSS 代码注⼊到 JavaScript 中，通过 DOM 操作去加载 CSS。 
- eslint-loader：通过 ESLint 检查 JavaScript 代码

## webpack 的 plugin 是什么？ 有哪些常见的 plugin

webpack 中的 plugin 赋予其各种灵活的功能，例如打包优化、资源管理、环境变量注入等，它们会运行在 webpack 的不同阶段（钩子 / 生命周期），贯穿了 webpack 整个编译周期

其本质是一个具有 apply 方法 javascript 对象，apply 方法会被 webpack compiler 调用，因此在整个编译生命周期都可以访问 compiler 对象

```js
class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    compiler.hooks.run.tap(pluginName, (compilation) => {
      console.log('webpack 构建过程开始！');
    });
  }
}

module.exports = ConsoleLogOnBuildWebpackPlugin;
```

关于整个编译生命周期钩子 compiler.hooks ，有如下：

- entry-option ：初始化 option
- run
- compile： 真正开始的编译，在创建 compilation 对象之前
- compilation ：生成好了 compilation 对象
- make 从 entry 开始递归分析依赖，准备对每个模块进行 build
- after-compile： 编译 build 过程结束
- emit ：在将内存中 assets 内容写到磁盘文件夹之前
- after-emit ：在将内存中 assets 内容写到磁盘文件夹之后
- done： 完成所有的编译过程
- failed： 编译失败的时候

一些常见的 plugin：

- define-plugin：定义环境变量 
- html-webpack-plugin：简化html⽂件创建 
- uglifyjs-webpack-plugin：通过 UglifyES 压缩 ES6 代码 
- webpack-parallel-uglify-plugin: 多核压缩，提⾼压缩速度 
- webpack-bundle-analyzer: 可视化webpack输出⽂件的体积 
- mini-css-extract-plugin: CSS提取到单独的⽂件中，⽀持按需加载 

## webpack 的 loader 和 plugin 的不同？

Loader的作⽤是让 webpack 拥有了加载和解析⾮ JavaScript ⽂件的能⼒；在 module.rules 中配置，也就是说他作为模块的解析规则⽽存在。 类型为数组，每⼀项都是⼀个 Object ，⾥⾯描述了对于什么类型的⽂件（ test ），使⽤什么加载( loader )和使⽤的参数（ options ） 

Plugin 可以扩展 webpack 的功能，让 webpack 具有更多的灵活性。 在 Webpack 运⾏的⽣命周期中会⼴播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果；在 plugins 中单独配置。 类型为数组，每⼀项是⼀个 plugin 的实例，参数都通过构造函数传⼊。

## webpack 的构建流程

初始化流程：从配置文件和 Shell 语句中读取与合并参数，并初始化需要使用的插件和配置插件等执行环境所需要的参数
编译构建流程：从 Entry 发出，针对每个 Module 串行调用对应的 Loader 去翻译文件内容，再找到该 Module 依赖的 Module，递归地进行编译处理；最终得到依赖关系
输出流程：根据依赖关系对编译后的 Module 组合成 Chunk，把 Chunk 转换成文件，输出到文件系统

## 如何编写一个 loader 或 plugin？

loader 实际做的工作是转义源文件的内容，配置的多个 loader 通过链式操作，将源文件翻译成目标形式

编写 loader 时需要遵循单一原则，每个 loader 只做一种转义工作；随后可以将翻译完成的结果以返回值的形式输出，交由下一个 loader 处理，或是调用 this.callback() 方法将内容返回给 webpack，也可以通过异步的方式 this.async() 生成一个 callback 函数，将处理结果输出

相比 loader ，plugin 则是按照 webpack 的不同生命周期所广播出的事件，来监听目标节点的事件来实现特定的功能

## webpack 的热更新

1. 通过webpack-dev-server创建两个服务器：提供静态资源的服务（express）和Socket服务
2. express server 负责直接提供静态资源的服务（打包后的资源直接被浏览器请求和解析）
3. socket server 是一个 websocket 的长连接，双方可以通信
4. 当 socket server 监听到对应的模块发生变化时，会生成两个文件.json（manifest文件）和.js文件（update chunk）
5. 通过长连接，socket server 可以直接将这两个文件主动发送给客户端（浏览器）
6. 浏览器拿到两个新的文件后，通过HMR runtime机制，加载这两个文件，并且针对修改的模块进行更新

## 利用 webpack 优化前端性能

实现了各类压缩以减小代码体积为主，还有按需加载分散加载压力

JS代码压缩
CSS代码压缩
Html文件代码压缩
文件大小压缩
图片压缩
Tree Shaking
代码分离 —— 按需加载
内联 chunk

## 如何提高 webpack 的构建速度？

通过优化配置项

优化 loader 配置 —— 针对目标目录处理
合理使用 resolve.extensions
优化 resolve.modules —— 采用绝对路径
优化 resolve.alias —— 用以减少查找过程
使用 DLLPlugin 插件 —— 将不常用的代码抽离成一个共享库
使用 cache-loader —— 性能开销较大的 loader 以提升二次构建速度
terser 启动多线程
合理使用 sourceMap

# SSR

## SSR 解决了什么问题


# 参考

https://github.com/lgwebdream/FE-Interview/issues/25