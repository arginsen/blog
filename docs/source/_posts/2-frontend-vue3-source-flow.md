---
title: 前端 | vue3 源码学习 - 流程
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

拉取 vue-next 源码

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

再来看 rollup.config.js , 其获取当下目录下的 package 路径，再在 package 中读取 rollup 注入环境变量的 TARGET，也就是 vue , 获取 vue 目录下的 package.json 并读取，得到 vue 的构建配置，以及命名，默认为 vue

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

# 调试配置

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

# 源码流程

# createApp

vue3 将创建 vue 实例的函数换成了 createApp , 首先就来到 runtime-dom 下的包里，此外也能看到该层还暴露出去了 render 函数

可以看到首先创建 app , 是调用 ensureRenderer 方法, 确保是否存在有 renderer , 若无则调用 createRenderer 方法创建一个新的 renderer , 而该方法会返回一个包含 render, createApp 方法的对象, 这也就解释了 `ensureRenderer()` 执行后接着调用 createApp() , createApp 方法指向的实际是 createAppAPI, 最后是调用该方法生成 app , 接着再进行拦截 mount;

以上就外层代码创建 app 走完了, 接下来会进行编译, 在 createApp 时赋予了 mount 方法, 并在初始化 app 后对 mount 方法进行了拦截处理

首先得到根节点, 得到当前的组件对象, 看是否是函数, 是否存在 render, template, 若均为则对属性进行拦截处理, 接着清空节点, 执行 mount

大致与 vue2 一样的编译生成代码, 再生成虚拟节点, 最后转为真实 dom 渲染上去的流程

```ts
// packages\runtime-dom\src\index.ts
import { createRenderer } from '@vue/runtime-core'

const rendererOptions = extend({ patchProp }, nodeOps)
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

  // 获取初始化 app 在 createAppAPI 时定义的 mount 方法
  const { mount } = app
  // 拦截 mount 进行代理
  app.mount = (containerOrSelector: Element | ShadowRoot | string): any => {
    // 得到根节点
    const container = normalizeContainer(containerOrSelector)
    if (!container) return

    // 创建 app 对象时令 _component 等于根组件 rootComponent
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

## createRenderer

createRenderer 函数来到运行时的核心, 该方法接收 render 的配置, RendererOptions 内部整合了各类打包渲染函数, 从类型来看包含有 Node, Element

接着来到 baseCreateRenderer 函数, 这是一个体量巨大的函数, 里边定义了许多方法, 如下所列, 最后用 internals 对象封装了一些方法的简称, 最终返回一个对象, 包含 render, hydrate, createApp;

到此就结束了, 至于以上的一堆方法, 等调用时我们自会知道, 先接着往下看 `createAppAPI(render, hydrate)` 的执行, 因为上一步执行到 `createApp(...args)`

```js
patch, processText, processCommentNode, mountStaticNode, patchStaticNode,
moveStaticNode, removeStaticNode, processElement, mountElement, setScopeId,
mountChildren, patchElement, patchBlockChildren, patchProps,
processFragment, processComponent, mountComponent, updateComponent,
setupRenderEffect, updateComponentPreRender, patchChildren,
patchKeyedChildren, move, mount, remove, removeFragment, unmountComponent,
unmountChildren, getNextHostNode, render

const internals: RendererInternals = {
  p: patch,
  um: unmount,
  m: move,
  r: remove,
  mt: mountComponent,
  mc: mountChildren,
  pc: patchChildren,
  pbc: patchBlockChildren,
  n: getNextHostNode,
  o: options
}
```

```ts
// packages\runtime-core\src\renderer.ts
export function createRenderer<
  HostNode = RendererNode,
  HostElement = RendererElement
>(options: RendererOptions<HostNode, HostElement>) {
  return baseCreateRenderer<HostNode, HostElement>(options)
}

function baseCreateRenderer(
  options: RendererOptions,
  createHydrationFns?: typeof createHydrationFunctions
): any {
  // compile-time feature flags check
  if (__ESM_BUNDLER__ && !__TEST__) {
    initFeatureFlags()
  }

  // 获取当前全局变量 window
  const target = getGlobalThis()
  target.__VUE__ = true
  if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
    setDevtoolsHook(target.__VUE_DEVTOOLS_GLOBAL_HOOK__, target)
  }

  // 从参数 render 的配置中提取下列方法
  const {
    insert: hostInsert,
    remove: hostRemove,
    patchProp: hostPatchProp,
    createElement: hostCreateElement,
    createText: hostCreateText,
    createComment: hostCreateComment,
    setText: hostSetText,
    setElementText: hostSetElementText,
    parentNode: hostParentNode,
    nextSibling: hostNextSibling,
    setScopeId: hostSetScopeId = NOOP,
    cloneNode: hostCloneNode,
    insertStaticContent: hostInsertStaticContent
  } = options

  // ...

  let hydrate: ReturnType<typeof createHydrationFunctions>[0] | undefined
  let hydrateNode: ReturnType<typeof createHydrationFunctions>[1] | undefined
  if (createHydrationFns) {
    ;[hydrate, hydrateNode] = createHydrationFns(
      internals as RendererInternals<Node, Element>
    )
  }

  return {
    render,
    hydrate,
    createApp: createAppAPI(render, hydrate)
  }
}
```

## createAppAPI

来到 apiCreateApp 文件, 可以看到 `createAppAPI(render, hydrate)` 刚好是我们上一步得到的, 返回实际的 createApp 函数, 在此接收我们从初始传入的参数 args, 也就是此时的 rootComponent 根组件;

接着开始创建 app, 可以看到 app 实际上也是一个塞满了方法的对象, 需要时调用其上方法, 这些也是改变全局 Vue 行为的 API, 如我们一般在项目中会书写:

```ts
const app = createApp(App)

// 注册各类插件
app.use(router).use(store).use(ElementPlus)

// 执行挂载方法
app.mount('#app')
```

```ts
// packages\runtime-core\src\apiCreateApp.ts
export function createAppAPI<HostElement>(
  render: RootRenderFunction,
  hydrate?: RootHydrateFunction
): CreateAppFunction<HostElement> {
  return function createApp(rootComponent, rootProps = null) {
    if (rootProps != null && !isObject(rootProps)) {
      __DEV__ && warn(`root props passed to app.mount() must be an object.`)
      rootProps = null
    }

    const context = createAppContext()
    // 缓存已安装的插件
    const installedPlugins = new Set()

    // 标记尚未挂载
    let isMounted = false

    // 将 app 同时存储在 context 下
    const app: App = (context.app = {
      _uid: uid++,
      _component: rootComponent as ConcreteComponent,
      _props: rootProps,
      _container: null,
      _context: context,
      _instance: null,

      version,

      get config() {
        return context.config
      },

      set config(v) {
        if (__DEV__) {
          warn(
            `app.config cannot be replaced. Modify individual options instead.`
          )
        }
      },

      use(plugin: Plugin, ...options: any[]) {
        if (installedPlugins.has(plugin)) {
          __DEV__ && warn(`Plugin has already been applied to target app.`)
        } else if (plugin && isFunction(plugin.install)) {
          installedPlugins.add(plugin)
          plugin.install(app, ...options)
        } else if (isFunction(plugin)) {
          installedPlugins.add(plugin)
          plugin(app, ...options)
        } else if (__DEV__) {
          warn(
            `A plugin must either be a function or an object with an "install" ` +
              `function.`
          )
        }
        return app
      },

      mixin(mixin: ComponentOptions) {
        if (__FEATURE_OPTIONS_API__) {
          if (!context.mixins.includes(mixin)) {
            context.mixins.push(mixin)
          } else if (__DEV__) {
            warn(
              'Mixin has already been applied to target app' +
                (mixin.name ? `: ${mixin.name}` : '')
            )
          }
        } else if (__DEV__) {
          warn('Mixins are only available in builds supporting Options API')
        }
        return app
      },

      component(name: string, component?: Component): any {
        if (__DEV__) {
          validateComponentName(name, context.config)
        }
        if (!component) {
          return context.components[name]
        }
        if (__DEV__ && context.components[name]) {
          warn(`Component "${name}" has already been registered in target app.`)
        }
        context.components[name] = component
        return app
      },

      directive(name: string, directive?: Directive) {
        if (__DEV__) {
          validateDirectiveName(name)
        }

        if (!directive) {
          return context.directives[name] as any
        }
        if (__DEV__ && context.directives[name]) {
          warn(`Directive "${name}" has already been registered in target app.`)
        }
        context.directives[name] = directive
        return app
      },

      mount(
        rootContainer: HostElement,
        isHydrate?: boolean,
        isSVG?: boolean
      ): any {
        if (!isMounted) {
          const vnode = createVNode(
            rootComponent as ConcreteComponent,
            rootProps
          )
          // store app context on the root VNode.
          // this will be set on the root instance on initial mount.
          vnode.appContext = context

          // HMR root reload
          if (__DEV__) {
            context.reload = () => {
              render(cloneVNode(vnode), rootContainer, isSVG)
            }
          }

          if (isHydrate && hydrate) {
            hydrate(vnode as VNode<Node, Element>, rootContainer as any)
          } else {
            render(vnode, rootContainer, isSVG)
          }
          isMounted = true
          app._container = rootContainer
          // for devtools and telemetry
          ;(rootContainer as any).__vue_app__ = app

          if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
            app._instance = vnode.component
            devtoolsInitApp(app, version)
          }

          return getExposeProxy(vnode.component!) || vnode.component!.proxy
        } else if (__DEV__) {
          warn(
            `App has already been mounted.\n` +
              `If you want to remount the same app, move your app creation logic ` +
              `into a factory function and create fresh app instances for each ` +
              `mount - e.g. \`const createMyApp = () => createApp(App)\``
          )
        }
      },

      unmount() {
        if (isMounted) {
          render(null, app._container)
          if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
            app._instance = null
            devtoolsUnmountApp(app)
          }
          delete app._container.__vue_app__
        } else if (__DEV__) {
          warn(`Cannot unmount an app that is not mounted.`)
        }
      },

      provide(key, value) {
        if (__DEV__ && (key as string | symbol) in context.provides) {
          warn(
            `App already provides property with key "${String(key)}". ` +
              `It will be overwritten with the new value.`
          )
        }
        // TypeScript doesn't allow symbols as index type
        // https://github.com/Microsoft/TypeScript/issues/24587
        context.provides[key as string] = value

        return app
      }
    })

        if (__COMPAT__) {
      installAppCompatProperties(app, context, render)
    }

    return app
  }
}

