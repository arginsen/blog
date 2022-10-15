---
title: 面试 | JavaScript
date: 2020-8-25 11:20:33
tags:
  - interview
  - Frontend
categories: notes
hide: false
photos:
    - /blog/img/interview.jpg
---


<br>
<!--more-->


<video class="vsc-initialized" preload="auto" controls="controls">
    <source src="/blog/img/2020/interview1/howtointerview.mp4" type="video/mp4">
    Your user agent does not support the HTML5 Video element.
</video>
<br>

# js 基础数据类型有哪些（字节）

Number、String、Boolean、null、undefined、Symbol、BigInt

补充：对象、数组、函数均为引用数据类型

基本类型的变量是存放在栈区的（栈区指内存里的栈内存），栈区包括了:变量的标识符和变量的值

引用类型的存储需要内存的栈区和堆区（堆区是指内存里的堆内存）共同完成，栈区内存保存 变量标识符 和 指向堆内存中该对象的指针


# 对数据类型的检测（字节）

1. typeof

其中 数组、对象、null 都会被判定为 object

```js
console.log(typeof 2);               // number
console.log(typeof true);            // boolean
console.log(typeof 'str');           // string
console.log(typeof []);              // object
console.log(typeof function(){});    // function
console.log(typeof {});              // object
console.log(typeof undefined);       // undefined
console.log(typeof null);            // object
console.log(typeof Symbol());        // object
```

2. instanceof

可以判断 **对象** 的类型，也就是引用数据类型，不能判断原始类型数据
原理是判断在其原型链中能否找到该类型的原型

```js
console.log(2 instanceof Number);                    // false
console.log(true instanceof Boolean);                // false
console.log('str' instanceof String);                // false

console.log([] instanceof Array);                    // true
console.log(function(){} instanceof Function);       // true
console.log({} instanceof Object);                   // true
```

3. constructor

constructor有两个作用，一是判断数据的类型，二是对象实例通过 constrcutor 对象访问它的构造函数。需要注意，如果创建一个对象来**改变**它的原型，constructor就不能用来判断数据类型了

```js
console.log((2).constructor === Number); // true
console.log((true).constructor === Boolean); // true
console.log(('str').constructor === String); // true
console.log(([]).constructor === Array); // true
console.log((function() {}).constructor === Function); // true
console.log(({}).constructor === Object); // true
```

4. Object.prototype.toString.call(obj)

toString是Object的原型方法，返回对象的具体类型；
Array、function等类型作为Object的实例，都重写了toString方法，所以不能直接调用 toString

```js
var a = Object.prototype.toString;

console.log(a.call(2)); // [object Number]
console.log(a.call(true)); // [object Boolean]
console.log(a.call('str')); // [object String]
console.log(a.call([])); // [object Array]
console.log(a.call(function(){})); // [object Function]
console.log(a.call({})); // [object Object]
console.log(a.call(undefined)); // [object Undefined]
console.log(a.call(null)); // [object Null]
```

# js如何判断数组是Array类型（字节）

```js
arr instanceof Array === true

Array.isArray(arr) === true

Object.prototype.toString.call(arr) === '[object Array]'

arr.constructor === Array

Array.prototype.isPrototypeOf(arr) // true
```

# 如何理解 [].slice.call(arguments)

通常用该方法使一个类数组对象能够使用数组的 slice 方法，这里也就是将类数组转为数组

实际上 call 的用途就是改变函数的 this 指向，这里也就可以理解成 slice 函数通过 call 将 this 指向 arguments 对象，slice 在该上下文执行，传入的参数为空。当然也可以在其后添加参数描述 start 和 end

# 如何检测一个空对象

```js
Object.keys(obj).length === 0

for (let i in obj) {
  return true
}
```

# 0.1 + 0.2 == 0.3 ?

```js
a = 0.1 + 0.2
b = 0.3

function numbersequal(a,b){
  return Math.abs(a-b) < Number.EPSILON;
}
```

# null 和 undefined 的区别

