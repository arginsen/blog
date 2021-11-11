---
title: 前端 | TS 基础类型
date: 2021-9-6 18:45:00
tags: 
- TypeScript
- Frontend
categories: notes
photos:
    - /blog/img/typescript.jpg
---

<br>
<!--more-->

# 概述

TS 是静态类型

类型系统按照【类型检查的时机】分为动态类型和静态类型

动态类型是指在**运行时**才会进行类型检查，这种语言的类型错误往往会导致运行时错误。JavaScript 是一门解释型语言，没有编译阶段，所以是动态类型

静态类型是指**编译阶段**就能确定每个变量的类型，这种语言的类型错误往往会导致语法错误。TypeScript 在运行前需要先编译为 JavaScript，因此在编译阶段就会进行类型检查，所以 TypeScript 是静态类型

# 原始数据类型

## 布尔值

```ts
let isDone: boolean = false

// new 创建出的 Boolean 是 Boolean 对象，并不是基础 boolean 类型
let createdByNewBoolean: Boolean = new Boolean(1)

// 直接调用构造函数 Boolean 返回的是 boolean 类型
let createdByNewBoolean: boolean = Boolean(1);
```

## 数值

```ts
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
// ES6 中的二进制表示法
let binaryLiteral: number = 0b1010; // 会被编译为十进制
// ES6 中的八进制表示法
let octalLiteral: number = 0o744; // 会被编译为十进制
let notANumber: number = NaN;
let infinityNumber: number = Infinity;
```

## 字符串

```ts
let name: string = 'Tom'

let sentence: string = `Hello, my name is ${name}`
```

## 空值

JS 中没有空值的概念，TS 可以用 void 标识没有任何值返回

```ts
function alertName(): void {
  alert('Tom')
}
```

## Null 和 Undefined

在 TS 中可以用 null 和 undefined 来定义这两个原始数据类型

```ts
let u: undefined = undefined
let n: null = null
```

null 和 undefined 是所有类型的子类型，可以赋值给其他变量

```ts
let num: number = undefined

let u: undefined
let num: number = u
```

# 任意值

任意值用来表示允许赋值为任意类型。变量在声明未指定类型时，会被识别为任意值类型

```ts
let myNumber: any = 'seven'
myNumber = 7 // 变更类型
```

声明一个变量为任意值后，对其所有操作返回的内容类型都是任意值

```ts
// 在任意值上访问任何属性都是允许的
let anyThing: any = 'hello'
console.log(anyThing.myName)

// 也允许调用任何方法
let anyThing: any = 'hello'
anyThing.setName('Terry')
anyThing.myName.setName('Terry')
```











































# 参考

https://vue3js.cn/es6/typeScript.html

https://ts.xcatliu.com/introduction/index.html