// 创建 app 的上下文对象
export function createAppContext(): AppContext {
  return {
    app: null as any,
    config: {
      isNativeTag: NO,
      performance: false,
      globalProperties: {},
      optionMergeStrategies: {},
      errorHandler: undefined,
      warnHandler: undefined,
      compilerOptions: {}
    },
    mixins: [],
    components: {},
    directives: {},
    provides: Object.create(null),
    optionsCache: new WeakMap(),
    propsCache: new WeakMap(),
    emitsCache: new WeakMap()
  }
}
```

# [mount]

首先进入 createAppApi 内定义 app 对象时传入的 mount 方法, 检查当前是否已经挂载, 若是初始挂载, 先传入根组件和根属性创建虚拟节点；

给根虚拟节点保存全局实例上下文; 开发环境下绑定根实例的重载的方法, 利用克隆节点再 render

再判断当前是否使 hydrate 状态, 如不是则进行 render

render 操作又会回到之前 baseCreateRenderer 阶段定义的 render 方法


```ts
// packages\runtime-core\src\apiCreateApp.ts
mount(
  rootContainer: HostElement,
  isHydrate?: boolean,
  isSVG?: boolean
): any {
  if (!isMounted) {
    const vnode = createVNode(
      rootComponent as ConcreteComponent,
      rootProps
    )
    // store app context on the root VNode.
    // this will be set on the root instance on initial mount.
    vnode.appContext = context

    // HMR root reload
    if (__DEV__) {
      context.reload = () => {
        render(cloneVNode(vnode), rootContainer, isSVG)
      }
    }

    if (isHydrate && hydrate) {
      hydrate(vnode as VNode<Node, Element>, rootContainer as any)
    } else {
      render(vnode, rootContainer, isSVG)
    }
    isMounted = true
    app._container = rootContainer
    // for devtools and telemetry
    ;(rootContainer as any).__vue_app__ = app

    if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
      app._instance = vnode.component
      devtoolsInitApp(app, version)
    }

    return getExposeProxy(vnode.component!) || vnode.component!.proxy
  } else if (__DEV__) {
    warn(
      `App has already been mounted.\n` +
        `If you want to remount the same app, move your app creation logic ` +
        `into a factory function and create fresh app instances for each ` +
        `mount - e.g. \`const createMyApp = () => createApp(App)\``
    )
  }
}
```

## createVNode

携带参数 根组件和根属性 创建虚拟节点, 如果为开发环境, 那么走 createVNodeWithArgsTransform 通道, 生产环境则走 `_createVNode`, 相比生产环境, 开发环境多了判断, 对当前参数进行转换, 无转换标记直接将参数参入 `_createVNode`

进入该函数后检查当前节点类型是否为 VNode 类型, 如果是则进入克隆, 再规范化其子节点

接着检查是否为 class 组件, 是否为异步或函数的组件形式

若存在 props, 则对响应式化的或已被代理的对象进行克隆以使其状态改变

接着进入到基础的虚拟节点创建函数 createBaseVNode, 除 _createVNode 函数返回调用此函数, 创建元素区块 createElementBlock 也会调用

最后返回 vnode

```ts
// packages\runtime-core\src\vnode.ts
// 创建虚拟节点, 依据环境检测匹配函数
export const createVNode = (
  __DEV__ ? createVNodeWithArgsTransform : _createVNode
) as typeof _createVNode

// vnodeArgsTransformer 定义, 默认为 undefined
let vnodeArgsTransformer:
  | ((
      args: Parameters<typeof _createVNode>,
      instance: ComponentInternalInstance | null
    ) => Parameters<typeof _createVNode>)
  | undefined

// 开发环境下创建虚拟节点函数参数转换
const createVNodeWithArgsTransform = (
  ...args: Parameters<typeof _createVNode>
): VNode => {
  return _createVNode(
    ...(vnodeArgsTransformer
      ? vnodeArgsTransformer(args, currentRenderingInstance)
      : args)
  )
}

// 创建虚拟节点
function _createVNode(
  type: VNodeTypes | ClassComponent | typeof NULL_DYNAMIC_COMPONENT,
  props: (Data & VNodeProps) | null = null,
  children: unknown = null,
  patchFlag: number = 0,
  dynamicProps: string[] | null = null,
  isBlockNode = false
): VNode {
  if (!type || type === NULL_DYNAMIC_COMPONENT) {
    if (__DEV__ && !type) {
      warn(`Invalid vnode type when creating vnode: ${type}.`)
    }
    type = Comment
  }

  if (isVNode(type)) {
    // createVNode receiving an existing vnode. This happens in cases like
    // <component :is="vnode"/>
    // #2078 make sure to merge refs during the clone instead of overwriting it
    const cloned = cloneVNode(type, props, true /* mergeRef: true */)
    if (children) {
      normalizeChildren(cloned, children)
    }
    return cloned
  }

  // class component normalization.
  if (isClassComponent(type)) {
    type = type.__vccOpts
  }

  // 2.x async/functional component compat
  if (__COMPAT__) {
    type = convertLegacyComponent(type, currentRenderingInstance)
  }

  // class & style normalization.
  if (props) {
    // for reactive or proxy objects, we need to clone it to enable mutation.
    props = guardReactiveProps(props)!
    let { class: klass, style } = props
    if (klass && !isString(klass)) {
      props.class = normalizeClass(klass)
    }
    if (isObject(style)) {
      // reactive state objects need to be cloned since they are likely to be
      // mutated
      if (isProxy(style) && !isArray(style)) {
        style = extend({}, style)
      }
      props.style = normalizeStyle(style)
    }
  }

  // encode the vnode type information into a bitmap
  const shapeFlag = isString(type)
    ? ShapeFlags.ELEMENT
    : __FEATURE_SUSPENSE__ && isSuspense(type)
    ? ShapeFlags.SUSPENSE
    : isTeleport(type)
    ? ShapeFlags.TELEPORT
    : isObject(type)
    ? ShapeFlags.STATEFUL_COMPONENT
    : isFunction(type)
    ? ShapeFlags.FUNCTIONAL_COMPONENT
    : 0

  if (__DEV__ && shapeFlag & ShapeFlags.STATEFUL_COMPONENT && isProxy(type)) {
    type = toRaw(type)
    warn(
      `Vue received a Component which was made a reactive object. This can ` +
        `lead to unnecessary performance overhead, and should be avoided by ` +
        `marking the component with \`markRaw\` or using \`shallowRef\` ` +
        `instead of \`ref\`.`,
      `\nComponent that was made reactive: `,
      type
    )
  }

  return createBaseVNode(
    type,
    props,
    children,
    patchFlag,
    dynamicProps,
    shapeFlag,
    isBlockNode,
    true
  )
}

