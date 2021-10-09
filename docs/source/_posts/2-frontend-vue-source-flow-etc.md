---
title: 前端 | Vue2 初始化流程 - 补充：其他功能的实现
date: 2021-8-12
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->

# 例子


hmtl

```html
<div id="app">
    <!-- v-if -->
    <button @click="switchShow">开关</button>
    <div v-if="show">$[ show ? 'bad ass' : 'motherfucker' ]</div>
    <!-- v-for -->
    <ul>
        <li v-for="food in foodList" :key="food.name">$[food.name]：￥ $[food.price]</li>
    </ul>
    <!-- v-model -->
    <div><input v-model="text" /></div>
    <!-- v-bind & event -->
    <div :class="bindClass" @click="update">$[sum]</div>
</div>
```

js

```js
let app = new Vue({
  el: '#app',
  delimiters: ['$[', ']'],
  watch: {
    sum(val) {
      console.log('the newVal of sum is' + val)
    }
  },
  data() {
    return {
      bindClass: 'date',
      base: 10,
      arr: ['num1', {'num2': 2}, ['num3', 3]],
      show: true,
      foodList: [{
          name: '葱',
          price: '10',
          discount: .8
        },
        {
          name: '姜',
          price: '8',
          discount: .6
        },
        {
          name: '蒜',
          price: '8',
          discount: null
        }
      ],
      logo: '查询菜单',
      text: ''
    }
  },
  computed: {
    sum() {
      return this.base * 10
    }
  },
  methods: {
    update() {
      this.base += 10
    },
    switchShow() {
      this.show = !this.show
      console.log(this.show ? '打开面板' : '关闭面板')
    }
  }
})
```

在 compile 阶段，compileToFunctions -> baseCompile -> compile -> parse 进行解析 html, 生成 AST 树。

总结一下，compileToFunctions 函数是 createCompilerCreator(baseCompile) 返回的 CreateCompiler(baseOptions) 执行返回的对象 { complie, createCompileToFunctionFn(compile) } 中提取的；

而 createCompileToFunctionFn(compile) 中由 compile(template, options) 进行编译，compile 中调用 baseCompile(template, finalOptions) 执行关键步骤：parse, optimize, generate 得到最后的 render 函数

## parse

parse 完成后, 我们的 AST 树如下, 其实逻辑也很清晰, 这个阶段完成的也是基本的准备工作，将节点按父子级排列出来, 每个节点都有自己对应的 attrsList、attrsMap、end、parent、plain、rawAttrsMap、start、tag、type , 表示当前节点的一些基本内容，属性也只是作以读取储存。

对于 vue 特定的一些属性指令作以特殊处理, 以 v- : @ 开头的属性均会添加 `hasBindings: true` 属性; 对于计算属性, 会添加 classBinding; 对于事件, 会添加 events 属性描述; 对于 v-if 指令, 会添加 if, ifConditions 属性; 对于 v-for , 会添加 for key 来描述; 对于 v-model , 会添加 directives 来描述指令

其实 parse 阶段也就是解析我们写的 html 为一个 Abstract Syntax Tree, 将写的一堆的东西按嵌套层作以归纳.

```js
ast: {
  attrs: Array(1)
    0: {name: "id", value: "\"app\"", dynamic: undefined, start: 5, end: 13}
  attrsList: Array(1)
    0: {name: "id", value: "app", start: 5, end: 13}
  attrsMap: {id: "app"}
  children: Array(9)
    0: {
      attrsList: Array(1)
        0: {name: "@click", value: "switchShow", start: 53, end: 72}
      attrsMap: {@click: "switchShow"}
        0: {type: 3, text: "开关", start: 73, end: 75}
      end: 84
      events:
        click: {value: "switchShow", dynamic: false, start: 53, end: 72}
      hasBindings: true
      parent: {type: 1, tag: "div", attrsList: Array(1), attrsMap: {…}, rawAttrsMap: {…}, …}
      plain: false
      rawAttrsMap: {@click: {…}}
      start: 45
      tag: "button"
      type: 1
    }
    1: {type: 3, text: " ", start: 84, end: 93}
    2: {
      attrsList: Array(0)
      length: 0
      attrsMap: {v-if: "show"}
      children: Array(1)
        0: {type: 2, expression: "_s(show ? 'bad ass' : 'motherfucker')", tokens: Array(1), text: "$[ show ? 'bad ass' : 'motherfucker' ]", start: 110, …}
      end: 154
      if: "show"
      ifConditions: Array(1)
        0: {exp: "show", block: {…}}
      parent: {type: 1, tag: "div", attrsList: Array(1), attrsMap: {…}, rawAttrsMap: {…}, …}
      plain: true
      rawAttrsMap: {v-if: {…}}
      start: 93
      tag: "div"
      type: 1
    }
  3: {type: 3, text: " ", start: 154, end: 163}
  4: {
    attrsList: []
    attrsMap: {}
    children: Array(1)
      0: {
        alias: "food"
        attrsList: []
        attrsMap: {v-for: "food in foodList", :key: "food.name"}
        children: Array(1)
          0: {type: 2, expression: "_s(food.name)+\"：￥ \"+_s(food.price)", tokens: Array(3), text: "$[food.name]：￥ $[food.price]", start: 249, …}
        end: 282
        for: "foodList"
        key: "food.name"
        parent: {type: 1, tag: "ul", attrsList: Array(0), attrsMap: {…}, rawAttrsMap: {…}, …}
        plain: false
        rawAttrsMap: {v-for: {…}, :key: {…}}
        start: 203
        tag: "li"
        type: 1
      }
    end: 296
    parent: {type: 1, tag: "div", attrsList: Array(1), attrsMap: {…}, rawAttrsMap: {…}, …}
    plain: true
    rawAttrsMap: {}
    start: 186
    tag: "ul"
    type: 1
  }
  5: {type: 3, text: " ", start: 296, end: 305}
  6: {
    attrsList: []
    attrsMap: {}
    children: Array(1)
      0: {
        attrsList: Array(1)
          0: {name: "v-model", value: "text", start: 342, end: 356}
        length: 1
        attrsMap: {v-model: "text"}
        children: []
        directives: Array(1)
          0: {name: "model", rawName: "v-model", value: "text", arg: null, isDynamicArg: false, …}
        length: 1
        end: 357
        hasBindings: true
        parent: {type: 1, tag: "div", attrsList: Array(0), attrsMap: {…}, rawAttrsMap: {…}, …}
        plain: false
        rawAttrsMap: {v-model: {…}}
        start: 335
        tag: "input"
        type: 1
      }
    end: 363
    parent: {type: 1, tag: "div", attrsList: Array(1), attrsMap: {…}, rawAttrsMap: {…}, …}
    plain: true
    rawAttrsMap: {}
    start: 330
    tag: "div"
    type: 1
  }
  7: {type: 3, text: " ", start: 363, end: 372}
  8: {
    attrsList: Array(1)
      0: {name: "@click", value: "update", start: 428, end: 443}
    attrsMap: {:class: "bindClass", @click: "update"}
    children: Array(1)
      0: {type: 2, expression: "_s(sum)", tokens: Array(1), text: "$[sum]", start: 444, …}
    classBinding: "bindClass"
    end: 456
    events:
      click: {value: "update", dynamic: false, start: 428, end: 443}
    hasBindings: true
    parent: {type: 1, tag: "div", attrsList: Array(1), attrsMap: {…}, rawAttrsMap: {…}, …}
    plain: false
    rawAttrsMap: {:class: {…}, @click: {…}}
    start: 404
    tag: "div"
    type: 1
  }
  length: 9
  end: 467
  parent: undefined
  plain: false
  rawAttrsMap: {id: {…}}
  start: 0
  tag: "div"
  type: 1
  }
```

## generate

将上边的 AST 转换为代码

从根节点开始, genElement, 在其内会判断节点上是否有各类属性, 根节点就一个 id, 所以会进入 genAttrs -> genProps, 接着判断属性是动态还是静态, 这里的 id 就直接写入 `"${prop.name}":${value},`, 以此就获得了根节点的代码; 

接着会读取根节点下 children, 使用 genChildren 生成子节点的代码, 在其内用 map 遍历 children, 每个 child 都使用 genNode 来创建代码; 根据当前节点的类型, 看是元素节点1, 还是注释节点2, 或者是文本节点3, 分别采用 genElement genComment genText 来创建对应的代码, 最后再拼接在一起.

```js
'with(this){return _c('div',{attrs:{"id":"app"}},[_c('button',{on:{"click":switchShow}},[_v("开关")]),_v(" "),(show)?_c('div',[_v(_s(show ? 'bad ass' : 'motherfucker'))]):_e(),_v(" "),_c('ul',_l((foodList),function(food){return _c('li',{key:food.name},[_v(_s(food.name)+"：￥ "+_s(food.price))])}),0),_v(" "),_c('div',[_c('input',{directives:[{name:"model",rawName:"v-model",value:(text),expression:"text"}],domProps:{"value":(text)},on:{"input":function($event){if($event.target.composing)return;text=$event.target.value}}})]),_v(" "),_c('div',{class:bindClass,on:{"click":update}},[_v(_s(sum))])])}'
```

继续将代码转成 render 函数返回, 也是涉及 _c, _v, _e, _l, _s 等工作, 在之后执行到 updateComponent 时, 调用 render 函数执行代码生成 VNode

