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

### createRenderer

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

### createAppAPI

来到 apiCreateApp 文件, 可以看到 `createAppAPI(render, hydrate)` 刚好是我们上一步得到的, 返回实际的 createApp 函数, 在此接收我们从初始传入的参数 args, 也就是此时的 rootComponent 跟组件; 

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

## mount

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

### createVNode

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

### render

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

### patch

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

### processComponent

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

### mountComponent

接收参数就直接接收新的虚拟节点, 真实 dom 根节点; 



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

  const setupResult = isStateful
    ? setupStatefulComponent(instance, isSSR)
    : undefined
  isInSSRComponentSetup = false
  return setupResult
}

// initProps
export function initProps(
  instance: ComponentInternalInstance,
  rawProps: Data | null,
  isStateful: number, // result of bitwise flag comparison
  isSSR = false
) {
  const props: Data = {}
  const attrs: Data = {}
  def(attrs, InternalObjectKey, 1)

  instance.propsDefaults = Object.create(null)

  setFullProps(instance, rawProps, props, attrs)

  // ensure all declared prop keys are present
  for (const key in instance.propsOptions[0]) {
    if (!(key in props)) {
      props[key] = undefined
    }
  }

  // validation
  if (__DEV__) {
    validateProps(rawProps || {}, props, instance)
  }

  if (isStateful) {
    // stateful
    instance.props = isSSR ? props : shallowReactive(props)
  } else {
    if (!instance.type.props) {
      // functional w/ optional props, props === attrs
      instance.props = attrs
    } else {
      // functional w/ declared props
      instance.props = props
    }
  }
  instance.attrs = attrs
}

// initSlots
```

### setupStatefulComponent

接收 instance 参数, 以及是否服务端渲染

从实例的 type 获取用户定义的根组件内容 Component, 如果是开发环境, 需要对组件进行检查, 查看其定义是否有效

接着从 Component 读取 setup, 如果用户侧用组合式 API , 那么就有用到 setup, 如果未定义, 那么就结束组件安装

如果写入了 setup 函数, 那么就开始安装, 先设定当前的实例, 全局变量 currentInstance 就等于当前活动的 instance, 接着进入 effectScope, 调用 on 方法, 将当前活跃的 effectScope 压入 effectScopeStack 栈中, 并标记 activeEffectScope 为 当前的 effectScope

pauseTracking 将 shouldTrack 压入 trackStack , 再将全局的 shouldTrack 变更为 false

接着处理 setup, 开发环境下得到浅层只读(shallow readonly)的 props 响应式代理, 将 props 作为参数, 传入我们定义的 setup 函数执行, 

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

### reactive

若是我们代码中有定义响应式 reactive 对象, 那么在安装组件时便会运行我们书写的 setup, 在此函数里完成整个组件的流程, 这里我们来主要看下响应式系统的构建流程

除 reactive 定义外, 还有 readonly(只读的响应式对象代理), 浅层的代理 shallow, 流程是一样的

reactive 函数接收一个待响应式化的对象, 如果传入的对象存在且对象存在只读属性的标记, 那么就直接返回该对象, 说明该对象是经过 readonly() 响应式化的对象; 若无此问题, 则调用 createReactiveObject 函数创建响应式对象

createReactiveObject 接收该对象, 并标记是否是 readonly , 同时传入可变的 mutableHandlers, 此handler 是提供给 Proxy 代理的配置, 此外也存在 readonly, 以及浅层对应的处理器; 此外还有针对 set、map、weakSet、weakMap 数据结构的 collectionHandlers, 最后的参数是 reactiveMap, 用于缓存根据当前的 target 创建的 proxy 对象的 weakMap 对象

createReactiveObject 首先判断传入的值是否为对象, 若不是则警告返回目标

```ts
// packages\reactivity\src\reactive.ts
export function reactive<T extends object>(target: T): UnwrapNestedRefs<T>
export function reactive(target: object) {
  // if trying to observe a readonly proxy, return the readonly version.
  if (target && (target as Target)[ReactiveFlags.IS_READONLY]) {
    return target
  }
  return createReactiveObject(
    target,
    false,
    mutableHandlers,
    mutableCollectionHandlers,
    reactiveMap
  )
}

// readonly
export function readonly<T extends object>(
  target: T
): DeepReadonly<UnwrapNestedRefs<T>> {
  return createReactiveObject(
    target,
    true,
    readonlyHandlers,
    readonlyCollectionHandlers,
    readonlyMap
  )
}

// createReactiveObject
function createReactiveObject(
  target: Target,
  isReadonly: boolean,
  baseHandlers: ProxyHandler<any>,
  collectionHandlers: ProxyHandler<any>,
  proxyMap: WeakMap<Target, any>
) {
  if (!isObject(target)) {
    if (__DEV__) {
      console.warn(`value cannot be made reactive: ${String(target)}`)
    }
    return target
  }
  // target is already a Proxy, return it.
  // exception: calling readonly() on a reactive object
  if (
    target[ReactiveFlags.RAW] &&
    !(isReadonly && target[ReactiveFlags.IS_REACTIVE])
  ) {
    return target
  }
  // target already has corresponding Proxy
  const existingProxy = proxyMap.get(target)
  if (existingProxy) {
    return existingProxy
  }
  // only a whitelist of value types can be observed.
  const targetType = getTargetType(target)
  if (targetType === TargetType.INVALID) {
    return target
  }
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  proxyMap.set(target, proxy)
  return proxy
}
```

### baseHandlers



```ts


```

### effect