// 基础创建虚拟节点函数
function createBaseVNode(
  type: VNodeTypes | ClassComponent | typeof NULL_DYNAMIC_COMPONENT,
  props: (Data & VNodeProps) | null = null,
  children: unknown = null,
  patchFlag = 0,
  dynamicProps: string[] | null = null,
  shapeFlag = type === Fragment ? 0 : ShapeFlags.ELEMENT,
  isBlockNode = false,
  needFullChildrenNormalization = false
) {
  // 定义 vnode 对象
  const vnode = {
    __v_isVNode: true,
    __v_skip: true,
    type,
    props,
    key: props && normalizeKey(props),
    ref: props && normalizeRef(props),
    scopeId: currentScopeId,
    slotScopeIds: null,
    children,
    component: null,
    suspense: null,
    ssContent: null,
    ssFallback: null,
    dirs: null,
    transition: null,
    el: null,
    anchor: null,
    target: null,
    targetAnchor: null,
    staticCount: 0,
    shapeFlag,
    patchFlag,
    dynamicProps,
    dynamicChildren: null,
    appContext: null
  } as VNode

  // 使子节点规范化
  if (needFullChildrenNormalization) {
    normalizeChildren(vnode, children)
    // normalize suspense children
    if (__FEATURE_SUSPENSE__ && shapeFlag & ShapeFlags.SUSPENSE) {
      ;(type as typeof SuspenseImpl).normalize(vnode)
    }
  } else if (children) {
    // compiled element vnode - if children is passed, only possible types are
    // string or Array.
    vnode.shapeFlag |= isString(children)
      ? ShapeFlags.TEXT_CHILDREN
      : ShapeFlags.ARRAY_CHILDREN
  }

  // validate key
  if (__DEV__ && vnode.key !== vnode.key) {
    warn(`VNode created with invalid key (NaN). VNode type:`, vnode.type)
  }

  // track vnode for block tree
  if (
    isBlockTreeEnabled > 0 &&
    // avoid a block node from tracking itself
    !isBlockNode &&
    // has current parent block
    currentBlock &&
    // presence of a patch flag indicates this node needs patching on updates.
    // component nodes also should always be patched, because even if the
    // component doesn't need to update, it needs to persist the instance on to
    // the next vnode so that it can be properly unmounted later.
    (vnode.patchFlag > 0 || shapeFlag & ShapeFlags.COMPONENT) &&
    // the EVENTS flag is only for hydration and if it is the only flag, the
    // vnode should not be considered dynamic due to handler caching.
    vnode.patchFlag !== PatchFlags.HYDRATE_EVENTS
  ) {
    currentBlock.push(vnode)
  }

  if (__COMPAT__) {
    convertLegacyVModelProps(vnode)
    convertLegacyRefInFor(vnode)
    defineLegacyVNodeProperties(vnode)
  }

  return vnode
}
```

## [render]

render 首先判断 vnode 是否为空, 如果根节点 dom 上存在 _vnode(vnode 父虚拟节点), 那么进行卸载

如果此时存在 vnode ,上一步我们生成好了 vnode, 那么这下就进入 patch 阶段, 传入父虚拟节点, 当前虚拟节点, 真实 dom

```ts
// packages\runtime-core\src\renderer.ts
const render: RootRenderFunction = (vnode, container, isSVG) => {
  if (vnode == null) {
    if (container._vnode) {
      unmount(container._vnode, null, null, true)
    }
  } else {
    patch(container._vnode || null, vnode, container, null, null, null, isSVG)
  }
  flushPostFlushCbs()
  container._vnode = vnode
}
```

## patch

patch 将传入的一老一新vnode进行比对, 如果一样直接返回, 如果 n1 存在且新旧类型不同, 那么就直接卸载掉旧树, 初始化则会直接跳过

提取新节点中的信息, 这里的 type 也就是我们在 createApp 时传入的第一个参数, 有文本节点的, 也有评论节点的, 也有组件, 碎片的, 这里我们传入的是根节点 rootComponent 对象

因此这里会进入默认的 type, 且我们这里的 shapeflag 存在且为组件 ShapeFlags.COMPONENT 也是为 true 的, 因此会走 processComponent 流程; 此外也有 ELEMENT TELEPORT SUSPENSE 流程

```ts
// packages\runtime-core\src\renderer.ts
const patch: PatchFn = (
  n1,
  n2,
  container,
  anchor = null,
  parentComponent = null,
  parentSuspense = null,
  isSVG = false,
  slotScopeIds = null,
  optimized = __DEV__ && isHmrUpdating ? false : !!n2.dynamicChildren
) => {
  if (n1 === n2) {
    return
  }

  // patching & not same type, unmount old tree
  if (n1 && !isSameVNodeType(n1, n2)) {
    anchor = getNextHostNode(n1)
    unmount(n1, parentComponent, parentSuspense, true)
    n1 = null
  }

  if (n2.patchFlag === PatchFlags.BAIL) {
    optimized = false
    n2.dynamicChildren = null
  }

  // 提取新节点的信息
  const { type, ref, shapeFlag } = n2
  // 按不同类型执行不同的进程
  switch (type) {
    case Text:
      processText(n1, n2, container, anchor)
      break
    case Comment:
      processCommentNode(n1, n2, container, anchor)
      break
    case Static:
      if (n1 == null) {
        mountStaticNode(n2, container, anchor, isSVG)
      } else if (__DEV__) {
        patchStaticNode(n1, n2, container, isSVG)
      }
      break
    case Fragment:
      processFragment(
        n1,
        n2,
        container,
        anchor,
        parentComponent,
        parentSuspense,
        isSVG,
        slotScopeIds,
        optimized
      )
      break
    default:
      if (shapeFlag & ShapeFlags.ELEMENT) {
        processElement(
          n1,
          n2,
          container,
          anchor,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized
        )
      } else if (shapeFlag & ShapeFlags.COMPONENT) {
        processComponent(
          n1,
          n2,
          container,
          anchor,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized
        )
      } else if (shapeFlag & ShapeFlags.TELEPORT) {
        ;(type as typeof TeleportImpl).process(
          n1 as TeleportVNode,
          n2 as TeleportVNode,
          container,
          anchor,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized,
          internals
        )
      } else if (__FEATURE_SUSPENSE__ && shapeFlag & ShapeFlags.SUSPENSE) {
        ;(type as typeof SuspenseImpl).process(
          n1,
          n2,
          container,
          anchor,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized,
          internals
        )
      } else if (__DEV__) {
        warn('Invalid VNode type:', type, `(${typeof type})`)
      }
  }

  // set ref
  if (ref != null && parentComponent) {
    setRef(ref, n1 && n1.ref, parentSuspense, n2 || n1, !n2)
  }
}
```

## processComponent

接收传递过来的新旧虚拟节点, 真实 dom 节点等;

判断旧节点是否不存在, 若存在则直接进行 updateComponent; 不存在则说明为初始化, 初始化接着判断 COMPONENT_KEPT_ALIVE 若组件没有开启 keep-alive, 则进入 mountComponent 环节

```ts
const processComponent = (
  n1: VNode | null,
  n2: VNode,
  container: RendererElement,
  anchor: RendererNode | null,
  parentComponent: ComponentInternalInstance | null,
  parentSuspense: SuspenseBoundary | null,
  isSVG: boolean,
  slotScopeIds: string[] | null,
  optimized: boolean
) => {
  n2.slotScopeIds = slotScopeIds
  if (n1 == null) {
    if (n2.shapeFlag & ShapeFlags.COMPONENT_KEPT_ALIVE) {
      ;(parentComponent!.ctx as KeepAliveContext).activate(
        n2,
        container,
        anchor,
        isSVG,
        optimized
      )
    } else {
      mountComponent(
        n2,
        container,
        anchor,
        parentComponent,
        parentSuspense,
        isSVG,
        optimized
      )
    }
  } else {
    updateComponent(n1, n2, optimized)
  }
}
```

## mountComponent

接收参数就直接接收新的虚拟节点, 真实 dom 根节点;

创建挂载的 vue 实例, 安装组件 setup, 安装 render 的 effect

```ts
// packages\runtime-core\src\renderer.ts
const mountComponent: MountComponentFn = (
  initialVNode,
  container,
  anchor,
  parentComponent,
  parentSuspense,
  isSVG,
  optimized
) => {
  // 2.x compat may pre-create the component instance before actually
  // mounting
  // compat 可以在实际操作之前预先创建组件实例
  const compatMountInstance =
    __COMPAT__ && initialVNode.isCompatRoot && initialVNode.component
  const instance: ComponentInternalInstance =
    compatMountInstance ||
    (initialVNode.component = createComponentInstance(
      initialVNode,
      parentComponent,
      parentSuspense
    ))

  if (__DEV__ && instance.type.__hmrId) {
    registerHMR(instance)
  }

  if (__DEV__) {
    pushWarningContext(initialVNode)
    startMeasure(instance, `mount`)
  }

  // inject renderer internals for keepAlive
  if (isKeepAlive(initialVNode)) {
    ;(instance.ctx as KeepAliveContext).renderer = internals
  }

  // resolve props and slots for setup context
  if (!(__COMPAT__ && compatMountInstance)) {
    if (__DEV__) {
      startMeasure(instance, `init`)
    }
    setupComponent(instance)
    if (__DEV__) {
      endMeasure(instance, `init`)
    }
  }

  // setup() is async. This component relies on async logic to be resolved
  // before proceeding
  if (__FEATURE_SUSPENSE__ && instance.asyncDep) {
    parentSuspense && parentSuspense.registerDep(instance, setupRenderEffect)

    // Give it a placeholder if this is not hydration
    // TODO handle self-defined fallback
    if (!initialVNode.el) {
      const placeholder = (instance.subTree = createVNode(Comment))
      processCommentNode(null, placeholder, container!, anchor)
    }
    return
  }

  setupRenderEffect(
    instance,
    initialVNode,
    container,
    anchor,
    parentSuspense,
    isSVG,
    optimized
  )

  if (__DEV__) {
    popWarningContext()
    endMeasure(instance, `mount`)
  }
}
```

### createComponentInstance

接收新的虚拟节点, 父级节点

获取虚拟节点的 type, 也就是根组件对象, 以及虚拟节点的 appContext

定义组件实例 instance , 同时给组件创建 effectScope, 定义在组件的 scope 属性上 ,在属性 ctx / root 保留实例对象

返回创建的实例, 此时的实例也就是在组件信息基础上, 转换为 vue 的组件实例

```ts
// packages\runtime-core\src\component.ts
export function createComponentInstance(
  vnode: VNode,
  parent: ComponentInternalInstance | null,
  suspense: SuspenseBoundary | null
) {
  const type = vnode.type as ConcreteComponent
  // inherit parent app context - or - if root, adopt from root vnode
  const appContext =
    (parent ? parent.appContext : vnode.appContext) || emptyAppContext

  const instance: ComponentInternalInstance = {
    uid: uid++,
    vnode,
    type,
    parent,
    appContext,
    root: null!, // to be immediately set
    next: null,
    subTree: null!, // will be set synchronously right after creation
    update: null!, // will be set synchronously right after creation
    scope: new EffectScope(true /* detached */),
    render: null,
    proxy: null,
    exposed: null,
    exposeProxy: null,
    withProxy: null,
    provides: parent ? parent.provides : Object.create(appContext.provides),
    accessCache: null!,
    renderCache: [],

    // local resovled assets
    components: null,
    directives: null,

    // resolved props and emits options
    propsOptions: normalizePropsOptions(type, appContext),
    emitsOptions: normalizeEmitsOptions(type, appContext),

    // emit
    emit: null!, // to be set immediately
    emitted: null,

    // props default value
    propsDefaults: EMPTY_OBJ,

    // inheritAttrs
    inheritAttrs: type.inheritAttrs,

    // state
    ctx: EMPTY_OBJ,
    data: EMPTY_OBJ,
    props: EMPTY_OBJ,
    attrs: EMPTY_OBJ,
    slots: EMPTY_OBJ,
    refs: EMPTY_OBJ,
    setupState: EMPTY_OBJ,
    setupContext: null,

    // suspense related
    suspense,
    suspenseId: suspense ? suspense.pendingId : 0,
    asyncDep: null,
    asyncResolved: false,

    // lifecycle hooks
    // not using enums here because it results in computed properties
    isMounted: false,
    isUnmounted: false,
    isDeactivated: false,
    bc: null,
    c: null,
    bm: null,
    m: null,
    bu: null,
    u: null,
    um: null,
    bum: null,
    da: null,
    a: null,
    rtg: null,
    rtc: null,
    ec: null,
    sp: null
  }
  // 给实例的 ctx 注入 _ 属性, 指向实例
  if (__DEV__) {
    instance.ctx = createDevRenderContext(instance)
  } else {
    instance.ctx = { _: instance }
  }
  // 保留实例的根
  instance.root = parent ? parent.root : instance
  // 实例的 emit 方法 this 指向 null
  instance.emit = emit.bind(null, instance)

  // apply custom element special handling
  if (vnode.ce) {
    vnode.ce(instance)
  }

  return instance
}
```

### setupComponent

初始化 props / slots

initProps 给实例 instance 添加 props / attrs 属性, 并做拦截

准备好的组件进行安装

```ts
export function setupComponent(
  instance: ComponentInternalInstance,
  isSSR = false
) {
  isInSSRComponentSetup = isSSR

  const { props, children } = instance.vnode
  const isStateful = isStatefulComponent(instance)
  initProps(instance, props, isStateful, isSSR)
  initSlots(instance, children)

  // 获得安装结果
  const setupResult = isStateful
    ? setupStatefulComponent(instance, isSSR)
    : undefined
  isInSSRComponentSetup = false
  return setupResult
}
```

### setupStatefulComponent

接收 instance 参数, 以及是否服务端渲染

从实例的 type 获取用户定义的根组件内容 Component, 如果是开发环境, 需要对组件进行检查, 查看其定义是否有效

接着从 Component 读取 setup, 如果用户侧用组合式 API , 那么就有用到 setup, 如果未定义, 那么就结束组件安装

如果写入了 setup 函数, 那么就开始安装, 先设定当前的实例, 全局变量 currentInstance 就等于当前活动的 instance, 接着进入 effectScope, 调用 on 方法, 将当前活跃的 effectScope 压入 effectScopeStack 栈中, 并标记 activeEffectScope 为 当前的 effectScope

pauseTracking 将 shouldTrack 压入 trackStack , 再将全局的 shouldTrack 变更为 false

接着处理 setup, 开发环境下得到浅层只读(shallow readonly)的 props 响应式代理, 将 props 作为参数, 传入我们定义的 setup 函数执行, 获取到 setupResult, 实际也就是我们在 setup() API 中设置的 return 的对象

因此也并不像 vue2 中, 首先初始化将所有数据响应式化, 进行编译 parse 代码成 AST 树, 再优化转成代码, 再由代码生成 vnode, 在此读取代码时捕获的变量触发响应式系统依赖收集

vue3 不是这样的, vue3 初始是先创建虚拟节点, 再进行 mount, 再进入render, 创建组件, 安装组件, 在此过程中再对代码中有书写到响应式的地方再进行响应式化, 或是定义了 computed, ref, watch, watchEffect 等, 有运行到再去操作, 两者的整个运行逻辑不同

```ts
// packages\runtime-core\src\component.ts
function setupStatefulComponent(
  instance: ComponentInternalInstance,
  isSSR: boolean
) {
  const Component = instance.type as ComponentOptions

  if (__DEV__) {
    if (Component.name) {
      validateComponentName(Component.name, instance.appContext.config)
    }
    if (Component.components) {
      const names = Object.keys(Component.components)
      for (let i = 0; i < names.length; i++) {
        validateComponentName(names[i], instance.appContext.config)
      }
    }
    if (Component.directives) {
      const names = Object.keys(Component.directives)
      for (let i = 0; i < names.length; i++) {
        validateDirectiveName(names[i])
      }
    }
    if (Component.compilerOptions && isRuntimeOnly()) {
      warn(
        `"compilerOptions" is only supported when using a build of Vue that ` +
          `includes the runtime compiler. Since you are using a runtime-only ` +
          `build, the options should be passed via your build tool config instead.`
      )
    }
  }
  // 0. create render proxy property access cache
  // 创建代理属性 accessCache 来访问缓存
  instance.accessCache = Object.create(null)
  // 1. create public instance / render proxy
  // also mark it raw so it's never observed
  // 创建实例的 proxy 属性,
  // 来标记一个并未被相应式化的原生对象
  instance.proxy = markRaw(new Proxy(instance.ctx, PublicInstanceProxyHandlers))
  if (__DEV__) {
    exposePropsOnRenderContext(instance)
  }
  // 2. call setup()
  const { setup } = Component
  if (setup) {
    const setupContext = (instance.setupContext =
      setup.length > 1 ? createSetupContext(instance) : null)

    setCurrentInstance(instance)
    pauseTracking()
    const setupResult = callWithErrorHandling(
      setup,
      instance,
      ErrorCodes.SETUP_FUNCTION,
      [__DEV__ ? shallowReadonly(instance.props) : instance.props, setupContext]
    )
    resetTracking()
    unsetCurrentInstance()

    if (isPromise(setupResult)) {
      setupResult.then(unsetCurrentInstance, unsetCurrentInstance)

      if (isSSR) {
        // return the promise so server-renderer can wait on it
        return setupResult
          .then((resolvedResult: unknown) => {
            handleSetupResult(instance, resolvedResult, isSSR)
          })
          .catch(e => {
            handleError(e, instance, ErrorCodes.SETUP_FUNCTION)
          })
      } else if (__FEATURE_SUSPENSE__) {
        // async setup returned Promise.
        // bail here and wait for re-entry.
        instance.asyncDep = setupResult
      } else if (__DEV__) {
        warn(
          `setup() returned a Promise, but the version of Vue you are using ` +
            `does not support it yet.`
        )
      }
    } else {
      handleSetupResult(instance, setupResult, isSSR)
    }
  } else {
    finishComponentSetup(instance, isSSR)
  }
}
```

#### setupResult

获取 setup() 运行后返回的结果

```ts
// packages\runtime-core\src\component.ts
const { setup } = Component
if (setup) {
  const setupResult = callWithErrorHandling(
    setup,
    instance,
    ErrorCodes.SETUP_FUNCTION,
    [__DEV__ ? shallowReadonly(instance.props) : instance.props, setupContext]
  )
}