```js
ƒ anonymous(
) {
  with(this){
    return _c('div',{
      attrs:{"id":"app"}},[
        _c('button',{on:{"click":switchShow}},[_v("开关")]), // event
        _v(" "),
        (show)?_c('div',[_v(_s(show ? 'bad ass' : 'motherfucker'))]):_e(), // v-if
        _v(" "),
        _c('ul',_l((foodList),function(food){
          return _c('li',{key:food.name},[
            _v(_s(food.name)+"：￥ "+_s(food.price))
          ])
        }),0), // v-for
        _v(" "),
        _c('div',[_c('input',{
          directives:[{name:"model",rawName:"v-model",value:(text),expression:"text"}],
          domProps:{"value":(text)},
          on:{
            "input":function($event){if($event.target.composing)return;text=$event.target.value}
          }})]
        ), // v-model
        _v(" "),
        _c('div',{class:bindClass,on:{"click":update}},[_v(_s(sum))])])} // v-bind & event
}
```

## render

因为 with(this) 的作用, 将当前代码块处于 vm._renderProxy 作用域下, 这时读取代码块中的变量就会在该作用域下进行查找, 而 vm 做了代理, 会使得在获取变量(也就是 vm 下的属性) 时会触发 hasHandler 进行检测 vm 当前是否有这个属性;

从根节点的 children 开始, 依次将 child 虚拟节点化; 从外到里获取变量, 从里到外执行函数, 比如第一个 child, 就是先获取 _c(之前还会获取一次根的), switchShow, _v; 然后接着从 _v 执行, 创建虚拟文本节点, 接着执行 _c, 创建元素节点

```js
// 第一段 事件
_c('button',{on:{"click":switchShow}},[_v("开关")])

// 第三段 v-if
(show)?_c('div',[_v(_s(show ? 'bad ass' : 'motherfucker'))]):_e()

// 第五段 v-for
_c('ul',_l((foodList),function(food){
  return _c('li',{key:food.name},[
    _v(_s(food.name)+"：￥ "+_s(food.price))
  ])
}),0)

// 第七段 v-model
_c('div',[_c('input',{
  directives:[{name:"model",rawName:"v-model",value:(text),expression:"text"}],
  domProps:{"value":(text)},
  on:{
    "input":function($event){if($event.target.composing)return;text=$event.target.value}
  }
})])

// 第九段 
_c('div',{class:bindClass,on:{"click":update}},[_v(_s(sum))])])}
```

## update

经过 render 函数, AST 树也就转换成了 VNode

接下来判断当前实例是否存在 prevVnode , 如无则说明是根节点挂载, 执行 `__patch__` 进行打包, 将 VNode 转成 dom 节点

判断传入 patch 的旧节点类型, 如是真实节点则以旧节点创建一个新的空虚拟节点, 保留真实的节点为 elm 属性; 同时记旧节点真实节点为 oldElm, 其父级节点为 parentElm; 接着 createElm 对 vnode 进行创建 dom 节点

给 vnode 创建真实节点 vnode.elm; 接着给其 children 创建节点; 遍历 children, 对每个子节点均进行 createElm, 完成属性等 data 绑定至新节点 vnode.elm; 最后将 children 也插入到根 vnode.elm

接着 removeVnodes 清除掉 oldVnode; 然后执行 invokeInsertHook, 对处理执行时 个别节点 data.hook.insert 赋值的进行 insert; 这里也就是给 input 继续绑定了 compositionStart, change 等方法

最后令 `vm.$el.__vue__ = vm`; `vm._isMounted = true` 作以标记, 至此挂载过程走完, 执行 mounted 钩子函数

```js
vnode: VNode {
  asyncFactory: undefined
  asyncMeta: undefined
  children: Array(9)
    0: VNode {tag: "button", data: {…}, children: Array(1), text: undefined, elm: undefined, …}
    1: VNode {tag: undefined, data: undefined, children: undefined, text: " ", elm: undefined, …}
    2: VNode {tag: "div", data: undefined, children: Array(1), text: undefined, elm: undefined, …}
    3: VNode {tag: undefined, data: undefined, children: undefined, text: " ", elm: undefined, …}
    4: VNode {tag: "ul", data: undefined, children: Array(3), text: undefined, elm: undefined, …}
    5: VNode {tag: undefined, data: undefined, children: undefined, text: " ", elm: undefined, …}
    6: VNode {tag: "div", data: undefined, children: Array(1), text: undefined, elm: undefined, …}
    7: VNode {tag: undefined, data: undefined, children: undefined, text: " ", elm: undefined, …}
    8: VNode {tag: "div", data: {…}, children: Array(1), text: undefined, elm: undefined, …}
    length: 9
    [[Prototype]]: Array(0)
  componentInstance: undefined
  componentOptions: undefined
  context: Vue {_uid: 0, _isVue: true, $options: {…}, _renderProxy: Proxy, _self: Vue, …}
  data: {attrs: {…}}
  elm: undefined
  fnContext: undefined
  fnOptions: undefined
  fnScopeId: undefined
  isAsyncPlaceholder: false
  isCloned: false
  isComment: false
  isOnce: false
  isRootInsert: true
  isStatic: false
  key: undefined
  ns: undefined
  parent: undefined
  raw: false
  tag: "div"
  text: undefined
  child: undefined
  [[Prototype]]: Object
}
```

# event

## processAttrs

在经过 processAttrs 处理后，v-on 或 @ 会被读取到，接着给当前节点层 AST 加入 events 属性, 对特殊的属性绑定也会增加 `hasBindings:true`

如下，例子中是在 button 上加入了点击事件 switchShow

```js
{
  attrsList:(1) [{…}]
  attrsMap:{@click: 'switchShow'}
  children:(1) [{…}]
  plain:false
  end:84
  start:45
  hasBindings:true
  parent:{type: 1, tag: 'div', attrsList: Array(1), attrsMap: {…}, rawAttrsMap: {…}, …}
  events:{click: {…}}
    click:{value: 'switchShow', dynamic: false, start: 53, end: 72}
  rawAttrsMap:{@click: {…}}
    @click:{name: '@click', value: 'switchShow', start: 53, end: 72}
  tag:'button'
  type:1
}
```

```js
if (onRE.test(name)) { // v-on
  name = name.replace(onRE, '')
  isDynamic = dynamicArgRE.test(name)
  if (isDynamic) {
    name = name.slice(1, -1)
  }
  addHandler(el, name, value, modifiers, false, warn, list[i], isDynamic)
}
```

## genElement

在生成事件节点代码时, 主要执行到以下函数: genData genChildren

```js
if (!el.plain || (el.pre && state.maybeComponent(el))) {
  data = genData(el, state)
}

const children = el.inlineTemplate ? null : genChildren(el, state, true)
code = `_c('${el.tag}'${
  data ? `,${data}` : '' // data
}${
  children ? `,${children}` : '' // children
})`
```

## genData

事件节点存在 events, 接着执行 genHandlers

```js
if (el.events) {
  data += `${genHandlers(el.events, false)},`
}
```

## genHandlers

遍历 events, 得到事件对应的方法, 拼接在一起, 再给前边加上 on: , 如例中是一个点击事件 '{on:{"click":switchShow},'

```js
export function genHandlers (
  events: ASTElementHandlers,
  isNative: boolean
): string {
  const prefix = isNative ? 'nativeOn:' : 'on:'
  let staticHandlers = ``
  let dynamicHandlers = ``
  for (const name in events) {
    const handlerCode = genHandler(events[name])
    if (events[name] && events[name].dynamic) {
      dynamicHandlers += `${name},${handlerCode},`
    } else {
      staticHandlers += `"${name}":${handlerCode},`
    }
  }
  staticHandlers = `{${staticHandlers.slice(0, -1)}}`
  if (dynamicHandlers) {
    return prefix + `_d(${staticHandlers},[${dynamicHandlers.slice(0, -1)}])`
  } else {
    return prefix + staticHandlers
  }
}
```

## createElement

```js
_c('button',{on:{"click":switchShow}},[_v("开关")])
```

针对以上代码, _c 实际上就是 createElement 方法, 在第一位参数传递了 vm 后, 剩下的参数由上边生成的代码传入, 因此 createElement 方法第二位参数就是 tag 标签; 第三位是 data, 也是当前节点的属性; 第四位是 children, 也是节点的子节点, 在此的 _v 表示这个子节点是文本节点.

当前节点 tag 为 html 内置标签, 那么就进行创建元素节点 vnode, 将这一段 _c 用vnode 替换掉, 完成后进入下一个虚拟节点的创建

```js
export function createElement (
  context: Component,
  tag: any,
  data: any, // 节点对应的属性数据
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

## createElm

给当前的子节点创建真实节点挂在 elm 属性下

在创建好 button 节点后, 进入 createChildren, 创建文本节点, 再执行插入, 通过父节点 button 执行 appendChild 完成插入操作; 

完成 createChildren 后, 进入 invokeCreateHooks, 通过执行 cbs 回调函数中的 create 下的系列创建方法, 完成新旧节点的更新; 这里的事件会执行 updateDOMListeners, 完成以下内容的同步更新; 接着将准备好的 children 插入到父级节点, 如此就算走完了当前这个事件节点

```js
cbs.create {
  0:ƒ updateAttrs(oldVnode, vnode)
  1:ƒ updateClass (oldVnode, vnode) 
  2:ƒ updateDOMListeners (oldVnode, vnode) 
  3:ƒ updateDOMProps(oldVnode, vnode)
  4:ƒ updateStyle(oldVnode, vnode)
  5:ƒ _enter (_, vnode) 
  6:ƒ create (_, vnode) 
  7:ƒ updateDirectives (oldVnode, vnode)
}
```

主要涉及到的环节: 

```js
vnode.elm = vnode.ns
  ? nodeOps.createElementNS(vnode.ns, tag)
  : nodeOps.createElement(tag, vnode)

// 创建子节点, 如果有属性 data, 初始化挂载属性 data, 后期更新属性 data
createChildren(vnode, children, insertedVnodeQueue)
if (isDef(data)) {
  invokeCreateHooks(vnode, insertedVnodeQueue)
}
insert(parentElm, vnode.elm, refElm)

// button 下子节点为文本节点
vnode.elm = nodeOps.createTextNode(vnode.text)

