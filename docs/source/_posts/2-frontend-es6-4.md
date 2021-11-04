---
title: 前端 | ES6笔记 | 四、Symbol
date: 2020-5-25 12:45:21
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

> - ES6 引入了一种新的**原始数据类型** Symbol，表示独一无二的值。它是 JavaScript 语言的第七种数据类型，前六种是：undefined、null、布尔值（Boolean）、字符串（String）、数值（Number）、对象（Object）。
> - Symbol 值通过 **Symbol 函数**生成。这就是说，对象的属性名现在可以有两种类型，一种是原来就有的字符串，另一种就是新增的 Symbol 类型。凡是属性名属于 Symbol 类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。
> - Symbol 函数前不能使用 new 命令，因为生成的 Symbol 是一个**原始类型的值**，不是对象。由于 Symbol 值不是对象，所以不能添加属性
> - Symbol 函数可以接受一个**字符串**作为参数，表示对 Symbol 实例的**描述**，如果 Symbol 的参数是一个对象，就会调用该对象的 toString 方法，将其转为字符串，然后才生成一个 Symbol 值。
> - Symbol 值不能与其他类型的值进行运算，Symbol 值可以显式转为**字符串**，也可以转为**布尔值**，但是**不能转为数值**


```js
// s 值表示一个独一无二的值
let s = Symbol()

// 变量 s 的数据类型为 Symbol
typeof s   // "symbol"
Object.prototype.toString.call(s)  // "[object Symbol]"

// 传入字符串参数对 Symbol 实例进行描述，便于区分
let s1 = Symbol('foo');
let s2 = Symbol('bar');

s1  // Symbol(foo)
s2  // Symbol(bar)

// 当参数为一个对象，调用 toString 方法，将对象转为字符串
const obj = {
  toString() {
    return 'abc';
  }
};
const sym = Symbol(obj);
sym // Symbol(abc)

// 理解 参数仅作为对当前 Symbol 值的描述
// 没有参数的情况
let s1 = Symbol();
let s2 = Symbol();

s1 === s2 // false

// 有参数的情况
let s1 = Symbol('foo');
let s2 = Symbol('foo');

s1 === s2 // false


// ES2019 提供了 Symbol 实例属性 description，
// 直接返回 Symbol 的描述
const sym = Symbol('foo');
sym.description   // "foo"
```


# 作为属性名

> - 每一个 Symbol 值都是不相等的，这意味着 Symbol 值可以作为**标识符**，用于对象的属性名，就能保证不会出现同名的属性。这对于一个对象由多个模块构成的情况非常有用，能防止某一个键被不小心改写或覆盖。
> - Symbol 值作为对象属性名时，不能用**点运算符**。
> - 在对象的内部，使用 Symbol 值定义属性时，Symbol 值必须放在**方括号**之中。

```js
let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
let a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"


// 在对象的内部使用 Symbol 值定义属性
let s = Symbol();

let obj = {
  [s]: function (arg) { ... }
};


// Symbol 类型还可以用于定义一组常量，保证这组常量的值都是不相等的
const log = {};

log.levels = {
  DEBUG: Symbol('debug'),
  INFO: Symbol('info'),
  WARN: Symbol('warn')
};
console.log(log.levels.DEBUG, 'debug message');
console.log(log.levels.INFO, 'info message');
```

# 消除魔术字符串


> 魔术字符串指的是，在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值，不利于将来的修改和维护。风格良好的代码，应该尽量消除魔术字符串，改由含义清晰的变量代替。


```js
// Triangle 作为一个魔术字符串
function getArea(shape, options) {
  let area = 0;

  switch (shape) {
    case 'Triangle': // 魔术字符串
      area = .5 * options.width * options.height;
      break;
    /* ... more code ... */
  }

  return area;
}

getArea('Triangle', { width: 100, height: 100 }); // 魔术字符串


// 一般消除魔术字符串，就是将它写为变量，有利于后期维护
const shapeType = {
  triangle: 'Triangle'
};

function getArea(shape, options) {
  let area = 0;
  switch (shape) {
    case shapeType.triangle:
      area = .5 * options.width * options.height;
      break;
  }
  return area;
}

getArea(shapeType.triangle, { width: 100, height: 100 });

// 将 triangle 的值设为 symbol 值，使每个 case 不同即可
const shapeType = {
  triangle: Symbol()
};
```

# 属性名的遍历

> - Symbol 作为属性名，遍历对象的时候，该属性不会出现在for...in、for...of循环中，也不会被 Object.keys()、Object.getOwnPropertyNames()、JSON.stringify() 返回。
> - Symbol 作为属性名，有一个 Object.getOwnPropertySymbols() 方法，可以获取指定对象的所有 Symbol 属性名。该方法返回一个数组，成员是当前对象的所有用作属性名的 Symbol 值

```js
let a = Symbol('a');
let b = Symbol('b');

// 将两个 symbol 值作为属性存入 obj 对象
const obj = {};
obj[a] = 'Hello';
obj[b] = 'World';

const objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols   // [Symbol(a), Symbol(b)]


// 应用一个新的 API，Reflect.ownKeys() 方法
// 可以返回所有类型的键名，包括常规键名和 Symbol 键名。
let obj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
};

Reflect.ownKeys(obj)
//  ["enum", "nonEnum", Symbol(my_key)]
```

# Symbol.for()，Symbol.keyFor()

> - Symbol.for() 接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建一个以该字符串为名称的 Symbol 值，并将其注册到全局。以此实现同一个 Symbol 值重新使用
> - Symbol.keyFor() 方法返回一个已登记的 Symbol 类型值的 key。

```js
// 应用 Symbol.for()，返回同一个 symbol 值
let s1 = Symbol.for('foo');
let s2 = Symbol.for('foo');

s1 === s2 // true


// Symbol() 写法没有登记机制，只是描述；
// Symbol.for() 会登记标识符key
let s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

let s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined
```



# 参考资料

[网道 - ES6教程 - Symbol](https://wangdoc.com/es6/symbol.htm)

