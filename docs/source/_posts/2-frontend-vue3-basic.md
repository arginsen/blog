---
title: 前端 | vue3 语法 - 相比 vue2 的升级
date: 2021-10-3 18:45:00
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->

主要对 vue3 新的语法及书写习惯作以整理。

# createApp

返回一个提供应用上下文的应用实例。应用实例挂载的整个组件树共享同一个上下文。

createApp 接收一个根组件选项对象作为第一个参数, 第二个参数是根 prop, 可以直接在 'app' 节点渲染出 username

```ts
import App from './App.vue'

const app = createApp(App, { username: 'Al' })
```

# setup

setup 函数是一个新的组件选项，作为在 组件内使用组合式 API 的入口

setup 接收两个参数： props、context

setup 函数中的 props 是响应式的，当传入新的 props 时会更新，同时也不能使用解构赋值来取值。当然也可以用 toRefs 将 props 转为一组 ref 值的对象，再获取。此外传入的 props 可能读取y一个可选的属性，若该属性没有，则应该使用 toRef 来为 props 的该属性新创建一个 ref

```ts
export default {
  props: {
    title?: String
  },
  setup(props) {
    console.log(props.title)

    const { title } = toRefs(props)

    const title = toRef(props, 'title')
  }
}
```

context 是一个普通的 js 对象，它暴露了一些值

```ts
export default {
  setup(props, context) {
    // Attribute (非响应式对象，等同于 $attrs)
    console.log(context.attrs)

    // 插槽 (非响应式对象，等同于 $slots)
    console.log(context.slots)

    // 触发事件 (方法，等同于 $emit)
    console.log(context.emit)

    // 暴露公共 property (函数)
    console.log(context.expose)
  }
}
```

可以对 context 直接使用解构

```ts
export default {
  setup(props, { attrs, slots, emit, expose }) {
    // ...
  }
}
```

# `<script setup>`

`<script setup>` 是在单文件组件（SFC）中使用组合式 API 的编译时语法糖。

1. 更少的样板内容，更简洁的代码
2. 能够使用纯 typescript 声明 props 和抛出事件
3. 更好的运行时性能
4. 更好的 IDE 类型推断性能

## 一般语法

里面的代码会被编译成组件 setup() 函数的内容。这意味着与普通的 `<script>` 只在组件被首次引入的时候执行一次不同，`<script setup>` 中的代码会在每次组件实例被创建的时候执行。

其内定义的变量均会直接暴露给模板，包括导入的组件，实际也是可以直接在模板使用

```html
<script setup>
  import { capitalize } from './helpers'
  import MyComponent from './MyComponent.vue'

  // 变量
  const msg = 'Hello!'

  // 函数
  function log() {
    console.log(msg)
  }
</script>

<template>
  <div @click="log">{{ msg }}</div>
  <div>{{ capitalize('hello') }}</div>
  <MyComponent />
</template>
```

## 动态组件

组件被引用为变量而不是作为字符串键来注册的，使用动态组件时用动态的 `:is` 来绑定

```html
<script setup>
import Foo from './Foo.vue'
import Bar from './Bar.vue'
</script>

<template>
  <component :is="Foo" />
  <component :is="someCondition ? Foo : Bar" />
</template>
```

## 递归组件

一个单文件组件可以通过它的文件名被自身所引用。但这种方式相比 import 导入的组件优先级更低

```html
<!-- ./FooBar.vue -->
<script setup>
  import { FooBar as FooBarChild } from './components'
</script>

<template>
  <!-- 直接书写 -->
  <FooBar/>
  <!-- 同名优先所以起个别名 -->
  <FooBarChild/>
</template>
```

## 命名空间组件

可以使用带点的组件标记, 来直接在模板中写入嵌套组件

```html
<script setup>
  import * as Form from './form-components'
</script>

<template>
  <Form.Input>
    <Form.Label>label</Form.Label>
  </Form.Input>
</template>
```

## defineprops 和 defineEmits

在 `<script setup>` 中必须使用 defineProps 和 defineEmits API 来声明 props 和 emits ，它们具备完整的类型推断并且在 `<script setup>` 中是直接可用的

两者是只有在 `<script setup>` 中才能使用的编译器宏。

