---
title: 前端 | Vue2 初始化流程 - mount
date: 2021-8-7
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->

## mount

在进行编译操作后，我们得到了 render 函数，那么此时才算真正进入到挂载阶段

在编译篇章，先保留了原来定义的 $mount 方法，在编译后接着调用 $mount 方法实现挂载

先查询到挂载节点 dom ，再调用 mountComponent 进行挂载

```js
// platforms/web/runtime/index.js

// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined // 查找dom
  return mountComponent(this, el, hydrating) // 返回当前的vue实例
}
```

### `__beforeMount__`

### mountComponent

接受实例、dom 节点 参数

首先保存待挂载的节点 dom 在实例 `vm.$el` 上，再判断是否上一步进行编译后得到 render 函数, 若没有的话，创建一个空的虚拟节点赋值给 `$options.render`

再判断 `config.performance` 和 mark 是否存在，若存在，则更新组件的函数 updateComponent 会被分开几步骤写，各环节作以标记；若不存在，则直接调用 _update 来更新 _render 生成的虚拟节点至 dom；但该函数会被传入 Watcher，当触发数据读取时再执行

接下来创建的 Watcher 在这里起到两个作用，一个是初始化的时候会执行回调函数（标记 `this.getter` 为回调函数，后执行 `this.get()` 中执行回调）；另一个是当 vm 实例中的监测的数据发生变化的时候执行回调函数

最后判断 `vm.$vnode` 是否为父节点，若 $vnode 为 null 那说明为根节点初始化，那么就标记 _isMounted 为 true，再执行 mounted 钩子函数，挂载就完成了

```js
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el // 挂在组件时存储 $el
  if (!vm.$options.render) { // 如果人为配置没有传入的 render
    vm.$options.render = createEmptyVNode // 返回一个虚拟节点
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }
  callHook(vm, 'beforeMount')

  let updateComponent
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name
      const id = vm._uid
      const startTag = `vue-perf-start:${id}`
      const endTag = `vue-perf-end:${id}`

      mark(startTag)
      const vnode = vm._render() // 生成虚拟节点
      mark(endTag)
      measure(`vue ${name} render`, startTag, endTag)

      mark(startTag)
      vm._update(vnode, hydrating) // 更新虚拟节点至dom
      mark(endTag)
      measure(`vue ${name} patch`, startTag, endTag)
    }
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  // Watcher 在这里起到两个作用，一个是初始化的时候会执行回调函数（标记this.getter为回调函数，后执行this.get()中执行回调），
  // 另一个是当 vm 实例中的监测的数据发生变化的时候执行回调函数
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  // $vnode 为父节点，为 null 那说明为根节点初始化
  // 不是组件的初始化，而是外边用户 new Vue 的初始化过程
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

### Watcher

现在来具体看下 Watcher ， 先只看与上文有关的部分；在编译阶段转换 template 得到 render 函数后，创建 new Watcher 传入 updateComponent 函数用来初始化时挂载以及观测数据之后变化后进行对比修改再渲染工作。

new Watcher 时，传入了 vm 实例、updateComponent 函数、 noop 回调函数、options 中有 before 方法、作为标记的 isRenderWatcher 为 true

初始化 watcher 时，保留 vm ， 当前 isRenderWatcher 为 true ， 保留当前的 watcher 为 `vm._watcher` ，同时将当前的 watcher 添加进 vm._watchers 组

接着初始化一些配置，重点是判断 expOrFn 是否为函数，也就是我们初始化 RenderWatcher 时传入的 updateComponent 函数，当前该函数自然为函数，里边有我们重要的挂载环节

这时会令 `this.getter = expOrFn` ， this.value 则由于是新 watcher 实例，则会直接走 `this.get()` , 而该方法涉及着初始化时执行我们的 updateComponent 函数

可以往下看下我们的 get 方法，pushTarget 操作将当前的 watcher 实例入栈，而此时全局的 Dep.target 就成了当前的 watcher 对象

接着获取 value 的值，会通过 `this.getter.call(vm, vm)` 的形式来调用 updateComponent 函数，将该函数的 this 指向当前的 Vue 实例

下边介绍 _render 和 _update

```js
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn // updateComponent 函数
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = noop
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get() // 新实例直接执行
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm) // 初始化时会执行回调函数 updateComponent
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }
```

### _render

首先是执行 _render() 操作，用来生成当前 render 对应的 虚拟节点

获取 `vm.$options` 中之前编译完放入的 render 函数和当前实例组件对应的 _parentVnode 。 当前为根层，自然也就没有 _parentVnode，`vm.$vnode = _parentVnode` 也是同样的道理；但是对于子组件，这里的 _parentVnode 就会起作用了

接着创建一个 vnode ，令其等于 render 的执行结果；render 的 this 指向挂载的代理 _renderProxy，参数 `vm.$createElement` 是初始化时定义的

保存当前正在 render 的 vm，接着执行 render() 函数，先从子节点开始计算，将其中的变量在 data 或计算属性中读取，这是便触发了响应式系统，进行依赖收集，将当前变量的 dep 添加进 deps，将当前的 watcher 添加进 subs，返回当前变量的值；接着获取下一个变量的值

```js
(function anonymous(
) {
with(this){return _c('div',{attrs:{"id":"app"}},[_c('div',{class:bindClass,on:{"click":update}},[_v(_s(sum))]),_v(" "),_c('balance')],1)}
})
```

在我们当前例子里，首先会依次读取变量，_c , bindClass, _v, _s, update, sum 等，这里因为做了代理 `vm._renderProxy` ，在读取 vm 上的属性时，会触发 hasHandler 来检查 vm 上是否有该变量。

接着检测是否允许 globals ， 或者查找当前 vm 上的 $data，而在初始化 Vue 时，stateMixin 做了工作，给 vue 原型的属性 $data 进行了属性限定，在读取该属性时的 get 方法是返回 当前实例的 `this._data`，检测当前这个变量是否在 this._data 上，如两项有一项确定了，那么就返回当前变量。

首先是 _c ，会在 vm 下进行查找，也就是 $createElement 方法; 接着是 bindClass ，因为 proxy 做了代理，vm 下直接能获取到，同时也存在于 _data , hasHandler 返回为 true，则直接从 `this[_data][bindClass]` 返回值; 接着是 update，在 initMethods 时，就将定义的方法挂在了 vue 实例下; 再下来是 _v，位于 vue 的原型上，_s 同; 

然后是计算属性 sum ，之前初始化了解到，计算属性的 get 也是经过限定的，通过 computedGetter 函数，获取计算属性对应的 _computedWatcher ， 再确定当前的 Dep.target ，将当前计算属性对应的依赖收集器 dep 存入 deps 里，再判断已有的依赖里是否有当前的这个依赖收集器 dep ， 若无则将当前观察者 watcher 存入 subs，再返回计算属性 _computedWatcher （watcher）初始化时获得的 value，如此便完成了 get 步骤

接着将获得的 sum 通过 _s 转为字符串，再通过 _v 创建文本 vnode，sum 在例子中也是需要被刷出来的变量, 最后返回生成的 vnode

下边将开始通过 _c 创建虚拟节点, 从子节点先开始创建

```js
// core/instance/render.js

