---
title: 前端 | Vue2 打包流程
date: 2021-7-15
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->

# Vue 初始化结构

Vue 在我们通过 script 引入后，会挂载在 window下，本身作为一个 function 函数，需要我们在使用时用 new 创建一个实例，此外挂载了诸多属性在 Vue 上 

```js
{
  Vue: ƒ Vue(options)
      cid: 0
      compile: ƒ compileToFunctions( template, options, vm )
      component: ƒ ( id, definition )
      delete: ƒ del(target, key)
      directive: ƒ ( id, definition )
      extend: ƒ (extendOptions)
      filter: ƒ ( id, definition )
      mixin: ƒ (mixin)
      nextTick: ƒ nextTick(cb, ctx)
      observable: ƒ (obj)
      options: {components: {…}, directives: {…}, filters: {…}, _base: ƒ}
      set: ƒ (target, key, val)
      use: ƒ (plugin)
      util: {warn: ƒ, extend: ƒ, mergeOptions: ƒ, defineReactive: ƒ}
      version: "2.6.12"
      FunctionalRenderContext: ƒ FunctionalRenderContext( data, props, children, parent, Ctor )
      arguments: (...)
      caller: (...)
      config: (...)
      length: 1
      name: "Vue"
      prototype:
        {
          $delete: ƒ del(target, key)
          $destroy: ƒ ()
          $emit: ƒ (event)
          $forceUpdate: ƒ ()
          $mount: ƒ ( el, hydrating )
          $nextTick: ƒ (fn)
          $off: ƒ (event, fn)
          $on: ƒ (event, fn)
          $once: ƒ (event, fn)
          $set: ƒ (target, key, val)
          $watch: ƒ ( expOrFn, cb, options )
          __patch__: ƒ patch(oldVnode, vnode, hydrating, removeOnly)
          _b: ƒ bindObjectProps( data, tag, value, asProp, isSync )
          _d: ƒ bindDynamicKeys(baseObj, values)
          _e: ƒ (text)
          _f: ƒ resolveFilter(id)
          _g: ƒ bindObjectListeners(data, value)
          _i: ƒ looseIndexOf(arr, val)
          _init: ƒ (options)
          _k: ƒ checkKeyCodes( eventKeyCode, key, builtInKeyCode, eventKeyName, builtInKeyName )
          _l: ƒ renderList( val, render )
          _m: ƒ renderStatic( index, isInFor )
          _n: ƒ toNumber(val)
          _o: ƒ markOnce( tree, index, key )
          _p: ƒ prependModifier(value, symbol)
          _q: ƒ looseEqual(a, b)
          _render: ƒ ()
          _s: ƒ toString(val)
          _t: ƒ renderSlot( name, fallback, props, bindObject )
          _u: ƒ resolveScopedSlots( fns, // see flow/vnode res, // the following are added in 2.6 hasDynamicKeys, contentHashKey )
          _update: ƒ (vnode, hydrating)
          _v: ƒ createTextVNode(val)
          $data: (...)
          $isServer: (...)
          $props: (...)
          $ssrContext: (...)
          constructor: ƒ Vue(options)
          get $data: ƒ ()
          set $data: ƒ ()
          get $isServer: ƒ ()
          get $props: ƒ ()
          set $props: ƒ ()
          get $ssrContext: ƒ ()
          [[Prototype]]: Object
        }
      get config: ƒ ()
      set config: ƒ ()
      [[FunctionLocation]]: index.js:8
      [[Prototype]]: ƒ ()
      [[Scopes]]: Scopes[3]
      0: Closure {emptyObject: {…}, isUndef: ƒ, isDef: ƒ, isTrue: ƒ, isFalse: ƒ, …}
      1: Script {vm: undefined}
      2: Global {window: Window, self: Window, document: document, name: "", location: Location, …}
}
```

# Vue 初始化

Vue 在浏览器导入加载初始化后，就会在 window 上挂载 Vue 构造函数，用于之后开发者调用；或者对于 vue 脚手架，直接从安装 vue 模块，再在环境中引入。
这些 vue 源码均会被打包处理，形成初始的 Vue 函数