// insert
function insert (parent, elm, ref) {
  if (isDef(parent)) {
    if (isDef(ref)) {
      if (nodeOps.parentNode(ref) === parent) {
        nodeOps.insertBefore(parent, elm, ref)
      }
    } else {
      nodeOps.appendChild(parent, elm)
    }
  }
}

// invokeCreateHooks
function invokeCreateHooks (vnode, insertedVnodeQueue) {
  for (let i = 0; i < cbs.create.length; ++i) {
    cbs.create[i](emptyNode, vnode)
  }
  i = vnode.data.hook // Reuse variable
  if (isDef(i)) {
    if (isDef(i.create)) i.create(emptyNode, vnode)
    if (isDef(i.insert)) insertedVnodeQueue.push(vnode) // 插入的 VNode 队列
  }
}

// updateDOMListeners
function updateDOMListeners (oldVnode: VNodeWithData, vnode: VNodeWithData) {
  if (isUndef(oldVnode.data.on) && isUndef(vnode.data.on)) {
    return
  }
  const on = vnode.data.on || {}
  const oldOn = oldVnode.data.on || {}
  target = vnode.elm
  normalizeEvents(on)
  updateListeners(on, oldOn, add, remove, createOnceHandler, vnode.context)
  target = undefined
}
```

## updateDOMListeners

获取新旧节点的 on 下的事件, 进行对比更新 updateListeners; 令 target 等于 vnode.elm, 也就是我们最终要获得新虚拟节点 dom 化后的真实节点;

在 updateListeners 函数中, 将新旧节点的 on 进行对比, 其实初始化时旧节点并没有什么, 只是生成的一个空的虚拟节点; 等到下次变动做更新时才是有内容的旧节点.

遍历新节点 on 中的方法, 对比新旧节点 on 中的事件: 新节点 on 中的方法是否未被定义? 若未定义则抛出时间错误; 新节点方法对应的函数有而旧节点没有, 则直接将该方法通过 addEventListener 添加至 target, 也就是新 vnode 的真实节点 elm; 最后一种情况是新旧节点的事件对应的函数不一致, 则令旧节点函数的 fns 等于新节点函数, 再更新新的为 old;

最后遍历旧节点的事件名, 倘若新节点里没有, 则移除这个事件

```js
function updateDOMListeners (oldVnode: VNodeWithData, vnode: VNodeWithData) {
  if (isUndef(oldVnode.data.on) && isUndef(vnode.data.on)) {
    return
  }
  const on = vnode.data.on || {}
  const oldOn = oldVnode.data.on || {}
  target = vnode.elm
  normalizeEvents(on)
  updateListeners(on, oldOn, add, remove, createOnceHandler, vnode.context)
  target = undefined
}

// updateListeners
export function updateListeners (
  on: Object,
  oldOn: Object,
  add: Function,
  remove: Function,
  createOnceHandler: Function,
  vm: Component
) {
  let name, def, cur, old, event
  for (name in on) {
    def = cur = on[name]
    old = oldOn[name]
    event = normalizeEvent(name)
    /* istanbul ignore if */
    if (__WEEX__ && isPlainObject(def)) {
      cur = def.handler
      event.params = def.params
    }
    if (isUndef(cur)) {
      process.env.NODE_ENV !== 'production' && warn(
        `Invalid handler for event "${event.name}": got ` + String(cur),
        vm
      )
    } else if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur, vm)
      }
      if (isTrue(event.once)) {
        cur = on[name] = createOnceHandler(event.name, cur, event.capture)
      }
      add(event.name, cur, event.capture, event.passive, event.params)
    } else if (cur !== old) {
      old.fns = cur
      on[name] = old
    }
  }
  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name)
      remove(event.name, oldOn[name], event.capture)
    }
  }
}

