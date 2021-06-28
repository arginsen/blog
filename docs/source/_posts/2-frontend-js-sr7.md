---
title: 前端 | JS深入 | 八、this 的原理
date: 2021-5-28
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

# 问题

```js
var obj = { foo:  5 };
```

如上，将一个对象赋值给变量`obj`，`JavaScript`引擎会先在内存里生成一个对象`{foo: 5}`，再把这个对象的内存地址赋值给变量`obj`

所以，变量`obj`是一个地址（reference），如果之后要读取`obj.foo`，引擎会先从`obj`拿到内存地址，然后再从该地址读出原始的对象，返回`foo`属性

原始对象以字典结构保存，每一个属性名都对应一个属性描述对象。如上`foo`属性，实际是以如下形式保存:

```js
{
  foo: {
    [[value]]: 5
    [[writable]]: true
    [[enumerable]]: true
    [[configurable]]: true
  }
}
```

# 缘由

对象的属性值可能是一个函数

```js
var obj = { foo: function () {} }
```

而此时引擎会将函数单独保存在内存中，然后再将**函数的地址**赋值给`foo`属性的`value`属性

由于函数是一个单独的值，所以它可以在不同的环境（上下文）执行

```js
var f = function () {};
var obj = { f: f };

// 单独执行
f()

// obj 环境执行
obj.f()
```

因此需要一种机制，能在函数内部提供当前的运行环境（context），这就是`this`，被用来在函数体内部，指代函数当前的运行环境

```js
var f = function () {
  console.log(this.x);
}

var x = 1;
var obj = {
  f: f,
  x: 2,
};

// 单独执行 当前运行环境即是全局
f() // 1

// obj 环境执行
obj.f() // 2
```

# 区别

```js
var obj = {
  foo: function () { console.log(this.bar) },
  bar: 1
};

// 将obj的属性foo函数赋值给全局的foo，实际是全局下变量foo指向了foo函数的地址，foo在全局下执行
var foo = obj.foo;
var bar = 2;

obj.foo() // 1
foo() // 2
```