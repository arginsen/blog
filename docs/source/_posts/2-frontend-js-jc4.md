---
title: 前端 | JS基础 | JS学习笔记Ⅳ：函数
date: 2020-4-26
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

# 函数定义

- 两种方式定义函数：函数定义表达式、函数申明语句
- 函数名称是函数声明语句必需的部分, 在函数定义表达式里是可选的
- 函数声明语句中函数名称为一个变量, 新定义的函数对象也会赋值给该变量, **函数被提前**到顶部, 可以被之前代码调用
- 函数定义表达式通过 var 将函数对象 f 赋值给另一个变量 a, 此时只有变量声明 a 被提前, 而变量初始化代码(函数对象f)仍在原来位置, 这个可选的函数名 f 仅作为该函数内部的一个局部变量

```js
var a = function f() {} // 将表达式赋值给变量 a, a 会被提前, 但函数表达式仍在原处
function f () {}        // 声明一个变量 f, 指向函数对象, 整个都会被提前
```

# 函数的作用

- 封装代码
- 控制数据流
- 作用域

```js
// 封装代码 控制数据流
function buy (money) {
    if(!money) {
        alert('警察叔叔就是这个人！')
    } else {
        alert('这是找您的钱，老板慢走')
    }
}

buy(110)


// 作用域
var a = 1       // 全局变量
console.log(a)

function fun () {
    var b = 2   // 局部变量
}
console.log(b)  // 此种状态下不能输出

// 同一作用域
function fun () {
    var b = 2
    console.log(b) // 此时可以输出
    console.log(a) // 变量a作为全局变量可以在函数内部获取
}

fun()

// 作用域污染, 在函数体里定义变量, 就不会污染全局变量
;(function() {
    // 填充代码
})
();     // () 触发函数


// 传参  JS 中没有语言层面的必传参数
function fun(a, b, c) {
  console.log(1)
}

fun();  // 1  没传参也可正常执行

// 定义必须 传参
function 买买买(钱) {
  if(钱 === undefined)           // 如果没有传入参数
    throw new Error('钱呢？！')  // 报错并停止执行

  // ...
  console.log('爸爸！')
}

买买买()    // Uncaught Error: 钱呢？！
买买买(666) // "爸爸！"


// return 返回执行结果
function 谁最美() {
  return '你'
}

var 美人 = 谁最美()
console.log(美人)  // "你"

// 在函数中一旦return，之后的所有语句都不会执行
// 不可多次返回
function yo() {
  return 'biu'

  // 以下内容通通不会执行
  console.log('yo')
  console.log('yoo')
  console.log('yooo')
}

yo()  // "biu"

function 想多次返回() {
  return 1  // 停！
  return 2  // 不存在的
  return 3  // 不存在的
}

想多次返回() // 1


// 回调函数
function 买(一旦交易成功) {
  console.log('交易成功')
  console.log(一旦交易成功)  // 输出回调函数 发货()

  /*检测是否有回调函数*/
  if (一旦交易成功) {
    /*如果有回调函数就触发它*/
    一旦交易成功()
  }
}

function 发货() {
  console.log('发货')
  console.log('短信通知买家已发货')
}

买(发货) // "交易成功" "发货" "短信通知买家已发货"
// --------- 过程 ---------
// 将发货作为 买() 函数的参数
// 执行了 买() 函数就会输出 "交易成功"
// 此时接着用 if 检测是否传入参数
// 此处的参数为 发货，存在该参数同名的回调函数，就执行该函数 发货()
// 执行了 发货() 函数就输出 "发货" "短信通知买家已发货"
```

# 函数调用

- 四种方式调用函数：
    - 作为函数
    - 作为方法
    - 作为构造函数
    - 通过函数的 call() 和 apply() 方法间接调用