// packages\runtime-core\src\errorHandling.ts
export function callWithErrorHandling(
  fn: Function,
  instance: ComponentInternalInstance | null,
  type: ErrorTypes,
  args?: unknown[]
) {
  let res
  try {
    res = args ? fn(...args) : fn()
  } catch (err) {
    handleError(err, instance, type)
  }
  return res
}
```

#### handleSetupResult

判断 setup 的结果是否为函数, 如果为函数则将结果赋值给实例 instance.render;

结果为对象时, 将 setupResult 用 proxyRefs 处理, 判断传入的结果是否时 reactive 对象, 若不是则进行 proxy 代理赋值给 instance.setupState

```ts
// packages\runtime-core\src\component.ts
export function handleSetupResult(
  instance: ComponentInternalInstance,
  setupResult: unknown,
  isSSR: boolean
) {
  if (isFunction(setupResult)) {
    // setup returned an inline render function
    if (__SSR__ && (instance.type as ComponentOptions).__ssrInlineRender) {
      // when the function's name is `ssrRender` (compiled by SFC inline mode),
      // set it as ssrRender instead.
      instance.ssrRender = setupResult
    } else {
      instance.render = setupResult as InternalRenderFunction
    }
  } else if (isObject(setupResult)) {
    if (__DEV__ && isVNode(setupResult)) {
      warn(
        `setup() should not return VNodes directly - ` +
          `return a render function instead.`
      )
    }
    // setup returned bindings.
    // assuming a render function compiled from template is present.
    if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
      instance.devtoolsRawSetupState = setupResult
    }
    instance.setupState = proxyRefs(setupResult)
    if (__DEV__) {
      exposeSetupStateOnRenderContext(instance)
    }
  } else if (__DEV__ && setupResult !== undefined) {
    warn(
      `setup() should return an object. Received: ${
        setupResult === null ? 'null' : typeof setupResult
      }`
    )
  }
  finishComponentSetup(instance, isSSR)
}

