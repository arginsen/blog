---
title: 面试 | Vue
date: 2020-8-27
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

# vue 的指令有哪些

`v-model`、`v-on`、`v-bind`、`v-if`、`v-for`、`v-show`、`v-slot`、`v-text`、`v-html`

# vue 组件间的通信方式

1. 通过 props 传递
2. 通过 $emit 触发自定义事件

```js
// 子
this.$emit('add', good)

// 父
<Children @add="cartAdd" />

cartAdd(good) {
  console.log(good)
}
```

3. 使用 ref

置于标签时获取当前 dom
置于组件上时获取整个组件对象

4. EventBus
5. $parent 或 $root

```js
this.$parent.on('add', this.add)
this.$parent.emit('add')
```

6. attrs 与 listeners
7. Provide 与 Inject

祖先组件定义 provide 属性，返回传递的值
后代组件通过 inject 接收值

8. Vuex

# vue 的生命周期函数有哪些

函数 | 周期
---- | ---
beforeCreate | 在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用。
created | 在实例创建完成后被立即调用。在这一步，实例已完成以下的配置：数据观测 (data observer)，property 和方法的运算，watch/event 事件回调。
beforeMount | 在挂载开始之前被调用：相关的 `render` 函数首次被调用。
mounted | 实例被挂载后调用，这时 el 被新创建的 `vm.$el` 替换了。
beforeUpdate | 数据更新时调用，发生在虚拟 DOM 打补丁之前。
updated | 由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。
activated | 被 keep-alive 缓存的组件激活时调用。
deactivated | 被 keep-alive 缓存的组件停用时调用。
beforeDestroy | 实例销毁之前调用。在这一步，实例仍然完全可用。
destroyed | 实例销毁后调用。该钩子被调用后，对应 Vue 实例的所有指令都被解绑，所有的事件监听器被移除，所有的子实例也都被销毁。
errorCaptured | 当捕获一个来自子孙组件的错误时被调用。

# vue 的 computed 和 watch 的区别

`computed` 是计算属性，是依赖其他属性的计算值，并且有缓存，只有当依赖的值变化时才会更新。
computed 会在初始化阶段针对我们定义的变量创建对应的 watcher, 并将书写的函数作为回调传入 watcher，初始化不会读取 computed 的 get 内容；等到后边读取内容时，需要获取 computed 依赖的值，此时依赖值的 dep 会将我们定义的 computed 这个方法（观察者）收集进 subs，同时返回经过 evaluate 后的值，而 watcher 的配置项 lazy 标记为 true，dirty 也为 true，等到 evaluate 结束后，就将 dirty 转为 false，那么在后续有其他地方有用到 computed 值时，便不再进行计算，直接返回当前缓存的值；
等到 computed 依赖的值更新后，dep 会 notify 各个观察者进行 update，其中就包括 computed 的变量，在 update 是会判断当前的 watcher.lazy 是否为 true，为 true 就说明是计算属性的 watcher ，那么就将其 dirty 再转变为 true，等到后续读取 computed 值时，就会再次触发 evaluate 重新计算值，再把 dirty 转为 false，其他地方使用 computed 时就不会再计算了。

`watch` 是在监听的属性发生变化时，在回调中执行一些逻辑。
watch 在初始化阶段对指定监听的变量创建 watcher，initWatch 是后于 initComputed 发生的，因此依赖同一变量时，dep 会先将 computed 的 watcher 先压入 subs，在之后依赖值变动 watcher 更新时也会先更新 computed，后更新 watch 的值，但此时更新的 computed 仅是打开 dirty 为 true 操作，之后便进行 watch 的更新，在后续页面或其他地方有用到 computed 时才会在触发 computedGetter 重新进行计算。因此在页面上的 computed 值表现是要晚于 watch 里的语句的，因为页面上的变量建立的 watcher 均是晚于 init 阶段压入 subs 的。 

computed 适合在模板渲染中，某个值是依赖了其他的响应式对象甚至是计算属性计算而来的；watch 适合监听某个值的变化去完成一些复杂的业务逻辑。

# v-show 和 v-if 的区别

`v-if` 会在切换过程中对条件块的事件监听器和子组件进行销毁和重建，如果初始条件是 false，则不改动什么，直到第一次为 true 时才开始渲染模块。
`v-show` 只是基于 css 进行切换，不管初始条件是什么，都会渲染。

v-if 在切换时的开销更大，而 v-show 初始化渲染开销更大；在需要频繁切换，或者切换的部分 dom 很复杂时，使用 v-show 更合适；在渲染后很少切换的则使用 v-if 更合适。

# v-if 和 v-for 可以一起使用吗