null 表示一个对象被定义了，但存放了空指针，转换为数值时为 0
undefined 表示声明的变量未初始化，转换为数值时为 NaN

```js
typeof null // object
typeof undefined // undefined
```

# js 中有几种类型的值

基本数据类型存储在栈中；引用数据类型（对象）存储在堆中，指针放在栈中。
两种类型的区别是：1. 存储位置不同；2. 原始数据类型是直接存储在栈中的简单数据段，占据空间小、大小固定，属于频繁操作的对象，所以放在栈中；3. 引用数据类型是存储在堆中的对象，占据空间大、大小不固定，如果在栈中存储将会影响程序运行的性能。
引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。

# == 操作符的强制类型转换规则

1. 首先判断两者类型是否相同，相同的话就比较两者的大小
2. 类型不同进行类型转换
3. 先判断是否在对比 null 和 undefined， 是的话就会返回 true
4. 再判断两者类型是否为 string 和 number，是的话将字符串转换为 number
5. 接着判断一方是否为 boolean，是的话把 boolean 转换为 number 再进行判断
6. 判断其中一方是否为 object 且另一方为 string、number、symbol，是的话就把  object 转换为原始类型再判断

# String 有哪些方法（实例方法）

方法 | 作用
---- | -----
String.prototype.concat()     | concat方法用于**连接**两个字符串，返回一个新字符串，不改变原字符串
String.prototype.slice(start, end)    | 从原字符串取出子字符串并返回，不改变原字符串，返回不包含 end 的字符，如不指定 end 则直到结束
String.prototype.substring(start, end)| 从原字符串取出子字符串并返回，不改变原字符串，参数同 slice，返回不包含 end 的字符
String.prototype.substr(start, len)   | 从原字符串取出子字符串并返回，不改变原字符串
String.prototype.indexOf()    | 用于确定一个字符串在另一个字符串中第一次出现的位置，返回结果是匹配开始的**位置**，如返回-1，就表示不匹配，可以在第二个参数指定起始位置
String.prototype.lastIndexOf()| 用法跟 indexOf 方法一致，主要的区别是 lastIndexOf 从尾部开始匹配
String.prototype.trim()       | **去除**字符串两端的空格（还有制表符`\t`、`\v`、换行符`\n`和回车符`\r`），返回一个新字符串，不改变原字符串。
String.prototype.toLowerCase()| 将一个字符串全部转为小写，返回一个新字符串，不改变原字符串
String.prototype.toUpperCase()| 将一个字符串全部转为大写，返回一个新字符串，不改变原字符串
String.prototype.match()      | 用于确定原字符串是否匹配某个子字符串，返回一个数组，包括**所有**匹配成功的结果。如果没有找到匹配，则返回 null
String.prototype.search()     | search 方法的用法基本等同于 match，但是返回值为匹配的**第一个位置**。如果没有找到匹配，则返回-1
String.prototype.replace()    | 用于**替换**匹配的子字符串，一般情况下只替换第一个匹配
String.prototype.split()      | 按照给定规则**分割**字符串，返回一个由分割出来的子字符串组成的数组，第二个参数可限定返回数组的最大成员数

# Array 有哪些方法