// packages\reactivity\src\ref.ts
export function proxyRefs<T extends object>(
  objectWithRefs: T
): ShallowUnwrapRef<T> {
  return isReactive(objectWithRefs)
    ? objectWithRefs
    : new Proxy(objectWithRefs, shallowUnwrapHandlers)
}
```

#### finishComponentSetup

期间会进行编译 compile

如果 instance.render 不存在, 则开始准备编译的 template, template 存在就调用 compile 进行处理, 完后赋值给 Component.render, 再赋值给 instance.render , 得到的即是整个组件的渲染语句

接着会根据是否有 vue2 的 options api 书写来按 vue2 的步骤来执行; 设置全局实例, 开始 track, 接着应用 options, 在此期间会先执行 vue2 的 beforeCreate 钩子函数, 接着判断是否有 inject, methods, data, computed, watch, provide, 完成数据初始化, 接着再执行 created 钩子函数; 再注册其他的钩子函数(接口至vue3这些钩子函数的地方), 而对于 vue3 组合式 api 的写法, 是没有 beforeCreate 和 created 钩子函数的, 而且此处的钩子函数均是从 options 获取, 组合式 api 是在 setup 期间使用 onMounted 这种 Api 来实现的

```ts
export function finishComponentSetup(
  instance: ComponentInternalInstance,
  isSSR: boolean,
  skipOptions?: boolean
) {
  const Component = instance.type as ComponentOptions

  if (__COMPAT__) {
    convertLegacyRenderFn(instance)

    if (__DEV__ && Component.compatConfig) {
      validateCompatConfig(Component.compatConfig)
    }
  }

  // template / render function normalization
  // could be already set when returned from setup()
  if (!instance.render) {
    // only do on-the-fly compile if not in SSR - SSR on-the-fly compliation
    // is done by server-renderer
    if (!isSSR && compile && !Component.render) {
      const template =
        (__COMPAT__ &&
          instance.vnode.props &&
          instance.vnode.props['inline-template']) ||
        Component.template
      if (template) {
        if (__DEV__) {
          startMeasure(instance, `compile`)
        }
        const { isCustomElement, compilerOptions } = instance.appContext.config
        const { delimiters, compilerOptions: componentCompilerOptions } =
          Component
        const finalCompilerOptions: CompilerOptions = extend(
          extend(
            {
              isCustomElement,
              delimiters
            },
            compilerOptions
          ),
          componentCompilerOptions
        )
        if (__COMPAT__) {
          // pass runtime compat config into the compiler
          finalCompilerOptions.compatConfig = Object.create(globalCompatConfig)
          if (Component.compatConfig) {
            extend(finalCompilerOptions.compatConfig, Component.compatConfig)
          }
        }
        Component.render = compile(template, finalCompilerOptions)
        if (__DEV__) {
          endMeasure(instance, `compile`)
        }
      }
    }

    instance.render = (Component.render || NOOP) as InternalRenderFunction

    // for runtime-compiled render functions using `with` blocks, the render
    // proxy used needs a different `has` handler which is more performant and
    // also only allows a whitelist of globals to fallthrough.
    if (installWithProxy) {
      installWithProxy(instance)
    }
  }

  // support for 2.x options
  if (__FEATURE_OPTIONS_API__ && !(__COMPAT__ && skipOptions)) {
    setCurrentInstance(instance)
    pauseTracking()
    applyOptions(instance)
    resetTracking()
    unsetCurrentInstance()
  }

  // warn missing template/render
  // the runtime compilation of template in SSR is done by server-render
  if (__DEV__ && !Component.render && instance.render === NOOP && !isSSR) {
    /* istanbul ignore if */
    if (!compile && Component.template) {
      warn(
        `Component provided template option but ` +
          `runtime compilation is not supported in this build of Vue.` +
          (__ESM_BUNDLER__
            ? ` Configure your bundler to alias "vue" to "vue/dist/vue.esm-bundler.js".`
            : __ESM_BROWSER__
            ? ` Use "vue.esm-browser.js" instead.`
            : __GLOBAL__
            ? ` Use "vue.global.js" instead.`
            : ``) /* should not happen */
      )
    } else {
      warn(`Component is missing template or render function.`)
    }
  }
}
```

##### [compile]

编译的函数是在初始化 vue 实例时被安装的, 在 vue/src/index.ts 加载 vue 源码时, 会从当前页开始; 使用 registerRuntimeCompiler 注册 compile 函数

```ts
// packages\vue\src\index.ts
function compileToFunction () {
  // ...
  const { code } = compile(
    template,
    extend(
      {
        hoistStatic: true,
        onError: __DEV__ ? onError : undefined,
        onWarn: __DEV__ ? e => onError(e, true) : NOOP
      } as CompilerOptions,
      options
    )
  )
  // ...
  const render = (
    __GLOBAL__ ? new Function(code)() : new Function('Vue', code)(runtimeDom)
  ) as RenderFunction
  // ...
}

registerRuntimeCompiler(compileToFunction)
```

registerRuntimeCompiler 在上边传入了 compileToFunction 的方法, 在下边就可以看到该函数被赋值给当前页的 compile, 在其后直接执行 compile()

```ts
// packages\runtime-core\src\component.ts
let compile: CompileFunction | undefined

export function registerRuntimeCompiler(_compile: any) {
  compile = _compile
  installWithProxy = i => {
    if (i.render!._rc) {
      i.withProxy = new Proxy(i.ctx, RuntimeCompiledPublicInstanceProxyHandlers)
    }
  }
}

// 上一步完成组件安装的函数中使用到的 compile
export function finishComponentSetup(
  instance: ComponentInternalInstance,
  isSSR: boolean,
  skipOptions?: boolean
) {
  const Component = instance.type as ComponentOptions
  // ...
  if (!instance.render) {
    const template =
      (__COMPAT__ &&
        instance.vnode.props &&
        instance.vnode.props['inline-template']) ||
      Component.template // 这里走的是组件的模板
    if (template) {
      // ...
      Component.render = compile(template, finalCompilerOptions)
    }
    // ...
  }
  // ...
}
```

##### compileToFunction

我们也就清楚了上边的 compile 也就是 compileToFunction

可以看到对 template 进行判断, 接着调用真正的 compile 进行解析 template, 最后提取 code, 通过 code 来生成函数执行, 最后赋值给 render, 最后缓存当前 template 对应的这个 render 方法, 并返回

ps: (render 函数的获得在 compileToFunction 之后展示)

```ts
// packages\vue\src\index.ts
function compileToFunction(
  template: string | HTMLElement,
  options?: CompilerOptions
): RenderFunction {
  if (!isString(template)) {
    if (template.nodeType) {
      template = template.innerHTML
    } else {
      __DEV__ && warn(`invalid template option: `, template)
      return NOOP
    }
  }

  const key = template
  const cached = compileCache[key]
  if (cached) {
    return cached
  }

  if (template[0] === '#') {
    const el = document.querySelector(template)
    if (__DEV__ && !el) {
      warn(`Template element not found or is empty: ${template}`)
    }
    // __UNSAFE__
    // Reason: potential execution of JS expressions in in-DOM template.
    // The user must make sure the in-DOM template is trusted. If it's rendered
    // by the server, the template should not contain any user data.
    template = el ? el.innerHTML : ``
  }

  const { code } = compile(
    template,
    extend(
      {
        hoistStatic: true,
        onError: __DEV__ ? onError : undefined,
        onWarn: __DEV__ ? e => onError(e, true) : NOOP
      } as CompilerOptions,
      options
    )
  )

  function onError(err: CompilerError, asWarning = false) {
    const message = asWarning
      ? err.message
      : `Template compilation error: ${err.message}`
    const codeFrame =
      err.loc &&
      generateCodeFrame(
        template as string,
        err.loc.start.offset,
        err.loc.end.offset
      )
    warn(codeFrame ? `${message}\n${codeFrame}` : message)
  }

  // The wildcard import results in a huge object with every export
  // with keys that cannot be mangled, and can be quite heavy size-wise.
  // In the global build we know `Vue` is available globally so we can avoid
  // the wildcard object.
  const render = (
    __GLOBAL__ ? new Function(code)() : new Function('Vue', code)(runtimeDom)
  ) as RenderFunction

  // mark the function as runtime compiled
  ;(render as InternalRenderFunction)._rc = true

  return (compileCache[key] = render)
}
```

呈接 [generate](######generate) 生成的 code, 接下来看如何执行:

```ts
const render = (
  __GLOBAL__ ? new Function(code)() : new Function('Vue', code)(runtimeDom)
)