当 v-for 和 v-if 处于同一个节点时，v-for 的优先级比 v-if 高，这意味着 v-if 将分别重复运行于每个 v-for 循环中。
如果要遍历的数组很大，而要真正展示的数据很少时，这将造成很大的性能浪费。这种场景可以用 computed 先对数据进行过滤。

# v-for 中的 key 的作用

key 是给每个 vnode 指定的唯一 id，在统计的 vnode diff 过程中，可以根据 key 快速的对比，来判断是否为相同节点，并利用 key 的唯一性生成 map 来更快地获取相应的节点

# keep-alive 的作用

`keep-alive` 可以在组件切换时，保存其包裹的组件的状态，使其不被销毁，防止多次渲染。

`keep-alive` 有两个独立的生命周期钩子函数 `actived` 和 `deactived`，使用 keep-alive 包裹的组件在切换时不会被销毁，而是缓存到内存中并执行 deactived 钩子函数，命中缓存渲染后会执行 actived 钩子函数。

# vue 的响应式原理

vue 的响应式是通过 `Object.defineProperty` 对数据进行劫持，并结合观察者模式实现。
vue 利用 `Object.defineProperty` 创建一个 observe 来劫持 监听所有的属性，把这些属性全部转为 getter 和 setter。
vue 中每个组件实例都会对应一个 watcher 实例，它会在组件渲染的过程中把使用过的数据属性通过 getter 收集为依赖；之后当触发 setter 时，会通知 watcher，从而使它关联的组件重新渲染。

# vue 的双向绑定原理

vue 的双向绑定基于 mvvm 架构, 用户侧来定义 model 和 view, 封装的框架来实现将两者关联起来, 使数据变化后更新到视图, 视图变化后更新到数据; 

实现双向绑定是再初始化 Vue 实例时, 通过 observe 对数据进行响应化处理, 对数据的访问与设定作以拦截, 在编译阶段最后创建 Watcher 和更新函数, 在 render 期间会获取绑定的数据触发数据 getter 收集依赖, 每个属性都对应一个依赖收集器 dep, 收集该属性在视图中出现的地方(计算属性方法, watch, render挂载时)时的全局 watcher, 一个 dep 的 subs 下往往也对应多个 watcher , 等到数据变动, 该数据对应的 dep 会 notify 它下边的 watcher 进行更新


# vue3 为什么使用 proxy 实现响应式

1. `Object.defineProperty` 只能劫持对象的属性，所以需要遍历对象的每个属性，而 proxy 是直接代理对象。
2. Object.defineProperty 对新增属性需要手动进行 Observe，由于 Object.ddefineProperty 劫持的是对象的属性，所以新增属性时，又需要重新遍历对象，对其新增属性再用 Object.ddefineProperty 劫持
3. Proxy 支持 13 种拦截操作，这时 Object.ddefineProperty 不具有的
4. Proxy 作为新标准，长远来看 js 引擎会继续优化 Proxy，但 getter 和 setter 则不会再有针对性优化
5. 此外 Proxy 兼容性较差，目前并没有一个完整支持 Proxy 所有拦截方法的 Polyfill 方案

# vue2 中如何检测数组变化

vue 的 Observer 对数组进行了单独的处理，对数组的方法进行编译，并赋值给数组属性的 `__proto__` 属性上，因为原型链的机制，找到对应方法就不会继续往上找了。
编译方法中会对一些可以增加索引的方法（push、unshift、splice）进行手动 observe。

# vue-router 的 hash 模式和 history 模式的区别

1. url 展示上，hash 模式有 `#`，history 模式则没有
2. 刷新页面时，hash 模式可以正常加载到 hash 值对应的页面，而 history 没有处理的话，会返回 404，一般需要后端将所有页面都配置重定向到首页路由
3. 兼容性问题，hash 支持低版本浏览器和 IE

# vue-router 的 hash 模式和 history 模式是如何实现的

`hash` 模式：`#` 后面 hash 值的变化，不会导致浏览器向服务器发出请求，浏览器不发出请求，就不会刷新页面。同时通过监听 `hashchange` 事件可以知道 hash 发生了哪些变化，然后根据 hash 变化来实现更新页面部分内容的操作。
`history` 模式：history 模式的实现，主要是 HTML5 标准发布的两个 API，`pushState` 和 `replaceState`，这两个 API 可以改变 url，但不会发送请求。这样就可以监听 url 变化来实现更新页面部分内容的操作


# 参考链接

[史上最强vue总结---面试开发全靠它了](https://juejin.im/post/6850037277675454478)
[金九银十，初中级前端面试复习总结「Vue篇」](https://juejin.im/post/6869908820353810445#heading-12)