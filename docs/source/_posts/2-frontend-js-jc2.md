---
title: 前端 | JS基础 | JS学习笔记Ⅱ：条件语句
date: 2020-4-24 15:25:21
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

# if & else

```js
// if else 适用于比较简单的判断情况
// 可以为一个 if 条件判断, 符和就执行, 不符合就直接跳过
var isAdmin = true;
// 如果花括号里只有一条语句, 可省去花括号
if (isAdmin) {
    console.log('他是管理员')
    // ...
}

// 使用 if else 判断
if (isAdmin) {
    console.log('他是管理员')
    // ...
} else {
    console.log('他不是管理员')
    // ...
}

// js 自动判断值为false
isAdmin = 0
isAdmin = ''           // 空字符串
isAdmin = NaN
isAdmin = null         // 空
isAdmin = undefined    // 未定义
```

# switch

```js
// 稍微复杂的情况用 if else 会嵌套很多层, 使结构十分臃肿, 不易读
// 例如：
if (今天是礼拜一) {
    // ...
} else {
    if(今天是礼拜二) {
        // ...
    } else {
        if(今天是礼拜三) {
            // ...
        } else {
            // ...
        }
    }
}

// 简化上边的写法, 将 else if 写在一起
if (今天是礼拜一) {
    // ...
} else if(今天是礼拜二) {
    // ...
} else if(今天是礼拜三) {
    // ...
}

// 使用 switch 写法
var day = 1
switch (day) {
    case 1:
    // ...
    break;  // 停止
    case 2:
    // ...
    break;
    case 3:
    // ...
    break;
}

// 变量 day 给定一个值, 判断在符和该值的情况下, 执行该值
// 例如：查询星期几吃什么
var day = 2
switch (day) {
    case 1:
    // ...
    console.log('今天吃牛肉面')
    break
    case 2:
    // ...
    console.log('今天吃羊肉泡馍')
    break
    case 3:
    // ...
    console.log('今天吃干锅千叶豆腐')
    break
    // ...
    default:    // 关键词, 如果不满足其中任何一个 case, 就输出默认值
    console.log('今天吃臊子面')
    break
}
```