方法 | 作用
---- | -----
valueOf()   | 返回数组本身
toString()  | 返回数组的字符串形式
push()      | 在数组的**末端**添加一或多个元素，并返回添加新元素后的数组**长度**
pop()       | 删除数组的**最后**一个元素，并返回该元素
shift()     | 删除数组的**第一个**元素，并返回该元素
unshift()   | 在数组的**第一个**位置添加元素，并返回添加新元素后的数组**长度**
join()      | 以指定参数作为分隔符（默认逗号），将所有数组成员连接为一个**字符串**返回
concat()    | 合并多个数组，添加在尾部，返回新数组，**原数组不变**
reverse()   | 颠倒排列数组元素，返回改变后的数组
slice(start, end) | 剪切，提取目标数组的一部分，返回一个新数组，原数组不变；还可将**类似数组对象**转为数组
splice(start, count, addElement1, ...) | 剪接，删除原数组的一部分成员，并可以在删除的位置添加新的数组成员，返回值是**被删除**的元素
sort()      | 对数组成员进行排序，默认是按照**字典顺序**排序
map()       | 将数组的所有成员依次传入第一个参数指定的**函数**（有三个参数：当前成员、当前位置和数组本身），同时第二个参数指定函数内部 this 指向的**对象**，然后把每一次的执行结果组成一个**新数组**返回
forEach()   | 对数组的所有成员依次执行参数函数，参数同 map，过程**无法中断**，结果无返回值
filter()    | 过滤数组成员，满足第一个参数指定函数为 true，第二个参数绑定函数内部 this 的成员组成一个新数组返回
some()      | 第一个参数指定一个函数，第二个参数函数绑定函数内部的 this，所有数组成员依次执行该函数。该函数接受三个参数：当前成员、当前位置和整个数组，同时然后返回一个布尔值，只要一个成员的返回值是true，则整个some方法的返回值就是true，否则返回false
every()     | 参数同 some，every 方法是所有成员的返回值都是 true，整个every方法才返回true，否则返回 false
reduce()    | 从左到右依次处理数组的每个成员，最终累计为一个值。第一个参数是一个函数，接受以下四个参数：累积变量，当前变量，当前位置（从0开始），原数组；第二个参数可以指定累积的初始值
reduceRight() | 从右到左依次处理数组的每个成员，最终累计为一个值。参数同 reduce
indexOf()     | 返回给定元素在数组中第一次出现的位置，如果没有出现则返回 -1，第二个参数指定开始位置
lastIndexOf() | 返回给定元素在数组中最后一次出现的位置，如果没有出现则返回-1

# call bind apply 有什么区别

三者作用均是改变函数执行时的上下文，简言之就是改变函数运行时的 this 指向

1. call 和 apply 直接执行函数，使函数 this 指向第一个参数对象，bind 是传入对象后返回一个新的函数，这个函数绑定了新的 this 环境等待调用
2. bind 和 apply 可以数组的形式传入参数，call 不可以

# js 创建新对象有哪些方法

通过对象直接量、关键字 new、和 Object.create() 函数来创建对象

# 什么是作用域, 什么是作用域链

作用域即是变量和函数生效的区域或集合。

在 ES6 之前，作用域分为全局作用域和函数作用域，而不存在块级作用域，这导致函数中的变量无论是在哪申明的，在编译阶段都将被提升到执行上下文的变量环境中，这些变量在函数体内部任何地方都能被访问到。

变量提升也有好处。一方面提高性能，js代码执行之前进行语法检查和预编译；另一方面容错性也更好，比如先赋值后定义
变量提升也有缺点。一方面会导致变量被覆盖；另一方面也会面临变量没有被销毁，如 for 循环时 var 定义的 i

作用域链是，在 js 中使用一个变量时，javascript 引擎会尝试在当前作用域下寻找该变量，若未找到，则向上层寻找，直到找到该变量或抵达全局作用域

# 什么是原型, 什么是原型链

JavaScript 被描述为一种基于原型的语言————每个对象都拥有一个原型对象

每个对象都有自己的原型对象，那么任何一个对象都可以充当其他对象的原型，且由于原型对象也是对象，所有它也有自己的原型，因此形成一个原型链

# `__proto__` 和 prototype 的区别

- 每个对象都有 `__proto__` ，只有函数对象才有 prototype
- js 在创建对象时，都会有一个 `__proto__` 内置属性，用于指向创建它的构造函数的原型对象
- 所有函数对象的 `__proto__` 都指向 Function.prototype

# js 实现继承的方式

1. 原型链继承
2. 借用构造函数（经典继承）
3. 组合式继承
4. 原型式继承
5. 寄生式继承
6. 寄生组合式继承

