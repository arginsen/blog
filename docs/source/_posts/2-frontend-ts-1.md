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

# 类型推论

TypeScript 会在没有明确的指定类型的时候推测出一个类型

```js
let xxx = 'seven';
xxx = 99;
// error TS2322: Type 'number' is not assignable to type 'string'
```

如果定义的时候没有赋值，不管之后有没有赋值，都会被推断成 any 类型而完全不被类型检查。

# 联合类型

取值可以为多种类型中的一种

```js
let myFavoriteNumber: string | number;

// toString 作为 string 和 number 的共有属性
function getString(something: string | number): string {
    return something.toString();
}

// 而如果不是共有的属性，则会报错
function getLength(something: string | number): number {
    return something.length;
}
```

# 对象的类型——接口

TS 中的接口，可以用于对类的一部分行为进行抽象以外，也可以用于对【对象的形状】进行描述

```ts
interface Person {
  name: string;
  age: number;
}

// 赋值时变量的形状必须与接口的形状保持一致
let tom: Person = {
  name: 'Tom',
  age: 25
};
```

## 可选属性

```ts
interface Person {
  name: string;
  age?: number;
}

let tom: Person = {
  name: 'Tom'
}
```

## 任意属性

接口存在一个任意的属性

```ts
interface Person {
  name: string;
  age?: number;
  [propName: string]: any;
}

let tom: Person = {
  name: 'Tom',
  gender: 'male'
}
```

定义了任意属性, 那么确定属性和可选属性的类型都必须时它的类型的子集

如下如果可选属性 age 被赋值数字,则会报错, 因为可选属性age的类型number, 不是任意属性的类型string的子集, 如果是any就没问题

```ts
interface Person {
  name: string;
  age?:number;
  [propName: string]: string;
}

let tom: Person = {
  name: 'Tom',
  age: 25,
  gender: 'male'
}
```

## 只读属性

对象的字段只在创建时被赋值, 随后仅用于访问

且只读的约束存在于第一次给对象赋值的时候, 而不是第一次给只读属性赋值的时候. 即第一次创建对象时就必须添加该属性

```ts
interface Person {
  readonly id: number;
  name: string;
  age?: number;
  [propName: string]: any;
}

let tom: Person = {
  id: 89757,
  name: 'Tom',
  gender: 'male'
}

tom.id = 9527 // error
```

# 数组的类型

采用 类型+方括号 表示

```ts
let fibonacci: number[] = [1, 1, 2, 3, 5];
```

## 数组泛型

```ts
let fibonacci: Array<number> = [1, 1, 2, 3, 5];
```

## 用接口表示数组

```ts
interface NumberArray {
  [index: number]: number;
}
let fibonacci: NumberArray = [1, 1, 2, 3, 5];
```

## 类数组

函数的 arguments 就是类数组, 这时不能使用数组来描述它, 得使用接口

```ts
interface IArguments {
  [index: number] = any;
  length: number;
  callee: function;
}
```

## 数组中的 any

```ts
let list: any[] = ['xca', 24, {website: 'https://lixupeng.cn'}]
```















# 参考

https://vue3js.cn/es6/typeScript.html

https://ts.xcatliu.com/introduction/index.html