// add
function add (
  name: string,
  handler: Function,
  capture: boolean,
  passive: boolean
) {
  // async edge case #6566: inner click event triggers patch, event handler
  // attached to outer element during patch, and triggered again. This
  // happens because browsers fire microtask ticks between event propagation.
  // the solution is simple: we save the timestamp when a handler is attached,
  // and the handler would only fire if the event passed to it was fired
  // AFTER it was attached.
  if (useMicrotaskFix) {
    const attachedTimestamp = currentFlushTimestamp
    const original = handler
    handler = original._wrapper = function (e) {
      if (
        // no bubbling, should always fire.
        // this is just a safety net in case event.timeStamp is unreliable in
        // certain weird environments...
        e.target === e.currentTarget ||
        // event is fired after handler attachment
        e.timeStamp >= attachedTimestamp ||
        // bail for environments that have buggy event.timeStamp implementations
        // #9462 iOS 9 bug: event.timeStamp is 0 after history.pushState
        // #9681 QtWebEngine event.timeStamp is negative value
        e.timeStamp <= 0 ||
        // #9448 bail if event is fired in another document in a multi-page
        // electron/nw.js app, since event.timeStamp will be using a different
        // starting reference
        e.target.ownerDocument !== document
      ) {
        return original.apply(this, arguments)
      }
    }
  }
  target.addEventListener(
    name,
    handler,
    supportsPassive
      ? { capture, passive }
      : capture
  )
}
```

# v-if

parseStartTag 中剪裁出标签 div 及属性 v-if="show", 将当前 attr 同步到 attrs 收集器, 再往下检测是否还有属性, 得知 end 也就是 > 存在, 说明后续就再没属性了, 将 html 又可以推进到 end 的 index

解析完开始标签，那么接下就要处理开始标签，handleStartTag 传入刚才匹配到的结果，获取标签名，然后开始处理属性 match.attrs; 将 attrs 遍历，将每一对属性转成 { name, value } 存入 attrs; 接着将 tag、attrs 生成 AST 树, 其实没多少操作, 就将 attrs 生成了 `attrsMap {v-if: 'show'}`; 接着走到 processIF, 给 v-if 添加 `IfConditions { exp: 'show', block: el }`

接着往下判断, 获取下一个 < 的位置, 将它之前的这段 text 剪下来, 将 html 推进; 接着将不是标签内容的 text 放入 chars 处理; 判断 currentParent(上段标签) 有无, 若有就说明 text 就是上一个标签内的内容; 接着检查 currentParent 是否是 style / script 标签, 若不是就解码这段 text; 然后解析处理好的 text, 看是否其中含有变量分隔符, 获取分隔符里的内容 exp, 用 _s() 包裹, 准备后续代码;最终得到该 text 的对象 child = {type: 2, expression, tokens, text}; 若是纯文本则没有 expression, tokens; 接着将 child 添加到父级的 children 中.

此时处理完的 v-if 的 AST 树为:

```js
{
  attrsList:(0) []
  attrsMap:{v-if: 'show'}
  children:(1) 
    0:{type: 2, expression: '_s(show ? 'bad ass' : 'motherfucker')', tokens: Array(1), text: '$[ show ? 'bad ass' : 'motherfucker' ]', start: 110, …}
  if:'show'
  ifConditions:(1)
    0:{exp: 'show', block: {…}}
  end:154
  start:93
  parent:{type: 1, tag: 'div', attrsList: Array(1), attrsMap: {…},}
  rawAttrsMap:{v-if:{name: 'v-if', value: 'show', start: 98, end: 109}}
  plain:true
  tag:'div'
  type:1
}
```



## parseHTML

从 while 开始处理 html, 不断推进生成 AST 树

```js
export function parseHTML (html, options) {
  const stack = []
  const expectHTML = options.expectHTML
  const isUnaryTag = options.isUnaryTag || no
  const canBeLeftOpenTag = options.canBeLeftOpenTag || no
  let index = 0
  let last, lastTag
  while (html) {
    last = html
    // Make sure we're not in a plaintext content element like script/style
    if (!lastTag || !isPlainTextElement(lastTag)) {
      let textEnd = html.indexOf('<') // 标记开始标签
      if (textEnd === 0) {
        // Comment:
        if (comment.test(html)) {
          const commentEnd = html.indexOf('-->')

          if (commentEnd >= 0) {
            if (options.shouldKeepComment) {
              options.comment(html.substring(4, commentEnd), index, index + commentEnd + 3)
            }
            advance(commentEnd + 3) // --> 占三位
            continue
          }
        }

        // http://en.wikipedia.org/wiki/Conditional_comment#Downlevel-revealed_conditional_comment
        if (conditionalComment.test(html)) {
          const conditionalEnd = html.indexOf(']>')

          if (conditionalEnd >= 0) {
            advance(conditionalEnd + 2)
            continue
          }
        }

        // Doctype:
        const doctypeMatch = html.match(doctype)
        if (doctypeMatch) {
          advance(doctypeMatch[0].length)
          continue
        }

        // End tag:
        const endTagMatch = html.match(endTag)
        if (endTagMatch) {
          const curIndex = index
          advance(endTagMatch[0].length)
          parseEndTag(endTagMatch[1], curIndex, index)
          continue
        }

        // Start tag:
        const startTagMatch = parseStartTag() // 解析开始的标签，把 template 转化成节点名和属性
        if (startTagMatch) {
          handleStartTag(startTagMatch)
          if (shouldIgnoreFirstNewline(startTagMatch.tagName, html)) {
            advance(1)
          }
          continue
        }
      }

      let text, rest, next
      if (textEnd >= 0) {
        rest = html.slice(textEnd) // 将结尾标签截出
        while (
          !endTag.test(rest) &&
          !startTagOpen.test(rest) &&
          !comment.test(rest) &&
          !conditionalComment.test(rest)
        ) {
          // < in plain text, be forgiving and treat it as text
          next = rest.indexOf('<', 1)
          if (next < 0) break
          textEnd += next
          rest = html.slice(textEnd)
        }
        text = html.substring(0, textEnd)
      }

      if (textEnd < 0) {
        text = html
      }

      if (text) {
        advance(text.length) // 更新 index html
      }

      if (options.chars && text) {
        options.chars(text, index - text.length, index)
      }
    } else {
      let endTagLength = 0
      const stackedTag = lastTag.toLowerCase()
      const reStackedTag = reCache[stackedTag] || (reCache[stackedTag] = new RegExp('([\\s\\S]*?)(</' + stackedTag + '[^>]*>)', 'i'))
      const rest = html.replace(reStackedTag, function (all, text, endTag) {
        endTagLength = endTag.length
        if (!isPlainTextElement(stackedTag) && stackedTag !== 'noscript') {
          text = text
            .replace(/<!\--([\s\S]*?)-->/g, '$1') // #7298
            .replace(/<!\[CDATA\[([\s\S]*?)]]>/g, '$1')
        }
        if (shouldIgnoreFirstNewline(stackedTag, text)) {
          text = text.slice(1)
        }
        if (options.chars) {
          options.chars(text)
        }
        return ''
      })
      index += html.length - rest.length
      html = rest
      parseEndTag(stackedTag, index - endTagLength, index)
    }

    if (html === last) {
      options.chars && options.chars(html)
      if (process.env.NODE_ENV !== 'production' && !stack.length && options.warn) {
        options.warn(`Mal-formatted tag at end of template: "${html}"`, { start: index + html.length })
      }
      break
    }
  }

  // Clean up any remaining tags
  parseEndTag()

  function advance (n) {
    index += n
    html = html.substring(n)
  }

  // 解析开始的标签
  function parseStartTag () {
    const start = html.match(startTagOpen)
    if (start) {
      const match = {
        tagName: start[1],
        attrs: [],
        start: index
      }
      advance(start[0].length)
      let end, attr
      while (!(end = html.match(startTagClose)) && (attr = html.match(dynamicArgAttribute) || html.match(attribute))) {
        attr.start = index
        advance(attr[0].length)
        attr.end = index
        match.attrs.push(attr)
      }
      if (end) {
        match.unarySlash = end[1]
        advance(end[0].length)
        match.end = index
        return match
      }
    }
  }

  // 处理开始标签
  function handleStartTag (match) {
    const tagName = match.tagName
    const unarySlash = match.unarySlash

    if (expectHTML) {
      if (lastTag === 'p' && isNonPhrasingTag(tagName)) {
        parseEndTag(lastTag)
      }
      if (canBeLeftOpenTag(tagName) && lastTag === tagName) {
        parseEndTag(tagName)
      }
    }

    const unary = isUnaryTag(tagName) || !!unarySlash // 判断标签是否是一元标签

    const l = match.attrs.length
    const attrs = new Array(l)
    for (let i = 0; i < l; i++) {
      const args = match.attrs[i]
      const value = args[3] || args[4] || args[5] || ''
      const shouldDecodeNewlines = tagName === 'a' && args[1] === 'href'
        ? options.shouldDecodeNewlinesForHref
        : options.shouldDecodeNewlines
      attrs[i] = {
        name: args[1],
        value: decodeAttr(value, shouldDecodeNewlines)
      }
      if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
        attrs[i].start = args.start + args[0].match(/^\s*/).length
        attrs[i].end = args.end
      }
    }

    // 将非一元的标签压入栈中，再由 parseRndTag 弹出匹配结束标签
    if (!unary) { 
      stack.push({ tag: tagName, lowerCasedTag: tagName.toLowerCase(), attrs: attrs, start: match.start, end: match.end })
      lastTag = tagName
    }

    if (options.start) {
      options.start(tagName, attrs, unary, match.start, match.end) // 将解析后的模板挂用传入的配置里 start 函数处理命令等
    }
  }

  function parseEndTag (tagName, start, end) { // 对于
    let pos, lowerCasedTagName
    if (start == null) start = index
    if (end == null) end = index

    // Find the closest opened tag of the same type
    if (tagName) {
      lowerCasedTagName = tagName.toLowerCase()
      for (pos = stack.length - 1; pos >= 0; pos--) { // 将栈内标签排出
        if (stack[pos].lowerCasedTag === lowerCasedTagName) {
          break
        }
      }
    } else {
      // If no tag name is provided, clean shop
      pos = 0
    }

    if (pos >= 0) {
      // Close all the open elements, up the stack
      for (let i = stack.length - 1; i >= pos; i--) {
        if (process.env.NODE_ENV !== 'production' &&
          (i > pos || !tagName) &&
          options.warn
        ) {
          options.warn(
            `tag <${stack[i].tag}> has no matching end tag.`,
            { start: stack[i].start, end: stack[i].end }
          )
        }
        if (options.end) {
          options.end(stack[i].tag, start, end)
        }
      }

      // Remove the open elements from the stack
      stack.length = pos // 清楚 pos 到栈顶的标签
      lastTag = pos && stack[pos - 1].tag
    } else if (lowerCasedTagName === 'br') {
      if (options.start) {
        options.start(tagName, [], true, start, end)
      }
    } else if (lowerCasedTagName === 'p') {
      if (options.start) {
        options.start(tagName, [], false, start, end)
      }
      if (options.end) {
        options.end(tagName, start, end)
      }
    }
  }
}
```

## parseHTML 的 option

涉及到 start end chars comment 处理方法, processIf 方法就在 start 中

```js
parseHTML(template, {
  warn,
  expectHTML: options.expectHTML,
  isUnaryTag: options.isUnaryTag,
  canBeLeftOpenTag: options.canBeLeftOpenTag,
  shouldDecodeNewlines: options.shouldDecodeNewlines,
  shouldDecodeNewlinesForHref: options.shouldDecodeNewlinesForHref,
  shouldKeepComment: options.comments,
  outputSourceRange: options.outputSourceRange,
  start (tag, attrs, unary, start, end) {
    // check namespace.
    // inherit parent ns if there is one
    const ns = (currentParent && currentParent.ns) || platformGetTagNamespace(tag)

    // handle IE svg bug
    /* istanbul ignore if */
    if (isIE && ns === 'svg') {
      attrs = guardIESVGBug(attrs)
    }

    let element: ASTElement = createASTElement(tag, attrs, currentParent) // 创建 AST 元素
    if (ns) {
      element.ns = ns
    }

    if (process.env.NODE_ENV !== 'production') {
      if (options.outputSourceRange) {
        element.start = start
        element.end = end
        element.rawAttrsMap = element.attrsList.reduce((cumulated, attr) => {
          cumulated[attr.name] = attr
          return cumulated
        }, {})
      }
      attrs.forEach(attr => {
        if (invalidAttributeRE.test(attr.name)) {
          warn(
            `Invalid dynamic argument expression: attribute names cannot contain ` +
            `spaces, quotes, <, >, / or =.`,
            {
              start: attr.start + attr.name.indexOf(`[`),
              end: attr.start + attr.name.length
            }
          )
        }
      })
    }

    // 处理 AST 元素
    // 判断是否为禁止的 tag 和服务端渲染
    if (isForbiddenTag(element) && !isServerRendering()) {
      element.forbidden = true
      process.env.NODE_ENV !== 'production' && warn(
        'Templates should only be responsible for mapping the state to the ' +
        'UI. Avoid placing tags with side-effects in your templates, such as ' +
        `<${tag}>` + ', as they will not be parsed.',
        { start: element.start }
      )
    }

    // apply pre-transforms
    // 对创建的 ASTElement 做预转换
    for (let i = 0; i < preTransforms.length; i++) {
      element = preTransforms[i](element, options) || element
    }

    if (!inVPre) {
      processPre(element)
      if (element.pre) {
        inVPre = true
      }
    }
    if (platformIsPreTag(element.tag)) {
      inPre = true
    }
    if (inVPre) {
      processRawAttrs(element)
    } else if (!element.processed) {
      // structural directives
      // 构建以前命令
      processFor(element) // 从元素中拿到 v-for 指令的内容，然后分别解析出 for、alias、iterator1、iterator2 等属性的值添加到 AST 的元素
      processIf(element) // 从元素中拿 v-if 指令的内容
      processOnce(element)
    }

    // 如果不存在 root 则将新建的 element 作为 root ,在检查 root 的约束条件 , 有问题的 tag 及 属性 v-for 均不能在 root 存在
    if (!root) {
      root = element
      if (process.env.NODE_ENV !== 'production') {
        checkRootConstraints(root)
      }
    }

    if (!unary) {
      currentParent = element
      stack.push(element)
    } else {
      closeElement(element)
    }
  },

  end (tag, start, end) {
    const element = stack[stack.length - 1]
    // pop stack
    stack.length -= 1
    currentParent = stack[stack.length - 1]
    if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
      element.end = end
    }
    closeElement(element)
  },

  chars (text: string, start: number, end: number) {
    if (!currentParent) {
      if (process.env.NODE_ENV !== 'production') {
        if (text === template) {
          warnOnce(
            'Component template requires a root element, rather than just text.',
            { start }
          )
        } else if ((text = text.trim())) {
          warnOnce(
            `text "${text}" outside root element will be ignored.`,
            { start }
          )
        }
      }
      return
    }
    // IE textarea placeholder bug
    /* istanbul ignore if */
    if (isIE &&
      currentParent.tag === 'textarea' &&
      currentParent.attrsMap.placeholder === text
    ) {
      return
    }
    const children = currentParent.children
    if (inPre || text.trim()) {
      text = isTextTag(currentParent) ? text : decodeHTMLCached(text)
    } else if (!children.length) {
      // remove the whitespace-only node right after an opening tag
      text = ''
    } else if (whitespaceOption) {
      if (whitespaceOption === 'condense') { // 如果此处文本样式 whitespace 配置为 condense，有换行符的取消，没有的单空格
        // in condense mode, remove the whitespace node if it contains
        // line break, otherwise condense to a single space
        text = lineBreakRE.test(text) ? '' : ' '
      } else {
        text = ' '
      }
    } else {
      text = preserveWhitespace ? ' ' : ''
    }
    if (text) {
      if (!inPre && whitespaceOption === 'condense') {
        // condense consecutive whitespaces into single space
        text = text.replace(whitespaceRE, ' ')
      }
      let res
      let child: ?ASTNode
      if (!inVPre && text !== ' ' && (res = parseText(text, delimiters))) {
        child = {
          type: 2, // 有表达式
          expression: res.expression,
          tokens: res.tokens,
          text
        }
      } else if (text !== ' ' || !children.length || children[children.length - 1].text !== ' ') {
        child = {
          type: 3, // 纯文本
          text
        }
      }
      if (child) {
        if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
          child.start = start
          child.end = end
        }
        children.push(child)
      }
    }
  },
  comment (text: string, start, end) {
    // adding anything as a sibling to the root node is forbidden
    // comments should still be allowed, but ignored
    if (currentParent) {
      const child: ASTText = {
        type: 3,
        text,
        isComment: true
      }
      if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
        child.start = start
        child.end = end
      }
      currentParent.children.push(child)
    }
  }
})
```

## parseText

```js
export function parseText (
  text: string,
  delimiters?: [string, string]
): TextParseResult | void {
  // 如果外界用户没有定义 delimiters 分隔符用来传变量，那么就使用 Vue 自带的双括号 {{}}
  // 如要使用此功能，可以在 new Vue 的时候传入 delimiters 属性： delimiters: ['', '']
  const tagRE = delimiters ? buildRegex(delimiters) : defaultTagRE 
  if (!tagRE.test(text)) {
    return
  }
  const tokens = []
  const rawTokens = []
  let lastIndex = tagRE.lastIndex = 0
  let match, index, tokenValue
  while ((match = tagRE.exec(text))) {
    index = match.index
    // push text token
    if (index > lastIndex) {
      rawTokens.push(tokenValue = text.slice(lastIndex, index))
      tokens.push(JSON.stringify(tokenValue))
    }
    // tag token
    const exp = parseFilters(match[1].trim())
    tokens.push(`_s(${exp})`)
    rawTokens.push({ '@binding': exp })
    lastIndex = index + match[0].length
  }
  if (lastIndex < text.length) {
    rawTokens.push(tokenValue = text.slice(lastIndex))
    tokens.push(JSON.stringify(tokenValue))
  }
  return {
    expression: tokens.join('+'),
    tokens: rawTokens
  }
}
```

## genElement

进入到 genIf 中, 在 genIfConditions 里获取 ast 时期建立的 IfConditions 对象 {exp, block} , 再进行展开

```js
if (el.if && !el.ifProcessed) {
  return genIf(el, state)
}

