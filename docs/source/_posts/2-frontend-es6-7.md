---
title: 前端 | ES6笔记 | 七、Reflect
date: 2020-5-27 8:09:00
tags: 
- ES6
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->

Reflect 对象与 Proxy 对象一样，也是 ES6 为了操作对象而提供的新 API 

# Reflect 对象的特点

Reflect 对象将 Object 对象的一明显属于语言内部的方法，放到 Reflect 对象上部署，未来的新方法将只部署在 Reflect 对象上。也就是说，从 Reflect 对象上可以拿到语言内部的方法

Reflect 对象修改某些 Object 方法的返回结果，让其变得更合理。比如 Object.defineProperty(obj, name, desc) 在无法定义属性时，会抛出一个错误，而 Reflect.defineProperty(obj, name, desc) 则会返回 false

Reflect 对象让 Object 操作都变成函数行为。某些 Object 操作是命令式，比如 `name in obj`和 `delete obj[name]`，而 `Reflect.has(obj, name)` 和 `Reflect.deleteProperty(obj, name)` 让它们变成了函数行为。

Reflect 对象的方法与 Proxy 对象的方法一一对应，只要是 Proxy 对象的方法，就能在 Reflect 对象上找到对应的方法。这就让 Proxy 对象可以方便的调用对应的 Reflect 方法，完成默认行为，也就是说，不管 Proxy 怎么修改默认行为，在 Reflect 上总可以获取默认行为

```js
// 每一个拦截操作，内部都用 Reflect 保证原生方法能够正常运行
var loggedObj = new Proxy(obj, {
  get(target, name) {
    console.log('get', target, name);
    return Reflect.get(target, name);
  },
  deleteProperty(target, name) {
    console.log('delete' + name);
    return Reflect.deleteProperty(target, name);
  },
  has(target, name) {
    console.log('has' + name);
    return Reflect.has(target, name);
  }
});
```

# Reflect 静态方法

## Reflect.get(target, name, receiver)

Reflect.get 方法查找并返回 target 对象的 name 属性，如果没有该属性，则返回 undefined 

```js
var myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar;
  },
}

Reflect.get(myObject, 'foo') // 1
Reflect.get(myObject, 'bar') // 2
Reflect.get(myObject, 'baz') // 3
```

如果 name 属性部署了读取函数 getter，则读取函数的 this 绑定 receiver

```js
var myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar;
  },
};

var myReceiverObject = {
  foo: 4,
  bar: 4,
};

Reflect.get(myObject, 'baz', myReceiverObject) // 8
```

## Reflect.set(target, name, value, receiver)

Reflect.set 方法设置 target 对象的 name 属性等于 value

```js
  foo: 1,
  set bar(value) {
    return this.foo = value;
  },
}

myObject.foo // 1

Reflect.set(myObject, 'foo', 2);
myObject.foo // 2

Reflect.set(myObject, 'bar', 3)
myObject.foo // 3
```

同 get 一样，如果 name 属性设置了赋值函数，则赋值函数的 this 绑定 receiver

```js
var myObject = {
  foo: 4,
  set bar(value) {
    return this.foo = value;
  },
};

var myReceiverObject = {
  foo: 0,
};

Reflect.set(myObject, 'bar', 1, myReceiverObject);
myObject.foo // 4
myReceiverObject.foo // 1
```

## Reflect.has(obj, name)

Reflect.has 方法对应 `name in obj` 里面的 in 运算符。

```js
var myObject = {
  foo: 1,
};

// 旧写法
'foo' in myObject // true

// 新写法
Reflect.has(myObject, 'foo') // true
```

## Reflect.deleteProperty(obj, name)

Reflect.deleteProperty 方法等同于 `delete obj[name]`，用于删除对象的属性。

```js
const myObj = { foo: 'bar' };

// 旧写法
delete myObj.foo;

// 新写法
Reflect.deleteProperty(myObj, 'foo');
```

该方法返回一个布尔值。如果删除成功，或者被删除的属性不存在，返回true；删除失败，被删除的属性依然存在，返回false。

如果 Reflect.deleteProperty() 方法的第一个参数不是对象，会报错。

