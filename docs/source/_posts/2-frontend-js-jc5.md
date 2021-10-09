---
title: 前端 | JS基础 | JS学习笔记Ⅴ：对象
date: 2020-4-27 8:45:21
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

# 对象

> 面向对象编程（Object Oriented Programming，OOP）是目前主流的编程范式。它将真实世界各种复杂的关系，抽象为一个个对象，然后由对象之间的分工与合作，完成对真实世界的模拟。
> 每一个对象都是功能中心，具有明确分工，可以完成接受信息、处理数据、发出信息等任务。对象可以复用，通过继承机制还可以定制。因此，面向对象编程具有灵活、代码可复用、高度模块化等特点，容易维护和开发，比起由一系列函数或指令组成的传统的过程式编程（procedural programming），更适合多人合作的大型软件项目。


**细分说:**
> - 对象是 JavaScript 的基本数据类型
> - 对象是一种复合值: 它将很多值聚合在一起， 通过名字访问这些值
> - 对象是属性的无序集合， 每个属性都是一个值对; 属性名是字符串， 对象即是从字符串到值的映射
> - 除了字符串、数字、布尔值、null、undefined，JavaScript 中的值都是对象
> - 除了保持自有的属性， 对象还可以从一个称为原型的对象继承属性


**对象包括:**
- **属性**
    - 名字
    - 值
    - 属性特性: 除名字和值外， 每个属性还有一些与之相关的值
        - 可写(writable attribute)， 表明是否可以设置该属性的值
        - 可枚举(enumerable attribute)， 表明是否可以通过 for/in 循环返回该函数
        - 可配置(configurable attibute)， 表明是否可以删除或修改该属性
- **对象特性**: 除属性外， 每个对象还拥有三个相关的对象特性
    - 对象的原型(prototype)， 指向另外一个对象， 本对象的属性继承自它的原型对象
    - 对象的类(class)， 一个标识对象类型的字符串
    - 对象的扩展标记(extensible flag)， 指明了是否可以向该对象添加新属性

# 对象/属性的分类

- 内置对象（native object）：由 ES 定义的对象或类。例如数组、函数、日期、正则表达式都是内置对象
- 宿主对象（host object）：由 JavaScript 解释器所嵌入的宿主环境（比如web浏览器）定义的
- 自定义对象（user-defined object）：由运行中的 JavaScript 代码创建的对象

- 自有属性（own property）：直接在对象中定义的属性
- 继承属性（inherited property）：在对象的原型对象中定义的属性

# 对象的操作

## 创建对象

> 可以通过对象直接量、关键字 new、和 Object.create() 函数来创建对象

### 对象直接量

> - 对象直接量是由若干个键值对组成的映射表
> - 属性名可以是 JavaScript 标识符，也可以是字符串直接量，保留字也可以用作不带引号的属性名，如 get、set
> - 属性值可以是任意类型的 JavaScript 表达式

### new 创建对象

> - new 运算符创建并初始化一个新对象
> - new 后跟随一个函数调用，这个函数称作 **构造函数**

```js
// 一些内置构造函数
var o = new Object()    // 创建一个空对象
var o = new Array()     // 创建一个空数组
var o = new Date()      // 创建一个表示当前时间的 Date 对象
```

### Object.create()

> - 每一个 JavaScript 对象(null除外) 都和另一个对象(原型)相关联，每一个对象都从原型继承属性。所有通过对象直接量创建的对象都具有同一个原型对象，并通过 Object.prototype 获取对原型对象的引用通过。
> - 通过 new 关键字和构造函数调用创建的对象的原型就是构造函数的 prototype 属性的值
> - Object.prototype 对象本身并没有原型，它不继承任何属性
> - 所有的内置构造函数(及大部分自定义的构造函数)都具有一个继承自 Object.prototype 的原型

```js
// ES5 定义了 Object.create() 的方法，它创建一个新对象，其中第一个参数是这个对象的原型
var o = Object.create({x:1, y:2})         // o 就继承了属性 x 和 y
var o = Object.create(Object.prototype)   // 创建一个空对象
```

## 属性的读取

> 通过 . 或 [] 来获取属性的值，使用方括号时，键名必须放在引号里

```js
var obj = {
  p: 'Hello World'
};

obj.p    // "Hello World"
obj['p'] // "Hello World"

// 引用对象obj的foo属性时，如果使用点运算符，foo就是字符串；
// 如果使用方括号运算符，但是不使用引号，那么foo就是一个变量，指向字符串bar
var foo = 'bar'
var obj = {
  foo: 1,
  bar: 2
};

obj.foo   // 1
obj[foo]  // 2

// 因此也可以利用反括号传入变量这种特性，进行传入变量
// 通过函数传入参数，讲股票名和购买股数存储在 portfolio 对象里
function addstock (portfolio, stockname, shares) {
    portfolio[stock] = shares   
}

// 使用 for/in 计算 portfolio 对象里的总股价
function getvalue () {
    var total = 0.0
    for (stock in protfolio) {          // 遍历 portfolio 中的每只股票
        var shares = portfolio[stock]   // 得到每只股票的份额
        var price = getquote(stock)     // 查找该股票价格
        total += shares * price         // 加结果累加
    }
    return total
}
```

