---
title: 前端 | ES6笔记 | 九、Promise 对象
date: 2020-5-28 13:45:21
tags: 
- ES6
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->

# 概述

> - Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。
> - 所谓 Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。
> - 从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 对象是一个**构造函数**，用来生成 Promise 实例。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。


# Promise 对象特点

> - 对象的状态不受外界影响。Promise 对象代表一个异步操作，有三种状态：**pending**（进行中）、**fulfilled**（已成功）和 **rejected**（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
> - 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise 对象的状态改变，只有两种可能：从 pending 变为 fulfilled 和从 pending 变为 rejected。
>   - 只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。
>   - 如果改变已经发生了，你再对 Promise 对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

# Promise 对象缺点

> - 首先，无法取消 Promise，一旦新建它就会立即执行，无法中途取消。
> - 其次，如果不设置回调函数，Promise 内部抛出的错误，不会反应到外部。
> - 第三，当处于 pending 状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。


# 基本用法


> - Promise 构造函数接受**一个函数**作为参数，该函数的两个参数分别是 resolve 和 reject。它们是**两个函数**，由 JavaScript 引擎提供，不用自己部署。如果调用 resolve 函数和 reject 函数时带有参数，那么它们的参数会被传递给回调函数。
>   - **resolve** 函数的作用是，将 Promise 对象的状态从**未完成变为成功**（即从 pending 变为 resolved），在异步操作**成功**时调用，并将异步操作的**结果**，作为**参数**传递出去；
>   - **reject** 函数的作用是，将 Promise 对象的状态从**未完成变为失败**（即从 pending 变为 rejected），在异步操作**失败**时调用，并将异步操作报出的**错误**，作为**参数**传递出去。
> - Promise 实例生成以后，可以用 **then** 方法接受**两个回调函数**作为参数，分别指定 **resolved** 状态和 **rejected** 状态的回调函数。其中，第二个函数是可选的，不一定要提供。这两个函数都接受 Promise 对象传出的值作为参数。
>   - 第一个回调函数是 Promise 对象的状态变为 resolved 时调用，
>   - 第二个回调函数是 Promise 对象的状态变为 rejected 时调用。


```js
// 生成一个 Promise 实例
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});

// 用 then 指定回调函数
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});


// 例一
// 过 100ms 后，函数 timeout 返回的 promise 实例状态变为 resolved，
// then 绑定的 resolved 状态回调函数被触发
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
// 'done'


// 例二
// promise 新建后立即执行 resovle 函数，将实例状态变为 resolved，首先输出
// then 方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行，最后输出
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// resolved


// 例三
// 使用 Promise 包装了一个图片加载的异步操作。
// 如果加载成功，就调用 resolve 方法，否则就调用 reject 方法。
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    const image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}



// 调用 resolve 函数和 reject 函数时带有参数，它们的参数会被传递给回调函数
// reject 函数的参数通常是 Error 对象的实例，表示抛出的错误；
// resolve 函数的参数除了正常的值以外，还可能是另一个 Promise 实例

// 实例 p2 调用 resolve 函数，实例 p1 作为参数
// 此时 p1 的状态决定了 p2 的状态。
// 如果 p1 的状态是 pending，那么 p2 的回调函数就会等待 p1 的状态改变；
// 如果 p1 的状态已经是 resolved 或者 rejected，那么 p2 的回调函数将会立刻执行。
const p1 = new Promise(function (resolve, reject) {
  // ...
});

const p2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})



// p1 在 3s 后状态变为 rejected，并传递一个 Error 对象 
// p2 的状态由 p1 决定，此时 p2 的 then 回调函数只定义了 resolved，因此无回调函数执行
// p2 中 resolve 函数的参数被传递到 catch 方法指定的回调函数，即 p1 实例返回的错误
const p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

const p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail

// catch() 方法用于指定发生错误时的回调函数。
```




















# 参考资料

[网道 - ES6教程 - Promise 对象](https://wangdoc.com/es6/promise.htm)