```html
<script setup>
  const props = defineProps({
    showText: { type: Boolean, default: true },
    reload: { type: Boolean },
  })
  const emit = defineEmits(['close'])

  if (props.reload) {
    emit('close')
  }
</script>

<template>
  <div v-if="showText"></div>
</template>
```

以上生命 props 和 emits 均使用 vue 运行时生命，可以提供默认值。

此外也可以使用纯类型语法作为参数给 defineProps 和 defineEmits 来声明。但是每次只能使用一种。使用 TS 类型声明时，静态分析会自动生成等效的运行时声明，以消除双重声明的需要并仍然保证正确的运行时行为

采用 TS 的类型声明时，可以通过 withDefault 编译器宏来提供默认值

```html
<script setup lang="ts">
  // 接口形式
  interface Props {
    showText?: boolean
    reload?: boolean
  }
  const props = withDeaults(defineProps<Props>(), {
    showText: true,
    reload: false
  })

  // 直接书写
  const props = withDeaults(defineProps<{
    showText?: boolean
    reload?: boolean
  }>(), {
    showText: true,
    reload: false
  })

  // emit 的书写
  const emit = defineEmits<{
    (e: 'close'): void
  }>()

  if (props.reload) {
    emit('close')
  }
</script>

<template>
  <div v-if="showText"></div>
</template>
```

## defineExpose

使用 '`<script setup>`' 的组件是默认关闭的，不会暴露任何在 '`<script setup>`' 中声明的绑定。为了在 '`<script setup>`' 组件中明确要暴露出去的属性，使用 defineExpose 编译器宏

```html
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

defineExpose({
  a,
  b
})
</script>
```

## 与普通的 `<script>` 一起使用

`<script setup>` 可以和普通的 `<script>` 一起使用。普通的 `<script>` 在有这些需要的情况下或许会被使用到：

1. 无法在 `<script setup>` 声明的选项，例如 inheritAttrs 或通过插件启用的自定义的选项。
2. 声明命名导出。
3. 运行副作用或者创建只需要执行一次的对象。

```html
<script>
// 普通 <script>, 在模块范围下执行(只执行一次)
runSideEffectOnce()

// 声明额外的选项
export default {
  inheritAttrs: false,
  customOptions: {}
}
</script>

<script setup>
// 在 setup() 作用域中执行 (对每个实例皆如此)
</script>
```

## 顶层 await

`<script setup>` 中可以使用顶层 `await`。结果代码会被编译成 `async setup()`; await 的表达式会自动编译成在 await 之后保留当前组件实例上下文的格式。

```html
<script setup>
  const post = await fetch(`/api/post/1`).then(r => r.json())
</script>
```


# 组合式生命周期钩子

从 vue 中导入所需钩子，在 setup 内直接使用

组合式 API 与 option API 大致一致，区别是取消掉了 beforeCreate 和 created ，其他钩子的在 setup 中使用，书写习惯多了前缀 on

因为 setup 是围绕 beforeCreate 和 created 生命周期钩子运行的，所以不要显示地定义它们

option          | composition
--------------- | -----------
beforeMount	    | onBeforeMount
mounted	        | onMounted
beforeUpdate    |	onBeforeUpdate
updated	        | onUpdated
beforeUnmount   |	onBeforeUnmount
unmounted	      | onUnmounted
errorCaptured   |	onErrorCaptured
renderTracked   |	onRenderTracked
renderTriggered |	onRenderTriggered
activated	      | onActivated
deactivated	    | onDeactivated

```ts
import { onMounted, onUpdated, onUnmounted } from 'vue'

const MyComponent = {
  setup() {
    onMounted(() => {
      console.log('mounted!')
    })
    onUpdated(() => {
      console.log('updated!')
    })
    onUnmounted(() => {
      console.log('unmounted!')
    })
  }
}
```



# 响应式书写

[vue3响应式系统](./2-frontend-vue3-reactivity.html)

## toRef

toRef 可以用来为一个 reactive 对象的属性创建一个 ref

被用来获取 props 内属性时，若不存在该属性就新定义一个，toRefs 则不能处理可选的 prop

```ts
export default {
  setup(props) {
    useSomeFeature(toRef(props, 'foo'))
  },
}
```

## toRefs

把一个响应式对象转换成普通对象，该普通对象的每个 property 都是一个 ref ，和响应式对象 property 一一对应。