// genIf
export function genIf (
  el: any,
  state: CodegenState,
  altGen?: Function,
  altEmpty?: string
): string {
  el.ifProcessed = true // avoid recursion
  return genIfConditions(el.ifConditions.slice(), state, altGen, altEmpty)
}

// genIfConditions
function genIfConditions (
  conditions: ASTIfConditions,
  state: CodegenState,
  altGen?: Function,
  altEmpty?: string
): string {
  if (!conditions.length) {
    return altEmpty || '_e()'
  }

  const condition = conditions.shift()
  if (condition.exp) {
    return `(${condition.exp})?${
      genTernaryExp(condition.block)
    }:${
      genIfConditions(conditions, state, altGen, altEmpty)
    }`
  } else {
    return `${genTernaryExp(condition.block)}`
  }

  // v-if with v-once should generate code like (a)?_m(0):_m(1)
  function genTernaryExp (el) {
    return altGen
      ? altGen(el, state)
      : el.once
        ? genOnce(el, state)
        : genElement(el, state)
  }
}
```

## createElement

```js
(show)?_c('div',[_v(_s(show ? 'bad ass' : 'motherfucker'))]):_e()
```

首先获取变量 show, 在 vm 的属性上查找, 触发 proxy 代理通过 proxyGetter 读取挂载 vm 上的该变量, 于是在 vm._data 上进行读取该变量, 从而触发该变量的响应式系统, 进入属性拦截 get, 接着检测当前全局是否由 Dep.target, 正是初始化整个最外层的 Watcher, 通过 watcher 将这个依赖 dep 添加进依赖收集器, 同时依赖 dep 也将当前的观察者添加进 subs, 当然也可能会有很多个观察者, 在 get 时将这些都添加到被观察者 dep 下, 等候该数据变动再通知所有 watcher 更新; 添加完依赖后, 弹出获得的变量值, 进行下一个变量的获取.

在读取完变量后, 开始执行代码, 从 _s 开始, 由里至外执行, 这里的三段式 show 为 true, 那么就会对应 'bad ass', 接着进行 toString 操作

继续进行 _v, 将刚才获得的字符串转为文本虚拟节点; 最后到 _c, 创建元素虚拟节点, 其子节点为刚才的文本虚拟节点

## createElm

还是首先给当前子节点创建真实节点挂到 vnode.elm 上, 接着进入到创建当前子节点的子节点, 而此时的 v-if 节点其内的子节点模板变量字符串已经在 createElement 阶段确定下来了, 此时为单纯的文本, 因此也就创建真实的文本节点, 再 appendChild 插入到上层节点;

而 v-if 在 render 执行代码阶段就已经读取到 if 后边的值了, 确定下来显示不显示, 其他不相关的属性就正常同步; 完成后将该节点插入到根节点下

```js
vnode.elm = vnode.ns
  ? nodeOps.createElementNS(vnode.ns, tag)
  : nodeOps.createElement(tag, vnode)