// 生成 vnode
Vue.prototype._render = function (): VNode {
  const vm: Component = this
  const { render, _parentVnode } = vm.$options

  if (_parentVnode) {
    vm.$scopedSlots = normalizeScopedSlots(
      _parentVnode.data.scopedSlots,
      vm.$slots,
      vm.$scopedSlots
    )
  }

  // set parent vnode. this allows render functions to have access
  // to the data on the placeholder node.
  vm.$vnode = _parentVnode
  // render self
  let vnode
  try {
    // There's no need to maintain a stack because all render fns are called
    // separately from one another. Nested component's render fns are called
    // when parent component is patched.
    currentRenderingInstance = vm
    // render的this指向挂载的代理，传入render函数的参数vm.$createElement是初始化时定义的，
    // 而render： h => h() 此时就为 vm.$createElement => vm.$createElement(vm, )
    vnode = render.call(vm._renderProxy, vm.$createElement)
  } catch (e) {
    handleError(e, vm, `render`)
    // return error render result,
    // or previous vnode to prevent render error causing blank component
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production' && vm.$options.renderError) {
      try {
        vnode = vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e)
      } catch (e) {
        handleError(e, vm, `renderError`)
        vnode = vm._vnode
      }
    } else {
      vnode = vm._vnode
    }
  } finally {
    currentRenderingInstance = null // 最后将正在render的实例对象重置
  }
  // if the returned array contains only a single node, allow it
  if (Array.isArray(vnode) && vnode.length === 1) {
    vnode = vnode[0]
  }
  // return empty vnode in case the render function errored out
  if (!(vnode instanceof VNode)) {
    if (process.env.NODE_ENV !== 'production' && Array.isArray(vnode)) {
      warn(
        'Multiple root nodes returned from render function. Render function ' +
        'should return a single root node.',
        vm
      )
    }
    vnode = createEmptyVNode()
  }
  // set parent
  vnode.parent = _parentVnode
  return vnode
}
```

#### createElement

首先从根节点下的子节点开始创建：

```js
_c('div',{class:bindClass,on:{"click":update}},[_v(_s(sum))])
```

createElement 作用是拦截判断当前被 _c 的节点对应的属性数据是否为数组或者原始值；以及判断 alwaysNormalize 是否为 true ， 判断完后再进行真正的 _createElement

_createElement 首先也是做数据校验，先判断 data 是否有定义，并且 data 存在自己的 `__ob__`, 如是则直接返回一个空的虚拟节点

接着判断 `data.is` 是否存在，是否有 tag（标签），再判断 `data.key` 是否存在，是为了避免使用非原始类型的值作为 key

接着是对 children 的判断，children 是第一个子节点内的节点，这里也就是 sum 对应的文本节点，已经生成为虚拟文本节点；先判断children 是否为数组，以及 children 第一个元素是否为函数；下边对 children 进行规范化，如果 normalizationType 存在

前置条件到此完成，接着进入创建 VNode 环节

按标签是否是字符串进行区分，不是字符串的那就是组件引入，直接调用 createComponent 创建一个组件类型的 VNode 节点

如果是字符串，继续进行划分；1)首先判断 tag 是否为内置的节点，也就是是否为传统 html 的 tag，或者 svg ；如果是，就解析这个平台的 tag ，创建虚拟节点。2)如不是则接着判断节点是否有属性，或者 pre，同时判断是否为已注册的组件名，在 `context.$options` 下的 components 中查找该标签，如是则通过 createComponent 创建一个组件类型的 VNode。3)如以上两种情况都不是，直接创建一个未知的标签的 VNode

那么给的例子是 div，自然走标签为字符串，且为内置标签的第一种情况，完成虚拟节点的创建

最后再判断生成的虚拟节点是否为数组形式，如是直接返回 vnode；接着判断 vnode 是否有定义，在有定义的基础上再判断 ns 是否有定义，如有则应用 ns，再判断节点的属性是否有定义，若有则检查是否有 deep binding 的变量，例如动态 class、style 等；完成之后返回 VNode，至此就创建完成了

然后继续给下一个节点生成 vnode

而我们例子中，第三个子节点是 balance ，这是个组件，下边也看下组件如何创建

```js
// core/vdom/create-element.js

