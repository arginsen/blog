---
title: 前端 | JS深入 | 六、数据类型的转换
date: 2020-5-12 16:12:21
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

> - JavaScript 是一种动态类型语言，变量没有**类型限制**，可以随时赋予任意值。
> - 运算符对数据类型是有要求的。如果运算符发现，运算子的类型与预期不符，就会自动转换类型。
> - 数据类型的转换分为**强制转换**和**自动转换**


```js
// x 的类型没法在编译阶段就知道，必须等到运行时才能知道
var x = y ? 1 : 'a';

// 减法运算符对数据类型的要求是数值，此时两个字符串相减，那么就会自动转换
'4' - '3' // 1
```

# 强制转换

> 强制转换主要指使用 Number()、String() 和 Boolean() 三个函数，手动将各种类型的值，分别转换成数字、字符串或者布尔值

## Number()

> 使用 Number 函数，可以将任意类型的值转换数值。主要分为两种情况，一种是参数是**原始类型的值**，另一种是参数是**对象**

### 原始类型值

```js
// 数值：转换后还是原来的值
Number(324) // 324

// 字符串：如果可以被解析为数值，则转换为相应的数值
Number('324') // 324

// 字符串：如果不可以被解析为数值，返回 NaN
Number('324abc') // NaN

// 空字符串转为0
Number('') // 0

// 布尔值：true 转成 1，false 转成 0
Number(true) // 1
Number(false) // 0

// undefined：转成 NaN
Number(undefined) // NaN

// null：转成 0
Number(null) // 0

// Number 函数将字符串转为数值，要比 parseInt 函数严格很多
// 只要有一个字符无法转成数值，整个字符串就会被转为 NaN
parseInt('42 cats') // 42
Number('42 cats')   // NaN

// parseInt 和 Number 函数都会自动过滤一个字符串前导和后缀的空格
```

### 对象

> Number 方法的参数是对象时，将返回 NaN，除非是包含单个数值的数组

```js
Number({a: 1}) // NaN
Number([1, 2, 3]) // NaN
Number([5]) // 5


// Number 背后的转换规则比较复杂：
// 第一步，调用对象自身的 valueOf 方法。如果返回原始类型的值，则直接对该值使用 Number 函数，不再进行后续步骤
// 第二步，如果 valueOf 方法返回的还是对象，则改为调用对象自身的 toString 方法。如果 toString 方法返回原始类型的值，则对该值使用 Number 函数，不再进行后续步骤
// 第三步，如果 toString 方法返回的是对象，就报错


// Number 将 obj 转为数值，首先调用 valueOf()，返回对象本身
// 满足条件，则调用 toString()，返回 [object Object]
// 最后执行 Number()，得到 NaN
var obj = {x: 1};
Number(obj) // NaN

// 等同于
if (typeof obj.valueOf() === 'object') {
  Number(obj.toString());
} else {
  Number(obj.valueOf());
}


// 按照 Number 背后的逻辑，在一二步会调用 valueOf、toString
// 那么这两个方法对象自身也是可以自定义的
// 自定义返回如果是数值，则会返回相应值，其他自定义会导致报错
var obj = {
  valueOf: function () {
    return {};
  },
  toString: function () {
    return {};
  }
};

Number(obj)  // TypeError

// 返回 valueOf 方法的值
Number({
  valueOf: function () {
    return 2;
  }
})  // 2

// 返回 toString 方法的值
Number({
  toString: function () {
    return 3;
  }
})  // 3

// valueOf 方法先于 toString 方法执行，
// 调用 valueOf 后对象直接返回数值
// 则直接执行 Number(obj.valueOf()) 
Number({
  valueOf: function () {
    return 2;
  },
  toString: function () {
    return 3;
  }
})  // 2
```

## String()

> String函数可以将任意类型的值转化成字符串

### 原始类型值

```js
// 数值：转为相应的字符串
// 字符串：转换后还是原来的值
// 布尔值：true转为字符串"true"，false转为字符串"false"
// undefined：转为字符串"undefined"
// null：转为字符串"null"
String(123) // "123"
String('abc') // "abc"
String(true) // "true"
String(undefined) // "undefined"
String(null) // "null"
```

### 对象

> String方法的参数如果是对象，返回一个类型字符串；如果是数组，返回该数组的字符串形式。

