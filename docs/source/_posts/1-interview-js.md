---
title: 面试 | JavaScript
date: 2020-8-25 11:20:33
tags: 
  - interview
  - Frontend
categories: notes
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

Number、String、Boolean、null、undefined、Symbol

# 对数据类型的检测（字节）

使用 `Object.prototype.toString.call(obj)`，返回一个字符串描述当前参数的数据类型

# js如何判断数组是Array类型（字节）

1. `arr instanceof Array === true`
2. `Array.isArray(arr) === true`
3. `Object.prototype.toString.call(arr) === '[object Array]'`
4. `arr.constructor === Array`

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

1. call 和 bind 直接执行函数，使函数 this 指向第一个参数对象，bind 是传入对象后返回一个新的函数，这个函数绑定了新的 this 环境等待调用
2. bind 和 apply 可以数组的形式传入参数，call 不可以

# js 创建新对象有哪些方法

通过对象直接量、关键字 new、和 Object.create() 函数来创建对象

# js 实现继承的方式

1. 原型链继承
2. 借用构造函数（经典继承）
3. 组合式继承
4. 原型式继承
5. 寄生式继承
6. 寄生组合式继承

# 对 this 的理解

1. this 总是指向函数的直接调用者，而非间接调用者
2. 如果有 new 关键字，this 指向 new 出来的那个对象
3. 在事件中，this 指向目标元素

# 什么是闭包，闭包的优点和缺点

闭包指的是一个函数可以访问另一个函数作用域中的变量，闭包的原理是作用域链。
优点是避免全局变量污染，缺点是容易造成内存泄露

# js 异步通信，对 Ajax 的了解

Ajax 是通过 JavaScript 的异步通信
在向服务器发送请求的时候，可以同时做其他事情，等实现特定结果，再进行后续操作，使页面发送局部更新。

过程：
1. 创建 XMLHTTPRequest 对象  `const xhr = new XMLHTTPRequest()`
2. 创建新的 HTTP 请求  `xhr.open('get', url, true)`
3. 设置响应 HTTP 请求状态变化的函数
4. 发送 HTTP 请求  `xhr.send(data)`
5. 获取异步通信的结果

# 跨域问题

## CORS（字节）

允许浏览器向跨域服务器发出 XMLHTTPRequest 请求。

1. 浏览器端会自动向请求头部添加 origin 字段，标注请求来源
2. 在服务器端设置响应头的允许方法、头部、源等
3. 请求分为两种：简单请求和非简单请求；简单请求的头部可包含 GET/POST/HEAD；非简单请求头部有 PUT/DELETE

> 对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。

```json
// 简单请求
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...

// 服务器返回的响应
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```

> 非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。
> 过程：
> 非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个Origin头信息字段
> 浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

1. 预检请求
除了Origin字段，"预检"请求的头信息包括两个特殊字段。
Access-Control-Request-Method 该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是PUT。
Access-Control-Request-Headers 该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段

2. 预检请求的回应
Access-Control-Allow-Methods 该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。
Access-Control-Allow-Headers 如果浏览器请求包含该字段，则该字段是必需的。表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。
Access-Control-Allow-Credentials
Access-Control-Max-Age 可选，用来指定本次预检请求的有效期，单位为秒

3. 浏览器的正常请求和回应
与简单请求一样，直接发送cors请求

```json
// 预检请求
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header

// 预检请求的回应
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000

// 正常cors请求
PUT /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
X-Custom-Header: value
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...

// 响应
Access-Control-Allow-Origin: http://api.bob.com
Content-Type: text/html; charset=utf-8
```

> 与 JSONP 相比，JSONP只支持GET请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。


# js 深浅拷贝

# 浅拷贝

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

# 深拷贝

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

# 参考链接

[金九银十，初中级前端面试复习总结「JavaScript篇」](https://juejin.im/post/6868843713729134599/)