---
title: 前端 | vue3 源码学习 - 初始化
date: 2021-10-10 18:45:00
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->

# 准备

vue3 安装依赖会走 preinstall，进入到预检，会要求使用 pnmp 包来进行安装，这里使用 yarn 忽略脚本安装。

安装过程会在命令行进行交互，选取安装的 @vue/reactivity , @vue/runtime-core , @vue/runtime-dom 的版本，选择最新版安装即可

```
yarn --ignore-scripts
```

可以在 package.json 看到，scripts 内的 dev 运行的是路径 scripts/dev.js

dev.js 内可以看到实现的是使用 rollup 打包，使用 minimist 库来收集命令行传来的参数，那么我们在此处是需要调试代码就需要开启 sourcemap，于是在 package.json 的 dev 指令追加 `--sourcemap` 参数

```js
// scripts/dev.js
const execa = require('execa')
const { fuzzyMatchTarget } = require('./utils')
const args = require('minimist')(process.argv.slice(2))
const target = args._.length ? fuzzyMatchTarget(args._)[0] : 'vue'
const formats = args.formats || args.f
const sourceMap = args.sourcemap || args.s
const commit = execa.sync('git', ['rev-parse', 'HEAD']).stdout.slice(0, 7)

execa(
  'rollup',
  [
    '-wc',
    '--environment',
    [
      `COMMIT:${commit}`,
      `TARGET:${target}`,
      `FORMATS:${formats || 'global'}`,
      sourceMap ? `SOURCE_MAP:true` : ``
    ]
      .filter(Boolean)
      .join(',')
  ],
  {
    stdio: 'inherit'
  }
)
```

```json
{
  "private": true,
  "version": "3.2.21",
  "scripts": {
    "dev": "node scripts/dev.js --sourcemap",
    // ...
  }
}
```

接下来进入开发模式, 开始用 rollup 打包

```
yarn run dev

rollup v2.38.5
bundles E:\vue-next\packages\vue\src\index.ts → packages\vue\dist\vue.global.js...
created packages\vue\dist\vue.global.js in 18.3s
```

再来分析下打包是接着怎么往下走的。可以在上边看到 dev.js 里获取的 target，也就是 rollup 要打包的目标，若是命令行用户没有指定目标，那么默认为 vue

再来看 rollup.config.js , 其下获取当下目录下的 package 路径，再在 package 中读取 rollup 注入环境变量的 TARGET，也就是 vue , 获取 vue 目录下的 package.json 并读取，得到 vue 的构建配置，以及命名，默认为 vue

接着就进入输出配置的读取，之前在 dev.js 中我们对打包的版式并未配置，也可以在配置中设置版式，默认为 global，也可以出运行时版本的，那样就少了编译环节。那么我们在此处获取的格式就是 global

最后开始构建，按照 vue/package.json 中的构建配置，从入口 vue/src/index.ts 开始，输出为 vue/dist/vue.global.js , 这个文件即是我们调试所要引用的 vue 文件

```js
// rollup.config.js

const packagesDir = path.resolve(__dirname, 'packages')
const packageDir = path.resolve(packagesDir, process.env.TARGET)
const resolve = p => path.resolve(packageDir, p)
const pkg = require(resolve(`package.json`))
const packageOptions = pkg.buildOptions || {}
const name = packageOptions.filename || path.basename(packageDir)

const outputConfigs = {
  'esm-bundler': {
    file: resolve(`dist/${name}.esm-bundler.js`),
    format: `es`
  },
  'esm-browser': {
    file: resolve(`dist/${name}.esm-browser.js`),
    format: `es`
  },
  cjs: {
    file: resolve(`dist/${name}.cjs.js`),
    format: `cjs`
  },
  global: {
    file: resolve(`dist/${name}.global.js`),
    format: `iife`
  },
  // runtime-only builds, for main "vue" package only
  'esm-bundler-runtime': {
    file: resolve(`dist/${name}.runtime.esm-bundler.js`),
    format: `es`
  },
  'esm-browser-runtime': {
    file: resolve(`dist/${name}.runtime.esm-browser.js`),
    format: 'es'
  },
  'global-runtime': {
    file: resolve(`dist/${name}.runtime.global.js`),
    format: 'iife'
  }
}

const defaultFormats = ['esm-bundler', 'cjs']
const inlineFormats = process.env.FORMATS && process.env.FORMATS.split(',')
const packageFormats = inlineFormats || packageOptions.formats || defaultFormats
const packageConfigs = process.env.PROD_ONLY
```

# 调试

在 vscode 创建一个调试环境 , 配置 launch.json

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "launch Chrome",
      "request": "launch",
      "type": "chrome",
      "file": "${workspaceFolder}/packages/vue/dist/index.html"
    }
  ]
}
```

在该目录下创建 index.html 和 index.js , 就可以开始调试了

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div id="app">
      {{ state.foo }}
    </div>
    <script src="../dist/vue.global.js"></script>
    <script src="practice.js"></script>
</body>
</html>
```

```js
const { createApp, reactive, ref } = Vue

const app = createApp({
  setup() {
    let state = reactive({
      foo: 'Reactive'
    })

    return {
      state
    }
  }
})

app.mount('#app')
```

## createApp

vue3 将创建 vue 实例的函数换成了 createApp , 首先就来到 runtime-dom 下的包里，此外也能看到该层还暴露出去了 render 函数

可以看到首先创建 app , 是调用 ensureRenderer 方法, 确保是否存在有 renderer , 若无则调用 createRenderer 方法创建一个新的 renderer , 而该方法会返回一个包含 render, createApp 方法的对象 , 这也就解释了 ensureRenderer() 执行后接着调用 createApp() , 而 createApp 方法指向的实际是 

```ts
import { createRenderer } from '@vue/runtime-core'

let renderer: Renderer<Element | ShadowRoot> | HydrationRenderer

// 定义 ensureRenderer
function ensureRenderer() {
  return (
    renderer ||
    (renderer = createRenderer<Node, Element | ShadowRoot>(rendererOptions))
  )
}

// 在运行时暴露 render 方法
export const render = ((...args) => {
  ensureRenderer().render(...args)
}) as RootRenderFunction<Element | ShadowRoot>

// 定义 createApp
export const createApp = ((...args) => {
  const app = ensureRenderer().createApp(...args)

  if (__DEV__) {
    injectNativeTagCheck(app)
    injectCompilerOptionsCheck(app)
  }

  const { mount } = app
  app.mount = (containerOrSelector: Element | ShadowRoot | string): any => {
    const container = normalizeContainer(containerOrSelector)
    if (!container) return

    const component = app._component
    if (!isFunction(component) && !component.render && !component.template) {
      // __UNSAFE__
      // Reason: potential execution of JS expressions in in-DOM template.
      // The user must make sure the in-DOM template is trusted. If it's
      // rendered by the server, the template should not contain any user data.
      component.template = container.innerHTML
      // 2.x compat check
      if (__COMPAT__ && __DEV__) {
        for (let i = 0; i < container.attributes.length; i++) {
          const attr = container.attributes[i]
          if (attr.name !== 'v-cloak' && /^(v-|:|@)/.test(attr.name)) {
            compatUtils.warnDeprecation(
              DeprecationTypes.GLOBAL_MOUNT_CONTAINER,
              null
            )
            break
          }
        }
      }
    }

    // clear content before mounting
    container.innerHTML = ''
    const proxy = mount(container, false, container instanceof SVGElement)
    if (container instanceof Element) {
      container.removeAttribute('v-cloak')
      container.setAttribute('data-v-app', '')
    }
    return proxy
  }

  return app
}) as CreateAppFunction<Element>
```