```ts
const state = reactive({
  foo: 1,
  bar: 2,
})

const stateAsRefs = toRefs(state)

// 可以进行解构赋值
const { foo, bar } = toRefs(state)
```

# defineComponent

defineComponent 只返回传递给它的对象。但是，就类型而言，返回的值有一个合成类型的构造函数，用于手动渲染函数、TSX 和 IDE 工具支持

此外为使 TypeScript 正确推断 Vue 组件选项中的类型，需要使用 defineComponent 全局方法定义组件：

```ts
<script lang="ts">
  import { defineComponent } from 'vue'
  export default defineComponent({
    // 已启用类型推断
  })
</script>
```

# typescript

1. 在 setup() 函数中，不需要将类型传递给 props 参数，会从 props 组件选项推断类型

```ts
const Component = defineComponent({
  props: {
    message: {
      type: String,
      required: true
    }
  },

  setup(props) {
    const result = props.message.split('') // 正确, 'message' 被声明为字符串
    const filtered = props.message.filter(p => p.value) // 将引发错误: Property 'filter' does not exist on type 'string'
  }
})
```

2. 定义 ref 值会根据初始值推断类型

```ts
const Component = defineComponent({
  setup() {
    const year = ref(2020)

    const result = year.value.split('') // => Property 'split' does not exist on type 'number'
  }
})

// 有时也会为 ref 值定义复杂类型
const year = ref<string | number>('2020')
year.value = 2020 // ok!
```

3. 声明 reactive 类型, 有三种表现接口的形式均可

```ts
interface Book {
  title: string
  year?: number
}

export default defineComponent({
  name: 'HelloWorld',
  setup() {
    const book = reactive<Book>({ title: 'Vue 3 Guide' })
    // or
    const book: Book = reactive({ title: 'Vue 3 Guide' })
    // or
    const book = reactive({ title: 'Vue 3 Guide' }) as Book
  }
})
```

4. computed 计算值会根据返回值自动推断类型，若返回值用法不对则报错

# 片段

vue3 支持多根节点的组件，在 template 下一级可以直接书写多个标签，不像 vue2 必须用 div 包裹它们

```html
<template>
  <header>...</header>
  <main v-bind="$attrs">...</main>
  <footer>...</footer>
</template>
```

# teleport

teleport 允许我们控制在 dom 中哪个父节点下渲染 html，而不必使用全局状态或将其拆分成两个组件.

```ts
// teleport 标签将被渲染到 body 节点下
app.component('modal-button', {
  template: `
    <button @click="modalOpen = true">
        Open full screen modal! (With teleport!)
    </button>

    <teleport to="body">
      <div v-if="modalOpen" class="modal">
        <div>
          I'm a teleported modal! 
          (My parent is "body")
          <button @click="modalOpen = false">
            Close
          </button>
        </div>
      </div>
    </teleport>
  `,
  data() {
    return { 
      modalOpen: false
    }
  }
})
```

此外，teleport 内如果包裹 vue 的组件，那其将依旧是 teleport 父组件的逻辑子组件

```ts
// 即使在不同的地方渲染 child-component，
// 它仍将是 parent-component 的子级，
// 并将从中接收 name prop
const app = Vue.createApp({
  template: `
    <h1>Root instance</h1>
    <parent-component />
  `
})

app.component('parent-component', {
  template: `
    <h2>This is a parent component</h2>
    <teleport to="body">
      <child-component name="John" />
    </teleport>
  `
})

app.component('child-component', {
  props: ['name'],
  template: `
    <div>Hello, {{ name }}</div>
  `
})
```

# provide/inject

用于父子组件传值，无论组件层次结构有多深，父组件都可以作为其所有子组件的依赖提供者。父组件使用 provide 提供数据，子组件使用 inject 来提取使用这些数据

provide 和 inject 启用依赖注入。只能在当前活动实例的 setup 中调用。

类型声明：

```ts
interface InjectionKey<T> extends Symbol {}

function provide<T>(key: InjectionKey<T> | string, value: T): void

```

Vue 提供了一个 InjectionKey 接口，该接口是扩展了 Symbol 的泛型类型

```ts
import { InjectionKey, provide, inject } from 'vue'

const key: InjectionKey<string> = Symbol()

provide(key, 'foo') // 若提供非字符串值将出错

const foo = inject(key) // foo 的类型: string | undefined
```