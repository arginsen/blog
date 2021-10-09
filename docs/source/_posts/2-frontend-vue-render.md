---
title: 前端 | Vue 语法进阶：render 函数
date: 2020-6-2
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->


# 原理

![](https://segmentfault.com/img/remote/1460000010352759)

> 除了使用摸板生成 Virtual DOM，通过 render 函数也可以生成 Virtual DOM，view 再根据 Virtual DOM 生成 Actual DOM。


## Actual DOM 和 Virtual DOM

```js
// Actual DOM 通过 document.createElement('div') 生成 DOM 节点。
document.createElement('div')

// 浏览器原生对象（开销大）
"[object HTMLDivElement]"


// Virtual DOM 通过 vm.$createElement('div')生成一个 JS 对象，
// VDOM 对象有一个表示 div 的 tag 属性，
// 有一个包含了所有可能特性的 data 属性，
// 可能还有一个包含更多虚拟节点的 children 列表。
vm.$createElement('div')

// 纯JS对象（轻量）
{ tag: 'div', data: { attrs: {}, ...}, children: [] }
```

## JSX 和 Template

> - JSX 和 Template 都是用于声明 DOM 和 state 之间关系的一种方式，在Vue中，Template 是默认推荐的方式，但是也可以使用 JSX 来做更灵活的事。
>   - JSX 更加动态化，对于使用编程语言是很有帮助的，可以做任何事，但是动态化使得编译优化更加复杂和困难。
>   - Template 更加静态化并且对于表达式有更多约束，但是可以快速复用已经存在的模板，模板约束意味着可以在编译时做更多的性能优化，相对于 JSX 在编译时间上有着更多优势。

## render 函数

> - render 函数依赖于页面上所有的数据 data，并且这些数据是响应式的，所有的数据作为组件 render 函数的依赖。一旦这些数据有所改变，那么 render 函数会被重新调用。
> - 在更新阶段，render 函数会重新调用并且返回一个新的 Virtual Dom，新旧Virtual DOM 之间会进行比较，把 diff 之后的最小改动应用到Actual DOM中。
> - Watcher 负责收集依赖，清除依赖和通知依赖。在大型复杂的组件树结构下，由于采用了精确的依赖追踪系统，所以会避免组件的过度渲染。


> - 在创建 Vue 实例的过程中，如果传入的选项中有 template 和 render 两个属性，render 会有更高的**优先级**
> - 通常使用 **h** 作为 createElement 的别名，这是 Vue 的通用惯例，也是JSX的要求


# 应用

> createElement('div', {}， [...]) 生成摸板

```js
// @returns {VNode}
createElement(
  // 第一个参数 {String | Object | Function}
  // 一个 HTML 标签字符串，组件选项对象，或者
  // 解析上述任何一种的一个 async 异步函数。必需参数。
  'div',

  // 第二个参数 {Object}
  // 一个包含模板相关属性的数据对象
  // 你可以在 template 中使用这些特性。可选参数。
  {
    
  },

  // 第三个参数 {String | Array}
  // 子虚拟节点 (VNodes)，由 `createElement()` 构建而成，
  // 也可以使用字符串来生成“文本虚拟节点”。可选参数。
  [
    '先写一些文字',
    createElement('h1', '一则头条'),
    createElement(MyComponent, {
      props: {
        someProp: 'foobar'
      }
    })
  ]
)
```

> 通常用 h 写

```html
<!-- 定义 template -->
<div id="app">
  <example :tags="['h1', 'h2', 'h3']"></example>
</div>

<script>
    // 定义example组件
    Vue.component('example', {
      props: ['tags'],
      render (h) {
        
        // 第二个参数是一个包含模板相关属性的数据对象，可选参数
        
        // 子虚拟节点（VNodes）参数可以传入字符串或者数字，
        // 通过createElement生成，可选参数
        return h('div', this.tags.map((tag, i) => h(tag, i)))
      }
    })
    
    // 实例化
    new Vue({ el: '#app' })
</script>
```


# render: h => h(App)

> App 为组件选项对象，用作第一个参数，App 本身作为 Vue 组件的入口组件，导入到 Vue 项目的 main.js，生成 Vue 实例时挂载到 `<div id="app"></div>` 上边去

```js
// render: h => h(App) 是下面内容的缩写：
render: function (createElement) {
    return createElement(App);
}

// 进一步缩写为(ES6 语法)：
render (createElement) {
    return createElement(App);
}

// 再进一步缩写为：
render (h) {
    return h(App);
}

// 按照 ES6 箭头函数的写法，就得到了：
render: h => h(App);
```







# 参考资料

[Vue 渲染函数](https://cn.vuejs.org/v2/guide/render-function.html)
[render 函数原理及实现](https://segmentfault.com/a/1190000016945248)
[render 函数的三个参数](https://segmentfault.com/a/1190000010913794)
[关于 render: h => h(App) 的解答](https://segmentfault.com/q/1010000007130348)