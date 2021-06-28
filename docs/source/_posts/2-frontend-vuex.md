---
title: 前端 | Vue 学习笔记Ⅳ：Vuex
date: 2020-6-8
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->


# 概述

> - Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。
> - 每一个 Vuex 应用的核心就是 **store**（仓库）。store 基本上就是一个容器，它包含着你的应用中大部分的**状态** (state)。Vuex 和单纯的全局对象有以下两点不同：
>   - Vuex 的状态存储是**响应式**的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
>   - 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。

# 安装

```js
// 直接引入
<script src="/path/to/vue.js"></script>
<script src="/path/to/vuex.js"></script>


// 安装在模块化的项目
yarn add vuex

// 显式地通过 Vue.use() 来安装 Vuex
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
```

# 一个简单的 vuex

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})


// 通过 store.commit 方法触发状态变更，
// 相当于执行 store 里 mutation 封装的各个函数
store.commit('increment')
// 通过 store.state 来获取状态对象
console.log(store.state.count) // -> 1


// 将 store 注入到 vue 实例中，
// 这样 vue 的组件就可以调用 this.$store，来定义改变一些状态
new Vue({
  el: '#app',
  store
})


// 在其他组件中提交变更
methods: {
  increment() {
    this.$store.commit('increment')  // 提交 mutation 中的 increment
    console.log(this.$store.state.count)
  }
}
```

# State

> Vuex 使用单一状态树——是的，用一个对象就包含了全部的应用层级状态。至此它便作为一个“唯一数据源 (SSOT)”而存在。这也意味着，每个应用将仅仅包含**一个 store 实例**。单一状态树让我们能够直接地定位任一特定的状态片段，在调试的过程中也能轻易地取得整个当前应用状态的快照。

```js
// Vuex 通过 store 选项，提供了一种机制将状态从根组件“注入”到每一个子组件中（需调用 Vue.use(Vuex)）：
const app = new Vue({
  el: '#app',
  // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})

// 通过在根实例中注册 store 选项，该 store 实例会注入到根组件下的所有子组件中，且子组件能通过 this.$store 访问到。让我们更新下 Counter 的实现：
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```


# Getter







# Mutation

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数

Vuex 的 store 中的状态是响应式的，那么当我们变更状态时，监视状态的 Vue 组件也会自动更新。这也意味着 Vuex 中的 mutation 也需要与使用 Vue 一样遵守一些注意事项：

1. 最好提前在你的 store 中初始化好所有所需属性。
2. 当需要在对象上添加新属性时，你应该
  - 使用 Vue.set(obj, 'newProp', 123), 
  - 以新对象替换老对象。例如，利用对象展开运算符：`state.obj = { ...state.obj, newProp: 123 }`

```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})

// 使用 store 的 commit 方法来提交此时需要状态更改
store.commit('increment')


// 第二个参数 payload，一般为一个对象，
// 这样可以包含多个字段并且记录的 mutation 会更易读
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}

store.commit('increment', {
  amount: 10
})

// 或者用如下方式提交，用 type 代替第一个 commit() 的参数
store.commit({
  type: 'increment',
  amount: 10
})
```


# Action

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含**任意异步操作**。




# Module



# 项目结构

Vuex 并不限制你的代码结构。但是，它规定了一些需要遵守的规则：

1. 应用层级的状态应该集中到单个 store 对象中。
2. 提交 mutation 是更改状态的唯一方法，并且这个过程是同步的。
3. **异步逻辑**都应该封装到 **action** 里面。

只要你遵守以上规则，如何组织代码随你便。如果你的 store 文件太大，只需将 action、mutation 和 getter 分割到单独的文件。
对于大型应用，我们会希望把 Vuex 相关代码分割到模块中。下面是项目结构示例：

```yml
├── index.html
├── main.js
├── api
│   └── ... # 抽取出 API 请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块
```


# 严格模式

> - 在严格模式下，无论何时发生了状态变更且不是由 mutation 函数引起的，将会抛出错误。这能保证所有的状态变更都能被调试工具跟踪到。
> - 不要在发布环境下启用严格模式！严格模式会深度监测状态树来检测不合规的状态变更——请确保在发布环境下关闭严格模式，以避免性能损失。

```js
// 开启严格模式，仅需在创建 store 的时候传入 strict: true：
const store = new Vuex.Store({
  // ...
  strict: true
})

 

// 发布环境下关闭严格模式，我们可以让构建工具来处理这种情况：
const store = new Vuex.Store({
  // ...
  strict: process.env.NODE_ENV !== 'production'
})
```

# 表单处理

> 当在严格模式中使用 Vuex 时，在属于 Vuex 的 state 上使用 v-model，用户输入时会直接修改绑定的对象，由于这个修改不是在 mutation 函数中执行的, 这里会抛出一个错误。

```js
// vuex 的逻辑

// 给 <input> 中绑定 value 来获取用户输入，
// 然后侦听 input 或者 change 事件，在事件回调中调用一个方法:

// ...components
<input :value="message" @input="updateMessage">

computed: {
  ...mapState({
    // value 绑定的 message 是计算属性返回的属于 Vuex store 的对象的一个属性
    message: state => state.obj.message  
  })
},
methods: {
  updateMessage (e) {
    // 提交 mutation 中的 updateMessage 方法，并传送改动的值 value
    this.$store.commit('updateMessage', e.target.value)
  }
}

// mutation 函数：
// ...store
mutations: {
  updateMessage (state, message) {
    state.obj.message = message
  }
}



// 上边的实现过于繁琐，另一种方法是使用带有 setter 的双向绑定计算属性
<input v-model="message">

computed: {
  message: {
    get () {
      return this.$store.state.obj.message
    },
    set (value) {
      this.$store.commit('updateMessage', value)
    }
  }
}
```





[Vuex 官方教程](https://vuex.vuejs.org/zh/)