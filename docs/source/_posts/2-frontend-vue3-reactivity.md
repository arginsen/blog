---
title: 前端 | vue3 响应式学习
date: 2021-10-8 18:45:00
tags: 
- Vue
- Frontend
categories: notes
photos:
    - /blog/img/vue.jpg
---

<br>
<!--more-->


# 实现响应式

## reactive

为 JavaScript 对象创建响应式状态，可以使用 reactive 方法

reactive 接收一个对象，返回一个经由 Proxy 代理的 proxy 对象

```ts
import { reactive } from 'vue'

// 响应式状态
const state = reactive({
  count: 0
})
```

## ref

如果针对原始值转变为响应式，可以使用 ref ，其内也就是创建一个相同值的 property 对象，将其传给 reactive 进行响应式化，之后返回一个可变的响应式对象，该对象作为一个响应式的引用维护着它内部的值。最后我们用该对象的 value 属性读取原来的值

```js
import { ref } from 'vue'

const count = ref(0)
console.log(count.value) // 0
```

### 解包

1. 当 ref 作为渲染上下文上的 property 返回并在模板使用时，它将自动浅层次解包内部值，在访问嵌套的 ref 时则需要再追加 value

```html
<template>
  <div>
    <span>{{ count }}</span>
    <button @click="count ++">Increment count</button>
    <button @click="nested.count.value ++">Nested Increment count</button>
  </div>
</template>

<script>
  import { ref } from 'vue'
  export default {
    setup() {
      const count = ref(0)
      return {
        count, // 直接访问

        nested: { // 嵌套
          count
        }
      }
    }
  }
</script>
```

2. 当 ref 作为响应式对象的 property 被访问或更改时，它也会自动解包内部值，不用写 value了

ref 解包进发生在被响应式 Object 嵌套的时候，当从 Array 或原生集合如 Map 访问 ref 时，不会进行解包

```ts
const count = ref(0)
const state = reactive({ count })

console.log(state.count) // 1
```

如果将新的 ref 赋值给现有 ref 的 property，将会替换旧的 ref, 但下方本来的 count 作为 ref 值并不改变

```ts
const otherCount = ref(2)

state.count = otherCount
console.log(state.count) // 2
console.log(count.value) // 1
```


## 响应式状态的解构

经由 reactive 包裹的响应式对象，如果用解构的方式单独获取内部属性时，属性会失去响应式，使用必须直接用响应式对象读取。

或者可以使用 toRefs 将响应式对象转换为一组 ref 对象，而这些 ref 将保留与源对象的响应式关联

```ts
import { reactive, toRefs } from 'vue'

const book = reactive({
    author: 'Evan',
    year: '2020',
    title: 'Human History'
})

let { author, title } = book // 两属性失去响应式关联
let { author, title } = toRefs(book) // 依旧保持响应式

title.value = 'Animals History'
console.log(book.title) // 'Animals History'
```

## 使用 readonly 创建只读不能更改的响应式对象

在 reactive 源码里，同样定义了 readonly 函数，其会返回 createReactiveObject() 创建的响应式对象，其中该函数第二个参数 isReadonly 标记为 true，因此会用 readonlyHandlers 返回只读的 proxy 对象

```ts
import { reactive, readonly } from 'vue'

const original = reactive({ count: 0 })
const copy = readonly(original)

original.count++
copy.count++ // 警告: "Set operation on key 'count' failed:
```

## 计算值

computed 函数接收 getter 函数并为 getter 返回值返回一个不可变的响应式 ref 对象

```ts
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2
plusOne.value++ // error
```

此外可以手动写入一个包含 get 和 set 函数的对象来创建一个可写的 ref 对象

```ts
const count = ref(1)
const plusOne = computed({
    get: () => count.value + 1,
    set: val => {
        count.value = val - 1
    }
})

plusOne.value // 1

plusOne.value = 1
console.log(count.value) // 0
```

## watchEffect

立即执行传入的一个函数，并响应式追踪其依赖，并在其依赖变更时重新运行该函数

目的：根据响应式状态 **自动应用** 和 **重新应用** 副作用

```ts
const count = ref(0)

watchEffect(() => console.log(count.value)) // 立即执行首次打印

setTimeout(() => {
    count.value++ // count 更新再次触发函数
}, 100)
```

### 停止侦听

当 watchEffect 在组件的 setup() 函数或生命周期钩子被调用时， 侦听器会被链接到该组件的生命周期，并在组件卸载时自动停止。

在一些情况下，也可以显式调用返回值以停止侦听：