// 封装 createElement 的接口，返回调用具体的创建步骤
export function createElement (
  context: Component,
  tag: any,
  data: any,
  children: any,
  normalizationType: any,
  alwaysNormalize: boolean
): VNode | Array<VNode> {
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children
    children = data
    data = undefined
  }
  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE
  }
  return _createElement(context, tag, data, children, normalizationType)
}

export function _createElement (
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: any,
  normalizationType?: number
): VNode | Array<VNode> {
  if (isDef(data) && isDef((data: any).__ob__)) {
    process.env.NODE_ENV !== 'production' && warn(
      `Avoid using observed data object as vnode data: ${JSON.stringify(data)}\n` +
      'Always create fresh vnode data objects in each render!',
      context
    )
    return createEmptyVNode()
  }
  // object syntax in v-bind
  if (isDef(data) && isDef(data.is)) {
    tag = data.is
  }
  if (!tag) {
    // in case of component :is set to falsy value
    return createEmptyVNode()
  }
  // warn against non-primitive key
  if (process.env.NODE_ENV !== 'production' &&
    isDef(data) && isDef(data.key) && !isPrimitive(data.key)
  ) {
    if (!__WEEX__ || !('@binding' in data.key)) {
      warn(
        'Avoid using non-primitive value as key, ' +
        'use string/number value instead.',
        context
      )
    }
  }
  // support single function children as default scoped slot
  if (Array.isArray(children) &&
    typeof children[0] === 'function'
  ) {
    data = data || {}
    data.scopedSlots = { default: children[0] }
    children.length = 0
  }
  // 规范化 children
  if (normalizationType === ALWAYS_NORMALIZE) { // 标准化-公共，1.render 函数是用户手写的；2.编译 slot、v-for 的时候会产生嵌套数组的情况
    children = normalizeChildren(children)
  } else if (normalizationType === SIMPLE_NORMALIZE) { // 简易标准化-内部， render 函数是编译生成的
    children = simpleNormalizeChildren(children)
  }
  // 创建 VNode
  let vnode, ns
  if (typeof tag === 'string') { // 标签为 string
    let Ctor
    ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag)
    // 判断是否为内置的一些节点
    if (config.isReservedTag(tag)) {
      // platform built-in elements
      if (process.env.NODE_ENV !== 'production' && isDef(data) && isDef(data.nativeOn)) {
        warn(
          `The .native modifier for v-on is only valid on components but it was used on <${tag}>.`,
          context
        )
      }
      // 如为内置的节点类型，创建一个普通 VNode
      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      )
    } else if ((!data || !data.pre) && isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
      // 如为已注册的组件名，则通过 createComponent 创建一个组件类型的 VNode
      // component
      vnode = createComponent(Ctor, data, context, children, tag)
    } else {
      // 创建一个未知的标签的 VNode
      // unknown or unlisted namespaced elements
      // check at runtime because it may get assigned a namespace when its
      // parent normalizes children
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      )
    }
  } else {
    // tag 不是 string 而是 Component 类型，直接调用 createComponent 创建一个组件类型的 VNode 节点
    // direct component options / constructor
    vnode = createComponent(tag, data, context, children)
  }
  if (Array.isArray(vnode)) {
    return vnode
  } else if (isDef(vnode)) {
    if (isDef(ns)) applyNS(vnode, ns)
    if (isDef(data)) registerDeepBindings(data)
    return vnode
  } else {
    return createEmptyVNode()
  }
}
```

#### createComponent

如果标签是已经注册过的组件，那么从 $options.component 取出我们之前配置的组件内容给 Ctor，进入 createComponent 开始创建组件。

倘若取到的 Ctor 是 undefined ， 那么直接返回。

接着通过 `context.$options._base` 取到当前实例在注入全局 API 时保留的构造函数 _base, 记为 baseCtor。然后使用 extend 将该组件扩展为 Vue 的一个子类 ：`baseCtor.extend(Ctor)`

构造完成后，Sub 又赋值给 Ctor ， 检测这时的 Ctor 是否为一个函数，不是则抛出警告；然后就是转换 model，提炼 props，

接着安装组件的钩子函数 init、prepatch、insert、destroy

创建一个 vnode，和普通元素节点的 vnode 不同，组件的 VNode 没有 children；而且配置第七项对应着 VNode 的属性 componentOptions，之后在组件实例化时会用到该配置。最后返回这个组件 vnode

```js
const vnode = new VNode(
  `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
  data, undefined, undefined, undefined, context,
  { Ctor, propsData, listeners, tag, children },
  asyncFactory
)
```

至此，children 里的三个字节点 _c 创建 vnode 的过程就完成了，接下来给根节点创建 vnode，返回最终的根 vnode

在 _render 里走到了 finally 步骤，将当前的 render 实例置空，判断 vnode 是否为数组，并且只有一个元素，若是则将 vnode 提出；再判断 vnode 是否为 VNode 的实例，若不是，则令 vnode 成为一个空的 vnode 节点

最后保存当前 vnode 的父级为 _parentVnode；返回 vnode；下一步将会走到 update

```js
// core/vdom/create-component.js

export function createComponent (
  Ctor: Class<Component> | Function | Object | void,
  data: ?VNodeData,
  context: Component,
  children: ?Array<VNode>,
  tag?: string
): VNode | Array<VNode> | void {
  if (isUndef(Ctor)) {
    return
  }

  // 实际 baseCtor 就是 Vue 构造函数
  const baseCtor = context.$options._base

  // plain options object: turn it into a constructor
  // extend 构造一个 Vue 的子类
  if (isObject(Ctor)) {
    Ctor = baseCtor.extend(Ctor)
  }

  // if at this stage it's not a constructor or an async component factory,
  // reject.
  if (typeof Ctor !== 'function') {
    if (process.env.NODE_ENV !== 'production') {
      warn(`Invalid Component definition: ${String(Ctor)}`, context)
    }
    return
  }

  // async component
  let asyncFactory
  if (isUndef(Ctor.cid)) {
    asyncFactory = Ctor
    Ctor = resolveAsyncComponent(asyncFactory, baseCtor)
    if (Ctor === undefined) {
      // return a placeholder node for async component, which is rendered
      // as a comment node but preserves all the raw information for the node.
      // the information will be used for async server-rendering and hydration.
      return createAsyncPlaceholder(
        asyncFactory,
        data,
        context,
        children,
        tag
      )
    }
  }

  data = data || {}

  // resolve constructor options in case global mixins are applied after
  // component constructor creation
  resolveConstructorOptions(Ctor)

  // transform component v-model data into props & events
  if (isDef(data.model)) {
    transformModel(Ctor.options, data)
  }

  // extract props
  const propsData = extractPropsFromVNodeData(data, Ctor, tag)

  // functional component
  if (isTrue(Ctor.options.functional)) {
    return createFunctionalComponent(Ctor, propsData, data, context, children)
  }

  // extract listeners, since these needs to be treated as
  // child component listeners instead of DOM listeners
  const listeners = data.on
  // replace with listeners with .native modifier
  // so it gets processed during parent component patch.
  data.on = data.nativeOn

  if (isTrue(Ctor.options.abstract)) {
    // abstract components do not keep anything
    // other than props & listeners & slot

    // work around flow
    const slot = data.slot
    data = {}
    if (slot) {
      data.slot = slot
    }
  }

  // install component management hooks onto the placeholder node
  // 安装组件钩子函数
  // 将 componentVNodeHooks 的钩子函数合并到 data.hook，在 VNode 执行 patch 时执行钩子函数
  installComponentHooks(data)

  // return a placeholder vnode
  const name = Ctor.options.name || tag
  // 实例化一个 vnode，和普通元素节点的 vnode 不同，组件的 VNode 没有 children
  const vnode = new VNode(
    `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
    data, undefined, undefined, undefined, context,
    { Ctor, propsData, listeners, tag, children },
    asyncFactory
  )

  // Weex specific: invoke recycle-list optimized @render function for
  // extracting cell-slot template.
  // https://github.com/Hanks10100/weex-native-directive/tree/master/component
  /* istanbul ignore if */
  if (__WEEX__ && isRecyclableComponent(vnode)) {
    return renderRecyclableComponentTemplate(vnode)
  }

  return vnode
}
```

##### extend

在创建组件时，会将组件 Ctor 扩展成 Vue 的子构造器，得到一个子实例，因为 Vue 实例本身也是一个组件。在获得 Vue 的方法继承后，就可以按 Vue 初始化程序来初始化当前的组件 Ctor

extend 使用一种非常经典的原型继承的方式把一个纯对象转换一个继承于 Vue 的构造器 Sub 并返回，然后对 Sub 这个对象本身扩展了一些属性，如扩展 options、添加全局 API 等；并且对配置中的 props 和 computed 做了初始化工作；


来看 extend 的运行

传入的参数是 extendOptions ，也就是 Ctor，是我们外界对这个组件的定义，包括它的模板、方法、数据等的一个对象；然后令 Vue 等于 Super，也就是 Vue 作为高级构造器，接着从缓存里找是否有基于 Vue 的子构造函数，若之前有建过，直接返回。

接下来获取组件 Ctor 的名字，检查命名是否有效；

创建组件的构造器 Sub，同 Vue 实际是一样的，里边有一个 init 步骤，之后在合并配置时也会检测当前是否时组件初始化，进而运行 `initInternalComponent(vm, options)` ；定义了 Sub 之后，接着给 Sub 添加 Vue 的原型，改变原型的构造函数为 Sub；随后创建 cid，合并 Vue 和 我们传入的 extendOptions 为 Sub 的最终 options；

给 Sub 添加 super 属性为 Super，初始化 Sub 配置中的 props、computed

继续给 Sub 增加属性 extend、mixin、use 方法；以及 component、filter、directive 属性；如果 组件有命名，令 `Sub.options.components` 下添加当前组件

接着在 Sub 的属性上保存 Super 的 options 为 superOptions，Sub 从外界传入的 extendOptions，Sub 合并后的 options 为 sealedOptions

最后缓存当前的 Sub ，`cachedCtors[SuperId] = Sub`，用当前 Vue 的 cid 标记；返回 Sub

```js
// core/global-api/extend.js