createChildren(vnode, children, insertedVnodeQueue)
if (isDef(data)) {
  invokeCreateHooks(vnode, insertedVnodeQueue)
}
insert(parentElm, vnode.elm, refElm) // 把 DOM 插入到父节点中
```

# v-for

v-for 的编译工作同上, 在 parse 完成后得到 AST 树为:

因为是 li 标签来执行 v-for, 外边还套了一层 ul, 所以此时的根节点就是 ul, 它的 parent 也就是 app; children 里是下一级节点, 也就是 li, 这里新增了 alias 属性, for 属性, key 属性, 均是在 processFor 中

```js
{
  attrsList:(0) []
  attrsMap:{}
  children:(1) [{…}]
    0: {
      alias:'food'
      attrsList:(0) []
      attrsMap:{v-for: 'food in foodList', :key: 'food.name'}
      children:(1) [{…}]
        0: {
          end:277
          expression:'_s(food.name)+"：￥ "+_s(food.price)'
          start:249
          text:'$[food.name]：￥ $[food.price]'
          tokens:(3) [{…}, '：￥ ', {…}]
            0:{@binding: 'food.name'}
            1:'：￥ '
            2:{@binding: 'food.price'}
          type:2
        }
      end:282
      for:'foodList'
      key:'food.name'
      parent:{type: 1, tag: 'ul', attrsList: Array(0), attrsMap: {…}, rawAttrsMap: {…}, …}
      plain:false
      rawAttrsMap:{v-for: {…}, :key: {…}}
      start:203
      tag:'li'
      type:1
    }
  rawAttrsMap:{}
  plain:true
  parent:{type: 1, tag: 'div', attrsList: Array(1), attrsMap: {…}, …}
  end:296
  start:186
  plain:true
  rawAttrsMap:{}
  tag:'ul'
  type:1
}
```

## processFor

```js
export function processFor (el: ASTElement) {
  let exp
  if ((exp = getAndRemoveAttr(el, 'v-for'))) {
    const res = parseFor(exp)
    if (res) {
      extend(el, res)
    } else if (process.env.NODE_ENV !== 'production') {
      warn(
        `Invalid v-for expression: ${exp}`,
        el.rawAttrsMap['v-for']
      )
    }
  }
}
```

## parseFor

```js
export function parseFor (exp: string): ?ForParseResult {
  const inMatch = exp.match(forAliasRE)
  if (!inMatch) return
  const res = {}
  res.for = inMatch[2].trim()
  const alias = inMatch[1].trim().replace(stripParensRE, '')
  const iteratorMatch = alias.match(forIteratorRE)
  if (iteratorMatch) {
    res.alias = alias.replace(forIteratorRE, '').trim()
    res.iterator1 = iteratorMatch[1].trim()
    if (iteratorMatch[2]) {
      res.iterator2 = iteratorMatch[2].trim()
    }
  } else {
    res.alias = alias
  }
  return res
}
```

## genFor

执行到 genFor 则会直接返回内容, 不会再向下执行, genFor 内部会读取之前的 ast 树添加的 for, 与 _l 拼接在一起, '_l((${exp}), function(key) ' ; 之后直接 genElement 继续处理其他属性, 以及 children, 且在 genFor 的时候, 改变 forProcessed 为 true, 则在再次调用 genElement 时不会重复执行 genFor; 按例中继续生成 children 的代码, 继而返回.

```js
export function genFor (
  el: any,
  state: CodegenState,
  altGen?: Function,
  altHelper?: string
): string {
  const exp = el.for
  const alias = el.alias
  const iterator1 = el.iterator1 ? `,${el.iterator1}` : ''
  const iterator2 = el.iterator2 ? `,${el.iterator2}` : ''

  if (process.env.NODE_ENV !== 'production' &&
    state.maybeComponent(el) &&
    el.tag !== 'slot' &&
    el.tag !== 'template' &&
    !el.key
  ) {
    state.warn(
      `<${el.tag} v-for="${alias} in ${exp}">: component lists rendered with ` +
      `v-for should have explicit keys. ` +
      `See https://vuejs.org/guide/list.html#key for more info.`,
      el.rawAttrsMap['v-for'],
      true /* tip */
    )
  }

  el.forProcessed = true // avoid recursion
  return `${altHelper || '_l'}((${exp}),` +
    `function(${alias}${iterator1}${iterator2}){` +
      `return ${(altGen || genElement)(el, state)}` +
    '})'
}
```

## genData

li 的 ast 树上有 key, 因此 genData 时会执行到下边内容

```js
// key
if (el.key) {
  data += `key:${el.key},`
}
```

## createElement

```js
_c('ul',_l((foodList),function(food){
  return _c('li',{key:food.name},[
    _v(_s(food.name)+"：￥ "+_s(food.price))
  ])
}),0)
```

接着上边创建好的函数, 依次读取变量 _c, _l, foodList, 然后由内而外执行.

## renderList

v-for 使用 _l 来渲染列表, 遍历第一个参数 foodList, 将元素传入第二个参数函数, 借 render 函数使每个元素对应生成一个虚拟节点;

依据上边的 li 节点, 在获取了 _s, 以及 food.name/food.price 后, 将两个变量转成字符串并合并在一起, 用 _v 来生成文本节点; 接着 _c 当前的 li 节点, 根据 'li', 属性 data, 其 children 三个参数来创建 li 的虚拟节点; 

如此遍历完 foodList, 就得到一个虚拟节点的数组, 给标记 _isVList = true, 作为 ul 的 chilren

接着回到 ul 的 _c 时刻, 其实就是按普通的内置元素创建虚拟节点, 下边 children 有几个 li 的虚拟节点

```js
export function renderList (
  val: any,
  render: (
    val: any,
    keyOrIndex: string | number,
    index?: number
  ) => VNode
): ?Array<VNode> {
  let ret: ?Array<VNode>, i, l, keys, key
  if (Array.isArray(val) || typeof val === 'string') {
    ret = new Array(val.length)
    for (i = 0, l = val.length; i < l; i++) {
      ret[i] = render(val[i], i)
    }
  } else if (typeof val === 'number') {
    ret = new Array(val)
    for (i = 0; i < val; i++) {
      ret[i] = render(i + 1, i)
    }
  } else if (isObject(val)) {
    if (hasSymbol && val[Symbol.iterator]) {
      ret = []
      const iterator: Iterator<any> = val[Symbol.iterator]()
      let result = iterator.next()
      while (!result.done) {
        ret.push(render(result.value, ret.length))
        result = iterator.next()
      }
    } else {
      keys = Object.keys(val)
      ret = new Array(keys.length)
      for (i = 0, l = keys.length; i < l; i++) {
        key = keys[i]
        ret[i] = render(val[key], key, i)
      }
    }
  }
  if (!isDef(ret)) {
    ret = []
  }
  (ret: any)._isVList = true
  return ret
}
```

## createElm

v-for 节点的父级设置了 ul, 因而在走到 li 时, 才能看到有 v-for 的一些特性, 但其实也没什么特别;

会在 checkDuplicateKeys 里检查每个 li 上的 key 是否有冲突; 此后再渲染出 li 的真实节点, 它的子节点也是已经准备好的文本节点, 直接创建真实的文本节点, 再添加到 li 上, 遍历 ul 的 children, 把所有 li 都创建完 挂在 ul 上; 再把 ul 挂载到上一级

# v-model

v-model 放在 input 上进行数据双向绑定，input 外边裹了一层 div

## processAttrs

parse 后生成的 AST 树如下：

```js
{
  attrsList:(1) [{…}]
    0:{name: 'v-model', value: 'text', start: 342, end: 356}
  attrsMap:{v-model: 'text'}
  children:(0) []
  directives: (1) [{…}]
    0: {
      arg:null
      end:356
      isDynamicArg:false
      modifiers:undefined
      name:'model'
      rawName:'v-model'
      start:342
      value:'text'
    }
  plain:false
  rawAttrsMap:{v-model: {…}}
    v-model:{name: 'v-model', value: 'text', start: 342, end: 356}
  start:335
  tag:'input'
  type:1
}
```

## genDirectives

model 会给 ast 树加上 directives 属性, 来描述 model, 经过 genDefaultModel, 完成了 model 指令对应 value 的 dom 属性绑定以及 input 事件绑定, 得到 

```js
directives:[{name:"model",rawName:"v-model",value:(text),expression:"text"},]
```

```js
function genDirectives (el: ASTElement, state: CodegenState): string | void {
  const dirs = el.directives
  if (!dirs) return
  let res = 'directives:['
  let hasRuntime = false
  let i, l, dir, needRuntime
  for (i = 0, l = dirs.length; i < l; i++) {
    dir = dirs[i]
    needRuntime = true
    const gen: DirectiveFunction = state.directives[dir.name]
    if (gen) {
      // compile-time directive that manipulates AST.
      // returns true if it also needs a runtime counterpart.
      needRuntime = !!gen(el, dir, state.warn)
    }
    if (needRuntime) {
      hasRuntime = true
      res += `{name:"${dir.name}",rawName:"${dir.rawName}"${
        dir.value ? `,value:(${dir.value}),expression:${JSON.stringify(dir.value)}` : ''
      }${
        dir.arg ? `,arg:${dir.isDynamicArg ? dir.arg : `"${dir.arg}"`}` : ''
      }${
        dir.modifiers ? `,modifiers:${JSON.stringify(dir.modifiers)}` : ''
      }},`
    }
  }
  if (hasRuntime) {
    return res.slice(0, -1) + ']'
  }
}
```

### genDefaultModel

执行 genAssignmentCode 后获得的代码, 拼接在 `if($event.target.composing)return;` 其后; 

接着调用 addProp 将 value 添加为 dom 属性 `props [{name: 'value', value: '(text)', dynamic: undefined}]`;

继续调用 addHandler 将 input 输入事件添加进当前 ast 节点, `events: {input:{value: 'if($event.target.composing)return;text=$event.target.value', dynamic: undefined}}`

```js
function genDefaultModel (
  el: ASTElement,
  value: string,
  modifiers: ?ASTModifiers
): ?boolean {
  const type = el.attrsMap.type

  // warn if v-bind:value conflicts with v-model
  // except for inputs with v-bind:type
  if (process.env.NODE_ENV !== 'production') {
    const value = el.attrsMap['v-bind:value'] || el.attrsMap[':value']
    const typeBinding = el.attrsMap['v-bind:type'] || el.attrsMap[':type']
    if (value && !typeBinding) {
      const binding = el.attrsMap['v-bind:value'] ? 'v-bind:value' : ':value'
      warn(
        `${binding}="${value}" conflicts with v-model on the same element ` +
        'because the latter already expands to a value binding internally',
        el.rawAttrsMap[binding]
      )
    }
  }

  const { lazy, number, trim } = modifiers || {}
  const needCompositionGuard = !lazy && type !== 'range'
  const event = lazy
    ? 'change'
    : type === 'range'
      ? RANGE_TOKEN
      : 'input'

  let valueExpression = '$event.target.value'
  if (trim) {
    valueExpression = `$event.target.value.trim()`
  }
  if (number) {
    valueExpression = `_n(${valueExpression})`
  }

  let code = genAssignmentCode(value, valueExpression)
  if (needCompositionGuard) {
    code = `if($event.target.composing)return;${code}`
  }

  addProp(el, 'value', `(${value})`)
  addHandler(el, event, code, null, true)
  if (trim || number) {
    addHandler(el, 'blur', '$forceUpdate()')
  }
}
```

### genAssignmentCode

解析 model 后, 看返回的结果 key 是否存在, 然后与传入的参数 valueExpression 进行拼接, 实际上也是 input 输入事件获取到的值, 同步到这里,  'text'='$event.target.value'

```js
export function genAssignmentCode (
  value: string,
  assignment: string
): string {
  const res = parseModel(value)
  if (res.key === null) {
    return `${value}=${assignment}`
  } else {
    return `$set(${res.exp}, ${res.key}, ${assignment})`
  }
}
```

### parseModel

解析 model 绑定的变量, 返回一个对象 {exp: 'text', key: null}

```js
export function parseModel (val: string): ModelParseResult {
  // Fix https://github.com/vuejs/vue/pull/7730
  // allow v-model="obj.val " (trailing whitespace)
  val = val.trim()
  len = val.length

  if (val.indexOf('[') < 0 || val.lastIndexOf(']') < len - 1) {
    index = val.lastIndexOf('.')
    if (index > -1) {
      return {
        exp: val.slice(0, index),
        key: '"' + val.slice(index + 1) + '"'
      }
    } else {
      return {
        exp: val,
        key: null
      }
    }
  }

  str = val
  index = expressionPos = expressionEndPos = 0

  while (!eof()) {
    chr = next()
    /* istanbul ignore if */
    if (isStringStart(chr)) {
      parseString(chr)
    } else if (chr === 0x5B) {
      parseBracket(chr)
    }
  }

  return {
    exp: val.slice(0, expressionPos),
    key: val.slice(expressionPos + 1, expressionEndPos)
  }
}
```

## genProps

经过 genDirectives, 给当前节点的 ast 树挂上了 props 属性, 用来描述 model 对应的 value, 判断是静态属性还是动态属性, 最终得到 `domProps:{"value":(text)},` , 与之前的指令进行拼接; 

```js
// DOM props
if (el.props) {
  data += `domProps:${genProps(el.props)},`
}

