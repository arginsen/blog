---
title: 前端 | Vue 学习笔记Ⅲ：Vue Router
date: 2020-6-5
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

> Vue Router 是 Vue.js 官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。

**包含的功能有：**
- 嵌套的路由/视图表
- 模块化的、基于组件的路由配置
- 路由参数、查询、通配符
- 基于 Vue.js 过渡系统的视图过渡效果
- 细粒度的导航控制
- 带有自动激活的 CSS class 的链接
- HTML5 历史模式或 hash 模式，在 IE9 中自动降级
- 自定义的滚动条行为

# 安装

```js
// 页面直接引用
<script src="/path/to/vue.js"></script>
<script src="/path/to/vue-router.js"></script>
// or
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

// npm 或 yarn 安装
npm install vue-router

yarn add vue-router 
```

# 路由使用

> 使用 Vue.js + Vue Router 创建单页应用，将组件映射到路由，指定路由在页面渲染的位置

1. 在 html 页面 通过 `<router-link>` 来匹配组件，使用 `<router-view>` 将匹配到的组件渲染到指定位置
```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <p>
    <!-- 使用 router-link 组件来导航.
        通过传入 to 属性指定链接.
        <router-link> 默认会被渲染成一个 <a> 标签
        <router-link> 对应的路由匹配成功，将自动设置 class 属性值 .router-link-active -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口，将匹配到的组件渲染输出 -->
  <router-view></router-view>
</div>
```

2. 创建组件并添加至定义好的路由，创建 router 实例并添加路由配置，最后挂载到跟实例
```js
// 0. 如果使用模块化机制编程，导入 Vue 和 VueRouter，要调用 Vue.use(VueRouter)

// 1. 定义 (路由) 组件。
// 可以从其他文件 import 进来
const ipone = { template: '<button>ipone 12</button>' }
const huawei = { template: '<button>huawei mate 40</button>' }

// 2. 定义路由
// 每个路由应该映射一个组件。 其中 "component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
const routes = [
  { path: '/ipone', component: ipone },
  { path: '/huawei', component: huawei }
]

// 3. 创建 router 实例，然后传 `routes` 配置
const router = new VueRouter({
  routes
})

// 4. 创建和挂载根实例。
// 通过 router 配置参数注入路由，让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app')

// 现在，应用已经启动了！
```

3. 通过注入路由器，可以在任何组件内通过 `this.$router` 访问路由器，也可以通过 `this.$route` 访问当前路由
```js
// Home.vue
export default {
  computed: {
    username() {
      return this.$route.params.username
    }
  },
  methods: {
    goBack() {
      window.history.length > 1 ? this.$router.go(-1) : this.$router.push('/')
    }
  }
}
```

## 样板

<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <router-link to="/ipone">ipone</router-link>
  <router-link to="/huawei">huawei</router-link>

  <router-view></router-view>
</div>

<script>
// 定义组件配置对象
const ipone = { template: '<button>ipone 12</button>' }
const huawei = { template: '<button>huawei mate 40</button>' }

// 定义路由
const routes = [
  { path: '/ipone', component: ipone },
  { path: '/huawei', component: huawei }
]

// 将定义的路由添加到 router 实例
const router = new VueRouter({
  routes
})

// 挂载 router 实例至元素节点
const app = new Vue({
  router
}).$mount('#app')
</script>


# 动态路由匹配


```js
const User = {
  template: '<div>User</div>'
}

const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})

//  /user/foo 和 /user/bar 都将映射到相同的路由
// 当匹配到一个路由时，参数值会被设置到 this.$route.params
// 可以在每个组件内使用：
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}


// 一个路由中可以设置多段路径参数，对应的值都会设置到 $route.params 中
// 模式
/user/:username/post/:post_id	
// 路径
/user/evan/post/123	
// $route.params 存储
{ username: 'evan', post_id: '123' }
```

## 响应路由参数的变化