Vue.extend = function (extendOptions: Object): Function {
  extendOptions = extendOptions || {}
  const Super = this
  const SuperId = Super.cid
  const cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {})
  if (cachedCtors[SuperId]) { // 从缓存中找 sub
    return cachedCtors[SuperId]
  }

  const name = extendOptions.name || Super.options.name
  if (process.env.NODE_ENV !== 'production' && name) {
    validateComponentName(name)
  }

  const Sub = function VueComponent (options) { // 定义构造器
    this._init(options)
  }
  Sub.prototype = Object.create(Super.prototype) // 原型继承
  Sub.prototype.constructor = Sub // 构造函数为其本身
  Sub.cid = cid++
  Sub.options = mergeOptions(
    Super.options, // 父级 Vue 构造函数的一些配置
    extendOptions // 子级
  )
  Sub['super'] = Super

  // For props and computed properties, we define the proxy getters on
  // the Vue instances at extension time, on the extended prototype. This
  // avoids Object.defineProperty calls for each instance created.
  if (Sub.options.props) {
    initProps(Sub)
  }
  if (Sub.options.computed) {
    initComputed(Sub)
  }

  // allow further extension/mixin/plugin usage
  Sub.extend = Super.extend
  Sub.mixin = Super.mixin
  Sub.use = Super.use

  // create asset registers, so extended classes
  // can have their private assets too.
  ASSET_TYPES.forEach(function (type) {
    Sub[type] = Super[type]
  })
  // enable recursive self-lookup
  if (name) {
    Sub.options.components[name] = Sub
  }

  // keep a reference to the super options at extension time.
  // later at instantiation we can check if Super's options have
  // been updated.
  Sub.superOptions = Super.options
  Sub.extendOptions = extendOptions
  Sub.sealedOptions = extend({}, Sub.options)

  // cache constructor
  cachedCtors[SuperId] = Sub
  return Sub
}
```

### _update

_update 起的作用是将 vnode 生成 dom 节点

将当前的 `vm.$el` 存为 prevEl，将 `vm._vnode` 存为 prevVnode，然后设定当前活跃的实例为 vm，在完成挂载后，会释放掉当前活跃的实例

保存当前的 vnode 为 `vm._vnode`，然后判断 prevVnode 是否存在，如不存在先前节点，说明为首次挂载，如有则做更新操作

这里走的是 initial render 操作

```js
// core/instance/lifecycle.js

