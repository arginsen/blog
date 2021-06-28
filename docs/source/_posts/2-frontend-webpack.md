---
title: 前端 | webpack 构建工具
date: 2020-6-21
tags: 
- webpack
- Frontend
categories: notes
photos:
    - /blog/img/webpack.jpg
---

<br>
<!--more-->

# 概述

> webpack 会把我们项目中使用到的多个代码模块（可以是不同文件类型），打包构建成项目运行仅需要的几个静态文件。webpack 有着十分丰富的配置项，提供了十分强大的扩展能力，可以在打包构建的过程中做很多事情。
> 

![](https://www.javazhiyin.com/wp-content/uploads/2019/11/java2-1572920946.jpg)

- webpack 与 grunt、gulp 的不同：
现在 webpack 相对来说比较主流，不过一些轻量化的任务还是会用 gulp 来处理，比如单独打包 CSS 文件。
**grunt** 和 **gulp** 是基于**任务和流**（Task、Stream）的一系列**链式操作**，更新流上的数据， 整条链式操作构成了一个任务，多个任务就构成了整个web的构建流程。
**webpack** 是基于**入口**的。webpack 会自动地**递归解析**入口所需要加载的所有资源文件，然后用不同的 Loader 来处理不同的文件，用 Plugin 来扩展webpack功能

- webpack 与 rollup、parcel 的不同：
三者都是基于入口的打包工具
**webpack** 适合大型复杂的前端站点构建；
**rollup** 适合基础库的打包，比如 vue，react；
**parcel** 适用于简单的实验室项目，但是打包出错很难调试。

# 安装

```bash
# 安装 node 环境
# 在 https://nodejs.org 下载安装

# 安装后查看 npm 版本
npm -v

# 安装后查看 node 版本
node -v

# 安装 webpack 至全局
npm i webpack -g

# 安装 webpack 至项目 确保版本
# 在项目目录初始化
npm init -y
# 安装 webpack 和 webpack-cli
npm i webpack webpack-cli -D


# 执行 1
# 项目初始化后生成 package.json，存放各依赖的信息
"devDependencies": {
    "webpack": "^3.5.6"
}

# 此时就可以在项目内运行 webpack
node_modules/.bin/webpack 入口.js 出口.js

# 当然我们也可以在 package.json 设置 webpack 的简化命令
"scripts": {
    "pack": "node_modules/.bin/webpack 入口.js 出口.js",
}

# 通过自定义命令运行 webpack 打包
npm run pack


# 执行 2
# webpack 的配置项非常多，因此可以集中用一个配置文件来设置
# 在项目目录创建 webpack.config.js
const path = require('path');

module.exports = {
  entry: './a.js',    # 入口
  output: {       # 出口
    filename: 'pack.js',   # 文件名
    path: path.resolve(__dirname, 'dist')   # 存放目录
  }
};

# 此时 config 指定了入口和出口，因此 pack 命令只用写到执行 webpack 即可
"scripts": {
    "pack": "node_modules/.bin/webpack",
}

# 进行打包
npm run pack

# 设定当文件发生变化时开始打包
npx webpack -w
```

# 核心功能

## entry 和 output

### entry

> 指定 webpack 由哪个模块作为项目构建的开始。通过配置 entry 属性，指定一个或多个起点，默认值 ./src

```js
module.exports = {
  entry: './path/leo/file.js'  // 无文件对象名称默认使用 main，之后输出 main.js
};

// 文件路径我们也可以传入一个数组，多文件一起注入
module.exports = {
  entry: {
    main: ['./path/leo/file.js', './path/leo/index.js', './path/leo/server.js']
  }
};


// 多文件入口
// 多个文件完全分离，互相独立
module.exports = {
  entry: {
    app: './src/app.js',    // 对每个文件进行单独的打包，后输出
    vendors: './src/vendors.js'
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].bundle.js'
  }
};
// 对于上边多文件打包，则会分别生成 app.bundle.js、vendors.bundle.js
```

### output

> 指定 webpack 最终输出的文件输出位置和文件名等信息。通过配置 output 属性，指定输出位置和文件名，默认输出位置为 ./dist

两个属性：
- path ：输出的目录绝对路径；
- filename ：输出的文件名称；

```js
const path = require('path');
module.exports = {
  entry: './path/leo/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].bundle.js'    // 使用占位符来为每个文件命名，保证名称唯一
  }
};


// 使用 CDN 和资源 hash
// 如果编译时不知道最终文件的 publicPath ，可以留空，并在入口文件中动态设置。
// 或者在入口起点设置 __webpack_public_path__ 来忽略它。
output: {
  path: "/home/proj/cdn/assets/[hash]",
  publicPath: "http://cdn.example.com/assets/[hash]/"
}
```

### 结合实现对多 js 文件打包

```js
// 建立 index.html signup.html 放进 /page
// 建立 base.js home.js signup.js 放进 /js

/* 目录结构为：
- node_modules
- js
  - base.js
  - home.js
  - signup.js
- page
  - index.html
  - signup.html
- package.json
- package-lock.json
- webpack.config.js
*/


// webpack.config.js
module.exports = {
    entry: {
        home: './js/home.js',
        signup: './js/signup.js',
    },
    output: {
        filename: '[name].bundle.js',   // [name] 动态获取 entry 键名
        path: __dirname + '/dist',
    }
}

// base.js
// ... 整个网站的逻辑
var open = true   // 对是否为内测用户作为判断

export { open }

// home.js
import { open } from './base'

if (open) {
    document.body.innerHTML =
        `<a href="signup.html">注册</a>`
}

// signup.js
import { open } from './base'

if (open) {
    document.body.innerHTML =
        `<h1">欢迎入坑</h1>`
} else {
    document.body.innerHTML =
        `<h1">暂不开放注册</h1>`
}

