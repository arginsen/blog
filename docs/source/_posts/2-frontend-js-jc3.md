---
title: 前端 | JS基础 | JS学习笔记Ⅲ：循环
date: 2020-4-25 14:25:21
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

# for

```js
var i
// for(initialize; test; increment) {statement} 初始化操作, 条件判断, 计数器变量更新, 语句
// 等同于 while 循环：
// initialize
// while (test) {
//     statement
//     increment
// }

// 初始值 1, 条件小于等于 100, 自增 1
for (i = 1; i <= 100; i++) {
    console.log(i)
}

// 当对象为数组, 获取数组中的每一个值
var result = [
    '百度',
    '百科',
    '百万',
    '百日'
]
// 单个获取
result[0]
result[1]
...
// 使用 for 循环
for (i = 0; i < 4; i++) {
    console.log(result[i])
}
// for 循环动态获取数组个数 result.length
for (i = 0; i < result.length; i++) {
    console.log(result[i])
}
// 或者使用 for/in 循环, 可更加方便的遍历
// for (variable in object) {statement}    // 变量名/产生左值的表达式/var语句申明的变量, 表达式 计算结果是一个对象, 语句
for (i in result) {
    console.log(result[i])
}
```

# while

```js
// while 语句是一个基本循环语句
// while(expression) {statement}    表达式, 语句
// 例如：请求尝试循环
var attempt = 0
while (attempt < 5) {
    // 继续尝试
    attempt++
    console.log(attempt)
}
console.log('loop stopped')
```