```js
// 方法调用  方法即是保存在一个对象的属性里的函数
var calculator = {
    operand1: 1,
    operand2: 1,
    add: function() {
        this.result = this operand1 + this.operand2 // this 指代当前对象(calculator)
    }
}
calculator.add()    // 调用作为属性的 add 函数, 进行计算, 继而返回 result
calculator.result   // 2

// 与函数调用的区别是 调用上下文:
// 属性访问表达式由 一个对象(calculator) 和 属性名称(add) 组成
// 此时对象成为调用上下文, 函数体可以使用关键字 this 引用该对象
var o = {
    m: function() {
        var self = this
        console.log(this === o)     // true  作为方法调用, this 即是对象 o
        console.log(self === o)     // true
        f()                         // 函数调用
        function f () {
        // this 没有作用域, 嵌套函数不会从调用它的函数(母函数)中继承 this
            console.log(this === o) // false  嵌套函数作为函数调用, this 为全局变量或者 undefined
            console.log(self === o) // true   self 为外部函数继承来的值
        }
    }
}
o.m()
```

# 回调函数

>回调函数的意义即是在已有的函数里，插入额外运行的函数，不影响已有函数的运行；有就运行，没有继续；插入是将额外运行函数的函数名称以已有函数的参数传入，再判断该参数对应的回调函数是否存在
>
>在已有的函数里插自己的逻辑进去

```js
function yo(callback) {
    console.log('Yo')
    if (callback) {
        callback()
    }
}

yo(function() { console.log('我肝帝，打钱') })
// Yo
// 我肝帝，打钱

var a = [2, 4, 6, 8]
function each(arr, callback) {
    for (var i = 0; i < arr.length; i++) {
        var item = arr[i]
        callback(item)
    }
}

// 回调函数会让主逻辑不变，每个回调函数可以自定义规则，且互不干扰
each(a, function(item) {if (item < 5) { console.log(item) }})
// 2
// 4

each(a, function(item) {if (item > 5) { console.log(item) }})
// 6
// 8

each(a, equal2)     // 将回调函数单独提出来声明
function equal2 (item) {
    if (item == 2) {
        console.log(item) 
    }
}
```

# 函数作用域 function scope

- 函数作用域：变量在声明它们的函数体以及这个函数体嵌套的任意函数体内都是有定义的
- 声明提前：函数内声明的所有变量在函数体内始终是可见的, 因此变量在声明之前已经可用
- 优先级：在函数体内, 局部变量的优先级高于**同名**的全局变量
- 全局作用域下可不用 var 语句, 声明局部变量时必须使用 var 语句


```js
// 同名的局部变量优先级高于全局变量
var scope = 'global'
function f() {
    console.log(scope)  // undefined 嵌套函数已存在同名 scope 局部变量, 但无初始值
    var scope = 'local' // scope 声明会被提前, 但初始值并不会提前
    console.log(scope)  // local 在执行了上边的 var 赋值语句后才能返回值
}
f()

// JavaScript 采用的词法作用域是静态作用域
// foo 函数内无局部变量，则会向上层查找变量
// 而动态作用域会从调用函数的作用域查找变量，也就是 bar()
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar()   // 1
```

# 作用域链 scope chain

- 作用域链：如果将一个局部变量看作是自定义实现的对象的属性, 那全局代码或函数都有一个与之关联的作用域链
- 作用域链是一个**对象列表**或者链表, 其组成：
    - JS 顶层代码中, 作用域链由一个全局对象组成
    - 在不包含嵌套的函数体内, 作用域链有两个对象：定义函数参数和局部变量的对象、全局对象
    - 在包含嵌套的函数体内, 作用域链上至少有三个对象
- 对象链的创建规则：
    - 定义一个函数时, 实际上保存了一个作用域链
    - 调用一个函数时, 又创建一个新的对象来存储它的局部变量, 并将这个对象添加保存到之前的作用域链, 同时创建一个新的更长的表示函数调用作用域的**链**
    - 函数返回时, 就从作用域链中将这个绑定变量的对象删除



# 参考资料

《JavaScript权威指南》
[冴羽的博客-JavaScript深入之词法作用域和动态作用域](https://github.com/mqyqingfeng/Blog/issues/3)
[冴羽的博客-JavaScript深入之作用域链](https://github.com/mqyqingfeng/Blog/issues/6)