// genProps
function genProps (props: Array<ASTAttr>): string {
  let staticProps = ``
  let dynamicProps = `` // 动态的
  for (let i = 0; i < props.length; i++) {
    const prop = props[i]
    const value = __WEEX__
      ? generateValue(prop.value)
      : transformSpecialNewlines(prop.value)
    if (prop.dynamic) {
      dynamicProps += `${prop.name},${value},`
    } else {
      staticProps += `"${prop.name}":${value},`
    }
  }
  staticProps = `{${staticProps.slice(0, -1)}}`
  if (dynamicProps) {
    return `_d(${staticProps},[${dynamicProps.slice(0, -1)}])`
  } else {
    return staticProps
  }
}
```

## genHandlers

经过 genDirectives, 给当前节点的 ast 树挂上了 events 属性, 用来描述 model 对应的输入事件, 将 model 事件的 value 加入函数, 接着绑定 input, 也就是说 input 事件的监听函数就是 value 的变动: 

```js
// 事件的 prefix
on: {"input":function($event){if($event.target.composing)return;text=$event.target.value}}
```

最后将 directives props events 的代码拼接起来:

```js
_c('input',{directives:[{name:"model",rawName:"v-model",value:(text),expression:"text"}],domProps:{"value":(text)},on:{"input":function($event){if($event.target.composing)return;text=$event.target.value}}})
```

```js
// event handlers
if (el.events) {
  data += `${genHandlers(el.events, false)},`
}

// genHandlers
export function genHandlers (
  events: ASTElementHandlers,
  isNative: boolean
): string {
  const prefix = isNative ? 'nativeOn:' : 'on:'
  let staticHandlers = ``
  let dynamicHandlers = ``
  for (const name in events) {
    const handlerCode = genHandler(events[name])
    if (events[name] && events[name].dynamic) {
      dynamicHandlers += `${name},${handlerCode},`
    } else {
      staticHandlers += `"${name}":${handlerCode},`
    }
  }
  staticHandlers = `{${staticHandlers.slice(0, -1)}}`
  if (dynamicHandlers) {
    return prefix + `_d(${staticHandlers},[${dynamicHandlers.slice(0, -1)}])`
  } else {
    return prefix + staticHandlers
  }
}

// genHandler
function genHandler (handler: ASTElementHandler | Array<ASTElementHandler>): string {
  if (!handler) {
    return 'function(){}'
  }

  if (Array.isArray(handler)) {
    return `[${handler.map(handler => genHandler(handler)).join(',')}]`
  }

  const isMethodPath = simplePathRE.test(handler.value)
  const isFunctionExpression = fnExpRE.test(handler.value)
  const isFunctionInvocation = simplePathRE.test(handler.value.replace(fnInvokeRE, ''))

  if (!handler.modifiers) {
    if (isMethodPath || isFunctionExpression) {
      return handler.value
    }
    /* istanbul ignore if */
    if (__WEEX__ && handler.params) {
      return genWeexHandler(handler.params, handler.value)
    }
    return `function($event){${
      isFunctionInvocation ? `return ${handler.value}` : handler.value
    }}` // inline statement
  } else {
    let code = ''
    let genModifierCode = ''
    const keys = []
    for (const key in handler.modifiers) {
      if (modifierCode[key]) {
        genModifierCode += modifierCode[key]
        // left/right
        if (keyCodes[key]) {
          keys.push(key)
        }
      } else if (key === 'exact') {
        const modifiers: ASTModifiers = (handler.modifiers: any)
        genModifierCode += genGuard(
          ['ctrl', 'shift', 'alt', 'meta']
            .filter(keyModifier => !modifiers[keyModifier])
            .map(keyModifier => `$event.${keyModifier}Key`)
            .join('||')
        )
      } else {
        keys.push(key)
      }
    }
    if (keys.length) {
      code += genKeyFilter(keys)
    }
    // Make sure modifiers like prevent and stop get executed after key filtering
    if (genModifierCode) {
      code += genModifierCode
    }
    const handlerCode = isMethodPath
      ? `return ${handler.value}.apply(null, arguments)`
      : isFunctionExpression
        ? `return (${handler.value}).apply(null, arguments)`
        : isFunctionInvocation
          ? `return ${handler.value}`
          : handler.value
    /* istanbul ignore if */
    if (__WEEX__ && handler.params) {
      return genWeexHandler(handler.params, code + handlerCode)
    }
    return `function($event){${code}${handlerCode}}`
  }
}
```

## createElement

```js
_c('div',[_c('input',{
  directives:[{name:"model",rawName:"v-model",value:(text),expression:"text"}],
  domProps:{"value":(text)},
  on:{
    "input":function($event){if($event.target.composing)return;text=$event.target.value}
  }
})])
```

依次读取完各变量, input 开始生成虚拟节点, 'input' 其后为属性抽象出来的 data 对象, 而此时 input 创建虚拟节点的 data 就直接为该对象, 只不过做了变量替换.

## createElm

创建 input 标签, 再看其 children, 没有则继续进行 invokeCreateHooks, 这也是 v-model 的重点, 因为当前节点的属性 data 有 directives, domProps, on; 也就是指令 model, input 的 dom 属性 value 用以事件读取, input 输入事件

接下来着重处理属性 data

```js
vnode.elm = vnode.ns
  ? nodeOps.createElementNS(vnode.ns, tag)
  : nodeOps.createElement(tag, vnode)

createChildren(vnode, children, insertedVnodeQueue)
if (isDef(data)) {
  invokeCreateHooks(vnode, insertedVnodeQueue)
}
insert(parentElm, vnode.elm, refElm)
```

## invokeCreateHooks

在这个方法里, 会遍历 cbs.create 里的方法来处理传入的 vnode 属性 data, 如属性, class, 事件, dom 属性, 行内样式, 指令等的挂载更新

这里会执行到 updateDOMListeners, updateDOMProps, 

```js
cbs.create {
  0:ƒ updateAttrs(oldVnode, vnode)
  1:ƒ updateClass (oldVnode, vnode) 
  2:ƒ updateDOMListeners (oldVnode, vnode) 
  3:ƒ updateDOMProps(oldVnode, vnode)
  4:ƒ updateStyle(oldVnode, vnode)
  5:ƒ _enter (_, vnode) 
  6:ƒ create (_, vnode) 
  7:ƒ updateDirectives (oldVnode, vnode)
}

function invokeCreateHooks (vnode, insertedVnodeQueue) {
  for (let i = 0; i < cbs.create.length; ++i) {
    cbs.create[i](emptyNode, vnode)
  }
  i = vnode.data.hook // Reuse variable
  if (isDef(i)) {
    if (isDef(i.create)) i.create(emptyNode, vnode)
    if (isDef(i.insert)) insertedVnodeQueue.push(vnode) // 插入的 VNode 队列
  }
}
```

### updateDOMListeners

updateDOMListeners 是因为存在 input 事件; 同之前说到的处理事件的方法一样, 会对比新旧节点属性 data 里 on 中的事件, 初始化时, 旧节点没有, 因此就直接将新节点 on 里的事件方法进行 createFnInvoker 处理, 接着直接在 add 方法里用 addEventListener 给当前节点 vnode.elm 绑定 input 事件

```js
function updateDOMListeners (oldVnode: VNodeWithData, vnode: VNodeWithData) {
  if (isUndef(oldVnode.data.on) && isUndef(vnode.data.on)) {
    return
  }
  const on = vnode.data.on || {}
  const oldOn = oldVnode.data.on || {}
  target = vnode.elm
  normalizeEvents(on)
  updateListeners(on, oldOn, add, remove, createOnceHandler, vnode.context)
  target = undefined
}

// updateListeners
export function updateListeners (
  on: Object,
  oldOn: Object,
  add: Function,
  remove: Function,
  createOnceHandler: Function,
  vm: Component
) {
  let name, def, cur, old, event
  for (name in on) {
    def = cur = on[name]
    old = oldOn[name]
    event = normalizeEvent(name)
    /* istanbul ignore if */
    if (__WEEX__ && isPlainObject(def)) {
      cur = def.handler
      event.params = def.params
    }
    if (isUndef(cur)) {
      process.env.NODE_ENV !== 'production' && warn(
        `Invalid handler for event "${event.name}": got ` + String(cur),
        vm
      )
    } else if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur, vm)
      }
      if (isTrue(event.once)) {
        cur = on[name] = createOnceHandler(event.name, cur, event.capture)
      }
      add(event.name, cur, event.capture, event.passive, event.params)
    } else if (cur !== old) {
      old.fns = cur
      on[name] = old
    }
  }
  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name)
      remove(event.name, oldOn[name], event.capture)
    }
  }
}
```

### updateDOMProps

接着进行 domProps 的处理, 获取当前的 domProps, 进行遍历, 对每个 prop 的名字进行判断, 看是否是 textContent, innerHTML, value, 按不同方式处理;

此时是 input 键入的值, 也就是 value, 设置当前节点 elm 的 _value 为 value 对应的值, 也就是已经存在与 input 框内的值; 接着检查 shouldUpdateValue 是否应该更新值, 避免值相同了还做重复设定

```js
function updateDOMProps (oldVnode: VNodeWithData, vnode: NodeWithData) 
  if (isUndef(oldVnode.data.domProps) && isUndef(vnode.data.domProps)) {
    return
  }
  let key, cur
  const elm: any = vnode.elm
  const oldProps = oldVnode.data.domProps || {}
  let props = vnode.data.domProps || {}
  // clone observed objects, as the user probably wants to mutate it
  if (isDef(props.__ob__)) {
    props = vnode.data.domProps = extend({}, props)
  }

  for (key in oldProps) {
    if (!(key in props)) {
      elm[key] = ''
    }
  }

  for (key in props) {
    cur = props[key]
    // ignore children if the node has textContent or innerHTML,
    // as these will throw away existing DOM nodes and cause removal errors
    // on subsequent patches (#3360)
    if (key === 'textContent' || key === 'innerHTML') {
      if (vnode.children) vnode.children.length = 0
      if (cur === oldProps[key]) continue
      // #6601 work around Chrome version <= 55 bug where single textNode
      // replaced by innerHTML/textContent retains its parentNode property
      if (elm.childNodes.length === 1) {
        elm.removeChild(elm.childNodes[0])
      }
    }

    if (key === 'value' && elm.tagName !== 'PROGRESS') {
      // store value as _value as well since
      // non-string values will be stringified
      elm._value = cur
      // avoid resetting cursor position when value is the same
      const strCur = isUndef(cur) ? '' : String(cur)
      if (shouldUpdateValue(elm, strCur)) {
        elm.value = strCur
      }
    } else if (key === 'innerHTML' && isSVG(elm.tagName) && isUndef(elm.innerHTML)) {
      // IE doesn't support innerHTML for SVG elements
      svgContainer = svgContainer || document.createElement('div')
      svgContainer.innerHTML = `<svg>${cur}</svg>`
      const svg = svgContainer.firstChild
      while (elm.firstChild) {
        elm.removeChild(elm.firstChild)
      }
      while (svg.firstChild) {
        elm.appendChild(svg.firstChild)
      }
    } else if (
      // skip the update if old and new VDOM state is the same.
      // `value` is handled separately because the DOM value may be temporarily
      // out of sync with VDOM state due to focus, composition and modifiers.
      // This  #4521 by skipping the unnecessary `checked` update.
      cur !== oldProps[key]
    ) {
      // some property updates can throw
      // e.g. `value` on <progress> w/ non-finite value
      try {
        elm[key] = cur
      } catch (e) {}
    }
  }
}
```

## updateDirectives

当前的节点有 Directives 命令, 将新节点的命令数组获取, 遍历, 如果不存在旧节点, 那么就将新命令绑定, 存入 dirsWithInsert; 如果存在旧节点, 那么就将命令作以更新;

在 _update 中, 如果旧节点等于空节点, 那么 isCreate 就为 true, 表示新创建; 因此之后会执行 `mergeVNodeHook(vnode, 'insert', callInsert)` 给当前的节点属性 data 插入 insert hook

在 中, oldHook 是 `vnode.data.hook['insert']`, 当前并不存在, 因此为 undefined; 在后续会执行 `invoker = createFnInvoker([wrappedHook])` , 也是用 callInsert 生成 invoker, 再令 `vnode.data.hook['insert'] = invoker`

如此就完成了; 再往下回到 invokeCreateHooks, 会检查 if (isDef(i.insert)) 是否定义, 而此时定义了, 那么就将当前的 VNode 加入 `insertedVnodeQueue.push(vnode)` 队列

```js
function updateDirectives (oldVnode: VNodeWithData, vnode: VNodeWithData) {
  if (oldVnode.data.directives || vnode.data.directives) {
    _update(oldVnode, vnode)
  }
}