## Reflect.construct(target, args)

Reflect.construct 方法等同于 `new target(...args)`，这提供了一种不使用 new，来调用构造函数的方法。

```js
function Greeting(name) {
  this.name = name;
}

// new 的写法
const instance = new Greeting('张三');

// Reflect.construct 的写法
const instance = Reflect.construct(Greeting, ['张三']);
```

如果Reflect.construct()方法的第一个参数不是函数，会报错。

## Reflect.getPrototypeOf(obj)

Reflect.getPrototypeOf 方法用于读取对象的 `__proto__` 属性，对应 Object.getPrototypeOf(obj)。

```js
const myObj = new FancyThing();

// 旧写法
Object.getPrototypeOf(myObj) === FancyThing.prototype;

// 新写法
Reflect.getPrototypeOf(myObj) === FancyThing.prototype;
```

Reflect.getPrototypeOf 和 Object.getPrototypeOf 的一个区别是:
如果参数不是对象，Object.getPrototypeOf 会将这个参数转为对象，然后再运行，而 Reflect.getPrototypeOf 会报错。

```js
Object.getPrototypeOf(1) // Number {[[PrimitiveValue]]: 0}
Reflect.getPrototypeOf(1) // 报错
```

## Reflect.setPrototypeOf(obj, newProto)

Reflect.setPrototypeOf 方法用于设置目标对象的原型（prototype），对应 Object.setPrototypeOf(obj, newProto) 方法。它返回一个布尔值，表示是否设置成功。

```js
const myObj = {};

// 旧写法
Object.setPrototypeOf(myObj, Array.prototype);

// 新写法
Reflect.setPrototypeOf(myObj, Array.prototype);

myObj.length // 0
```

如果无法设置目标对象的原型（比如，目标对象禁止扩展），Reflect.setPrototypeOf 方法返回 false。

```js
Reflect.setPrototypeOf({}, null)
// true
Reflect.setPrototypeOf(Object.freeze({}), null)
// false
```

如果第一个参数不是对象，Object.setPrototypeOf 会返回第一个参数本身，而 Reflect.setPrototypeOf 会报错。

```js
Object.setPrototypeOf(1, {})
// 1

Reflect.setPrototypeOf(1, {})
// TypeError: Reflect.setPrototypeOf called on non-object
```

如果第一个参数是 undefined 或 null，Object.setPrototypeOf 和 Reflect.setPrototypeOf 都会报错。

## Reflect.apply(func, thisArg, args)

Reflect.apply方法等同于Function.prototype.apply.call(func, thisArg, args)，用于绑定this对象后执行给定函数。

一般来说，如果要绑定一个函数的this对象，可以这样写fn.apply(obj, args)，但是如果函数定义了自己的apply方法，就只能写成Function.prototype.apply.call(fn, obj, args)，采用Reflect对象可以简化这种操作。

```js
const ages = [11, 33, 12, 54, 18, 96];

// 旧写法
const youngest = Math.min.apply(Math, ages);
const oldest = Math.max.apply(Math, ages);
const type = Object.prototype.toString.call(youngest);

// 新写法
const youngest = Reflect.apply(Math.min, Math, ages);
const oldest = Reflect.apply(Math.max, Math, ages);
const type = Reflect.apply(Object.prototype.toString, youngest, []);
```

## Reflect.ownKeys(target)

Reflect.ownKeys 方法用于返回对象的所有属性，基本等同于 Object.getOwnPropertyNames 与 Object.getOwnPropertySymbols 之和。

```js
var myObject = {
  foo: 1,
  bar: 2,
  [Symbol.for('baz')]: 3,
  [Symbol.for('bing')]: 4,
};

// 旧写法
Object.getOwnPropertyNames(myObject)
// ['foo', 'bar']

Object.getOwnPropertySymbols(myObject)
//[Symbol(baz), Symbol(bing)]

// 新写法
Reflect.ownKeys(myObject)
// ['foo', 'bar', Symbol(baz), Symbol(bing)]
```

如果 Reflect.ownKeys() 方法的第一个参数不是对象，会报错。
