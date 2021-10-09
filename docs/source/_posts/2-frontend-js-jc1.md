---
title: 前端 | JS基础 | JS学习笔记Ⅰ：数据类型、变量、运算符、关键字
date: 2020-4-23 10:25:21
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

# 导论

> - JavaScript 是一种轻量级的脚本语言。所谓“脚本语言”（script language），指的是它不具备开发操作系统的能力，而是只用来编写控制其他大型应用程序（比如浏览器）的“脚本”。
> - JavaScript 也是一种嵌入式（embedded）语言。它本身提供的核心语法不算很多，只能用来做一些数学和逻辑运算。JavaScript 本身不提供任何与 I/O（输入/输出）相关的 API，都要靠宿主环境（host）提供，所以 JavaScript 只合适嵌入更大型的应用程序环境，去调用宿主环境提供的底层 API。
> - 目前，已经嵌入 JavaScript 的宿主环境有多种，最常见的环境就是浏览器，另外还有服务器环境，也就是 Node 项目。
> - 从语法角度看，JavaScript 语言是一种“对象模型”语言。各种宿主环境通过这个模型，描述自己的功能和操作接口，从而通过 JavaScript 控制这些功能。但是，JavaScript 并不是纯粹的“面向对象语言”，还支持其他编程范式（比如函数式编程）。这导致几乎任何一个问题，JavaScript 都有多种解决方法。阅读本书的过程中，你会诧异于 JavaScript 语法的灵活性。
> - JavaScript 的核心语法部分相当精简，只包括两个部分：基本的语法构造（比如操作符、控制结构、语句）和标准库（就是一系列具有各种功能的对象比如Array、Date、Math等）。

# 细则

