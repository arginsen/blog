---
title: 前端 | ES6笔记 | 十一、async 函数
date: 2020-5-30 6:45:21
tags: 
- ES6
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->


> - async 函数是什么？一句话，它就是 Generator 函数的语法糖。
> - async 函数就是将 Generator 函数的星号（*）替换成async，将yield替换成await，仅此而已。
> - async 函数对 Generator 函数的改进，体现在以下四点：
>   - （1）**内置执行器**。Generator 函数的执行必须靠执行器，所以才有了 co 模块，而 async 函数自带执行器。也就是说，async函数的执行，与普通函数一模一样，只要一行。Generator 函数，需要调用 next 方法，或者用 co 模块，才能真正执行，得到最后结果。
>   - （2）**更好的语义**。async 和 await，比起星号和 yield，语义更清楚了。async 表示函数里有异步操作，await 表示紧跟在后面的表达式需要等待结果。
>   - （3）**更广的适用性**。co 模块约定，yield 命令后面只能是 Thunk 函数或 Promise 对象，而 async 函数的 await 命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 resolved 的 Promise 对象）
>   - （4）**返回值是 Promise**。async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。
> - 进一步说，async 函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而 await 命令就是内部 then 命令的语法糖。


```js
// 使用 Generator 函数依次读取两个文件
const fs = require('fs');

const readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};

const gen = function* () {
  const f1 = yield readFile('/etc/fstab');
  const f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};


// gen 函数写成 async 函数
const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

asyncReadFile();  // 直接执行
```


# 语法

## 定义

```js
// async 函数多种使用形式

// 函数声明
async function foo() {}

// 函数表达式
const foo = async function () {};

// 对象的方法
let obj = { async foo() {} };
obj.foo().then(...)

// Class 的方法
class Storage {
  constructor() {
    this.cachePromise = caches.open('avatars');
  }

  async getAvatar(name) {
    const cache = await this.cachePromise;
    return cache.match(`/avatars/${name}.jpg`);
  }
}

const storage = new Storage();
storage.getAvatar('jake').then(…);

// 箭头函数
const foo = async () => {};
```

## 返回 Promise 对象


```js
// async 函数返回一个 Promise 对象。
// async 函数内部 return 语句返回的值，会成为 then 方法回调函数的参数。
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"


// async 函数内部抛出错误，会导致返回的 Promise 对象变为 reject 状态。
// 抛出的错误对象会被 catch 方法回调函数接收到。

async function f() {
  throw new Error('出错了');
}

f().then(
  v => console.log(v),
  e => console.log(e)
)
// Error: 出错了
```

## Promise 对象的状态变化


> async 函数返回的 Promise 对象，必须等到内部所有 await 命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到 return 语句或者抛出错误。也就是说，只有 async 函数内部的异步操作执行完，才会执行 **then** 方法指定的回调函数。

```js
// getTitle 函数内抓取网页，返回文本，匹配标题（return）三个操作
// 此三步完成后，then 方法才会打印
async function getTitle(url) {
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
// "ECMAScript 2017 Language Specification"

```

## await 命令

> 正常情况下，await 命令后面是一个 Promise 对象，返回该对象的结果。如果不是 Promise 对象，就直接返回对应的值。