Vue.prototype._update = function (vnode: VNode, hydrating?: oolean) {
  const vm: Component = this
  const prevEl = vm.$el // 将当前的元素节点存为前一个
  const prevVnode = vm._vnode
  const restoreActiveInstance = setActiveInstance(vm)
  vm._vnode = vnode // vm._render() 返回的组件渲染 VNode
  // Vue.prototype.__patch__ is injected in entry points
  // based on the rendering backend used.
  if (!prevVnode) { // 如不存在先前节点，说明为首次挂载，如有则做更新
    // initial render
    vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
  } else {
    // updates
    vm.$el = vm.__patch__(prevVnode, vnode) // 前两个参数分别为待替换节点、用来替换的节点；其后均为可选参数，首次传入
  }
  restoreActiveInstance() // 当前 vm 完成子组件的 patch 后恢复活动实例至父实例
  // update __vue__ reference
  if (prevEl) { // 如有前节点，去掉
    prevEl.__vue__ = null
  }
  if (vm.$el) { // 
    vm.$el.__vue__ = vm
  }
  // if parent is an HOC, update its $el as well
  if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
    vm.$parent.$el = vm.$el
  }
  // updated hook is called by the scheduler to ensure that children are
  // updated in a parent's updated hook.
}

```

#### patch

接着来看下 patch 方法，patch 方法在引入 Vue 时就已经初始化好了，

```js
// platforms/web/runtime/index.js

import { patch } from './patch'

Vue.prototype.__patch__ = inBrowser ? patch : noop
```

再来看一下 patch 文件, 可见 patch 来源于 core/vdom/patch

```js
// platforms/web/runtime/patch.js

import * as nodeOps from 'web/runtime/node-ops'
import { createPatchFunction } from 'core/vdom/patch'
import baseModules from 'core/vdom/modules/index'
import platformModules from 'web/runtime/modules/index'

// the directive module should be applied last, after all
// built-in modules have been applied.
// 将虚拟节点下的指令模块（增删更节点、属性等）打包
const modules = platformModules.concat(baseModules)

export const patch: Function = createPatchFunction({ nodeOps, modules })
```

来看 `core/vdom/patch.js` , createPatchFunction 函数里定义了一系列函数，最后返回 patch 这个函数，这样就找到了我们进程中的 patch 步骤的来源。

再来看 _update 中初始化打包是如何进行的

`vm.__patch__(vm.$el, vnode, hydrating, false)`

patch 的参数，初始化时第一个参数为当前待挂载的 dom 节点，第二个为整理好的根虚拟节点；而更新时第一个传入的参数是先前的 vnode，第二个则为新的 vnode，两者便会进行对比，在有改动的地方上进行修改。

进入到 patch 里边，首先检测 vnode 是否定义。再做一个 isInitialPatch 判断标记。

接着判断 oldVnode 是否定义，这里也就是 el dom 节点，是有定义的；

然后判断给出的 oldVnode 是否为真实的节点，若不是真实的节点，且新旧节点相同，那么就走更新的路线；若为真实节点，将真实节点转化成空的虚拟节点。然后缓存 oldNode 的 dom 节点以及它的父 dom 节点到 oldElm、parentElm

接下来将虚拟节点转换成 dom 节点，通过 createElm 。

```js
// core/vdom/patch.js 