```js
String({a: 1}) // "[object Object]"
String([1, 2, 3]) // "1,2,3"


// String 方法背后的转换规则，与 Number 方法基本相同，
// 只是互换了 valueOf 方法和 toString 方法的执行顺序：
// 第一步，先调用对象自身的 toString 方法。如果返回原始类型的值，则对该值使用 String 函数，不再进行以下步骤。
// 第二步，如果 toString 方法返回的是对象，再调用原对象的 valueOf 方法。如果 valueOf 方法返回原始类型的值，则对该值使用 String 函数，不再进行以下步骤。
// 第三步，如果 valueOf 方法返回的是对象，就报错。

// String 参数是对象，首先调用 toString()，
// 返回字符串 "[object Object]"，就不再向下执行了
String({a: 1})
// "[object Object]"

// 等同于
String({a: 1}.toString())
// "[object Object]"


// 同样，
// 如果 toString 法和 valueOf 方法，返回的都是对象，就会报错
// 如下就相当于对象自己定义了方法，不是这两个原来继承的同名方法
var obj = {
  valueOf: function () {
    return {};
  },
  toString: function () {
    return {};
  }
};

String(obj)
// TypeError


// 也可以自定义，令对象返回指定的值
// 定义 toString 方法直接返回数值（原始类型）

// 返回 toString 方法的值 
String({
  toString: function () {
    return 3;
  }
})
// "3"

// 返回的还是toString方法的值 [object Object]
String({
  valueOf: function () {
    return 2;
  }
})
// "[object Object]"

// 说明相比 Number，String 这两个方法 toString() 优先
String({
  valueOf: function () {
    return 2;
  },
  toString: function () {
    return 3;
  }
})
// "3"
```


## Boolean()

> Boolean() 函数可以将任意类型的值转为布尔值

```js
// 除了以下五个值的转换结果为 false，其他的值全部为 true
Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean(NaN) // false
Boolean('') // false
```

# 自动转换

> 自动转换以强制转换为基础
>
> 预期什么类型的值，就调用该类型的**转换函数**。比如，某个位置预期为字符串，就调用 String 函数进行转换

```js
// 以下三类情况会自动转换数据类型

// 第一种情况，不同类型的数据互相运算
123 + 'abc' // "123abc"

// 第二种情况，对非布尔值类型的数据求布尔值
if ('abc') {
  console.log('hello')
}  // "hello"

// 第三种情况，对非数值类型的值使用一元运算符（+、-）
+ {foo: 'bar'} // NaN
- [1, 2, 3] // NaN
```

## 自动转换为布尔值

> JavaScript 遇到预期为布尔值的地方（比如 if 语句的条件部分），就会将非布尔值的参数自动转换为布尔值。系统内部会自动调用 Boolean 函数
> 
> 除了 undefined/null/+0 或-0/NaN/''（空字符串），其他都是自动转为 true。

```js
// 条件部分全为 true
if ( !undefined
  && !null
  && !0
  && !NaN
  && !''
) {
  console.log('true');
}  // true


// 将一个表达式转为布尔值
// 写法一
expression ? true : false

// 写法二
!! expression
```

## 自动转换为字符串

> JavaScript 遇到预期为字符串的地方，就会将非字符串的值自动转为字符串。具体规则是，先将复合类型的值转为原始类型的值，再将原始类型的值转为字符串

```js
// 加法运算时，当一个值为字符串，另一个值为非字符串
'5' + 1 // '51'
'5' + true // "5true"
'5' + false // "5false"
'5' + {} // "5[object Object]"
'5' + [] // "5"
'5' + function (){} // "5function (){}"
'5' + undefined // "5undefined"
'5' + null // "5null"
```

## 自动转换为数值

> JavaScript 遇到预期为数值的地方，就会将参数值自动转换为数值。系统内部会自动调用 Number 函数

```js
// 除了加法运算符（+）有可能把运算子转为字符串，
// 其他运算符都会把运算子自动转成数值
'5' - '2' // 3
'5' * '2' // 10
true - 1  // 0
false - 1 // -1
'1' - 1   // 0
'5' * []    // 0
false / '5' // 0
'abc' - 1   // NaN
null + 1 // 1
undefined + 1 // NaN

// 一元运算符也会把运算子转成数值
+'abc' // NaN
-'abc' // NaN
+true // 1
-false // 0
```

# 参考资料

[网道- JavaScript教程 - 数据类型的转换](https://wangdoc.com/javascript/features/conversion.html)
[冴羽的博客 - JavaScript深入之头疼的类型转换(上)](https://github.com/mqyqingfeng/Blog/issues/159)
[冴羽的博客 - JavaScript深入之头疼的类型转换(下)](https://github.com/mqyqingfeng/Blog/issues/164)
