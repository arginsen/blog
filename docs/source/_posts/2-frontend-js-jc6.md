---
title: 前端 | JS基础 | JS学习笔记Ⅵ：标准库
date: 2020-4-27 20:45:21
tags: 
- JavaScript
- Frontend
- regex
- JSON
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->


# Object对象

> - JavaScript 原生提供 Object 对象，JavaScript 的所有其他对象都继承自Object对象，即那些对象都是 Object 的实例。
> - Object 对象的原生方法分成两类：[Object 本身的方法](#Object-的静态方法) 与 [Object 的实例方法](#Object-的实例方法)。
> - Object 对象本身的方法，即直接定义在 **Object 对象**的方法；Object 的实例方法，即定义在 **Object 原型对象** Object.prototype 上的方法。

```js
// Object 对象本身的方法
// 即直接定义在 Object 对象的方法
Object.print = function (o) { console.log(o) };


// Object 的实例方法
// 即定义在 Object 原型对象 Object.prototype 上的方法。它可以被 Object 实例直接使用
Object.prototype.print = function () {
  console.log(this);
};

var obj = new Object();
obj.print() // Object   实例对象 obj 的方法继承自其构造函数的原型
```

## Object()

> - Object 本身作为一个构造函数，可以当作**工具方法**使用，将**任意值转为对象**。这个方法常用于保证某个值一定是对象
> - 如果参数为空（或者为 undefined 和 null），Object() 返回一个空对象

```js
// Object 方法将其转为对应的包装对象的实例
// 参数是各种原始类型的值，转换成对象就变成原始类型值对应的包装对象
// 参数是对象，则返回该对象，不用转换
var obj = Object(1);
obj instanceof Object // true
obj instanceof Number // true

var obj = Object('foo');
obj instanceof Object // true
obj instanceof String // true

var obj = Object(true);
obj instanceof Object // true
obj instanceof Boolean // true


// 定义一个判断变量是否为对象的函数
function isObject(value) {
  return value === Object(value);
}

isObject([]) // true
isObject(true) // false
```

## Object 构造函数

> - Object不仅可以当作工具函数使用，还可以当作构造函数使用
> - 接受一个参数，如果该参数是一个对象，则直接返回这个**对象**；如果是一个原始类型的值，则返回该值对应的**包装对象**

```js
// 生成新对象
var obj = new Object();

// 等价于
var obj = {}

// o1 对象作为参数返回 o1
var o1 = {a: 1};
var o2 = new Object(o1);
o1 === o2 // true

// 参数为数值时，返回数值的包装对象，该对象即是 Number 的实例，又是 Object 的实例
var obj = new Object(123);
obj instanceof Number // true
```

## Object 的静态方法


一般方法 | 作用
---- | -----
Object.keys()               | 遍历对象的属性，返回一个数组包含参数指定对象的所有属性名（可枚举的属性）
Object.getOwnPropertyNames()| 遍历对象的属性，返回一个数组包含参数指定对象的所有属性名

```js
// 遍历 a 的属性，返回 a 中所有属性名
var a = ['Hello', 'World'];

Object.keys(a) // ["0", "1"]    只返回可枚举的属性
Object.getOwnPropertyNames(a)  // ["0", "1", "length"]    还返回不可枚举属性 length

// 计算对象属性，两个方法均可使用 length 属性
var obj = {
  p1: 123,
  p2: 456
};

Object.keys(obj).length // 2
Object.getOwnPropertyNames(obj).length // 2
```

对象属性模型的相关方法 | 作用
---- | -----
Object.getOwnPropertyDescriptor() | 获取某个属性的描述对象
Object.defineProperty()           | 通过描述对象，定义某个属性
Object.defineProperties()         | 通过描述对象，定义多个属性


控制对象状态的方法 | 作用
---- | -----
Object.preventExtensions()  | 防止对象扩展
Object.isExtensible()       | 判断对象是否可扩展
Object.seal()               | 禁止对象配置
Object.isSealed()           | 判断一个对象是否可配置
Object.freeze()             | 冻结一个对象
Object.isFrozen()           | 判断一个对象是否被冻结


原型链相关方法 | 作用
---- | -----
Object.create()         | 该方法可以指定原型对象和属性，返回一个新的对象
Object.getPrototypeOf() | 获取对象的Prototype对象


## Object 的实例方法


继承自原型的几个主要方法 | 作用
---- | -----
Object.prototype.valueOf()              | 返回当前对象对应的值，自动类型转换默认调用
Object.prototype.toString()             | 返回当前对象对应的字符串形式，判断数据类型
Object.prototype.toLocaleString()       | 返回当前对象对应的本地字符串形式
Object.prototype.hasOwnProperty()       | 判断某个属性是否为当前对象自身的属性，还是继承自原型对象的属性
Object.prototype.isPrototypeOf()        | 判断当前对象是否为另一个对象的原型
Object.prototype.propertyIsEnumerable() | 判断某个属性是否可枚举


```js
// Object.prototype.valueOf() 
// -----------------------------------------
// 自动类型转换默认调用 valueOf 方法
var obj = new Object();
1 + obj // "1[object Object]"   // 第二个 Object 表示构造函数

// 例
var obj = new Object();
obj.valueOf = function () {
  return 2;
};

1 + obj // 3
// -----------------------------------------


// Object.prototype.toString()
// -----------------------------------------
// 返回一个对象的字符串形式，默认情况下返回类型字符串
var obj = new Object();

obj.toString = function () {
  return 'hello';
};

obj + ' ' + 'world' // "hello world"  用于加法时自动调用 toString 方法

// toString 也可用于判断数据类型
// 使用 Object.prototype.toString 方法，
// 通过函数的 call 方法，可以在任意值上调用这个方法来判断这个值的类型
// 注：call 的作用是将函数的 this 指向参数指定的对象，
//    可以理解为函数在参数对象的环境下运行

// 不同数据类型的值将会得到特定的返回值  
Object.prototype.toString.call(2) // "[object Number]"
Object.prototype.toString.call('') // "[object String]"
Object.prototype.toString.call(true) // "[object Boolean]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(Math) // "[object Math]"
Object.prototype.toString.call({}) // "[object Object]"
Object.prototype.toString.call([]) // "[object Array]"
// -----------------------------------------
```


# 属性描述对象

> - JavaScript 提供一个内部数据结构，用来描述对象的属性，控制它的行为，比如该属性是否可写、可遍历等等，这个内部数据结构称为“属性描述对象”（attributes object）
> - **每个属性**都有自己对应的属性描述对象，保存该属性的一些元信息

## 属性描述对象的元属性

属性描述对象提供 **6** 个元属性：

元属性 | 说明
----- | ----
value | value 是该属性的属性值，默认为undefined。
writable | writable 是一个布尔值，表示属性值（value）是否可改变（即是否可写），默认为true
enumerable | enumerable 是一个布尔值，表示该属性是否可遍历，默认为true。如果设为false，会使得某些操作（比如for...in循环、Object.keys()）跳过该属性
configurable | configurable 是一个布尔值，表示可配置性，可写性，默认为true
get | get 是一个函数，表示该属性的取值函数（getter），默认为undefined
set | set 是一个函数，表示该属性的存值函数（setter），默认为undefined

## 属性描述对象的几个方法


方法 | 说明
----- | ----
Object.getOwnPropertyDescriptor() | 获取当前对象自身属性的属性描述对象，第一个参数是目标对象，第二个参数是属性名
Object.getOwnPropertyNames() | 返回一个包含参数对象自身全部属性的属性名数组，不管该属性是否可遍历
Object.defineProperty() | 通过属性描述对象，定义或修改一个属性，返回修改后的对象，接受三个参数：object(属性所在的对象)、propertyName(字符串，表示属性名)、attributesObject(属性描述对象)
Object.defineProperties() | 一次性定义或修改多个属性
Object.prototype.propertyIsEnumerable() | 返回一个布尔值，用来判断对象自身某个属性是否可遍历




# Array对象

## Array 构造函数

> - Array 是 JavaScript 的原生对象，同时也是一个构造函数，可以用它生成新的数组，但建议使用直接定义数组的方法
> - 当参数是一个正整数，返回数组的成员都是空位，读取时返回 undefined，但实际该位置没有任何值。这时可以读取到 length 属性，但是取不到键名（索引）


## Array 的静态方法

```js
// Array.isArray()
// -------------------------------
// 返回一个布尔值，表示参数是否为数组
var arr = [1, 2, 3];

typeof arr // "object"
Array.isArray(arr) // true
Object.prototype.toString.call(arr) // "[object Array]"
```


## Array 的实例方法


方法 | 作用
---- | -----
valueOf()   | 返回数组本身
toString()  | 返回数组的字符串形式
push()      | 在数组的**末端**添加一或多个元素，并返回添加新元素后的数组**长度**
pop()       | 删除数组的**最后**一个元素，并返回该元素
shift()     | 删除数组的**第一个**元素，并返回该元素
unshift()   | 在数组的**第一个**位置添加元素，并返回添加新元素后的数组**长度**
join()      | 以指定参数作为分隔符（默认逗号），将所有数组成员连接为一个字符串返回
concat()    | 合并多个数组，添加在尾部，返回新数组，**原数组不变**
reverse()   | 颠倒排列数组元素，返回改变后的数组
slice(start, end) | **提取**目标数组的一部分，返回一个新数组，原数组不变；还可将**类似数组对象**转为数组
splice(start, count, addElement1, ...) | **删除**原数组的一部分成员，并可以在删除的位置**添加**新的数组成员，返回值是被删除的元素
sort()      | 对数组成员进行排序，默认是按照**字典顺序**排序
map()       | 将数组的所有成员依次传入第一个参数指定的**函数**（有三个参数：当前成员、当前位置和数组本身），同时第二个参数指定函数内部 this 指向的**对象**，然后把每一次的执行结果组成一个**新数组**返回
forEach()   | 对数组的所有成员依次执行参数函数，参数同 map，过程**无法中断**，结果无返回值
filter()    | 过滤数组成员，满足第一个参数指定函数为 true，第二个参数绑定函数内部 this 的成员组成一个新数组返回
some()      | 第一个参数指定一个函数，第二个参数函数绑定函数内部的 this，所有数组成员依次执行该函数。该函数接受三个参数：当前成员、当前位置和整个数组，同时然后返回一个布尔值，只要一个成员的返回值是true，则整个some方法的返回值就是true，否则返回false
every()     | 参数同 some，every 方法是所有成员的返回值都是 true，整个every方法才返回true，否则返回 false
reduce()    | 从左到右依次处理数组的每个成员，最终累计为一个值。第一个参数是一个函数(必须有返回值)，接受以下四个参数：累积变量(默认arr[0])，当前变量(默认arr[1])，当前位置（从0开始），原数组；第二个参数可以指定累积的初始值
reduceRight() | 从右到左依次处理数组的每个成员，最终累计为一个值。参数同 reduce
indexOf()     | 返回给定元素在数组中第一次出现的位置，如果没有出现则返回 -1，第二个参数指定开始位置
lastIndexOf() | 返回给定元素在数组中最后一次出现的位置，如果没有出现则返回-1


# 包装对象


> - 数值、字符串、布尔值在一定条件下，也会自动转为对象，也就是原始类型的“包装对象”
> - 所谓“包装对象”，指的是与数值、字符串、布尔值分别相对应的Number、String、Boolean三个原生对象
> - 这三个对象作为**构造函数**使用（带有new）时，可以将原始类型的值转为对象；作为**普通函数**使用时（不带有new），可以将任意类型的值，转为原始类型的值。



```js
// 分别将三个值转为包装对象
var v1 = new Number(123);
var v2 = new String('abc');
var v3 = new Boolean(true);

typeof v1 // "object"
typeof v2 // "object"
typeof v3 // "object"

v1 === 123 // false
v2 === 'abc' // false
v3 === true // false

// 字符串转为数值
Number('123') // 123
// 数值转为字符串
String(123)   // "123"
// 数值转为布尔值
Boolean(123)  // true
```

## 包装对象的实例方法


方法 | 作用
---- | -----
valueOf() | 返回包装对象实例对应的原始类型的值
toString()| 返回对应的字符串形式

```js
new Number(123).valueOf()   // 123
new String('abc').valueOf() // "abc"
new Boolean(true).valueOf() // true

new Number(123).toString()   // "123"
new String('abc').toString() // "abc"
new Boolean(true).toString() // "true"
```

## 原始类型与实例对象的自动转换

> 某些场合，原始类型的值会自动当作包装对象调用，即调用包装对象的属性和方法。这时，JavaScript 引擎会自动将原始类型的值转为包装对象实例，并在使用后立刻销毁实例，所以是暂时性的。但自动转换生成的包装对象是只读的，无法修改




# Boolean 对象

> Boolean 对象是 JavaScript 的三个包装对象之一。作为构造函数，它主要用于生成布尔值的包装对象实例


```js
// b 是一个 Boolean 对象的实例，类型是对象，值为布尔值true。
var b = new Boolean(true);

typeof b // "object"
b.valueOf() // true
```


## Boolean 函数的类型转换作用

> - Boolean 对象除了可以作为构造函数，还可以单独使用，将任意值转为布尔值。这时Boolean 就是一个单纯的工具方法
> - 此外使用双重的否运算符（!）也可以将任意值转为对应的布尔值

```js
// 将任意值转为布尔值
// 一些值的默认布尔值
Boolean(undefined)  // false
Boolean(null)       // false
Boolean(0)          // false
Boolean('')         // false
Boolean(NaN)        // false

Boolean(1)          // true
Boolean('false')    // true
Boolean([])         // true
Boolean({})         // true  对象对应的布尔值是 true
Boolean(function () {}) // true
Boolean(/foo/)      // true

// 双重的否运算符（!）将值转为布尔值
!!1         // true
!!'false'   // true
!![]        // true
!!{}        // true
```



# Number 对象

> - Number 对象是数值对应的包装对象，可以作为构造函数使用，也可以作为工具函数使用
> - 作为构造函数时，Number 对象用于生成值为数值的对象
> - 作为工具函数时，Number 对象可以将任何类型的值转为数值（见数据类型转换章节）


## Number 对象的本身属性


属性 | 作用
---- | -----
Number.POSITIVE_INFINITY | 正的无限，指向Infinity。
Number.NEGATIVE_INFINITY | 负的无限，指向-Infinity。
Number.NaN               | 表示非数值，指向 NaN。
Number.MIN_VALUE         | 表示最小的正数（即最接近0的正数，在64位浮点数体系中为5e-324），相应的，最接近0的负数为 -Number.MIN_VALUE。
Number.MAX_SAFE_INTEGER  | 表示能够精确表示的最大整数，即 9007199254740991
Number.MIN_SAFE_INTEGER  | 表示能够精确表示的最小整数，即 -9007199254740991


## Number 对象的实例方法


方法 | 作用
---- | -----
Number.prototype.toString() | 将一个数值转为字符串形式，接受一个参数，表示输出的进制，数值需要放进圆括号
Number.prototype.toFixed()  | 将一个数转为指定位数的小数，接受一个参数，指定小数位数，返回对应的**字符串**
Number.prototype.toExponential() | 将一个数转为科学计数法形式，参数是小数点后有效数字的位数，范围为0到100
Number.prototype.toPrecision() | 将一个数转为指定位数的有效数字，参数为有效数字的位数，范围是1到100
Number.prototype.toLocaleString()| 接受一个地区码作为参数，返回一个字符串，表示当前数字在该地区的当地书写形式



# String 对象


> - String 对象是 JavaScript 原生提供的三个包装对象之一，用来生成字符串对象
> - 字符串对象是一个类似数组的对象

## String 对象的静态方法

```js
// String.fromCharCode()
// 参数是一个或多个数值，代表 Unicode 码点，返回值是这些码点组成的字符串
```

## String 对象的实例属性

```js
// String.prototype.length
// 返回字符串的长度
```

## String 对象的实例方法


方法 | 作用
---- | -----
String.prototype.charAt()     | 返回指定位置的字符，参数是从0开始编号的位置
String.prototype.charCodeAt() | 返回字符串指定位置的 Unicode 码点
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
String.prototype.split()      | 按照给定规则**分割**字符串，返回一个由分割出来的子字符串组成的数组，可为空字符串，第二个参数可限定返回数组的最大成员数
String.prototype.localeCompare()| 用于**比较**两个字符串。它返回一个整数表示被比较与参数指定的比较字符串的长度差，此外还会考虑自然语言的顺序


# Math 对象

> Math 是 JavaScript 的原生对象，提供各种数学功能。该对象不是构造函数，不能生成实例，所有的属性和方法都必须在 **Math 对象**上调用

## Math 对象的静态属性

属性（只读） | 作用
---- | -----
Math.E       | 常数e
Math.LN2     | 2 的自然对数
Math.LN10    | 10 的自然对数
Math.LOG2E   | 以 2 为底的e的对数
Math.LOG10E  | 以 10 为底的e的对数
Math.PI      | 常数 π
Math.SQRT1_2 | 0.5 的平方根
Math.SQRT2   | 2 的平方根


## Math 对象的静态方法

方法 | 作用
---- | -----
Math.abs()   | 绝对值
Math.ceil()  | 向上取整
Math.floor() | 向下取整
Math.max()   | 最大值
Math.min()   | 最小值
Math.pow()   | 幂运算
Math.sqrt()  | 平方根
Math.log()   | 自然对数，以 e 为底
Math.exp()   | e 的指数
Math.round() | 四舍五入
Math.random()| 随机数


```js
// Math.abs()
// ----------------------------------------
// 绝对值
Math.abs(-1) // 1
// ----------------------------------------


// Math.max()
// Math.min()
// ----------------------------------------
// 取最值
Math.max(2, -1, 5) // 5
Math.min(2, -1, 5) // -1
Math.min() // Infinity
Math.max() // -Infinity
// ----------------------------------------


// Math.floor()
// Math.ceil()
// ----------------------------------------
// 返回小于参数值的最大整数 地板值
Math.floor(3.2)  // 3
Math.floor(-3.2) // -4
// 返回大于参数值的最小整数 天花板值
Math.ceil(3.2)  // 4
Math.ceil(-3.2) // -3

// 结合定义一个取整值的函数
// 正值调用 floor 保留整数位，负值调用 ceil 保留整数位
function ToInteger(x) {
  x = Number(x);
  return x < 0 ? Math.ceil(x) : Math.floor(x);
}

ToInteger(3.2) // 3
ToInteger(3.8) // 3
ToInteger(-3.2) // -3
ToInteger(-3.8) // -3
// ----------------------------------------


// Math.round()
// ----------------------------------------
// 四舍五入
Math.round(0.1) // 0
Math.round(0.5) // 1
Math.round(0.6) // 1

Math.round(-1.1) // -1
Math.round(-1.5) // -1
Math.round(-1.6) // -2
// ----------------------------------------


// Math.pow()
// ----------------------------------------
// 第一个参数为底数、第二个参数为指数的幂运算
// 等同于 2 ** 2
Math.pow(2, 2) // 4
// 等同于 2 ** 3
Math.pow(2, 3) // 8
// 计算圆面积
var radius = 20
var area = Math.PI * Math.pow(radius, 2)
// ----------------------------------------


// Math.sqrt()
// ----------------------------------------
// 返回参数值的平方根。如果参数是一个负值，则返回 NaN
Math.sqrt(4) // 2
Math.sqrt(-4) // NaN
// ----------------------------------------


// Math.log()
// ----------------------------------------
// 返回以 e 为底的自然对数值
Math.log(Math.E) // 1
Math.log(10)     // 2.302585092994046

// 计算以 10 为底的对数
Math.log(100)/Math.LN10 // 2
// 相当于 ln100/ln10 = lg100 = 2

// 计算以 2 为底的对数
Math.log(8)/Math.LN2 // 3
// 相当于 ln8/ln2 = log₂8 = 3
// ----------------------------------------


// Math.exp()
// ----------------------------------------
// 返回常数 e 的参数次方
Math.exp(1) // 2.718281828459045
Math.exp(3) // 20.085536923187668
// ----------------------------------------


// Math.random()
// ----------------------------------------
// 返回 0 到 1 之间的一个伪随机数，可能等于 0，但是一定小于1
Math.random() // 0.7151307314634323

// 任意范围的随机数生成函数如下
// 将 0 到 1 的范围扩大到指定范围，再加最小边界值
function getRandomArbitrary(min, max) {
  return Math.random() * (max - min) + min;  
}

getRandomArbitrary(1.5, 6.5)
// 2.4942810038223864

// 任意范围的随机整数生成函数如下
function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

getRandomInt(1, 6) // 5
// ----------------------------------------


// 三角函数
// ----------------------------------------
Math.sin()  // 返回参数的正弦（参数为弧度值）
Math.cos()  // 返回参数的余弦（参数为弧度值）
Math.tan()  // 返回参数的正切（参数为弧度值）
Math.asin() // 返回参数的反正弦（返回值为弧度值）
Math.acos() // 返回参数的反余弦（返回值为弧度值）
Math.atan() // 返回参数的反正切（返回值为弧度值）

Math.sin(Math.PI / 2)  // 1
// ----------------------------------------
```


# Date 对象

> Date对象是 JavaScript 原生的时间库。它以国际标准时间（UTC）1970年1月1日00:00:00作为时间的零点，可以表示的时间范围是前后各1亿天（单位为毫秒）

## Date()

> Date对象可以作为普通函数直接调用，返回一个代表当前时间的字符串。即使带有参数，Date作为普通函数使用时，返回的还是当前时间。

## Date 构造函数

> 使用 new 命令，会返回一个 Date 对象的实例。如果不加参数，实例代表的就是当前时间。

```js
// Date对象可以接受多种格式的参数，返回一个该参数对应的时间实例。
// 参数为年、月、日等多个整数时，年和月是不能省略的

// 参数为时间零点开始计算的毫秒数
new Date(1378218728000)
// Tue Sep 03 2013 22:32:08 GMT+0800 (CST)

// 参数为日期字符串
new Date('January 6, 2013');
// Sun Jan 06 2013 00:00:00 GMT+0800 (CST)

// 参数为多个整数，
// 代表年、月、日、小时、分钟、秒、毫秒
new Date(2013, 0, 1, 0, 0, 0, 0)
// Tue Jan 01 2013 00:00:00 GMT+0800 (CST)
```

## Date 对象的静态方法


方法 | 作用
---- | -----
Date.now()  | 返回当前时间距离时间零点（1970年1月1日 00:00:00 UTC）的毫秒数
Date.parse()| 解析日期字符串，返回该时间距离时间零点（1970年1月1日 00:00:00）的毫秒数
Date.UTC()  | 接受年、月、日等变量作为参数，返回该时间距离时间零点（1970年1月1日 00:00:00 UTC）的毫秒数


## Date 对象的实例方法

Date 的实例对象，有几十个自己的方法，除了 valueOf，可以分为以下三类。
- to类：从Date对象返回一个字符串，表示指定的时间。
- get类：获取Date对象的日期和时间。
- set类：设置Date对象的日期和时间。
具体可在 [网道 - Date 对象](https://wangdoc.com/javascript/stdlib/date.html#%E5%AE%9E%E4%BE%8B%E6%96%B9%E6%B3%95) 查看

方法 | 作用
---- | -----
Date.prototype.valueOf() | 返回实例对象距离时间零点（1970年1月1日00:00:00 UTC）对应的毫秒数

to 类方法 | 作用
------- | -----
Date.prototype.toString()      | 返回一个完整的日期字符串
Date.prototype.toUTCString()   | 返回对应的 UTC 时间，也就是比北京时间晚8个小时
Date.prototype.toISOString()   | 返回对应时间的 ISO8601 写法
Date.prototype.toJSON()        | 返回一个符合 JSON 格式的 ISO 日期字符串
Date.prototype.toDateString()  | 返回日期字符串（不含小时、分和秒）
Date.prototype.toTimeString()  | 返回时间字符串（不含年月日）
Date.prototype.toLocaleString()| 将 Date 实例转为表示完整的本地时间的字符串
Date.prototype.toLocaleDateString()| 将 Date 实例转为表示本地日期（不含小时、分和秒）
Date.prototype.toLocaleTimeString()| 将 Date 实例转为表示本地时间（不含年月日）

get 类方法（当前时区） | 作用
-------- | -----
getTime()          | 返回实例距离1970年1月1日00:00:00的毫秒数，等同于valueOf方法
getDate()          | 返回实例对象对应每个月的几号（从1开始）
getDay()           | 返回星期几，星期日为0，星期一为1，以此类推
getFullYear()      | 返回四位的年份
getMonth()         | 返回月份（0表示1月，11表示12月）
getHours()         | 返回小时（0-23）
getMilliseconds()  | 返回毫秒（0-999）
getMinutes()       | 返回分钟（0-59）
getSeconds()       | 返回秒（0-59）
getTimezoneOffset()| 返回当前时间与 UTC 的时区差异，以分钟表示，返回结果考虑到了夏令时因素

set 类方法（当前时区） | 作用
-------- | -----
setDate(date)       |设置实例对象对应的每个月的几号（1-31），返回改变后毫秒时间戳。
setFullYear(year [, month, date])   |设置四位年份。
setHours(hour [, min, sec, ms])     |设置小时（0-23）。
setMilliseconds()           |设置毫秒（0-999）。
setMinutes(min [, sec, ms]) |设置分钟（0-59）。
setMonth(month [, date])    |设置月份（0-11）。
setSeconds(sec [, ms])      |设置秒（0-59）。
setTime(milliseconds)       |设置毫秒时间戳。



# RegExp 对象

> - 正则表达式（regular expression）是一种表达文本模式（即字符串结构）的方法，有点像字符串的模板，常常用来按照“给定模式”**匹配**文本。
> - 新建正则表达式有两种方法：一种是使用字面量（常用），以斜杠表示开始和结束；另一种是使用 RegExp 构造函数。
> - 正则表达式的具体规则见[正则表达式规则](http://lixupeng.cn/blog/2020/05/01/2-regex1/)

```js
var regex = /xyz/;    // 效率高 便利直观 常用
var regex = new RegExp('xyz');

// RegExp 构造函数还可以接受第二个参数，表示修饰符
var regex = new RegExp('xyz', 'i');
// 等价于
var regex = /xyz/i;
```

## RegExp 对象的实例属性

修饰符相关属性（只读） | 作用
---------- | -------
RegExp.prototype.ignoreCase | 返回一个布尔值，表示是否设置了 **i 修饰符**
RegExp.prototype.global     | 返回一个布尔值，表示是否设置了 **g 修饰符**
RegExp.prototype.multiline  | 返回一个布尔值，表示是否设置了 **m 修饰符**
RegExp.prototype.flags      | 返回一个字符串，包含了已经设置的**所有修饰符**，按字母排序


```js
var r = /abc/igm;

r.ignoreCase // true
r.global // true  全局搜索
r.multiline // true
r.flags // 'gim'
```


与修饰符无关的属性 | 作用
----------- | ---------
RegExp.prototype.lastIndex| 返回一个整数，表示下一次**开始搜索的位置**。该属性可读写，但是只在进行连续搜索时有意义，且只对同一个正则表达式有效。
RegExp.prototype.source   | 返回正则表达式的字符串形式（不包括反斜杠），该属性只读


```js
var r = /abc/igm;
var s = '_abc_abc';

r.lastIndex         // 0     默认从头开始
r.lastIndex = 4;    // 指定从第五位开始
r.test(s)           // true  可以匹配到 abc

r.source // "abc"
```


## RegExp 对象的实例方法


方法 | 作用
---- | -----
RegExp.prototype.test() | 返回一个布尔值，表示当前模式是否能匹配参数字符串
RegExp.prototype.exec() | 用来返回匹配结果。如果发现匹配，就返回一个包含匹配成功的子字符串数组，否则返回 null


```js
var s = '_x_x';
var r1 = /x/;
var r2 = /y/;

r1.exec(s) // ["x"]   返回匹配到的字符串数组
r2.exec(s) // null    匹配不到

// 正则表示式包含圆括号（即含有“组匹配”），则返回的数组会包括多个成员
// 第一个元素字符串是整个匹配的结果
// 第二个元素字符串是圆括号匹配的结果
var s = '_x_x';
var r = /_(x)/;

r.exec(s) // ["_x", "x"]
```

# JSON 对象

> JSON 对象是 JavaScript 的原生对象，用来处理 JSON 格式数据

## JSON 格式

> - JSON（JavaScript Object Notation）格式是一种用于数据交换的文本格式，2001年由 Douglas Crockford 提出，目的是取代繁琐笨重的 XML 格式。
>
> - 相比 XML 格式，JSON 格式有两个显著的优点：书写简单，一目了然；符合 JavaScript 原生语法，可以由解释引擎直接处理，不用另外添加解析代码。
>
> - 每个 JSON 对象就是一个值，可能是一个数组或对象，也可能是一个原始类型的值。总之，只能是一个值，不能是两个或更多的值

> **JSON 对值的类型和格式的规定:**
> - **复合类型**的值只能是数组或对象，不能是函数、正则表达式对象、日期对象。
> - **原始类型**的值只有四种：字符串、数值（必须以十进制表示）、布尔值和 null（不能使用**NaN, Infinity, -Infinity和undefined**）。
> - 字符串必须使用**双引号**表示，不能使用单引号。
> - 对象的**键名**必须放在双引号里面。
> - 数组或对象最后一个成员的后面，不能加**逗号**。


## JSON 对象的静态方法

方法 | 作用
---- | -----
JSON.stringify() | 用于将一个值转为 JSON 字符串
JSON.parse()     | 用于将 JSON 字符串转换成对应的值


```js
// JSON.stringify()
// ------------------------------
// 将第一个参数指定的值转为 JSON 字符串
// 该方法接受一个数组，作为第二个参数，指定需要转成字符串的属性
// 接受第三个参数，用于增加返回的 JSON 字符串的可读性
//   如果是数字，表示每个属性前面添加的空格（最多不超过10个）；
//   如果是字符串（不超过10个字符），则该字符串会添加在每行前面
JSON.stringify('abc')  // ""abc""
JSON.stringify(1)      // "1"
JSON.stringify(false)  // "false"
JSON.stringify([])     // "[]"
JSON.stringify({})     // "{}"

JSON.stringify([1, "false", false])  // '[1,"false",false]'

JSON.stringify({ name: "张三" })     // '{"name":"张三"}'

// 
JSON.stringify('abc') === "\"abc\"" // true
// ------------------------------


// JSON.parse()
// ------------------------------
// 将 JSON 字符串转换成对应的值
// JSON.parse方法可以接受一个处理函数，作为第二个参数
JSON.parse('{}') // {}
JSON.parse('true') // true
JSON.parse('"foo"') // "foo"
JSON.parse('[1, 5, "false"]') // [1, 5, "false"]
JSON.parse('null') // null

var o = JSON.parse('{"name": "张三"}');
o.name // 张三
// ------------------------------
```


# 参考资料

[网道- JavaScript教程 - 标准库](https://wangdoc.com/javascript/stdlib/index.html)
[网道- JavaScript教程 - Object对象](https://wangdoc.com/javascript/stdlib/object.html)
[网道- JavaScript教程 - Array对象](https://wangdoc.com/javascript/stdlib/array.html)
[网道- JavaScript教程 - RegExp对象](https://wangdoc.com/javascript/stdlib/regexp.html)
[网道- JavaScript教程 - JSON对象](https://wangdoc.com/javascript/stdlib/json.html)