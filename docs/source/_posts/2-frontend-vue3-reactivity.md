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

除 reactive 定义外, 还有 readonly(只读的响应式对象代理), 浅层的代理 shallow, 流程是一样的

reactive 函数接收一个待响应式化的对象, 如果传入的对象存在且对象存在只读属性的标记, 那么就直接返回该对象, 说明该对象是经过 readonly() 响应式化的对象; 若无此问题, 则调用 createReactiveObject 函数创建响应式对象

createReactiveObject 接收该对象, 并标记是否是 readonly , 同时传入可变的 mutableHandlers, 此handler 是提供给 Proxy 代理的配置, 此外也存在 readonly, 以及浅层对应的处理器; 此外还有针对 set、map、weakSet、weakMap 数据结构的 collectionHandlers, 最后的参数是 reactiveMap, 用于缓存根据当前的 target 创建的 proxy 对象的 weakMap 对象

createReactiveObject 首先判断传入的值是否为对象, 若不是则警告返回目标; 接着检测传入的 target 是否已经为一个 proxy 对象, 并且排除传入 readonly 的 proxy 对象, 返回 target 本身;

target 的 proxy 对象有进行缓存设计, 经过 proxy 处理的 target 会被存入 proxyMap, 如果缓存中存在, 则直接提取返回; 

若是 target 的 type 无效, 也会被直接返回

最后给 target 创建 proxy 对象, 判断 target 的类型是否为 COLLECTION, 也就是判断是否为 map set 等数据类型; 来给予不同的 handler; 最后将创建的 proxyMap 缓存

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



```ts

```


## effect



## ref

创建一个 ref 值时, 默认观测是浅层的, 这里 shallow 为 false, 接着创建 RefImpl 实例, 将传入的值用 toRaw toReactive 方法处理;

toRaw 方法首先判断传入的值是否是一个经过 reactive 处理过后的 proxy 对象, 如果是则直接返回; 如果不是则将 raw 传入 toRaw 递归

toReactive 方法判断传入的是否为一个对象, 如果是对象就用 reactive 处理, 如果不是则直接返回

在获取 ref 对象值时, 触发 get() 函数, 首先收集 trackRefValue 依赖, 再返回当前对象的 this._value 值

这里是对外开放 value, 提供了 get 和 set 方法, 所以我们获取 ref 对象后的值需要用 value 获取, 也就是 `ref(100).value`

```ts
// packages\reactivity\src\ref.ts
export function ref(value?: unknown) {
  return createRef(value, false)
}

function createRef(rawValue: unknown, shallow: boolean) {
  if (isRef(rawValue)) {
    return rawValue
  }
  return new RefImpl(rawValue, shallow)
}

class RefImpl<T> {
  private _value: T
  private _rawValue: T

  public dep?: Dep = undefined
  public readonly __v_isRef = true

  constructor(value: T, public readonly _shallow: boolean) {
    this._rawValue = _shallow ? value : toRaw(value)
    this._value = _shallow ? value : toReactive(value)
  }

  get value() {
    trackRefValue(this)
    return this._value
  }

  set value(newVal) {
    newVal = this._shallow ? newVal : toRaw(newVal)
    if (hasChanged(newVal, this._rawValue)) {
      this._rawValue = newVal
      this._value = this._shallow ? newVal : toReactive(newVal)
      triggerRefValue(this, newVal)
    }
  }
}

export function toRaw<T>(observed: T): T {
  const raw = observed && (observed as Target)[ReactiveFlags.RAW]
  return raw ? toRaw(raw) : observed
}

export const toReactive = <T extends unknown>(value: T): T =>
  isObject(value) ? reactive(value) : value

```

## computed

使用 computed 定义一个计算属性

首先会检查传入的第一个参数是否为一个函数, 一般我们只会传入一个基于依赖值返回计算值的函数, 当然也可以传入一个含有 get 与 set 的对象, 指定该计算属性的读取与赋值, 再将函数提取

接着创建一个 computedRefImpl 实例, 每个计算属性 ref 实例会有自己 dep, effect, 以及计算属性缓存 dirty 标识; 

初始 get 获取时 dirty 为 true, 会调用自身的 effect 进行计算 run, 得到经过计算依赖后的值, 同时将 dirty 转为 false, 其他地方有获取该计算属性 ref 的地方, 直接返回当前的值; 会等到再次依赖值变化后, 触发 triggerRefValue, 执行压入 effect 的回调函数 scheduler 再次令 dirty 为 true

```ts
// packages\reactivity\src\computed.ts
export function computed<T>(
  getterOrOptions: ComputedGetter<T> | WritableComputedOptions<T>,
  debugOptions?: DebuggerOptions
) {
  let getter: ComputedGetter<T>
  let setter: ComputedSetter<T>

  const onlyGetter = isFunction(getterOrOptions)
  if (onlyGetter) {
    getter = getterOrOptions
    setter = __DEV__
      ? () => {
          console.warn('Write operation failed: computed value is readonly')
        }
      : NOOP
  } else {
    getter = getterOrOptions.get
    setter = getterOrOptions.set
  }

  const cRef = new ComputedRefImpl(getter, setter, onlyGetter || !setter)

  if (__DEV__ && debugOptions) {
    cRef.effect.onTrack = debugOptions.onTrack
    cRef.effect.onTrigger = debugOptions.onTrigger
  }

  return cRef as any
}

// ComputedRefImpl 类
class ComputedRefImpl<T> {
  public dep?: Dep = undefined

  private _value!: T
  private _dirty = true
  public readonly effect: ReactiveEffect<T>

  public readonly __v_isRef = true
  public readonly [ReactiveFlags.IS_READONLY]: boolean

  constructor(
    getter: ComputedGetter<T>,
    private readonly _setter: ComputedSetter<T>,
    isReadonly: boolean
  ) {
    this.effect = new ReactiveEffect(getter, () => {
      if (!this._dirty) {
        this._dirty = true
        triggerRefValue(this)
      }
    })
    this[ReactiveFlags.IS_READONLY] = isReadonly
  }

  get value() {
    // the computed ref may get wrapped by other proxies e.g. readonly() #3376
    const self = toRaw(this)
    trackRefValue(self)
    if (self._dirty) {
      self._dirty = false
      self._value = self.effect.run()!
    }
    return self._value
  }

  set value(newValue: T) {
    this._setter(newValue)
  }
}
```

# 参考

https://v3.cn.vuejs.org/guide/reactivity.html

https://vue3js.cn/vue-composition-api