```ts
const linsen = watchEffect(() => {
  /* ... */
})

// 调用返回值停止
linsen()
```

### 清除副作用

有时副作用函数会执行一些异步的副作用，这些响应需要在其失效时清除（达成完成条件），所以侦听副作用传入的函数可以接收一个 onInvalidate 函数作入参，用来注册清理失效时的回调。

onInvalidate 函数会在副作用即将重新执行时、侦听器被停止 (如果在 `setup()` 或生命周期钩子函数中使用了 watchEffect，则在组件卸载时) 触发

之所以是通过传入一个函数去注册失效回调，而不是从回调返回它，是因为返回值对于异步错误处理很重要

```ts
watchEffect(onInvalidate => {
  const token = performAsyncOperation(id.value)
  onInvalidate(() => {
    token.cancel()
  })
})
```

### 副作用刷新时机

vue 的响应式系统会缓存副作用函数，并异步地进行刷新，这样可以避免在同一个 tick 中多个状态改变导致的不必要的重复调用。

组件的 update 函数也是一个被侦听的副作用，当一个用户定义的副作用函数进入队列时，默认会在所有组件 update 前

```html
<template>
  <div>{{ count }}</div>
</template>

<script>
export default {
  setup() {
    const count = ref(0)

    watchEffect(() => {
      console.log(count.value) // 在初始运行时会同步打印，count 更改时，先于组件更新被执行
    })

    // 给 watchEffect 传入第二个参数：附加的 option 对象配置 flush
    // pre 即是在组件更新前执行，post 即为在更新后执行
    watchEffect(
      () => {
        /* ... */
      },
      {
        flush: 'post'
      }
    )

    return {
      count
    }
  }
}
</script>
```

## watch

watch 需要侦听特定的数据源，并在回调函数中执行副作用。默认情况下，它是惰性的，即只有当被侦听的源发生变化时才执行回调

与 watchEffect 相比，watch ：

1. 懒执行副作用
2. 更具体地说明什么状态应该触发侦听器重新运行
3. 访问侦听状态变化前后的值

此外 watch 与 watchEffect 共享着此类行为：停止侦听、清除副作用（相应 onInvalidate 会作为回调的第三个参数传入 watch）、副作用刷新时机、侦听器调试

### 侦听单个数据源

侦听器数据源可以是返回值的 getter 函数，也可以直接是 ref

```ts
// 侦听一个 getter
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)

// 直接侦听ref
const count = ref(0)
watch(count, (count, prevCount) => {
  /* ... */
})
```

### 侦听多个数据源

侦听器可以使用数组同时侦听多个源

```ts
const firstName = ref('')
const lastName = ref('')

watch([firstName, lastName], (newValues, prevValues) => {
  console.log(newValues, prevValues)
})

firstName.value = 'John' // logs: ["John", ""] ["", ""]
lastName.value = 'Smith' // logs: ["John", "Smith"] ["John", ""]
```

### 侦听响应式对象

使用侦听器来比较一个数组或对象的值，这些值是响应式的，要求它有一个由值构成的副本

```ts
const numbers = reactive([1,2,3,4])

watch(
  () => [...numbers],
  (numbers, prevNumbers) => {
    console.log(numbers, prevNumbers)
  }
)

numbers.push(5)
```

尝试检查深度嵌套对象或数组中的 property 变化时，需要 deep 选项设置为 true，但是可以读取到值并不能侦听到变化。因为在侦听响应式对象或数组将始终返回该对象的当前值和上一个状态值的引用。

```ts
const state = reactive({
  id: 1,
  attributes: {
    name: '',
  }
})

watch(
  () => state,
  (state, prevState) => {
    console.log('deep', state.attributes.name, prevState.attributes.name)
  },
  { deep: true }
)

state.attributes.name = 'Alex' // "deep" "Alex" "Alex"
```

想要完全侦听深度嵌套的对象和数组，可能需要对值进行深拷贝。可以通过 lodash.cloneDeep 工具函数实现

```ts
import _ from 'lodash'

const state = reactive({
  id: 1,
  attributes: {
    name: '',
  }
})

watch(
  () => _.cloneDeep(state),
  (sate, prevState) => {
    console.log(state.attributes.name, prevState.attributes.name)
  }
)

state.attributes.name = 'Alex' // 日志: "Alex" ""
```


# 响应式原理

vue3 的响应式系统放在 package 下的 reactivity 目录。

reactivity 单独作为一个包存在，是内嵌到 vue 渲染器中，它也可以单独发布或者被第三方引用。

