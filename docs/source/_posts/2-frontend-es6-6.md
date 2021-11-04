---
title: 前端 | ES6笔记 | 六、Proxy
date: 2020-5-26 18:45:00
tags: 
- ES6
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->

# 概述

Proxy 用于修改某些操作的默认行为，等同于在语言层面做出修改，所有属于一种元编程，即对编程语言进行编程。

Proxy 可以理解成，在目标对象之前架设一层拦截，外界对该对象的访问，都必须先通过这层拦截，因此提供了可以对外界访问进行过滤的改写。

```js
var proxy = new Proxy(target, handler)
```

# proxy 的拦截方法

Proxy 接收两个参数。第一个参数是所要代理的目标对象，第二个参数是一个配置对象，对于每个被代理的操作，需要提供一个对应的处理函数，该函数将拦截对应的操作

Proxy 一共支持的拦截操作有 13 种：

get(target, propKey, receiver) 
拦截对象属性的读取，比如 proxy.foo

set(target, propKey, value, receiver) 
拦截对象属性的设置，比如 proxy.foo = v

has(target, propKey) 
拦截 `propKey in proxy` 的操作，返回一个布尔值

deleteProperty(target, propKey) 
拦截 `delete proxy[propKey]` 的操作，返回一个布尔值

ownKeys(target)
拦截 `Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in` 循环对对象自身属性的读取操作，返回一个数组。该方法返回目标对象所有自身的属性的属性名

getOwnPropertyDescriptor(target, propKey) 
拦截 `Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。

defineProperty(target, propKey, propDesc)
拦截 `Object.defineProperty(proxy, propKey, propDesc)`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。

preventExtensions(target)
拦截 `Object.preventExtensions(proxy)`，返回一个布尔值。

getPrototypeOf(target)
拦截 `Object.getPrototypeOf(proxy)`，返回一个对象。

isExtensible(target)
拦截 `Object.isExtensible(proxy)`，返回一个布尔值。

setPrototypeOf(target, proto)
拦截 `Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。

apply(target, object, args)
拦截 Proxy 实例作为函数调用的操作，比如 `proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。

construct(target, args)
拦截 Proxy 实例作为构造函数调用的操作，比如 `new proxy(...args)`。


## get(target, propKey, receiver) 

get 方法用于拦截某个属性的读取操作，可以接受三个参数，依次为目标对象、属性名和 proxy 实例本身（可选）

例：

```js
var person = {
  name: "张三"
};

var proxy = new Proxy(person, {
  get: function(target, propKey) {
    if (propKey in target) {
      return target[propKey];
    } else {
      throw new ReferenceError("Prop name \"" + propKey + "\" does not exist.");
    }
  }
});

proxy.name // "张三"
proxy.age // 抛出一个错误
```

get 方法可以继承：
proxy 实例可以作为其他对象的实例，拦截操作定义在 prototype 对象上，如果读取 obj 对象继承的属性时，拦截也会生效

```js
let proto = new Proxy({}, {
  get(target, propertyKey, receiver) {
    console.log('GET ' + propertyKey);
    return target[propertyKey];
  }
});

let obj = Object.create(proto);
obj.foo // "GET foo"
```

利用 proxy 实现数组读取负数的索引

```js
function createArray(...elements) {
  let handler = {
    get(target, propKey, receiver) {
      let index = Number(propKey)
      if (index < 0) {
        propKey = String(target.length + index)
      }
      return Reflect.get(target, propKey, receiver)
    }
  }

  let target = []
  target.push(...elements)
  return new Proxy(target, handler)
}

let arr = createArray('a', 'b', 'c')
arr[-1] // 'c'
```

利用 proxy，将读取属性的操作转为执行某个函数，实现属性的链式操作

```js
// 根据闭包的特性，用 funcStack 存储被调用到的方法，
// 再用 reduce 依次执行各方法进行累积，起始值是初始传入的 value
// 直接调用 pipe(3) 会返回一个 proxy 对象，在 proxy 基础上调用其他方法就是通过 get 读取属性
// 如果当前读取的属性不为 get ，那么就将该属性方法缓存起来继续返回当前的 proxy 对象，直到 get
function pipe (value) {
  let funcStack = []
  let proxy = new Proxy({}, {
    get(pipeObject, fnName) {
      if (fnName === 'get') {
        return funcStack.reduce(function (val, fn) {
          return fn(val)
        }, value)
      }
      funcStack.push(window[fnName])
      return proxy
    }
  })
  return proxy
}