- **使用两个空格** – 进行缩进
- **字符串使用单引号** – 需要转义的地方除外
- **不要有冗余的变量** – 这是导致 大量 bug 的源头!
- **无分号**
- 行首不要以 **(, [, or `** 开头 - 这是省略分号时唯一会造成问题的地方
- **关键字**后加空格 if (condition) { ... }
- **函数名**后加空格 function name (arg) { ... }


# 数据类型

```js
// Number
123         // 整型
1.2         // 浮点数
-1          // 复数
1e3         // 1000
1.2e4       // 12000
NaN         // Not a Number
Infinity    // 无限大

// String 字符串
'王花花'
"王花花"
'王花花说: "你好"'
'he\'s'    // 转义符
'啊' +      // 换行1
'祖国'
`           // ES6 反单引号可以直接换行写
啊
祖国
`

// Boolean 布尔值
true
false

// Array 数组
[
    'a',
    1,
    true,
    {a: b}
]

// Object 对象
var o = {
    a:1,
    b:2
}

o.a  // 1
var x = o.a + o.b 
x    // x = 3
```

## Number

```js
// 在其它类型前使用 + 可以将其他类型转换为数字类型
+'1'    // 1
+''     // 0
+'1.1'  // 1.1
+true   // 1
+false  // 0
```

### 整数(整型)

```js
// 判断一个值为整型  Number.isInteger()
Number.isInteger(1)     // true
Number.isInteger(1.1)   // false

// 其他类型转整型  parseInt()
parseInt('1')   // 1
parseInt('1.1') // 1
parseInt('1.9') // 1
parseInt('1a')  // 1
parseInt('a1')  // NaN
parseInt('.9')  // NaN
parseInt('0.9') // 0
```

### 小数(浮点数)

```js
// 0.1，1.1，.1 如果小于1，小数点前的 0 可以省略

// 其他类型转整型  parseFloat()
parseFloat('1')     // 1
parseFloat('1.0')   // 1
parseFloat('1.1')   // 1.1
parseFloat('1.1a')  // 1.1
parseFloat('a1.1')  // NaN
parseFloat('.1')    // 0.1

// JS中采用的IEEE 754的双精度标准，计算机内部存储数据的编码的时候，
// IEEE 754 标准的 64 位双精度浮点数的小数部分最多支持53位二进制位，
// 所以两者相加后，因浮点数小数位的限制而截断的二进制数字，再转换为十进制，就成了 0.30000000000000004
0.2 + 0.1 // 0.30000000000000004  1/5 + 1/10 其中 5 不是 2 的倍数
0.1 + 0.3 // 0.4
```

### NaN

```js
// NaN 不是一个数, 但它是种数字类型
// 一般在出错或不可预料的结果中出现 如：'a' * 'b'，0 / 0
NaN === NaN     // false 永不相等

// 判断是否为NaN  isNaN()
isNaN(1)        // false
isNaN(0 / 0)    // true
```

## String

```js
// 使用引号定义一段字符串, 不加引号会识别为变量
// js中单双引号没有任何区别
// 使用反引号定义模板字符串，可以传入变量，还可以直接断行
let name = 'arginsen'
let str = `我是个模板字符串，我叫${name}`
console.log(str)

// 获得字符串中的某一个字符 .charAt()
'yo'.charAt(0); // "y" 
'yo'.charAt(1); // "o"
'yo'[0];        // "y"

// 获取字符串中的某一段字符
//  .substring(begin, end) .slice(begin, end) .substr(begin, len)
// substring 支持负数, slice 不支持负数，substr 的 len 指字符串的长度
// begin 包括, end 不包括
let name = 'arginsen'
let str = `我是个模板字符串，我叫${name}`
console.log(str.substring(3, 8))  // "模板字符串"
console.log(str.slice(3, 8))      // "模板字符串"
console.log(str.substr(3, 5))     // "模板字符串"
// 结束位置可选
console.log(str.substring(9))     // "我叫 arginsen "

// 检查一段字符是否包含另一段字符 .includes()
'花花你好'.includes('花花');     // true
'花花你好'.includes('拴蛋');     // false

// 用字符串将字符串分割为数组 .split()
'花花→_→拴蛋→_→背背'.split('→_→');  // ["花花", "拴蛋", "背背"]
'→_→拴蛋→_→背背'.split('→_→');  // ["", "拴蛋", "背背"]

// 连接字符串 .concat()
// 依次连接传入的字符, 传参数量不限
'yo'.concat('ooo', 'oo', 'o');     // "yooooooo"

// 移除两头的空格 .trim()
'  yo  '.trim();    // "yo"
'  yo'.trim();      // "yo"
```

## Object

```js
// 定义一个对象
// 花括号包起来, 每一项称为键值对, 可嵌套对象
// 键值对左边是键(key，或者叫属性名), 右边是值(value), 每一组用英文逗号隔开
// 键名一般使用英文和数字，有空格和特殊字符时外部需要加引号包住
const o = {
    key1: 'value1',
    key2: 'value2'
}

// 获取对象的属性
const o = {
    a: 1,
    b: {
        b1: 666,
        b2: 'hhh'
    }
}
// 通过 . 或 [] 访问对象属性
o.a;     // 1
o["a"]   // 1
o.b.b1   // 666

// 针对一些奇葩的键名, 或动态键名, 可用函数绑定 id 动态调用
const o = {
    key1: 'value1',
    key2: 'value2'
}

function get_value(id) {
    let key_name = 'key' + id
    return o[key_name]
}

get_value(1)

// 对象可存储所有数据类型
const o = {
  a: [1],   // 数组
  b: {},    // 对象
  c: 1,     // 数字
  d: true,  // 布尔值
  e: 'yo',  // 字符串
}

// 给对象加入新键, 同调用的方式
var obj = {};
obj.a = 1; 
console.log(obj)
```

## Array

```js
// 定义一个数组
// 方括号包起来, 每一项叫 元素, 元素类型不限, 可嵌套
const a = [2, 4, [1, 2]]
const a = [
  2, 
  true,
  'a', 
  {}, 
  function() {}
]

// 获取元素 数组通过索引可获取元素
var letter = ['a', 'b', 'c'];
letter[0] // 'a'
letter[1] // 'b'
letter[2] // 'c'

// 获取数组长度
[5, 10, 15].length // 3

// 添加元素
// 从末尾添加 .push() 
var arr = [3, 4, 5]
arr.push(6)        // 4 返回修改后的长度
console.log(arr)   // [3, 4, 5, 6]
// 从开头添加 .upshift()
var arr = [3, 4, 5]
arr.unshift(2)     // 4 返回修改后的长度
console.log(arr)   // [2, 3, 4, 5]

// 删除元素
// 从末尾删除 .pop()
var arr = [3, 4, 5, 6]
arr.pop()          // 6 返回被删除的数
console.log(arr)   // [3, 4, 5]
// 从开头删除 .shift()
var arr = [2, 3, 4, 5]
arr.shift()        // 2 返回被删除的数
console.log(arr)   // [3, 4, 5]

// 连接元素 .join() 返回字符串，默认用逗号隔开
[5, 10, 15].join()   // "5,10,15"

// 元素排序，并返回排序后的数组 .sort()
var a = ["banana", "cherry", "apple"]
a.sort()             // ["apple", "banana", "cherry"]
var s = a.join()     // "apple,banana,cherry"

// 颠倒元素的顺序 .reverse()
[1, 2, 3].reverse()  // [3, 2, 1]

// 创建并返回一个数组 .concat() 
// 元素包括调用 concat() 的原始数组的元素和 concat() 的每个参数
// 当参数中任何一个自身是数组，则连接的是数组的元素
var a = [1, 2, 3]
a.concat(4, 5)              // [1, 2, 3, 4, 5]
a.concat([4, 5])            // [1, 2, 3, 4, 5]
a.concat([4, 5], [6, 7])    // [1, 2, 3, 4, 5, 6, 7]
a.concat(4, [5, [6, 7])     // [1, 2, 3, 4, 5, [6, 7]]

// 剪裁数组, 返回剪裁的新数组, 不影响原数组 .slice(start，end)
const group = ['a', 'b', 1, 2, 'c'] 
const order = group.slice(2, 4)
console.log(order)      // [1, 2]

// 剪接数组, 返回剪后的数组 .splice(start, len, 替换元素1, 替换元素2)
const group = ['a', 'b', 1, 2, 'c'] 
// 从索引 2 开始剪, 剪掉 2 个元素
group.splice(2, 2)
console.log(group)      // ["a", "b", "c"]
// 继续从索引 1 开始剪, 剪掉 1 个元素, 加入两个新元素
group.splice(1, 1, 'aa', 'bb')
console.log(group)      // ["a", "aa", "bb", "c"]

// 迭代(回调函数) .forEach() 
['a', 'b', 'c'].forEach(function(element, index) { 
  console.log('第' + index + '条：' + element);
})
//------Console------
// 第0条：a
// 第1条：b
// 第2条：c

// 过滤器(回调函数), 返回满足条件的新数组 .filter()
const old = [1, 2, 3, 4]
const now = old.filter(
    // 传入一个函数, 每迭代一个元素就执行一次
    function(element, index, origin_array) {
        const condition = element > 2     // 大于 2 的元素
        return condition
    }
)
console.log(now)    // [3, 4]

// 检验数组是否满足给定的条件 .every()
const old = [1, 2, 3, 4]
const content = old.every(
    // 传入一个函数, 每迭代一个元素就执行一次
    function(element, index, origin_array) {
        const condition = element < 6    // 小于 6 的元素
        return condition
    }
)
console.log(content)    // true  
```

## Null

```js
// null只有在需要明确指定（或清空）一个量时才使用，如删除用户介绍：user.intro = null。
// null只能手动设置，JS本身不会将任何量的默认值设为null

// typeof null 会返回 object
// null 是一个假值
if(null) {
    // 这里永远都不会执行
}
```

## undefined

```js
// 注意
undefined == null   // true
// undefined 是一个假值
if(undefined) {
    // 这里永远都不会执行
}
```

# 变量

```js
var name = 'Yo.'  // 变量可以任意定义关键词之外的数据类型，也可以为中文
name = 1   // 此时变量 name 不为 'Yo.', 一个变量在一个点上只能等于一个值
console.log(name)  // 输出变量

console.log(typeof name)  // 输出变量的类型
```

# 运算符

```js
// + 
1 + 1 // 2 加法运算
'1' + '1' // '11' 连接字符串

// - * / 减乘除

// % 求余
10 % 3 // 1
11 % 3 // 2
12 % 3 // 0

// ++ 自增
// 每登陆一次, 登录次数累计加一
var loginCount = 0
loginCount = loginCount + 1    // 简写成下一条
loginCount++

// -- 自减
loginCount = 1
loginCount--
// 上边输出运算结果：
console.log("loginCount:", loginCount) // loginCount: 0

// > < >= <=
// 判断大小, 输出布尔值
console.log("1 > 2 :", 1 > 2)      // 1 > 2 : false
console.log("1 < 2 :", 1 < 2)      // 1 < 2 : true
console.log("1 >= 2 :", 1 >= 2)    // 1 >= 2 : false
console.log("1 <= 2 :", 1 <= 2)    // 1 <= 2 : true

// = 赋值
var a = 1

// == 判断是否相等
console.log("1 == 2 :", 1 == 2)    // 1 == 2 : false
console.log("1 == 1 :", 1 == 1)    // 1 == 1 : true
console.log("'1' == 1 :", '1' == 1)// '1' == 1 : true

// === 判断是否相等（严格模式）  加入类型分辨
console.log("'1' === 1 :", '1' === 1)   // '1' === 1 : false

// != 不等于
console.log("1 != 2 :", 1 != 2)    // 1 != 2 : true
console.log("1 != 1 :", 1 != 1)    // 1 != 1 : false

// += -= *= /=
loginCount = loginCount + 2    // 可简写为下一条
loginCount += 2

// && || 和 或
console.log("true && false :", true && false)    // true && false : true
console.log("true && false :", true && false)    // true && false : false
console.log("true || false :", true || false)    // true || false : true
console.log("true || false :", true || false)    // true || false : false
// 例如：
var 他是制单人 = true,
    他是审核员 = false

if(他是制单人 && 他是审核员) {
    console.log('转账成功')
} else {
    console.log('转账失败，权限不足')
}
// 转账失败，权限不足

var 他是制单人 = true,
    他是审核员 = false

if(他是制单人 || 他是审核员) {
    console.log('转账成功')
} else {
    console.log('转账失败，权限不足')
}
// 转账成功
```

# 关键字

## this

```js
// this 的三种情况
// 1.所在的函数被直接调用 (全局环境中的函数、匿名函数)
function user() {
    console.log(this)
}
uuser() // window{}
// 严格模式下
'use strict';
function user() {
    console.log(this)
}
user() // undefined

// 2.所在函数被作为构造函数被调用，用来新生成对象
function user() {
    console.log(this)   
}
new user() // user{}    此处的 this 指即将生成的对象

// 3.函数以方法的形式被调用
function yo() {
  console.log('Yo, 我是' + this.name);
}

var whh = {
  name: '王花花',
}

var lsd = {
  name: '李拴蛋',
}

whh.yo = yo;    // 将函数 yo 作为对象 whh 的一个方法
lsd.yo = yo;

whh.yo();       // 将 yo 作为 whh 的方法调用, this 指向此时的对象
lsd.yo();       // this 具有动态赋能的作用
// this 的值取决于它是如何被调用的
// 直接调用就指全局对象，以方法调用指方法所在的对象(父级对象)
// ----------------------------------------------------


// this 的 call apply bind
// 使用 call() 可以将 this 与函数绑定，在括号指定 this 指向的对象
function yo(name) {
  console.log('Yo, ' + name + ', 我是' + this.name);    // this 指全局对象 window
}

var whh = {
  name: '王花花',
}

var lsd = {
  name: '李拴蛋',
}

yo.call(whh, '赵爽爽')  // 函数 yo 中的 this 指向对象 whh，也是一种动态赋能的方式
yo.call(lsd, '刘蓓蓓')  // 对象后边函数的参数 name 在逗号后边传入

// apply() 与 call() 相仿，在后边的传参方式上有区别
// 必须以数组的形式传入参数，多个参数用逗号隔开
yo.apply(lsd, ['刘蓓蓓'])

// apply() 与 call() 是直接执行
// bind() 是传入对象后返回一个新的函数，这个函数绑定了新的 this 环境等待调用
yo2 = yo.bind(lsd)  // 传入对象
yo2('刘蓓蓓')       // 再传入其他参数
```

## return

> - return 语句只能在函数体内出现，当执行到 return 语句的时候，函数终止执行，并返回**表达式的值**给调用程序
> - 当函数体内不用 return 时，函数调用仅依次执行函数体内的每一条语句直到函数结束，最后返回调用程序，但这种调用表达式的结果也是undefined
> - return 语句可以单独使用而不必带有 expression，这样函数会向调用程序返回 undefined

```js
return expression;

// return 返回表达式
function square(x) {
    return x*x;
}

square(2);    // 4

// 函数不使用 return
function square(x) {
    x*x;
}

square(2);    // undefined

// 单独使用 return
function square(x) {
    x*x;
    return;
}

square(2);    // undefined
```

## throw

> - 当产生运行错误或者程序使用 throw 语句时就会显式地抛出异常
> - 当抛出异常时，JavaScript 解释器会立即停止当前正在执行的逻辑，并跳转至就近的异常处理程序

```js
throw expression

// expression 的值可以是任意的
// 通常采用 Error 类型和其子类型
// 一个 Error 对象有一个 name 属性表示错误类型，
// 一个message 属性用来存放传递给构造函数的字符串
function 买买买(钱) {
  if(钱 === undefined)           // 如果没有传入参数
    throw new Error('钱呢？！')  // 报错并停止执行

  // ...
  console.log('爸爸！')
}

买买买()    // Uncaught Error: 钱呢？！
买买买(666) // "爸爸！"
```

# 参考资料

[网道- JavaScript教程 - 导论](https://wangdoc.com/javascript/basic/introduction.html)