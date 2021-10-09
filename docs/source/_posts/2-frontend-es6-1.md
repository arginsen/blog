---
title: 前端 | ES6笔记 | 一、let 与 const
date: 2020-5-20 15:45:21
tags: 
- ES6
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->

# 块级作用域

> ES5只有 全局作用域 和 函数作用域，没有块级作用域，这会带来两种不合理的情况：

```js
// 一、内层变量可能覆盖外层变量 
var tmp = new Date();

function f() {
  console.log(tmp);             // 本应输出外部的 tmp
  if (false) {                  // 此时加入条件语句
    var tmp = 'hello world';    // 条件语句中的变量被提前至函数作用域顶部
  }                             // 但变量初始值并未提前
}                               // 因而输出 undefined

f();                            // undefined 

// 二、用来计数的循环变量泄露为全局变量
var s = 'hello';

for (var i = 0; i < s.length; i++) {
  console.log(s[i]);            // 此时循环定义了变量 i, 并赋予初始值
}                               // 但循环结束后的 i = 5 会保留下
                                // 条件语句下 i 不是局部变量, 因此称为全局变量
console.log(i);                 // 5
```

>ES6 中，let 为JavaScript 新增了块级作用域，ES6 的块级作用域必须有大括号，let 声明的变量只在当前代码块内有效

```js
// 块级作用域的特点：
// 控制变量作用区域
function f1() {
  let n = 5;
  if (true) {
    let n = 10;   // 块级作用域的变量保留在原代码块中
  }
  console.log(n); // 5
}

// 块级作用域可以任意嵌套
{{{{
  {let insane = 'Hello World'}
  console.log(insane); // 报错
}}}};

// 内层作用域可以不影响外层作用域的同名变量
{{{{
  let insane = 'Hello World';
  {let insane = 'Hello World'}
}}}};

// 取代匿名立即执行函数表达式
// (function () {
//   var tmp = ...;
//   ...
// }());
// 块级作用域写法
{
  let tmp = ...;
  ...
}

// 块级作用域中可以声明函数 (ES5 规定函数只能在顶层作用域和函数作用域中声明)
// 但函数声明类似于var，即会提升到全局作用域或函数作用域的头部
// 函数声明还会提升到所在的块级作用域的头部
// but
// 也因此考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数
// 如果确实需要，也应该写成函数表达式，不是函数声明语句
function f() { console.log('I am outside!'); }

(function () {
  // var f = undefined;     // ES6 中, 在块级作用域声明的函数 f 的名称, 被提升到函数作用域顶部
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());                       // Uncaught TypeError: f is not a function
```

# let

## 基本用法

```js
// 声明变量 变量仅在代码块内有效
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1


// 使循环中的变量不泄露为全局变量
// var 在 for 循环中声明的变量会成为全局变量
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);  // 这里的 i 其实指向的是全局的 i
  };                 // 因此等循环体结束后的 i = 10 被数组所有元素获取
}
console.log(i);      // 10  
a[6]();              // 10

// let 在 for 循环中声明的变量仅在循环内有效
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i); // 当前的 i 只在本轮循环有效
  };
}
console.log(i);     // ReferenceError: i is not defined
a[6]();             // 6

// ---------- * ---------- * ---------- * ---------- * ---------- * ---- 
// for 循环设置循环变量的部分是一个父作用域，而循环体内部是一个单独的子作用域
// 每次迭代循环时都创建一个新变量，并以之前迭代中同名变量的值将其初始化
// ---------- * ---------- * ---------- * ---------- * ---------- * ---- 
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc

// 上边的就相当于：
// 伪代码
(let i = 0) {
    funcs[0] = function() {
        console.log(i)
    };
}

(let i = 1) {   // 创建一个 i, 用迭代中的 i++ 后的值 1 将 i 初始化, 因此 i = 1
    funcs[1] = function() {
        console.log(i)
    };
}

(let i = 2) {
    funcs[2] = function() {
        console.log(i)
    };
};
```

## 不存在变量提升

```js
console.log(bar);   // 报错ReferenceError
let bar = 2;        // bar 未被提前
```

## 暂时性死区 trmporal dead zone

```js
// ES6 中规定，如果区块存在 let、const 命令
// 其所声明的变量就绑定这个区域，不再受外部影响
// 这个区块对这些命令声明的变量，从一开始就形成了封闭作用域
// 不能在声明之前使用这些变量，之前的部分称为暂时性死区 TDZ
if (true) {
  // TDZ开始
  tmp = 'abc';      // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp;          // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}

// 因为 let 声明形成的变量死区，使得 typeof 也需要注意
// let 声明之前，都是变量 x 的死区，typeof 报错
typeof x; // ReferenceError
let x;

// 但不声明一个变量，typeof 反而不会报错
typeof x; // undefined

// 其他死区情况
// 此时 y 未声明，属于死区
function bar(x = y, y = 2) {    
  return [x, y];
}

bar(); // 报错

// 变量 x 的声明语句还没执行完，就去取 x 的值
let x = x;  // 报错
```

## 不允许重复声明

```js
// let 不允许在相同作用域重复声明同一个变量
function func() {
  let a = 10;
  var a = 1;    // var 会被提前，let 声明的 a 就已经存在了
}
// 报错

function func() {
  let a = 10;
  let a = 1;
}
// 报错

// 不能在函数内部重新声明参数
function func(arg) {
  let arg;
}
func() 
// 报错

function func(arg) {
  {
    let arg;
  }
}
func() 
// 不报错
```

# const

## const 的用法

> - const 声明一个只读的**常量** - 那么声明变量需**立即初始化**，不能留到以后赋值
> - const 与 let 声明都是块级作用域有效
> - const 声明的常量不提升
> - const 也存在暂时性死区，只能在声明的后边使用
> - const 不可重复声明

## const 的本质

>const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。

- 对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于**常量**。
- 对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向**实际数据**的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。

```js
// 常量 foo 储存的是一个地址，指向一个对象，
// 不可变的只是这个地址，对象本身可变可写入，但不能把 foo 指向另一个地址(对象)
const foo = {};
foo.prop = 123;     // 为 foo 添加一个属性，可以成功
foo.prop            // 123

// 再将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only

// 数组同样也是储存一个地址，不能改变其指向地址(将另一个数组赋值给 a)
const a = [];
a.push('Hello'); // 可执行
a.length = 0;    // 可执行
a = ['Dave'];    // 报错
```

# 顶层对象的属性

>顶层对象，在浏览器环境指的是window对象，在 Node 指的是global对象。ES5 之中，顶层对象的属性与全局变量是**等价**的。

```js
window.a = 1;
a        // 1

a = 2;
window.a // 2
// 顶层对象的属性与全局变量等价可能引发一系列问题：
// 1.有时全局变量可能由顶层对象的属性创造(莫名其妙多了)，属性的创造还是动态的
// 2.顶层对象的属性随处可读写，不利于模块化编程
```

>ES6规定：一、var 和 function 声明的全局变量，依旧是顶层对象的属性；二、let、const 声明的全局变量，不属于顶层对象的属性



# 参考资料
https://wangdoc.com/es6/let.html
https://github.com/mqyqingfeng/Blog/issues/82