```js
// 寄生组合式继承
function clone (parent, child) {
    // 这里改用 Object.create 就可以减少组合继承中多进行一次构造的过程
    child.prototype = Object.create(parent.prototype);
    child.prototype.constructor = child;
}

function Parent() {
    this.name = 'parent';
    this.play = [1, 2, 3];
}

Parent.prototype.getName = function () {
    return this.name;
}

function Child() {
    Parent.call(this);
    this.friends = 'child';
}

clone(Parent, Child);

Child.prototype.getFriends = function () {
    return this.friends;
}

let person = new Child();
```

# 对 this 的理解

this 是函数运行时内部生成的一个对象，只能在函数内部使用，总是指向调用它的对象,在函数体内部，指代函数当前的运行环境。

1. this 总是指向函数的直接调用者，而非间接调用者
2. 如果有 new 关键字，this 指向 new 出来的那个对象
3. 在事件中，this 指向目标元素

# 箭头函数和普通函数的区别

1. 没有 this。需要通过查找作用域链来确定 this 的值，this 也就成了最近一层非箭头函数的 this；此外箭头函数没有 this，所以也不能用 call、apply、bind 这些方法 改变 this 指向

2. 没有 arguments 。箭头函数没有自己的 arguments 对象，访问到的是外层函数的 arguments 对象，想要访问箭头函数的参数，可以使用 rest 参数的形式

3. 不能通过 new 关键字调用。JavaScript 函数有两个内部方法：`[[Call]]` 和 `[[Construct]]`。

当通过 new 调用函数时，执行 `[[Construct]]` 方法，创建一个实例对象，然后再执行函数体，将 this 绑定到实例上。

当直接调用的时候，执行 `[[Call]]` 方法，直接执行函数体。

箭头函数并没有 `[[Construct]]` 方法，不能被用作构造函数，如果通过 new 的方式调用，会报错。

4. 没有 new.target。不能使用 new 调用，也就没有 new.target

5. 没有原型。不能使用 new 调用箭头函数，所以也没有构建原型的需求，不存在 prototype 属性

6. 没有 super。没有原型

# 什么是闭包，闭包的优点和缺点

闭包指的是一个函数可以访问另一个函数作用域中的变量，闭包的原理是作用域链。
优点是避免全局变量污染，缺点是容易造成内存泄露

可以应用于函数柯里化，柯里化的目的在于避免频繁调用具有相同参数的函数，同时又方便重用

```js
// 假设我们有一个求长方形面积的函数
function getArea(width, height) {
    return width * height
}
// 如果我们碰到的长方形的宽老是10
const area1 = getArea(10, 20)
const area2 = getArea(10, 30)
const area3 = getArea(10, 40)

// 我们可以使用闭包柯里化这个计算面积的函数
function getArea(width) {
    return height => {
        return width * height
    }
}

const getTenWidthArea = getArea(10)
// 之后碰到宽度为10的长方形就可以这样计算面积
const area1 = getTenWidthArea(20)

// 而且如果遇到宽度偶尔变化也可以轻松复用
const getTwentyWidthArea = getArea(20)
```

# 如何用一个函数生成对象时设置不用 new 关键字报错

```js
function Person(name) {
  this.name = name
}
Person("jake") // 报错
new Person("jake") // 正确
```

answer：

```js
function Person(name) {
  if(!(this instanceof Person)) {
    throw Error('error msg')
  }
  this.name = name
}
```

其实在 vue 中我们就有见到过, Vue 本身是一个函数，只不过在其上定义了很多属性，以及原型上定义了各类方法，我们在使用时需要用 new 来创建一个 vue 实例

```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```


# js 异步通信，对 Ajax 的了解

Ajax 是通过 JavaScript 的异步通信（jquery 对 xhr 的封装）
在向服务器发送请求的时候，可以同时做其他事情，等实现特定结果，再进行后续操作，使页面发送局部更新。

过程：
1. 创建 XMLHTTPRequest 对象  `const xhr = new XMLHTTPRequest()`
2. 创建新的 HTTP 请求  `xhr.open('get', url, true)`
3. 设置响应 HTTP 请求状态变化的函数 onreadystatechange
4. 发送 HTTP 请求  `xhr.send(data)`
5. 获取异步通信的结果