接下来，从源码看一下 Vue 是如何初始化的，又完成了哪些工作


## 入口

先找一下源码的入口，如之前所示，package.json 会指出运行的入口文件：

```
"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev --sourcemap"
```

可以看到运行了 scripts/config.js, 来到该文件最下边可以看到：

```js
if (process.env.TARGET) {
  module.exports = genConfig(process.env.TARGET)
} else {
  exports.getBuild = genConfig
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}
```

而此时命令中配置了 TARGET:web-full-dev , 那么就会执行：

```js
genConfig('web-full-dev')
```

再看 genConfig 的定义

```js
function genConfig (name) {
  const opts = builds[name]
  const config = {
    input: opts.entry,
    external: opts.external,
    plugins: [
      flow(),
      alias(Object.assign({}, aliases, opts.alias))
    ].concat(opts.plugins || []),
    output: {
      file: opts.dest,
      format: opts.format,
      banner: opts.banner,
      name: opts.moduleName || 'Vue'
    },
    onwarn: (msg, warn) => {
      if (!/Circular/.test(msg)) {
        warn(msg)
      }
    }
  }

  // built-in vars
  const vars = {
    __WEEX__: !!opts.weex,
    __WEEX_VERSION__: weexVersion,
    __VERSION__: version
  }
  // feature flags
  Object.keys(featureFlags).forEach(key => {
    vars[`process.env.${key}`] = featureFlags[key]
  })
  // build-specific env
  if (opts.env) {
    vars['process.env.NODE_ENV'] = JSON.stringify(opts.env)
  }
  config.plugins.push(replace(vars))

  if (opts.transpile !== false) {
    config.plugins.push(buble())
  }

  Object.defineProperty(config, '_name', {
    enumerable: false,
    value: name
  })

  return config
}
```

传入的参数 'web-full-dev' , 则会在 builds 中匹配，获得对应的信息：

```js
'web-full-dev': {
  entry: resolve('web/entry-runtime-with-compiler.js'),
  dest: resolve('dist/vue.js'),
  format: 'umd',
  env: 'development',
  alias: { he: './entity-decoder' },
  banner
},
```

获取对应命令的配置后，再进一步配置 config ，用于 rollup 打包。
在上边我们可以看到入口为 `web/entry-runtime-with-compiler.js`, 打包后放在 dist 下命名为 vue.js

## 流程

### platforms/web/entry-runtime-with-compiler.js

来到 `web/entry-runtime-with-compiler.js`, 我们可以发现，该文件是给引入的 Vue 再进一步添加方法，给原型上的 $mount 方法拦截加入编译处理(而运行时中定义的mount仅是作为挂载方法)，再给构造函数添加 compile 方法，最后导出为 Vue，也就是供我们调用的 Vue。

也可以在 mount 时看到，挂载方法中用到了编译方法 `compileToFunctions`, 但此时并没有被调用，而在给构造函数添加 compile 方法时，则需要加载 `compileToFunctions` 方法。

暂时先不用看编译模块，先以 Vue 构造函数为主，只需知道当前文件加载了这个东西，会挂载到 Vue 的 compile 属性上即可

而在文件顶端也可以看到，当前文件的 Vue 是从 `./runtime/index` 中引入

```js
// 拦截挂载方法重新定义
const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (el, hydrating) { 
  // ...
  return mount.call(this, el, hydrating)
}

Vue.compile = compileToFunctions
```

### platforms/web/runtime/index.js

可以看到当前文件的 Vue 又是从 `core/index` 引入。那么我们下一站也明确了。

再来看 `runtime/index` 做了哪些初始化工作：

安装一些方法，工具函数。主要完成的是在 Vue 原型上安装平台打包函数，和定义挂载的方法。
而此时的打包函数 `__patch__` 对应的函数也会从模块 `./patch` 中被加载出来（实现虚拟节点至dom节点的转换）; 而挂载的具体方法 `mountComponent` 未被调用则不会加载

至此，Vue 的编译，挂载，平台打包方法初始化差不多了，接下来进入 Vue 的核心