export function createPatchFunction (backend) {
  let i, j
  const cbs = {}

  const { modules, nodeOps } = backend

  for (i = 0; i < hooks.length; ++i) {
    cbs[hooks[i]] = []
    for (j = 0; j < modules.length; ++j) {
      if (isDef(modules[j][hooks[i]])) {
        cbs[hooks[i]].push(modules[j][hooks[i]])
      }
    }
  }

  // ...

  return function patch (oldVnode, vnode, hydrating, removeOnly) {
    if (isUndef(vnode)) {
      if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
      return
    }

    let isInitialPatch = false // 初始打包
    const insertedVnodeQueue = []

    if (isUndef(oldVnode)) { // 旧节点未被定义
      // empty mount (likely as component), create new root element
      isInitialPatch = true
      createElm(vnode, insertedVnodeQueue)
    } else { // 旧节点已被定义
      const isRealElement = isDef(oldVnode.nodeType)
      // 判断旧节点类型是否为真的dom元素节点，且新老节点为同一节点，直接进行比对 patchVnode
      if (!isRealElement && sameVnode(oldVnode, vnode)) {
        // patch existing root node
        patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
      } else {
        if (isRealElement) { // 如为真实节点，将真实节点转化成虚拟节点
          // mounting to a real element
          // check if this is server-rendered content and if we can perform
          // a successful hydration.
          if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
            oldVnode.removeAttribute(SSR_ATTR)
            hydrating = true
          }
          if (isTrue(hydrating)) {
            if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
              invokeInsertHook(vnode, insertedVnodeQueue, true)
              return oldVnode
            } else if (process.env.NODE_ENV !== 'production') {
              warn(
                'The client-side rendered virtual DOM tree is not matching ' +
                'server-rendered content. This is likely caused by incorrect ' +
                'HTML markup, for example nesting block-level elements inside ' +
                '<p>, or missing <tbody>. Bailing hydration and performing ' +
                'full client-side render.'
              )
            }
          }
          // either not server-rendered, or hydration failed.
          // create an empty node and replace it
          oldVnode = emptyNodeAt(oldVnode) // 将真是节点转化为空的虚拟节点
        }

        // replacing existing element
        const oldElm = oldVnode.elm
        const parentElm = nodeOps.parentNode(oldElm)

        // create new node
        // 创建新的节点，通过传入的虚拟节点创建真实的 DOM 并插入到它的父节点中
        createElm(
          vnode,
          insertedVnodeQueue,
          // extremely rare edge case: do not insert if old element is in a
          // leaving transition. Only happens when combining transition +
          // keep-alive + HOCs. (#4590)
          oldElm._leaveCb ? null : parentElm,
          nodeOps.nextSibling(oldElm)
        )

        // update parent placeholder node element, recursively
        if (isDef(vnode.parent)) {
          let ancestor = vnode.parent
          const patchable = isPatchable(vnode)
          while (ancestor) {
            for (let i = 0; i < cbs.destroy.length; ++i) {
              cbs.destroy[i](ancestor)
            }
            ancestor.elm = vnode.elm
            if (patchable) {
              for (let i = 0; i < cbs.create.length; ++i) {
                cbs.create[i](emptyNode, ancestor)
              }
              // #6513
              // invoke insert hooks that may have been merged by create hooks.
              // e.g. for directives that uses the "inserted" hook.
              const insert = ancestor.data.hook.insert
              if (insert.merged) {
                // start at index 1 to avoid re-invoking component mounted hook
                for (let i = 1; i < insert.fns.length; i++) {
                  insert.fns[i]()
                }
              }
            } else {
              registerRef(ancestor)
            }
            ancestor = ancestor.parent
          }
        }

        // destroy old node
        if (isDef(parentElm)) {
          removeVnodes([oldVnode], 0, 0)
        } else if (isDef(oldVnode.tag)) {
          invokeDestroyHook(oldVnode)
        }
      }
    }

    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
    return vnode.elm
  }
}
```

#### createElm

createElm 函数传入参数为之前准备好的 vnode 节点；insertedVnodeQueue，父 dom 节点用以插入当前节点；refElm 是当前节点同一级的下一个 nextSibling 节点 

接着检查 vnode.elm、ownerArray` 是否有定义，`vnode.elm` 是当前虚拟节点转换成的 dom 节点，ownerArray 是当前是 children 里的 vnode 创建 dom 节点是带过来的 children 参数

试创建创建组件 createComponent ，看返回值是否为 true，如为 true 直接返回，那么说明当前 vnode 节点也是个组件; 如果不为 true，那么继续向下执行

判断当前 vnode 的 tag 是否有定义，如果是不知名的 tag ，那么就抛出警告；或者当前 vnode 是注释节点，那么就直接创建注释节点 createComment，再执行插入；要么就是当前是文本节点，就直接创建文本节点就行，再做插入

接着操作 dom 创建对应 tag 的元素节点，若该 tag 不等于 select ，就直接返回创建好的元素节点

设置 vnode 的 scope ，对于有配置 scoped 的 css、slot 来说

接着创建 vnode 的 children ，将它们转换为 dom 节点；创建完接着激活创建的钩子函数，执行所有的 `cbs.create` 时的回调函数，将 vnode 的属性、class、事件、props、style 等都更新到 `vnode.elm`上，也就是 vnode 对应的 dom 节点，这时 属性、事件、class 等都同步上去了。