// new 函数立即执行
(function anonymous(
) {
const _Vue = Vue
// 从构造函数获得创建虚拟节点的方法
const { createElementVNode: _createElementVNode, createTextVNode: _createTextVNode } = _Vue

// 创建三个静态子节点的虚拟节点
const _hoisted_1 = /*#__PURE__*/_createElementVNode("span", null, "reactive:", -1 /* HOISTED */)
const _hoisted_2 = /*#__PURE__*/_createElementVNode("span", null, "ref:", -1 /* HOISTED */)
const _hoisted_3 = /*#__PURE__*/_createElementVNode("span", null, "computed:", -1 /* HOISTED */)

return function render(_ctx, _cache) {
  with (_ctx) {
    const { createElementVNode: _createElementVNode, toDisplayString: _toDisplayString, createTextVNode: _createTextVNode, Fragment: _Fragment, openBlock: _openBlock, createElementBlock: _createElementBlock } = _Vue

    return (_openBlock(), _createElementBlock(_Fragment, null, [
      _createElementVNode("div", null, [
        _hoisted_1,
        _createTextVNode(_toDisplayString(state.title), 1 /* TEXT */)
      ]),
      _createElementVNode("div", null, [
        _hoisted_2,
        _createTextVNode(_toDisplayString(price), 1 /* TEXT */)
      ]),
      _createElementVNode("div", null, [
        _hoisted_3,
        _createTextVNode(_toDisplayString(currTime), 1 /* TEXT */)
      ])
    ], 64 /* STABLE_FRAGMENT */))
  }
}
})
```


##### compile

这里的 compile 才是实际执行编译的方法, 会将传入的 template 解析生成 code

在这里就利用了函数柯里化, compile 获取参数 template 和 option, 直接返回 baseCompile 函数传入 template 及其他配置

```ts
// packages\compiler-dom\src\index.ts
export function compile(
  template: string,
  options: CompilerOptions = {}
): CodegenResult {
  return baseCompile(
    template,
    extend({}, parserOptions, options, {
      nodeTransforms: [
        // ignore <script> and <tag>
        // this is not put inside DOMNodeTransforms because that list is used
        // by compiler-ssr to generate vnode fallback branches
        ignoreSideEffectTags,
        ...DOMNodeTransforms,
        ...(options.nodeTransforms || [])
      ],
      directiveTransforms: extend(
        {},
        DOMDirectiveTransforms,
        options.directiveTransforms || {}
      ),
      transformHoist: __BROWSER__ ? null : stringifyStatic
    })
  )
}
```

##### baseCompile

baseCompile 的工作就是解析 template, vue3 是先用 baseParse 解析成 ast 树, 再用 transform 来将生成的 ast 树处理, 最后返回用 generate 生成的代码 code

```ts
// packages\compiler-core\src\compile.ts
export function baseCompile(
  template: string | RootNode,
  options: CompilerOptions = {}
): CodegenResult {
  const onError = options.onError || defaultOnError
  const isModuleMode = options.mode === 'module'
  /* istanbul ignore if */
  if (__BROWSER__) {
    if (options.prefixIdentifiers === true) {
      onError(createCompilerError(ErrorCodes.X_PREFIX_ID_NOT_SUPPORTED))
    } else if (isModuleMode) {
      onError(createCompilerError(ErrorCodes.X_MODULE_MODE_NOT_SUPPORTED))
    }
  }

  const prefixIdentifiers =
    !__BROWSER__ && (options.prefixIdentifiers === true || isModuleMode)
  if (!prefixIdentifiers && options.cacheHandlers) {
    onError(createCompilerError(ErrorCodes.X_CACHE_HANDLER_NOT_SUPPORTED))
  }
  if (options.scopeId && !isModuleMode) {
    onError(createCompilerError(ErrorCodes.X_SCOPE_ID_NOT_SUPPORTED))
  }

  // 解析 template 为 ast 树
  const ast = isString(template) ? baseParse(template, options) : template
  const [nodeTransforms, directiveTransforms] =
    getBaseTransformPreset(prefixIdentifiers)

  if (!__BROWSER__ && options.isTS) {
    const { expressionPlugins } = options
    if (!expressionPlugins || !expressionPlugins.includes('typescript')) {
      options.expressionPlugins = [...(expressionPlugins || []), 'typescript']
    }
  }

  transform(
    ast,
    extend({}, options, {
      prefixIdentifiers,
      nodeTransforms: [
        ...nodeTransforms,
        ...(options.nodeTransforms || []) // user transforms
      ],
      directiveTransforms: extend(
        {},
        directiveTransforms,
        options.directiveTransforms || {} // user transforms
      )
    })
  )

  return generate(
    ast,
    extend({}, options, {
      prefixIdentifiers
    })
  )
}
```

###### baseParse

首先创建一个解析的上下文 context, 作以整个的记录; 包含有配置, 如 delimeters, isNativeTag 等, 以及缓存 source(即原 template)

getCursor 来获取开始解析的一个定位

先解析子节点, 再创建 root, 生成完整的 ast 树,

```ts
// packages\compiler-core\src\parse.ts
export function baseParse(
  content: string,
  options: ParserOptions = {}
): RootNode {
  const context = createParserContext(content, options)
  const start = getCursor(context)
  return createRoot(
    parseChildren(context, TextModes.DATA, []),
    getSelection(context, start)
  )
}

// createParserContext
function createParserContext(
  content: string,
  rawOptions: ParserOptions
): ParserContext {
  const options = extend({}, defaultParserOptions)

  let key: keyof ParserOptions
  for (key in rawOptions) {
    // @ts-ignore
    options[key] =
      rawOptions[key] === undefined
        ? defaultParserOptions[key]
        : rawOptions[key]
  }
  return {
    options,
    column: 1,
    line: 1,
    offset: 0,
    originalSource: content,
    source: content,
    inPre: false,
    inVPre: false,
    onWarn: options.onWarn
  }
}
```

###### transform

将 ast 作为 root , 给 root 上添加 codegenNode 描述

```ts
// packages\compiler-core\src\transform.ts
export function transform(root: RootNode, options: TransformOptions) {
  const context = createTransformContext(root, options)
  traverseNode(root, context)
  if (options.hoistStatic) {
    hoistStatic(root, context)
  }
  if (!options.ssr) {
    createRootCodegen(root, context)
  }
  // finalize meta information
  root.helpers = [...context.helpers.keys()]
  root.components = [...context.components]
  root.directives = [...context.directives]
  root.imports = context.imports
  root.hoists = context.hoists
  root.temps = context.temps
  root.cached = context.cached

  if (__COMPAT__) {
    root.filters = [...context.filters!]
  }
}
```

###### generate

创建 codegen 上下文, 开始根据 ast 所有的属性来生成 code, 通过遍历所有的属性, 来拼贴成字符串, 在 compile 后提取 code

生成 code 后流程就回到 [compileToFunction](#####compileToFunction) 函数, 利用 code 来 new 一个函数并执行, 赋值给 render, 这就是 render 方法了;

可以看到我此处的 code 为:

```ts
'const _Vue = Vue\nconst { createElementVNode: _createElementVNode, createTextVNode: _createTextVNode } = _Vue\n\nconst _hoisted_1 = /*#__PURE__*/_createElementVNode("span", null, "reactive:", -1 /* HOISTED */)\nconst _hoisted_2 = /*#__PURE__*/_createElementVNode("span", null, "ref:", -1 /* HOISTED */)\nconst _hoisted_3 = /*#__PURE__*/_createElementVNode("span", null, "computed:", -1 /* HOISTED */)\n\nreturn function render(_ctx, _cache) {\n  with (_ctx) {\n    const { createElementVNode: _createElementV…ock(), _createElementBlock(_Fragment, null, [\n      _createElementVNode("div", null, [\n        _hoisted_1,\n        _createTextVNode(_toDisplayString(state.title), 1 /* TEXT */)\n      ]),\n      _createElementVNode("div", null, [\n        _hoisted_2,\n        _createTextVNode(_toDisplayString(price), 1 /* TEXT */)\n      ]),\n      _createElementVNode("div", null, [\n        _hoisted_3,\n        _createTextVNode(_toDisplayString(currTime), 1 /* TEXT */)\n      ])\n    ], 64 /* STABLE_FRAGMENT */))\n  }\n}'
```

格式化下:

```ts
const _Vue = Vue
const { createElementVNode: _createElementVNode, createTextVNode: _createTextVNode } = _Vue

const _hoisted_1 = /*#__PURE__*/_createElementVNode("span", null, "reactive:", -1 /* HOISTED */)
const _hoisted_2 = /*#__PURE__*/_createElementVNode("span", null, "ref:", -1 /* HOISTED */)
const _hoisted_3 = /*#__PURE__*/_createElementVNode("span", null, "computed:", -1 /* HOISTED */)