```js
// 安装平台特定的工具方法
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement

// 安装平台运行时指令和组件
extend(Vue.options.directives, platformDirectives)
extend(Vue.options.components, platformComponents)

// 安装平台打包函数
Vue.prototype.__patch__ = inBrowser ? patch : noop

Vue.prototype.$mount = function (el, hydrating) { 
  // ...
  return mountComponent(this, el, hydrating)
}
```

### core/index.js

core 内是 Vue 的核心代码，实现了 Vue 的响应式，虚拟节点，组件化，全局 api ，实例生命周期，事件等功能。

core 的 index.js 首先给 Vue 注入全局 api，接着在 Vue 原型上限定了三个属性 `$isServer`、`$ssrContext`、`FunctionalRenderContext`，最后给 Vue 添加版本信息


```js
// core/index.js

initGlobalAPI(Vue)

Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

Vue.version = '__VERSION__'
```

注入全局 api 过程中，直接给 Vue 添加属性 util、set、delete、nextTick、observable；其中 set、delete、nextTick 三者也会被添加在 Vue 原型上，前加 $

接着创建 options 属性，再在 options 属性上添加了 'component', 'directive', 'filter' 三个属性，还将 Vue 的构造函数赋值给 options 的 _base 属性；

再扩展了 options.components，继续完成 use、mixin、extend、 AssetRegisters 的初始化

```js
// initGlobalAPI(Vue)

Vue.util = {
  warn,
  extend,
  mergeOptions,
  defineReactive
}

Vue.set = set
Vue.delete = del
Vue.nextTick = nextTick

// 2.6 explicit observable API
Vue.observable = <T>(obj: T): T => {
  observe(obj)
  return obj
}

Vue.options = Object.create(null)
Vue.options._base = Vue

extend(Vue.options.components, builtInComponents)

initUse(Vue)
initMixin(Vue)
initExtend(Vue)
initAssetRegisters(Vue)
```

### core/instance/index.js

接着来到实例 Vue，此时才是明确定义了 Vue 这个构造函数，当我们在外界传入配置，便能 new 一个 Vue 实例出来。

定义 Vue 函数之后，接着完成其他功能的 mixin

```js
// core/instance/index.js

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)
```

initMixin 给 Vue 的原型上添加 _init 方法，也就是 function Vue() 中执行的初始化实例的方法。

```js
Vue.prototype._init = function (options?: Object) {}
```

stateMixin 限定了 Vue 原型上 $data 和 $props 属性，还给原型上添加了 $set、$delete、$watch 方法

```js
// stateMixin
Object.defineProperty(Vue.prototype, '$data', dataDef)
Object.defineProperty(Vue.prototype, '$props', propsDef)

Vue.prototype.$set = set
Vue.prototype.$delete = del

Vue.prototype.$watch = function () {}
```

eventsMixin 给原型上添加 $on、$once、$off、$emit 方法

```js
Vue.prototype.$on = function () {}
Vue.prototype.$once = function () {}
Vue.prototype.$off = function () {}
Vue.prototype.$emit = function () {}
```

lifecycleMixin 给原型上添加了 _update、$forceUpdate、$destroy 方法，其中 _update 中会调用 `__patch__` 方法将 虚拟节点转换为 dom 节点

```js
Vue.prototype._update = function () {}
Vue.prototype.$forceUpdate = function () {}
Vue.prototype.$destroy = function () {}
```

renderMixin 给 Vue 原型上添加 $nextTick、_render 两个方法，此外还在安装了 render 助手，也是在 render 过程中使用到的一些方法或工具。

```js
installRenderHelpers(Vue.prototype)

Vue.prototype.$nextTick = function () {}
Vue.prototype._render = function () {}


// installRenderHelpers 给 Vue 原型安装诸下方法
export function installRenderHelpers (target: any) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
  target._d = bindDynamicKeys
  target._p = prependModifier
}
```

至此，Vue 的打包过程就走完了，实际上就是定义了 Vue 函数的基础上，给 Vue 本身添加一系列属性方法，给 Vue 原型上添加一系列属性方法。最终导出的 Vue 也就是在 [Vue 初始化结构](#Vue 初始化结构) 所展示的那样。完成之后，等待我们调用初始化具体的实例。