然后再把 dom 插入父节点下

在 createElm 后，返回递归转 dom 的根节点 `vnode.elm` 赋值给 $el

```js
// cbs.create
0:ƒ updateAttrs(oldVnode, vnode)
1:ƒ updateClass (oldVnode, vnode)
2:ƒ updateDOMListeners (oldVnode, vnode)
3:ƒ updateDOMProps(oldVnode, vnode)
4:ƒ updateStyle(oldVnode, vnode)
5:ƒ _enter (_, vnode)
6:ƒ create (_, vnode) {\n      registerRef(vnode);\n    }
7:ƒ updateDirectives (oldVnode, vnode)
[[Prototype]]:Object
```

```js
// core/vdom/patch.js

function createElm (
  vnode,
  insertedVnodeQueue,
  parentElm,
  refElm,
  nested,
  ownerArray,
  index
) {
  if (isDef(vnode.elm) && isDef(ownerArray)) {
    // This vnode was used in a previous render!
    // now it's used as a new node, overwriting its elm would cause
    // potential patch errors down the road when it's used as an insertion
    // reference node. Instead, we clone the node on-demand before creating
    // associated DOM element for it.
    vnode = ownerArray[index] = cloneVNode(vnode)
  }

  vnode.isRootInsert = !nested // for transition enter check
  // 如果组件的根节点是个普通元素，那么 vm._vnode 也是普通的 vnode
  if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
    return
  }

  const data = vnode.data // VNode 的 data
  const children = vnode.children
  const tag = vnode.tag
  if (isDef(tag)) {
    if (process.env.NODE_ENV !== 'production') {
      if (data && data.pre) {
        creatingElmInVPre++
      }
      if (isUnknownElement(vnode, creatingElmInVPre)) {
        warn(
          'Unknown custom element: <' + tag + '> - did you ' +
          'register the component correctly? For recursive components, ' +
          'make sure to provide the "name" option.',
          vnode.context
        )
      }
    }

    vnode.elm = vnode.ns
      ? nodeOps.createElementNS(vnode.ns, tag)
      : nodeOps.createElement(tag, vnode)
    setScope(vnode)

    /* istanbul ignore if */
    if (__WEEX__) {
      // in Weex, the default insertion order is parent-first.
      // List items can be optimized to use children-first insertion
      // with append="tree".
      const appendAsTree = isDef(data) && isTrue(data.appendAsTree)
      if (!appendAsTree) {
        if (isDef(data)) {
          invokeCreateHooks(vnode, insertedVnodeQueue)
        }
        insert(parentElm, vnode.elm, refElm)
      }
      createChildren(vnode, children, insertedVnodeQueue)
      if (appendAsTree) {
        if (isDef(data)) {
          invokeCreateHooks(vnode, insertedVnodeQueue)
        }
        insert(parentElm, vnode.elm, refElm)
      }
    } else {
      createChildren(vnode, children, insertedVnodeQueue) // 遍历子虚拟节点，递归调用 createElm，把 vnode.elm 作为父容器的 DOM 节点占位符传入。
      if (isDef(data)) {
        invokeCreateHooks(vnode, insertedVnodeQueue) // 执行所有的 create 的钩子并把 vnode push 到 insertedVnodeQueue 中。
      }
      insert(parentElm, vnode.elm, refElm) // 把 DOM 插入到父节点中
    }

    if (process.env.NODE_ENV !== 'production' && data && data.pre) {
      creatingElmInVPre--
    }
  } else if (isTrue(vnode.isComment)) {
    vnode.elm = nodeOps.createComment(vnode.text)
    insert(parentElm, vnode.elm, refElm)
  } else {
    vnode.elm = nodeOps.createTextNode(vnode.text)
    insert(parentElm, vnode.elm, refElm)
  }
}
```

##### createComponent

获取当前 vnode 的 data，再判断 data 下是否存在 hook，并且 hook 中有 init，那说明当前 vnode 为组件虚拟节点，在 render 创建虚拟节点时期，有给组件安装 hook，`installComponentHooks(data)`；而此时能获取到 hook.init ，那说明当前就是组件虚拟节点转 dom 节点的过程，接着用 init 狗子函数初始化当前的组件 vnode，因为组件实例 Sub 被定义了但还没有初始化展开。

```js
// core/vdom/patch.js

function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
  let i = vnode.data
  if (isDef(i)) {
    const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
    // i 就为 init 钩子函数，初始化子组件
    if (isDef(i = i.hook) && isDef(i = i.init)) {
      i(vnode, false /* hydrating */) // init(vnode, false)
    }
    // after calling the init hook, if the vnode is a child component
    // it should've created a child instance and mounted it. the child
    // component also has set the placeholder vnode's elm.
    // in that case we can just return the element and be done.
    if (isDef(vnode.componentInstance)) {
      initComponent(vnode, insertedVnodeQueue)
      insert(parentElm, vnode.elm, refElm)
      if (isTrue(isReactivated)) {
        reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm)
      }
      return true
    }
  }
}
```

###### init

在组件虚拟节点转 dom 节点时，组件初始化。

`createComponentInstanceForVnode(vnode, activeInstance)`

获取当前活跃的 vue 实例，设置 _isComponent 为 true，这样在初始化合并配置环境就直接走组件流程；`_parentVnode` 记为当前的 vnode，parent 也就是传入的 vue 实例

最后调用 vnode.componentOptions.Ctor ，也就是组件创建时，在 extend 里创建里组件的函数 Sub，同 Vue 函数一样，它用来 new 一个组件实例，传入配置 options 开始创建实例。