```js
// /user/foo 导航到 /user/bar，原来的组件实例会被复用，
// 这意味着组件的生命周期钩子不会再被调用

// 复用组件时可以通过 watch $route 对象对路由参数的变化作出相应
const User = {
  template: '...',
  watch: {
    $route(to, from) {
      // 对路由变化作出响应...
    }
  }
}

// 或使用 beforeRouteUpdate 导航守卫
const User = {
  template: '...',
  beforeRouteUpdate (to, from, next) {
    // react to route changes...
    // don't forget to call next()
  }
}
```

## 捕获所有路由

```js
// 常规参数只会匹配被 / 分隔的 URL 片段中的字符
// 匹配任意路径，得使用通配符 (*)
{
  // 会匹配所有路径
  path: '*'  // 通常用于客户端 404 错误
}
{
  // 会匹配以 `/user-` 开头的任意路径
  path: '/user-*'
}
```

## 匹配优先级

> 同一个路径可以匹配多个路由，此时，匹配的优先级就按照路由的定义顺序：谁先定义的，谁的优先级就最高。


# 嵌套路由

```html
<!-- 在 html 页面写路由出口，输出组件 -->
<div id="app">
  <router-view></router-view>
</div>

<script>
// 定义第一层路由的组件
// 被渲染组件中再嵌套 <router-view>
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}

// 再定义子路由的组件
const UserHome = {
    template: ''
}

const UserProfile = {
    template: ''
}

const UserPosts = {
    template: ''
}

// 创建 router 实例，第一层动态路由，及嵌套的子路由写在 children

// 如 children 只定义各子路由，不接子路由直接访问，是无法进行渲染的
// 那么可以定义一个空的子路由，指向 /user/:id 对应的组件
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        // 定义一个空路由，对应第一层路由渲染的组件
        { path: '', component: UserHome },
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})

// 将路由挂载到根实例
const app = new Vue({
  router
}).$mount('#app')
</script>
```


# 命名路由

> 通过一个名称来标识一个路由，在链接一个路由，或者是执行一些跳转的时候会更方便

```js
// 通过 name 属性来命名路由
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})

// 通过 to，链接到 user 路由，导航至 /user/123
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```

# 命名视图

```js
// 同级 展示多个视图
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>

// 对不同视图出口，给出不同的组件渲染
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

## 嵌套命名视图

```js
// UserSettings.vue 
<div>
  <h1>User Settings</h1>
  <NavBar/>
  <router-view/>
  <router-view name="helper"/>
</div>


// 定义顶级路由 /settings，下有二级路由 emails 和 profile，分别有各自的组件
// 其中 profile 子路由在页面有多个视图出口，通过命名视图来指定输出组件
{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions
  }, {
    path: 'profile',
    components: {
      default: UserProfile,
      helper: UserProfilePreview
    }
  }]
}
```

# 重定向和别名

## 重定向

> 重定向即为，当用户访问 /a 时，URL 将会被替换成 /b，然后匹配路由为 /b

```js
// 从 /a 重定向到 /b
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})

// 重定向的目标也可以是一个命名的路由
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: { name: 'foo' }}
  ]
})

// 重定向的目标可以为一个方法，动态返回重定向目标
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: to => {
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}
  ]
})
```

## 别名

> /a 的别名是 /b，意味着，当用户访问 /b 时，URL 会保持为 /b，但是路由匹配则为 /a，就像用户访问 /a 一样。类似 tomcat 配置文件路径，path 为虚拟地址，别名，docBase 为文件真实路径

```js
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```


# 路由组件传参

> 在组件中使用 $route 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性。

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

> 可使用 props 将组件和路由解耦

```js
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const router = new VueRouter({
  routes: [
    { 
      path: '/user/:id', 
      component: User, 
      props: true 
    },
  ]
})

// 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
{
  path: '/user/:id',
  components: { default: User, sidebar: Sidebar },
  props: { default: true, sidebar: false }
}
```

# History 模式

> - vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。
> - 如果不想要很丑的 hash，我们可以用路由的 history 模式，这种模式充分利用 history.pushState API 来完成 URL 跳转而无须重新加载页面。

# 参考资料

[Vue Router 官方教程](https://router.vuejs.org/zh/)