# js 深浅拷贝

1. 浅拷贝

- Object.assign 实现浅拷贝，Object.assign 只会拷贝所有的属性值到新的对象中，如果属性值是对象的话，拷贝的则是地址。

```js
let a = {
  age: 1
}
let b = Object.assign({}, a)
a.age = 2
console.log(b.age) // 1
```
- 展开运算符 `...` 实现浅拷贝

2. 深拷贝

- 通常可以用 `JSON.parse(JSON.stringify(object))` 实现深拷贝，但是该方法也有局限性
  - 会忽略 undefined
  - 会忽略 symbol
  - 不能序列化函数
  - 不能解决循环引用的对象

```js
let a = {
  age: undefined,
  sex: Symbol('male'),
  jobs: function() {},
  name: 'yck'
}

let b = JSON.parse(JSON.stringify(a))
console.log(b) // {name: "yck"}
```

- 所需拷贝的对象含有内置类型并且不包含函数，可以使用 MessageChannel

```js
function structuralClone(obj) {
  return new Promise(resolve => {
    const { port1, port2 } = new MessageChannel()
    port2.onmessage = ev => resolve(ev.data)
    port1.postMessage(obj)
  })
}

var obj = {
  a: 1,
  b: {
    c: 2
  }
}

obj.b.d = obj.b

// 注意该方法是异步的
// 可以处理 undefined 和循环引用对象
const test = async () => {
  const clone = await structuralClone(obj)
  console.log(clone)
}
test()
```

- lodash 的深拷贝函数

```js
function deepClone(obj) {
  function isObject(o) {
    return (typeof o === 'object' || typeof o === 'function') && o !== null
  }

  if (!isObject(obj)) {
    throw new Error('非对象')
  }

  let isArray = Array.isArray(obj)
  let newObj = isArray ? [...obj] : { ...obj }

  // 对属性进行遍历，如果属性为对象，递归处理
  Reflect.ownKeys(newObj).forEach(key => {
    newObj[key] = isObject(obj[key]) ? deepClone(obj[key]) : obj[key]
  })
  return newObj
}

//
let obj = {
  a: [1, 2, 3],
  b: {
    c: 2,
    d: 3
  }
}

let newObj = deepClone(obj)
newObj.b.c = 1
console.log(obj.b.c) // 2
```

- 循环递归

```js
function deepClone(obj, hash = new WeakMap()) {
  if (obj === null) return obj; // 如果是null或者undefined我就不进行拷贝操作
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  // 可能是对象或者普通的值  如果是函数的话是不需要深拷贝
  if (typeof obj !== "object") return obj;
  // 是对象的话就要进行深拷贝
  if (hash.get(obj)) return hash.get(obj);
  let cloneObj = new obj.constructor();
  // 找到的是所属类原型上的constructor,而原型上的 constructor指向的是当前类本身
  hash.set(obj, cloneObj);
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      // 实现一个递归拷贝
      cloneObj[key] = deepClone(obj[key], hash);
    }
  }
  return cloneObj;
}
```

# async 是什么的语法糖

async 函数是 Generator 函数的语法糖。实际上就是将 Generator 函数的星号（*）替换成 async，将 yield 替换成 await

async 函数对 对 Generator 函数的改进，体现在四点：

1. Generator 函数的执行必须靠执行器，所以才有了co模块，而async函数自带执行器，与普通函数一样执行

2. async 和 await，比起星号和 yield，语义更清楚了。

3. async 函数的 await 命令后面，可以是 Promise 对象和原始类型的值

4. async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作

# promise 有几种状态, 又是怎么转变状态的

有 pending、fulfilled、rejected 三种状态，promise 函数执行进入 pending 状态，通过 resolve 函数 会将 fulfilled 成功状态，通过 reject 函数将进行状态转为失败状态












# 参考链接

[金九银十，初中级前端面试复习总结「JavaScript篇」](https://juejin.im/post/6868843713729134599/)