此时 Sub 调用的 _init 与 Vue 的一样，是原型上的方法，而 Sub 等于了 Vue的原型，只是调整了构造函数。

初始化到合并 options 阶段，会检查 `options && options._isComponent` 是否为 true，之后 `initInternalComponent(vm, options)` 初始化实例内部组件

记组件构造函数的 options 为 `vm.$options`， `options._parentVnode` 为 parentVnode, 接着又将 parentVnode 赋值给 `vm.$options._parentVnode` ，这里流转的 parentVnode 也就是当前组件的 vnode。将初始化传入的 options 都过继到 vm.$options。再保留 render 所需的 render、staticRenderFns；然后就是继续走 vue 初始化走过的路

最后将组件实例记为 child ，先进行编译，获取当前组件的 template，判断 template 是否为 string 。接着继续生成 AST 树，再转为代码，再将代码生成函数。获得当前的 render 函数，接着进入 mountComponent 环节，开始挂载。

继续给组件也创建 Watcher，进行初始化，执行 updateComponent 回调，进入到 _render 创建虚拟节点，创建完成后进入到 _update，开始打包环节。如此递归直到当前节点没有 children

```js
// core/vdom/create-component.js

init (vnode: VNodeWithData, hydrating: boolean): ?boolean {
  if (
    vnode.componentInstance &&
    !vnode.componentInstance._isDestroyed &&
    vnode.data.keepAlive
  ) {
    // kept-alive components, treat as a patch
    const mountedNode: any = vnode // work around flow
    componentVNodeHooks.prepatch(mountedNode, mountedNode)
  } else {
    // 创建 Vue 实例
    // activeInstance 从 lifecycle 获取，指当前上下文的 vue 实例，在此作为父实例传入函数
    const child = vnode.componentInstance = createComponentInstanceForVnode(
      vnode,
      activeInstance
    )
    child.$mount(hydrating ? vnode.elm : undefined, hydrating) // 挂载组件
  }
}

// createComponentInstanceForVnode
export function createComponentInstanceForVnode (
  // we know it's MountedComponentVNode but flow doesn't
  vnode: any,
  // activeInstance in lifecycle state
  parent: any
): Component {
  const options: InternalComponentOptions = {
    _isComponent: true, // 表示为组件，通过创建子组件来的
    _parentVnode: vnode,
    parent
  }
  // check inline-template render functions
  const inlineTemplate = vnode.data.inlineTemplate
  if (isDef(inlineTemplate)) {
    options.render = inlineTemplate.render
    options.staticRenderFns = inlineTemplate.staticRenderFns
  }
  return new vnode.componentOptions.Ctor(options) // new Sub(options) 实为继承了 Vue 的子组件构造函数
}

// vnode.componentOptions.Ctor = Sub
const Sub = function VueComponent (options) { // 定义构造器
  this._init(options)
}

// initInternalComponent
export function initInternalComponent (vm: Component, options: InternalComponentOptions) {
  const opts = vm.$options = Object.create(vm.constructor.options) // 组件构造函数的配置
  // doing this because it's faster than dynamic enumeration.
  const parentVnode = options._parentVnode
  opts.parent = options.parent
  opts._parentVnode = parentVnode

  const vnodeComponentOptions = parentVnode.componentOptions
  opts.propsData = vnodeComponentOptions.propsData
  opts._parentListeners = vnodeComponentOptions.listeners
  opts._renderChildren = vnodeComponentOptions.children
  opts._componentTag = vnodeComponentOptions.tag

  if (options.render) {
    opts.render = options.render
    opts.staticRenderFns = options.staticRenderFns
  }
}

```

##### createElement

创建元素节点

```js
// platforms/web/runtime/node-ops.js

export function createElement (tagName: string, vnode: VNode): Element {
  const elm = document.createElement(tagName)
  if (tagName !== 'select') {
    return elm
  }
  // false or null will remove the attribute but undefined will not
  if (vnode.data && vnode.data.attrs && vnode.data.attrs.multiple !== undefined) {
    elm.setAttribute('multiple', 'multiple')
  }
  return elm
}
```

##### createTextNode

创建文本节点

```js
export function createTextNode (text: string): Text {
  return document.createTextNode(text)
}
```

##### createComment

创建注释节点

```js
export function createComment (text: string): Comment {
  return document.createComment(text)
}
```

##### createChildren

创建当前 vnode 的子节点，将它们转换为 dom 节点

检查 children 是否为数组格式，如果是再检查是否有重复的 key；接着遍历 children，给其中每个 vnode 都执行 createElm，将它们转换为 dom 节点，而这时的 vnode.elm 也就成了上层根节点；如果 children 不为数组格式，检测 vnode.text 是否为原始值，如是则直接创建文本节点执行插入。

```js
function createChildren (vnode, children, insertedVnodeQueue) {
  if (Array.isArray(children)) {
    if (process.env.NODE_ENV !== 'production') {
      checkDuplicateKeys(children)
    }
    for (let i = 0; i < children.length; ++i) {
      createElm(children[i], insertedVnodeQueue, vnode.elm, null, true, children, i)
    }
  } else if (isPrimitive(vnode.text)) {
    nodeOps.appendChild(vnode.elm, nodeOps.createTextNode(String(vnode.text)))
  }
}
```

这个例子套的太深了，，打算重写个看下 v-model , watch , v-if , v-for, 及事件