// 在 index.html 引入 js
<script src="../dist/home.bundle.js"></script>

// 在 signup.html 引入 js
<script src="../dist/signup.bundle.js"></script>


/* 执行打包 npm run back，目录结构改变为
- dist
  - home.bundle.js
  - signup.bundle.js
- node_modules
- js
  - base.js
  - home.js
  - signup.js
- page
  - index.html
  - signup.html
- package.json
- package-lock.json
- webpack.config.js
*/
```


## loader


> 让 webpack 能够处理非 JS 文件，在 import 或 “加载”模块时预处理文件。
> 编写 Loader 时要遵循单一原则，每个Loader 只做一种"转义"工作。 每个Loader的拿到的是源文件内容（source），可以通过返回值的方式将处理后的内容输出，也可以调用 this.callback() 方法，将内容返回给 webpack。 还可以通过 this.async() 生成一个 callback 函数，再用这个 callback 将处理后的内容输出出去。


```js
module: {
  // ...
  rules: [
    {
      test: /.jsx?/, // 匹配文件路径的正则表达式，通常我们都是匹配文件类型后缀
      include: [
        path.resolve(__dirname, 'src') // 指定哪些路径下的文件需要经过 loader 处理
      ],
      use: 'babel-loader', // 指定使用的 loader
    },
  ],
}
```


### 属性

- test 属性，用来标识出应该被对应的 loader 进行转换的某个或多个文件；
- use 属性，表示转换时要用哪个 loader；

### 特性

- loader 支持链式传递。能够对资源使用流水线(pipeline)。一组链式的 loader 将按照相反的顺序执行。loader 链中的第一个 loader 返回值给下一个 loader。在最后一个 loader，返回 webpack 所预期的 JavaScript。
- loader 可以是同步的，也可以是异步的。
- loader 运行在 Node.js 中，并且能够执行任何可能的操作。
- loader 接收查询参数。用于对 loader 传递配置。
- loader 也能够使用 options 对象进行配置。
- 除了使用 package.json 常见的 main 属性，还可以将普通的 npm 模块导出为 - loader，做法是在 package.json 里定义一个 loader 字段。
- 插件(plugin)可以为 loader 带来更多特性。
- loader 能够产生额外的任意文件。

### 常用的 loader

- babel-loader：把 es6 转成 es5；
- css-loader：加载 css，支持模块化，压缩，文件导入等特性；
- style-loader：把 css 代码注入到 js 中，通过 dom 操作去加载 css；
- eslint-loader：通过 Eslint 检查 js 代码；
- image-loader：加载并且压缩图片；
- file-loader：文件输出到一个文件夹中，在代码中通过相对 url 去引用输出的文件；
- url-loader：和 file-loader 类似，文件很小的时候可以 base64 方式吧文件内容注入到代码中。
- source-map-loader：加载额外的 source map 文件，方便调试




### 打包 css

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
     {
       test: /.css$/,
       use: [
         'style-loader',
         'css-loader'
       ]
     }
   ]
 }
};

// 应用上边直接用 js 引入 css，在浏览器中审查元素不好定位 css 的原始文件
// 这时就需要 sourceMap 使源文件的地址映射出来
// 本来是通过 js 给 head 标签添加 style 标签，应用 sourceMap 后会添加 link 标签
module.exports = {
  mode   : 'development',
  entry  : './index.js',
  output : {
    filename : 'bundle.js',
    path     : __dirname + '/dist',
  },
  module : {
    rules : [
      {
        test : /\.css$/i,
        use  : [
          {
            loader : 'style-loader', // 用<style>元素插入到<head>中
          },
          {
            loader  : 'css-loader', // 解析CSS语法
            options : {
              sourceMap : true, // 源码映射
            },
          },
        ],
      },
    ],
  },
};

// 或者使用 file-loader 将 css 文件通过 link 引入
module.exports = {
  mode   : 'development',
  entry  : './index.js',
  output : {
    filename : 'bundle.js',
    path     : __dirname + '/dist',
  },
  module : {
    rules : [
      {
        test : /\.css$/i,
        use  : [
          {
            loader : 'style-loader/url',  // 加入 url 参数，使
          },
          {
            loader  : 'file-loader', // 解析文件
            options : {
              publicPath : './dist',
              name       : '[name].bundle.css',  // 如不命名默认用 [hash].css
            },
          },
        ],
      },
    ],
  },
};
```