function _update (oldVnode, vnode) {
  const isCreate = oldVnode === emptyNode
  const isDestroy = vnode === emptyNode
  const oldDirs = normalizeDirectives(oldVnode.data.directives, oldVnode.context)
  const newDirs = normalizeDirectives(vnode.data.directives, vnode.context)

  const dirsWithInsert = []
  const dirsWithPostpatch = []

  let key, oldDir, dir
  for (key in newDirs) {
    oldDir = oldDirs[key]
    dir = newDirs[key]
    if (!oldDir) {
      // new directive, bind
      callHook(dir, 'bind', vnode, oldVnode)
      if (dir.def && dir.def.inserted) {
        dirsWithInsert.push(dir)
      }
    } else {
      // existing directive, update
      dir.oldValue = oldDir.value
      dir.oldArg = oldDir.arg
      callHook(dir, 'update', vnode, oldVnode)
      if (dir.def && dir.def.componentUpdated) {
        dirsWithPostpatch.push(dir)
      }
    }
  }

  if (dirsWithInsert.length) {
    const callInsert = () => {
      for (let i = 0; i < dirsWithInsert.length; i++) {
        callHook(dirsWithInsert[i], 'inserted', vnode, oldVnode)
      }
    }
    if (isCreate) {
      mergeVNodeHook(vnode, 'insert', callInsert)
    } else {
      callInsert()
    }
  }

  if (dirsWithPostpatch.length) {
    mergeVNodeHook(vnode, 'postpatch', () => {
      for (let i = 0; i < dirsWithPostpatch.length; i++) {
        callHook(dirsWithPostpatch[i], 'componentUpdated', vnode, oldVnode)
      }
    })
  }

  if (!isCreate) {
    for (key in oldDirs) {
      if (!newDirs[key]) {
        // no longer present, unbind
        callHook(oldDirs[key], 'unbind', oldVnode, oldVnode, isDestroy)
      }
    }
  }
}

// mergeVNodeHook
export function mergeVNodeHook (def: Object, hookKey: string, hook: Function) {
  if (def instanceof VNode) {
    def = def.data.hook || (def.data.hook = {})
  }
  let invoker
  const oldHook = def[hookKey]

  function wrappedHook () {
    hook.apply(this, arguments)
    // important: remove merged hook to ensure it's called only once
    // and prevent memory leak
    remove(invoker.fns, wrappedHook)
  }

  if (isUndef(oldHook)) {
    // no existing hook
    invoker = createFnInvoker([wrappedHook])
  } else {
    /* istanbul ignore if */
    if (isDef(oldHook.fns) && isTrue(oldHook.merged)) {
      // already a merged invoker
      invoker = oldHook
      invoker.fns.push(wrappedHook)
    } else {
      // existing plain hook
      invoker = createFnInvoker([oldHook, wrappedHook])
    }
  }

  invoker.merged = true
  def[hookKey] = invoker
}
```

# v-bind

## processAttrs

在 processAttrs 对 v-bind 的处理

```js
if (bindRE.test(name)) { // v-bind
  name = name.replace(bindRE, '')
  value = parseFilters(value)
  isDynamic = dynamicArgRE.test(name)
  if (isDynamic) {
    name = name.slice(1, -1)
  }
  if (
    process.env.NODE_ENV !== 'production' &&
    value.trim().length === 0
  ) {
    warn(
      `The value for a v-bind expression cannot be empty. Found in "v-bind:${name}"`
    )
  }
  if (modifiers) {
    if (modifiers.prop && !isDynamic) {
      name = camelize(name)
      if (name === 'innerHtml') name = 'innerHTML'
    }
    if (modifiers.camel && !isDynamic) {
      name = camelize(name)
    }
    if (modifiers.sync) {
      syncGen = genAssignmentCode(value, `$event`)
      if (!isDynamic) {
        addHandler(
          el,
          `update:${camelize(name)}`,
          syncGen,
          null,
          false,
          warn,
          list[i]
        )
        if (hyphenate(name) !== camelize(name)) {
          addHandler(
            el,
            `update:${hyphenate(name)}`,
            syncGen,
            null,
            false,
            warn,
            list[i]
          )
        }
      } else {
        // handler w/ dynamic event name
        addHandler(
          el,
          `"update:"+(${name})`,
          syncGen,
          null,
          false,
          warn,
          list[i],
          true // dynamic
        )
      }
    }
  }
  if ((modifiers && modifiers.prop) || (
    !el.component && platformMustUseProp(el.tag, el.attrsMap.type, name)
  )) {
    addProp(el, name, value, list[i], isDynamic)
  } else {
    addAttr(el, name, value, list[i], isDynamic)
  }
} 
```

## genElement

dataGenFns 中的方法使动态属性及动态样式表示出来

```js
// module data generation functions
for (let i = 0; i < state.dataGenFns.length; i++) {
  data += state.dataGenFns[i](el)
}

// 调用 dataGenFns 中的方法
function genData (el: ASTElement): string {
  let data = ''
  if (el.staticClass) {
    data += `staticClass:${el.staticClass},`
  }
  if (el.classBinding) {
    data += `class:${el.classBinding},`
  }
  return data
}

// style
function genData (el: ASTElement): string {
  let data = ''
  if (el.staticStyle) {
    data += `staticStyle:${el.staticStyle},`
  }
  if (el.styleBinding) {
    data += `style:(${el.styleBinding}),`
  }
  return data
}
```

## createElement

```js
_c('div',{class:bindClass,on:{"click":update}},[_v(_s(sum))])])}
```

sum 通过 computedGetter 获取, 会得到 initComputed 时期创建的计算属性 watcher , 通过 watcher 添加依赖, 其实 在对数据的每次 getter 时, 都会触发相应的依赖收集, 将新的订阅添加完，会移除掉旧的订阅, newDeps 表示新添加的 Dep 实例数组，而 deps 表示上一次添加的 Dep 实例数组。

同时我们也能看到, 计算属性 sum 对应的依赖 dep 当前已有三个观察者, 一个是初始化计算属性, 一个是初始化 watch 时我们写入了 sum, 再有就是刚才做 render 生成虚拟节点时读取到了 sum;

读取完 sum 后将其转为字符串, 接着生成文本虚拟节点.

此时 div 标签 _c 的准备工作就做完了, 标签, 属性 data, children, 创建元素虚拟节点; 动态属性也在之前读取变量时得到了对应的值

## createElm

计算属性到这一步, 和一般的其实也差不多, 这里说下上边没有的 class 的更新;

判断如果和之前旧的节点的 class 不同, 那么就用 setAttribute 设定现在的 class, 同时再用 _prevClass 存储当前的 class, 以便之后 class 改变了与 _prevClass 再进行对比

```js
function updateClass (oldVnode: any, vnode: any) {
  const el = vnode.elm
  const data: VNodeData = vnode.data
  const oldData: VNodeData = oldVnode.data
  if (
    isUndef(data.staticClass) &&
    isUndef(data.class) && (
      isUndef(oldData) || (
        isUndef(oldData.staticClass) &&
        isUndef(oldData.class)
      )
    )
  ) {
    return
  }

  let cls = genClassForVnode(vnode)

  // handle transition classes
  const transitionClass = el._transitionClasses
  if (isDef(transitionClass)) {
    cls = concat(cls, stringifyClass(transitionClass))
  }

  // set the class
  if (cls !== el._prevClass) {
    el.setAttribute('class', cls)
    el._prevClass = cls
  }
}
```