return function render(_ctx, _cache) {
  with (_ctx) {
    const { createElementVNode: _createElementVNode, toDisplayString: _toDisplayString, createTextVNode: _createTextVNode, Fragment: _Fragment, openBlock: _openBlock, createElementBlock: _createElementBlock } = _Vue

    return (_openBlock(), _createElementBlock(_Fragment, null, [
      _createElementVNode("div", null, [
        _hoisted_1,
        _createTextVNode(_toDisplayString(state.title), 1 /* TEXT */)
      ]),
      _createElementVNode("div", null, [
        _hoisted_2,
        _createTextVNode(_toDisplayString(price), 1 /* TEXT */)
      ]),
      _createElementVNode("div", null, [
        _hoisted_3,
        _createTextVNode(_toDisplayString(currTime), 1 /* TEXT */)
      ])
    ], 64 /* STABLE_FRAGMENT */))
  }
}
```

generate 函数:

```ts
// packages\compiler-core\src\codegen.ts
export function generate(
  ast: RootNode,
  options: CodegenOptions & {
    onContextCreated?: (context: CodegenContext) => void
  } = {}
): CodegenResult {
  const context = createCodegenContext(ast, options)
  if (options.onContextCreated) options.onContextCreated(context)
  const {
    mode,
    push,
    prefixIdentifiers,
    indent,
    deindent,
    newline,
    scopeId,
    ssr
  } = context

  const hasHelpers = ast.helpers.length > 0
  const useWithBlock = !prefixIdentifiers && mode !== 'module'
  const genScopeId = !__BROWSER__ && scopeId != null && mode === 'module'
  const isSetupInlined = !__BROWSER__ && !!options.inline

  // preambles
  // in setup() inline mode, the preamble is generated in a sub context
  // and returned separately.
  const preambleContext = isSetupInlined
    ? createCodegenContext(ast, options)
    : context
  if (!__BROWSER__ && mode === 'module') {
    genModulePreamble(ast, preambleContext, genScopeId, isSetupInlined)
  } else {
    genFunctionPreamble(ast, preambleContext)
  }
  // enter render function
  const functionName = ssr ? `ssrRender` : `render`
  const args = ssr ? ['_ctx', '_push', '_parent', '_attrs'] : ['_ctx', '_cache']
  if (!__BROWSER__ && options.bindingMetadata && !options.inline) {
    // binding optimization args
    args.push('$props', '$setup', '$data', '$options')
  }
  const signature =
    !__BROWSER__ && options.isTS
      ? args.map(arg => `${arg}: any`).join(',')
      : args.join(', ')

  if (isSetupInlined) {
    push(`(${signature}) => {`)
  } else {
    push(`function ${functionName}(${signature}) {`)
  }
  indent()

  if (useWithBlock) {
    push(`with (_ctx) {`)
    indent()
    // function mode const declarations should be inside with block
    // also they should be renamed to avoid collision with user properties
    if (hasHelpers) {
      push(
        `const { ${ast.helpers
          .map(s => `${helperNameMap[s]}: _${helperNameMap[s]}`)
          .join(', ')} } = _Vue`
      )
      push(`\n`)
      newline()
    }
  }

  // generate asset resolution statements
  if (ast.components.length) {
    genAssets(ast.components, 'component', context)
    if (ast.directives.length || ast.temps > 0) {
      newline()
    }
  }
  if (ast.directives.length) {
    genAssets(ast.directives, 'directive', context)
    if (ast.temps > 0) {
      newline()
    }
  }
  if (__COMPAT__ && ast.filters && ast.filters.length) {
    newline()
    genAssets(ast.filters, 'filter', context)
    newline()
  }

  if (ast.temps > 0) {
    push(`let `)
    for (let i = 0; i < ast.temps; i++) {
      push(`${i > 0 ? `, ` : ``}_temp${i}`)
    }
  }
  if (ast.components.length || ast.directives.length || ast.temps) {
    push(`\n`)
    newline()
  }

  // generate the VNode tree expression
  if (!ssr) {
    push(`return `)
  }
  if (ast.codegenNode) {
    genNode(ast.codegenNode, context)
  } else {
    push(`null`)
  }

  if (useWithBlock) {
    deindent()
    push(`}`)
  }

  deindent()
  push(`}`)

  return {
    ast,
    code: context.code,
    preamble: isSetupInlined ? preambleContext.code : ``,
    // SourceMapGenerator does have toJSON() method but it's not in the types
    map: context.map ? (context.map as any).toJSON() : undefined
  }
}
```

### setupRenderEffect

先定义组件更新的函数 componentUpdateFn(实际就是 this.fn, 后边在 update 时触发), 接着创建 effect 准备 render, ReactiveEffect 构造函数传入之前的更新函数, 以及将实例更新的队列作为 scheduler 函数压入 effect, 最后将实例的 scope 作为 effect scope

接着给 instance.update 赋值 effect.run.bind(effect), 实际也就是 effect 副作用更新的 run 函数绑定 this 指向后的函数, instance.update 赋值给 update, 在最后执行 update(), 进行初始化 render

```ts
const setupRenderEffect: SetupRenderEffectFn = (
  instance,
  initialVNode,
  container,
  anchor,
  parentSuspense,
  isSVG,
  optimized
) => {
  // 组件更新函数, 作为 fn 被传入 ReactiveEffect
  const componentUpdateFn = () => {
    // ...
  }

  // create reactive effect for rendering
  const effect = new ReactiveEffect(
    componentUpdateFn,
    () => queueJob(instance.update),
    instance.scope // track it in component's effect scope
  )

  // 给 update 和 instance.update 赋值 effect.run 方法
  const update = (instance.update = effect.run.bind(effect) as SchedulerJob)
  update.id = instance.uid
  // allowRecurse
  // #1801, #2043 component render effects should allow recursive updates
  effect.allowRecurse = update.allowRecurse = true

  if (__DEV__) {
    effect.onTrack = instance.rtc
      ? e => invokeArrayFns(instance.rtc!, e)
      : void 0
    effect.onTrigger = instance.rtg
      ? e => invokeArrayFns(instance.rtg!, e)
      : void 0
    // @ts-ignore (for scheduler)
    update.ownerInstance = instance
  }

  // 相当于执行 effect.run()
  update()
}
```

#### update() == effect.run()

run 作为 effect 实例的方法, 用以触发 effect 进行依赖收集, setupRenderEffect

```ts
// packages\reactivity\src\effect.ts

run() {
  // 如果当前的 effect 副作用时激活的, 则直接返回 fn(), 也就是组件更新的函数
  if (!this.active) {
    return this.fn()
  }
  if (!effectStack.includes(this)) {
    try {
      // 将当前活跃的 effect 添加进栈
      effectStack.push((activeEffect = this))
      // 开始 track
      enableTracking()

      trackOpBit = 1 << ++effectTrackDepth

      // effect track 的深度要小于 30, 否则就开始清理 effect
      if (effectTrackDepth <= maxMarkerBits) {
        initDepMarkers(this)
      } else {
        cleanupEffect(this)
      }
      return this.fn()
    } finally {
      if (effectTrackDepth <= maxMarkerBits) {
        finalizeDepMarkers(this)
      }

      trackOpBit = 1 << --effectTrackDepth

      resetTracking()
      effectStack.pop()
      const n = effectStack.length
      activeEffect = n > 0 ? effectStack[n - 1] : undefined
    }
  }
}
```

#### componentUpdateFn

在 effect.run 中 this.fn() 里被调用, 用以更新或初始化组件; 通过 renderComponentRoot 构建树

```ts
// packages\runtime-core\src\renderer.ts

