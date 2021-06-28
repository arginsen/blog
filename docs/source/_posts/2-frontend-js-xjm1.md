---
title: 前端 | 萧井陌JS学习笔记Ⅰ：变量、循环、函数
date: 2020-4-11 11:25:00
categories: notes
tags:
 - Frontend
 - JavaScript
photo: 
    - /blog/img/javascript.jpg
---

<br>
<!--more-->


# js的变量

> - 变量名（函数名也是变量名的一种）命名规则
> - 变量名只能包含字母（汉字也是字母）、下划线、数字、$
> - 不能以数字开头
> - 现代的 JavaScript 中汉字才是合法的变量名字母
> - 变量名不能是系统保留字（while、function、var等）


# js的数据类型

> - 程序中的数据不仅仅只有数字一种
> - 还有很多其他类型的数据，可以将它们显示出来

> - 数字类型是 Number，又可以分为 整型(整数) 和 浮点型(小数) 
> - 双引号或单引号包起来的字符就是字符串

# js实例讲解循环、函数、判断

## var变量的用法

```js
// 普通画一个三角形
forward(50)
right(120)
forward(50)
right(120)
forward(50)
right(120)

// 使用变量简介表达
var x = 50
var angle = 120
// 执行三次
forward(x)
right(angle)
forward(x)
right(angle)
forward(x)
right(angle)
```

## 循环

```js
// 使用循环画一个三角形
var n = 3
var angle = 120
var x = 50
var i = 0
while (i < n) {
    i = i + 1
    forward(x)
    right(angle)
}

// 使用循环画一个多边形
var x = 50
var angle = - ((n - 2) * 180 / n) + 180
var n = 9
var i = 0
while (i < n) {
    forward(x)
    right(angle)
    i = i + 1
}
```

## 函数

### 定义一个函数square

```js
// 画一个三角形
var square = function() {
    var x = 50
    var n = 3
    var angle = 120
    var i = 0
    while (i < n) {
        forward(x)
        right(angle)
        i = i + 1
    }
}

square()
// 可以多次调用函数
forward(50)
square()
```

### 可以定义函数的参数

```js
// 画一个三角形
// square(length)
var square = function(length) {
    // length 作为函数的参数，只能在函数内部可见
    var x = length
    var n = 3
    var angle = 120
    var i = 0
    while (i < n) {
        forward(x)
        right(angle)
        i = i + 1
    }
}

square(50)
```

### 函数可以有多个参数

```js
// 多个参数用逗号隔开
// 画一个正方形，x, y 是正方形左上角坐标，len为边长
// square(x, y, len)
var square = function(x, y, len) {
    jump (x, y)
    setHeading(0)
    var i = 0
    while (i < 4) {
        forward(len)
        right(90)
        i = i + 1
    }
}

square(0, 0, 50)
```

### 在其他函数中调用之前的函数

```js
// 从 0, 0 点开始，边长为30，正方形间隔为 10 像素，画五个
// x, y 为第一个正方形左上角坐标 
// n 是正方形个数 
// space 是正方形间距
// len 是正方形的边长
// square5(x, y, n, space, len)
var square = function(x, y, len) {
    jump(x, y)
    setHeading(0)
    var i = 0
    while (i < 4) {
        forward(len)
        right(90)
        i = i + 1
    }
}

var square5 = function(x, y, n, space, len) {
    jump(x, y)
    setHeading(0)
    // 0 40 80 120 160
    var i = 0
    while (i < n) {
        // 画 n 个正方形
        var x = i * (len + space)
        var y = 0
        square(x, y, len)
        i = i + 1
    }
}

square5(0, 0, 5, 10, 30)


// 以上边函数为基础，画 3 * 3 正方形矩阵
// x, y 为第一个正方形左上角坐标 
// n 是横向正方形个数 
// m 是纵向正方形个数
// space 是正方形间距
// len 是正方形的边长
// square_square(x, y, space, len, n, m)
var square = function(x, y1, len) {
    jump(x, y1)
    setHeading(0)
    var i = 0
    while(i < 4) {
        forward(len)
        right(90)
        i = i + 1
    }
}

var square5 = function(x, y1, n, space, len) {
    jump(x, y1)
    setHeading(0)
    // 0 40 80 120 160
    var i = 0
    while(i < n) {
        // 画 n 个正方形
        var x = i * (len + space)
        var y = 0
        square(x, y1, len)
        i = i + 1
    }
}

var square_square = function(x, y, space, len, n, m) {
    jump(x, y1)
    setHeading(0)
    var i = 0
    while(i < m) {
        // 对新的坐标进行赋值，相应的改变上边两个变量的参数、jump值
        var x1 = x
        var y1 = i * (len + space) + y
        square5(x1, y1, n, space, len)
        i = i + 1
    }
}

square_square(0, 0, 10, 30, 3, 3)
```

### 调用函数画正多边形

```js
// length 表示边长，bmuu 表示边数
// polygon(length, bmuu)
var polygon = function(length, bmuu) {
    // length 作为函数的参数，只能在函数内部可见
    var x = length
    var n = bmuu
    var angle = 180 - ((n - 2) * 180 / n)
    var i = 0
    while(i < n) {
        forward(x)
        right(angle)
        i = i + 1
    }
}

polygon(50, 4)
```

### 定义函数两种形式

```js
// 函数申明语句
function square () {
}
// 函数表达式
var square = function() {
}
```

## 判断

### 选择控制

```js
// if 可以根据情况选择性执行某些语句
var grade = 3
if(grade < 7) {
    // 采用浏览器的console.log, 得到返回值
    var log = function() {
        console.log.apply(console, arguments)
    }
    log('小学生')
}


// 选择控制的多种用法

// 只有 if
if (1 > 2) {
    log('条件成立')
}

// if 带 else
if (1 > 2) {
    log('条件成立')
} else {
    log('条件不成立')
}

// 多个 if else
var grade = 8
if (grade < 7) {
    log('小学生')
} else if (grade < 10) {
    log('初中生')
} else {
    log('高中生')
}
```

### 判断语句的用途

```js
// 求绝对值
var n = -1
if (n < 0) {
    n = -n
}

// 判断奇偶
var n = 1
if (n % 2 == 0) {
    log('偶数')
} else {
    log('奇数')
}

// 比较运算和逻辑操作
// if 判断的条件其实是一个值, 叫布尔值
// 布尔值只有两种结果: 真, 假
// JavaScript 中, 这两种值为: true, false

// 一共有 7 种比较运算
//    ===   严格相等
//    ==    相等
//    !=    不等
//    <     小于
//    >     大于
//    <=   小于等于
//    >=   大于等于
1 === '1'
false

1 == '1'
true


// 三中逻辑操作
//    &&    与 
//    ||    或
//    !     非
1 == 1 && 2 == 2
true
1 == 1 && 1 == 2
false
1 == 1 && 1 == 2
true
! 1 == 1
false
```

## 函数返回值

> - 函数不仅可以合并操作重复性的代码, 还可通过计算得到一个结果
> - 返回值的意思是函数调用的结果是一个值, 可以被赋值给变量, 这个结果就是函数的返回值