var double = n => n * 2
var pow = n => n * n
var reverseInt = n => n.toString.split("").reverse().join("") | 0

pipe(3).double.pow.reverseInt.get // 63
```

## set(target, propKey, value, receiver) 

set 方法用来拦截某个属性的赋值操作，可以接受四个参数，以此为目标对象、属性名、属性值、Proxy 实例（可选）

例：

在 set 存值作以拦截，禁止设置下划线开头的内部属性
在 get 读值作以拦截，禁止读取下划线开头的内部属性

```js
const handler = {
  get (target, key) {
    invariant(key, 'get');
    return target[key];
  },
  set (target, key, value) {
    invariant(key, 'set');
    target[key] = value;
    return true;
  }
};
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}
const target = {};
const proxy = new Proxy(target, handler);
proxy._prop
// Error: Invalid attempt to get private "_prop" property
proxy._prop = 'c'
// Error: Invalid attempt to set private "_prop" property
```

set 代理应该返回一个布尔值。在严格模式下，set 代理如果没有返回 true ，就会报错

## has(target, propKey) 

has 方法用来拦截HasProperty操作，即判断对象是否具有某个属性时，这个方法会生效。典型的操作就是in运算符。

has 方法接受两个参数，分别是目标对象、需查询的属性名。

例：

使用 has 方法隐藏下划线开头的内部属性，不被in运算符发现。

```js
var handler = {
  has (target, key) {
    if (key[0] === '_') {
      return false;
    }
    return key in target;
  }
};
var target = { _prop: 'foo', prop: 'foo' };
var proxy = new Proxy(target, handler);
'_prop' in proxy // false
```

has() 方法拦截的是 HasProperty 操作，而不是 HasOwnProperty 操作，即 has() 方法不判断一个属性是对象自身的属性，还是继承的属性。

另外，虽然 for...in 循环也用到了 in 运算符，但是 has() 拦截对 for...in 循环不生效。

```js
let stu1 = {name: '张三', score: 59};
let stu2 = {name: '李四', score: 99};

let handler = {
  has(target, prop) {
    if (prop === 'score' && target[prop] < 60) {
      console.log(`${target.name} 不及格`);
      return false;
    }
    return prop in target;
  }
}

let oproxy1 = new Proxy(stu1, handler);
let oproxy2 = new Proxy(stu2, handler);

'score' in oproxy1
// 张三 不及格
// false

'score' in oproxy2
// true

// for in 无变化
for (let a in oproxy1) {
  console.log(oproxy1[a]);
}
// 张三
// 59

for (let b in oproxy2) {
  console.log(oproxy2[b]);
}
// 李四
// 99
```

## ownKeys(target) 

ownKeys 方法用来拦截对象自身属性的读取操作，具体来说拦截以下操作

`Object.getOwnPropertyNames()`
`Object.getOwnPropertySymbols()`
`Object.keys()`
`for...in` 循环

对于 Object.keys() 方法，有三类属性会被 ownKeys() 自动过滤，不会返回：

1.目标对象上不存在的属性
2.属性名为 Symbol 值
3.不可遍历（enumerable）的属性

## apply(target, object, args)

apply 方法拦截函数的调用、call 操作、apply 操作

apply 方法接受三个参数，分别是目标对象、目标对象的上下文对象（this）、目标对象的参数数组

例：

对函数对象 sum 进行代理，给函数执行返回的结果乘 2 倍

```js
var twice = {
  apply (target, ctx, args) {
    return Reflect.apply(...arguments) * 2;
  }
};
function sum (left, right) {
  return left + right;
};
var proxy = new Proxy(sum, twice);
proxy(1, 2) // 6
proxy.call(null, 5, 6) // 22
proxy.apply(null, [7, 8]) // 30
```