## 属性的赋值

```js
// 赋值与读取操作一样
var obj = {};

obj.foo = 'Hello';
obj['bar'] = 'World'

// JavaScript 对象的属性是可动态添加的
// 没必要在定义对象的时候，就定义好属性
```

## 属性的继承

- JavaScript 对象具有自由属性，也有一些属性是从原型对象继承来的
- 只有在查询属性时会体会到继承的存在，而设置属性则和继承无关
    - 如果一个对象的属性不是继承来的，那么赋值操作只改变这个自有属性的值
    - 如果这个对象不存在此属性，那么赋值操作会给对象添加一个新属性
    - 如果这个属性是对象继承的，那么这个继承的属性会被新创建的同名属性覆盖(但也仅是继承的，并不会对原型对象做出改动)
    - 如果对象继承了一个只读属性，那么就不能进行赋值操作
    - 如果允许属性赋值操作，那么也只是在原始对象上创建属性或对已有的属性赋值，而不会去修改原型链

## 属性的查看

```js
// 使用 Object.keys() 查看对象本身的所有属性
var obj = {
  key1: 1,
  key2: 2
};

Object.keys(obj)   // ['key1', 'key2']
```

## 属性的删除

> - 删除一个不存在的属性，delete不报错，而且返回true
> - 因此不能根据delete命令的结果，认定某个属性是存在的
> - delete 命令会返回false，那就是该属性存在，且不得删除
> - delete 命令只能删除对象本身的属性，无法删除继承的属性

```js
// 使用 delete 删除对象的属性，删除成功后返回 true
var obj = { p: 1 }
Object.keys(obj)    // ["p"]

delete obj.p        // true
obj.p               // undefined
Object.keys(obj)    // []
```

## 属性的检测

> - in 运算符用于检查对象是否包含某个属性，有 true，没有 false
> - in 运算符不能识别哪些属性是对象自身的，哪些属性是继承的，这个得用 hasOwnProperty 方法

```js
// in，属性是字符串
var obj = { p: 1 }
'p' in obj          // true
'toString' in obj   // true

// hasOwnProperty
var obj = {}
if ('toString' in obj) {
  console.log(obj.hasOwnProperty('toString'))  // false
}
```

## 属性的遍历

> - for...in 遍历的是对象所有可遍历（enumerable）的属性，会跳过不可遍历的属性。
> - 它不仅遍历对象自身的属性，还遍历继承的属性。但是 toString 则是默认不可便利的继承的属性，这时应该结合 hasOwnProperty 方法进行判断
```js
var obj = {
    a: 1, 
    b: 2, 
    c: 3
}

for (var i in obj) {
  console.log('键名：', i)
  console.log('键值：', obj[i])
}
// 键名： a
// 键值： 1
// 键名： b
// 键值： 2
// 键名： c
// 键值： 3
```




# window 对象

>window 对象为浏览器内最底层的对象，里边已存在很多个方法(函数)，隶属于 window 的函数在调用时不用写 window，直接写函数名即可使用

## alert confirm和prompt 

```js
// 此三个函数暂停当前运行, 等用户反馈后再运行
// alert 作为纯提醒的信息
alert('你的电脑将遭受病毒攻击');

// confirm 作为确认一个条件, 有两个选项 是/否
const r = confirm('您是否年满18周岁')  // 是 or 否

// prompt 用于弹出对话框, 输入并提交信息
const name = prompt('你叫什么名字')    // 键入内容
console.log('name:', name)          // 返回内容
```

## setTimeout setInterval和clearInterval

```js
// setTimeout 定时 设置一个时间间隔, 用于未来去执行
setTimeout(function() {
    console.log('2s到了')   // 2s 后返回信息
}, 2000)

// setInterval 按时返回内容
setInterval(function() {
    console.log("Yo.")      // 每隔 2s 返回一次
}, 2000)

// 按时返回递增内容
const count = 0
setInterval(function() {
    count++
    console.log(`Yo. ${count}`)     // 返回递增内容, 使用 ES6 模板字符串拼接
}, 2000)

// clearInterval 按时返回递增内容, 满足指定条件后停止
const count = 0
const timer = setInterval(function() {
    count++
    if (count > 10) {
        clearInterval(timer)    // 达成条件后清除状态
        return
    }
    console.log(`Yo. ${count}`)    // 继续返回递增内容
}, 2000)

// 优先度
setTimeout(function() {
    console.log('时间到了')    // 窗口外
}, 0)
console.log('yo')   // 窗口内, 优先执行
```