reactivity 的目录结构：

```ts
.
├── LICENSE
├── README.md
├── __tests__  // 单元测试目录
│   ├── collections
│   │   ├── Map.spec.ts
│   │   ├── Set.spec.ts
│   │   ├── WeakMap.spec.ts
│   │   └── WeakSet.spec.ts
│   ├── computed.spec.ts
│   ├── effect.spec.ts
│   ├── reactive.spec.ts
│   ├── reactiveArray.spec.ts
│   ├── readonly.spec.ts
│   └── ref.spec.ts
├── api-extractor.json
├── index.js
├── package.json
└── src
    ├── baseHandlers.ts // 基本类型的处理器
    ├── collectionHandlers.ts  // Set Map WeakSet WeckMap的处理器
    ├── computed.ts // 计算属性，同 Vue2
    ├── DeferredComputed.ts 
    ├── dep.ts 
    ├── effect.ts // reactive 核心，处理依赖收集，依赖更新
    ├── effectScope.ts // effect 作用域对象，用以捕获在其内部创建的响应式 effect
    ├── index.ts
    ├── operations.ts // 定义依赖收集，依赖更新的类型
    ├── reactive.ts // reactive 入口，内部主要以 Proxy 实现
    ├── ref.ts // Proxy 处理不了基础值类型的响应，Ref 来处理
    └── warning.ts // 定义一个 warn 的方法用以抛出 vue 警告
```

## reactive

通常我们在 setup() 中使用 reactive 来创建响应式对象

```ts
export defineComponent({
  setup() {
    const obj = reactive({
      count: 0
    })

    return {
      obj
    }
  }
})
```

reactive 的实现是由 proxy 和 effect 组合形成的

对传入的对象作以筛选过滤，已有、不符合的直接返回目标，否则新建一个代理

```ts
// 使用 weakMap 存储目标对象+目标对象的代理
export const reactiveMap = new WeakMap<Target, any>()
export const readonlyMap = new WeakMap<Target, any>()

export function reactive<T extends object>(target: T): UnwrapNestedRefs<T>
export function reactive(target: object) {
  // if trying to observe a readonly proxy, return the readonly version.
  // 已经过 readonly 处理的响应式对象直接返回
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

// readonly 创建只读不能更改的响应式对象，
// 返回的创建函数第二个参数 isReadonly 为 true，
// 同时传给 proxy 的 handler 也是 readonly 专有的
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

function createReactiveObject(
  target: Target,
  isReadonly: boolean,
  baseHandlers: ProxyHandler<any>,
  collectionHandlers: ProxyHandler<any>,
  proxyMap: WeakMap<Target, any>
) {
  if (!isObject(target)) { // 如传入目标不是对象，抛出警告
    if (__DEV__) {
      console.warn(`value cannot be made reactive: ${String(target)}`)
    }
    return target
  }
  // target is already a Proxy, return it.
  // exception: calling readonly() on a reactive object
  // 传入目标是 proxy 对象，且使用 readonly 处理已代理对象的除外，直接返回
  if (
    target[ReactiveFlags.RAW] &&
    !(isReadonly && target[ReactiveFlags.IS_REACTIVE])
  ) {
    return target
  }
  // target already has corresponding Proxy
  // 传入目标已存在相匹配的代理对象则直接返回
  const existingProxy = proxyMap.get(target)
  if (existingProxy) {
    return existingProxy
  }
  // only a whitelist of value types can be observed.
  // 仅有传入值类型白名单内的才可以响应式化，无效目标直接返回
  const targetType = getTargetType(target)
  if (targetType === TargetType.INVALID) {
    return target
  }
  // 创建 proxy 代理
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  proxyMap.set(target, proxy)
  return proxy
}

// 可见白名单的 target 有 Object,Array,Map,Set,WeakMap,WeakSet
function targetTypeMap(rawType: string) {
  switch (rawType) {
    case 'Object':
    case 'Array':
      return TargetType.COMMON
    case 'Map':
    case 'Set':
    case 'WeakMap':
    case 'WeakSet':
      return TargetType.COLLECTION
    default:
      return TargetType.INVALID
  }
}

function getTargetType(value: Target) {
  return value[ReactiveFlags.SKIP] || !Object.isExtensible(value)
    ? TargetType.INVALID
    : targetTypeMap(toRawType(value))
}
```




## baseHandler


## effect


# 参考

https://v3.cn.vuejs.org/guide/reactivity.html

https://vue3js.cn/vue-composition-api