[style-loader/url](https://www.webpackjs.com/loaders/style-loader/#url)

## plugin

> 在 webpack 的构建流程中，plugin 用于处理更多其他的一些构建任务。可以这么理解，模块代码转换的工作由 loader 来处理，除此之外的其他任何工作都可以交由 plugin 来完成。
> webpack 在运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。

```js
const UglifyPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
  plugins: [
    new UglifyPlugin()
  ],
}
```

## loader 和 plugin 的不同

- 作用不同：
（1）loader 让 webpack 有加载和解析非 js 的能力；
（2）plugin 可以扩展 webpack 功能，在 webpack 运行周期中会广播很多事件，Plugin 可以监听一些事件，通过 webpack 的 api 改变结果。
- 用法不同：
（1）loader 在 module.rule （**rules**）中配置。类型为**数组**，每一项都是 Object，其中包含 test 与 use；
（2）plugin 是（**plugins**）单独配置的，类型为**数组**，每一项都是 **plugin 实例**，参数通过构造函数传入。

# webpack 的构建流程

- Webpack 的运行流程是一个**串行**的过程，从启动到结束会依次执行以下流程：
（1）**初始化参数**：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
（2）**开始编译**：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
（3）**确定入口**：根据配置中的 entry 找出所有的入口文件；
（4）**编译模块**：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
（5）**完成模块编译**：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
（6）**输出资源**：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
（7）**输出完成**：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。


# webpack 优化前端性能

1. 压缩代码
    - 删除多余的代码、注释、简化代码的写法等等方式。
    - 利用 webpack 的 UglifyJsPlugin 和 ParallelUglifyPlugin 来压缩 JS 文件。
    - 利用 cssnano（css-loader?minimize）来压缩 css。
    - 使用 webpack4，打包项目使用 production 模式，会自动开启代码压缩。
2. 利用CDN加速
    - 在构建过程中，将引用的静态资源路径修改为 CDN 上对应的路径。
    - 可以利用 webpack 对于 output 参数和各 loader 的 publicPath 参数来修改资源路径。
3. 删除死代码（Tree Shaking）
    - 将代码中永远不会走到的片段删除掉。可以通过在启动webpack时追加参数 --optimize-minimize 来实现或者使用es6模块开启删除死代码。
4. 优化图片，对于小图可以使用 base64 的方式写入文件中。
5. 按照路由拆分代码，实现按需加载，提取公共代码。
6. 给打包出来的文件名添加哈希，实现浏览器缓存文件。


# 参考资料

[webpack 中文文档](https://www.webpackjs.com/concepts/)
[表严肃 - Webpack精讲](https://biaoyansu.com/21.x)
[思语 - Webpack的基本使用](https://www.javazhiyin.com/50321.html)
[思语 - Webpack的运行机制](https://www.javazhiyin.com/50313.html)
[Webpack 核心](https://github.com/pingan8787/Leo-JavaScript/blob/master/Cute-Webpack/guide/README.md)
[前端自习课 - Webpack4 入门手册（上）](https://mp.weixin.qq.com/s/CnQs8dLS0M4dvDFTksh57g)
[前端自习课 - Webpack4 入门手册（下）](https://mp.weixin.qq.com/s/sYFqSc_Xx2hwrrnyWwiCdA)
[webpack 常见面试题](https://www.yuque.com/wangpingan/cute-frontend/lptos6)