const componentUpdateFn = () => {
  // 按实例是否挂载来判断接下来执行初始化还是更新
  if (!instance.isMounted) {
    let vnodeHook: VNodeHook | null | undefined
    const { el, props } = initialVNode
    const { bm, m, parent } = instance
    const isAsyncWrapperVNode = isAsyncWrapper(initialVNode)

    effect.allowRecurse = false
    // beforeMount hook
    // 执行 beforeMount 定义的钩子函数
    if (bm) {
      invokeArrayFns(bm)
    }
    // onVnodeBeforeMount
    if (
      !isAsyncWrapperVNode &&
      (vnodeHook = props && props.onVnodeBeforeMount)
    ) {
      invokeVNodeHook(vnodeHook, parent, initialVNode)
    }
    if (
      __COMPAT__ &&
      isCompatEnabled(DeprecationTypes.INSTANCE_EVENT_HOOKS, instance)
    ) {
      instance.emit('hook:beforeMount')
    }
    effect.allowRecurse = true

    if (el && hydrateNode) {
      // vnode has adopted host node - perform hydration instead of mount.
      const hydrateSubTree = () => {
        if (__DEV__) {
          startMeasure(instance, `render`)
        }
        instance.subTree = renderComponentRoot(instance)
        if (__DEV__) {
          endMeasure(instance, `render`)
        }
        if (__DEV__) {
          startMeasure(instance, `hydrate`)
        }
        hydrateNode!(
          el as Node,
          instance.subTree,
          instance,
          parentSuspense,
          null
        )
        if (__DEV__) {
          endMeasure(instance, `hydrate`)
        }
      }

      if (isAsyncWrapperVNode) {
        ;(initialVNode.type as ComponentOptions).__asyncLoader!().then(
          // note: we are moving the render call into an async callback,
          // which means it won't track dependencies - but it's ok because
          // a server-rendered async wrapper is already in resolved state
          // and it will never need to change.
          () => !instance.isUnmounted && hydrateSubTree()
        )
      } else {
        hydrateSubTree()
      }
    } else {
      if (__DEV__) {
        startMeasure(instance, `render`)
      }
      const subTree = (instance.subTree = renderComponentRoot(instance))
      if (__DEV__) {
        endMeasure(instance, `render`)
      }
      if (__DEV__) {
        startMeasure(instance, `patch`)
      }
      patch(
        null,
        subTree,
        container,
        anchor,
        instance,
        parentSuspense,
        isSVG
      )
      if (__DEV__) {
        endMeasure(instance, `patch`)
      }
      initialVNode.el = subTree.el
    }
    // mounted hook
    if (m) {
      queuePostRenderEffect(m, parentSuspense)
    }
    // onVnodeMounted
    if (
      !isAsyncWrapperVNode &&
      (vnodeHook = props && props.onVnodeMounted)
    ) {
      const scopedInitialVNode = initialVNode
      queuePostRenderEffect(
        () => invokeVNodeHook(vnodeHook!, parent, scopedInitialVNode),
        parentSuspense
      )
    }
    if (
      __COMPAT__ &&
      isCompatEnabled(DeprecationTypes.INSTANCE_EVENT_HOOKS, instance)
    ) {
      queuePostRenderEffect(
        () => instance.emit('hook:mounted'),
        parentSuspense
      )
    }

    // activated hook for keep-alive roots.
    // #1742 activated hook must be accessed after first render
    // since the hook may be injected by a child keep-alive
    if (initialVNode.shapeFlag & ShapeFlags.COMPONENT_SHOULD_KEEP_ALIVE) {
      instance.a && queuePostRenderEffect(instance.a, parentSuspense)
      if (
        __COMPAT__ &&
        isCompatEnabled(DeprecationTypes.INSTANCE_EVENT_HOOKS, instance)
      ) {
        queuePostRenderEffect(
          () => instance.emit('hook:activated'),
          parentSuspense
        )
      }
    }
    instance.isMounted = true

    if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
      devtoolsComponentAdded(instance)
    }

    // #2458: deference mount-only object parameters to prevent memleaks
    initialVNode = container = anchor = null as any
  } else {
    // updateComponent
    // This is triggered by mutation of component's own state (next: null)
    // OR parent calling processComponent (next: VNode)
    let { next, bu, u, parent, vnode } = instance
    let originNext = next
    let vnodeHook: VNodeHook | null | undefined
    if (__DEV__) {
      pushWarningContext(next || instance.vnode)
    }

    // Disallow component effect recursion during pre-lifecycle hooks.
    effect.allowRecurse = false

    if (next) {
      next.el = vnode.el
      updateComponentPreRender(instance, next, optimized)
    } else {
      next = vnode
    }

    // beforeUpdate hook
    if (bu) {
      invokeArrayFns(bu)
    }
    // onVnodeBeforeUpdate
    if ((vnodeHook = next.props && next.props.onVnodeBeforeUpdate)) {
      invokeVNodeHook(vnodeHook, parent, next, vnode)
    }
    if (
      __COMPAT__ &&
      isCompatEnabled(DeprecationTypes.INSTANCE_EVENT_HOOKS, instance)
    ) {
      instance.emit('hook:beforeUpdate')
    }

    effect.allowRecurse = true

    // render
    if (__DEV__) {
      startMeasure(instance, `render`)
    }
    const nextTree = renderComponentRoot(instance)
    if (__DEV__) {
      endMeasure(instance, `render`)
    }
    const prevTree = instance.subTree
    instance.subTree = nextTree

    if (__DEV__) {
      startMeasure(instance, `patch`)
    }
    patch(
      prevTree,
      nextTree,
      // parent may have changed if it's in a teleport
      hostParentNode(prevTree.el!)!,
      // anchor may have changed if it's in a fragment
      getNextHostNode(prevTree),
      instance,
      parentSuspense,
      isSVG
    )
    if (__DEV__) {
      endMeasure(instance, `patch`)
    }
    next.el = nextTree.el
    if (originNext === null) {
      // self-triggered update. In case of HOC, update parent component
      // vnode el. HOC is indicated by parent instance's subTree pointing
      // to child component's vnode
      updateHOCHostEl(instance, nextTree.el)
    }
    // updated hook
    if (u) {
      queuePostRenderEffect(u, parentSuspense)
    }
    // onVnodeUpdated
    if ((vnodeHook = next.props && next.props.onVnodeUpdated)) {
      queuePostRenderEffect(
        () => invokeVNodeHook(vnodeHook!, parent, next!, vnode),
        parentSuspense
      )
    }
    if (
      __COMPAT__ &&
      isCompatEnabled(DeprecationTypes.INSTANCE_EVENT_HOOKS, instance)
    ) {
      queuePostRenderEffect(
        () => instance.emit('hook:updated'),
        parentSuspense
      )
    }

    if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
      devtoolsComponentUpdated(instance)
    }

    if (__DEV__) {
      popWarningContext()
    }
  }
}
```

##### renderComponentRoot

构建 subTree, nextTree

```ts
// packages\runtime-core\src\componentRenderUtils.ts
export function renderComponentRoot(
  instance: ComponentInternalInstance
): VNode {
  const {
    type: Component,
    vnode,
    proxy,
    withProxy,
    props,
    propsOptions: [propsOptions],
    slots,
    attrs,
    emit,
    render,
    renderCache,
    data,
    setupState,
    ctx,
    inheritAttrs
  } = instance

  let result
  let fallthroughAttrs
  const prev = setCurrentRenderingInstance(instance)
  if (__DEV__) {
    accessedAttrs = false
  }

  try {
    if (vnode.shapeFlag & ShapeFlags.STATEFUL_COMPONENT) {
      // withProxy is a proxy with a different `has` trap only for
      // runtime-compiled render functions using `with` block.
      const proxyToUse = withProxy || proxy
      result = normalizeVNode(
        render!.call(
          proxyToUse,
          proxyToUse!,
          renderCache,
          props,
          setupState,
          data,
          ctx
        )
      )
      fallthroughAttrs = attrs
    } else {
      // functional
      const render = Component as FunctionalComponent
      // in dev, mark attrs accessed if optional props (attrs === props)
      if (__DEV__ && attrs === props) {
        markAttrsAccessed()
      }
      result = normalizeVNode(
        render.length > 1
          ? render(
              props,
              __DEV__
                ? {
                    get attrs() {
                      markAttrsAccessed()
                      return attrs
                    },
                    slots,
                    emit
                  }
                : { attrs, slots, emit }
            )
          : render(props, null as any /* we know it doesn't need it */)
      )
      fallthroughAttrs = Component.props
        ? attrs
        : getFunctionalFallthrough(attrs)
    }
  } catch (err) {
    blockStack.length = 0
    handleError(err, instance, ErrorCodes.RENDER_FUNCTION)
    result = createVNode(Comment)
  }

  // attr merging
  // in dev mode, comments are preserved, and it's possible for a template
  // to have comments along side the root element which makes it a fragment
  let root = result
  let setRoot: ((root: VNode) => void) | undefined = undefined
  if (
    __DEV__ &&
    result.patchFlag > 0 &&
    result.patchFlag & PatchFlags.DEV_ROOT_FRAGMENT
  ) {
    ;[root, setRoot] = getChildRoot(result)
  }

  if (fallthroughAttrs && inheritAttrs !== false) {
    const keys = Object.keys(fallthroughAttrs)
    const { shapeFlag } = root
    if (keys.length) {
      if (shapeFlag & (ShapeFlags.ELEMENT | ShapeFlags.COMPONENT)) {
        if (propsOptions && keys.some(isModelListener)) {
          // If a v-model listener (onUpdate:xxx) has a corresponding declared
          // prop, it indicates this component expects to handle v-model and
          // it should not fallthrough.
          // related: #1543, #1643, #1989
          fallthroughAttrs = filterModelListeners(
            fallthroughAttrs,
            propsOptions
          )
        }
        root = cloneVNode(root, fallthroughAttrs)
      } else if (__DEV__ && !accessedAttrs && root.type !== Comment) {
        const allAttrs = Object.keys(attrs)
        const eventAttrs: string[] = []
        const extraAttrs: string[] = []
        for (let i = 0, l = allAttrs.length; i < l; i++) {
          const key = allAttrs[i]
          if (isOn(key)) {
            // ignore v-model handlers when they fail to fallthrough
            if (!isModelListener(key)) {
              // remove `on`, lowercase first letter to reflect event casing
              // accurately
              eventAttrs.push(key[2].toLowerCase() + key.slice(3))
            }
          } else {
            extraAttrs.push(key)
          }
        }
        if (extraAttrs.length) {
          warn(
            `Extraneous non-props attributes (` +
              `${extraAttrs.join(', ')}) ` +
              `were passed to component but could not be automatically inherited ` +
              `because component renders fragment or text root nodes.`
          )
        }
        if (eventAttrs.length) {
          warn(
            `Extraneous non-emits event listeners (` +
              `${eventAttrs.join(', ')}) ` +
              `were passed to component but could not be automatically inherited ` +
              `because component renders fragment or text root nodes. ` +
              `If the listener is intended to be a component custom event listener only, ` +
              `declare it using the "emits" option.`
          )
        }
      }
    }
  }

  if (
    __COMPAT__ &&
    isCompatEnabled(DeprecationTypes.INSTANCE_ATTRS_CLASS_STYLE, instance) &&
    vnode.shapeFlag & ShapeFlags.STATEFUL_COMPONENT &&
    root.shapeFlag & (ShapeFlags.ELEMENT | ShapeFlags.COMPONENT)
  ) {
    const { class: cls, style } = vnode.props || {}
    if (cls || style) {
      if (__DEV__ && inheritAttrs === false) {
        warnDeprecation(
          DeprecationTypes.INSTANCE_ATTRS_CLASS_STYLE,
          instance,
          getComponentName(instance.type)
        )
      }
      root = cloneVNode(root, {
        class: cls,
        style: style
      })
    }
  }

  // inherit directives
  if (vnode.dirs) {
    if (__DEV__ && !isElementRoot(root)) {
      warn(
        `Runtime directive used on component with non-element root node. ` +
          `The directives will not function as intended.`
      )
    }
    root.dirs = root.dirs ? root.dirs.concat(vnode.dirs) : vnode.dirs
  }
  // inherit transition data
  if (vnode.transition) {
    if (__DEV__ && !isElementRoot(root)) {
      warn(
        `Component inside <Transition> renders non-element root node ` +
          `that cannot be animated.`
      )
    }
    root.transition = vnode.transition
  }

  if (__DEV__ && setRoot) {
    setRoot(root)
  } else {
    result = root
  }

  setCurrentRenderingInstance(prev)
  return result
}
```

