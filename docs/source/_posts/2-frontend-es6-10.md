---
title: 前端 | ES6笔记 | 十、Generator 函数
date: 2020-5-29 11:45:21
tags: 
- ES6
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->


# 异步

> 所谓"异步"，简单说就是一个任务不是连续完成的，可以理解成该任务被人为分成两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头执行第二段。
>
> 比如，有一个任务是读取文件进行处理，任务的第一段是向操作系统发出请求，要求读取文件。然后，程序执行其他任务，等到操作系统返回文件，再接着执行任务的第二段（处理文件）。这种不连续的执行，就叫做异步。
>
> 相应地，连续的执行就叫做同步。由于是连续执行，不能插入其他任务，所以操作系统从硬盘读取文件的这段时间，程序只能干等着。



# Generator

> - Generator 函数有多种理解角度。语法上，首先可以把它理解成，Generator 函数是一个**状态机**，封装了多个内部状态。执行 Generator 函数会返回一个遍历器对象，也就是说，Generator 函数除了状态机，还是一个**遍历器对象**生成函数。返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。
> - 形式上，Generator 函数是一个普通函数，但是有两个特征。一是，function 关键字与函数名之间有一个**星号**；二是，函数体内部使用 **yield** 表达式，定义不同的内部状态（yield在英语里的意思就是“产出”）。
> - 调用 Generator 函数，返回一个遍历器对象，代表 Generator 函数的内部指针。以后，每次调用遍历器对象的 next 方法，就会返回一个有着 value 和 done 两个属性的对象。value 属性表示当前的内部状态的值，是 yield 表达式后面那个表达式的值；done 属性是一个布尔值，表示是否遍历结束。

```js
// 定义一个 Generator 函数，
// 内部有两个 yeild 表达式，共有三个状态，到 return 结束
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

// 调用 Generator 函数后，该函数并不执行，返回遍历器对象
var hw = helloWorldGenerator();

// 遍历器包含了函数的各个状态，
// 调用遍历器的 next 方法，函数会进入下一个状态，到下一个 yield 表达式为止
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

// 再执行就没状态了，但函数运行完毕 done: true
hw.next()
// { value: undefined, done: true }
```

# yield 表达式

> - Generator 函数可以不用yield表达式，这时就变成了一个单纯的暂缓执行函数
> - yield 表达式只能用在 Generator 函数里面，用在其他地方都会报错。
> - yield 表达式如果用在另一个表达式之中，必须放在圆括号里面。
> - yield 表达式用作函数参数或放在赋值表达式的右边，可以不加括号。

```js
function* demo() {
  console.log('Hello' + yield); // SyntaxError
  console.log('Hello' + yield 123); // SyntaxError

  console.log('Hello' + (yield)); // OK
  console.log('Hello' + (yield 123)); // OK
}


function* demo() {
  foo(yield 'a', yield 'b'); // OK
  let input = yield; // OK
}
```



# for...of 循环

> - for...of 循环可以自动遍历 Generator 函数运行时生成的Iterator对象，且此时不再需要调用 **next** 方法。一旦 next 方法的返回对象的 done 属性为 true，for...of 循环就会中止，且不包含该返回对象
> - 利用 for...of 循环，可以写出遍历任意对象（object）的方法。原生的 JavaScript 对象没有遍历接口，无法使用 for...of 循环，通过 Generator 函数为它加上这个接口，就可以用了。

```js
// return 执行后 done 为 true，不包含 6
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5


// 利用 Generator 函数和 for...of 循环，实现斐波那契数列
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

for (let n of fibonacci()) {
  if (n > 1000) break;
  console.log(n);
}


// Generator 函数给对象 jane 加上遍历器接口，可以用 for...of 遍历
function* objectEntries(obj) {
  let propKeys = Reflect.ownKeys(obj);

  for (let propKey of propKeys) {
    yield [propKey, obj[propKey]];
  }
}

let jane = { first: 'Jane', last: 'Doe' };

for (let [key, value] of objectEntries(jane)) {
  console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe
```


# co 模块


```js
// co 模块可以让你不用编写 Generator 函数的执行器。
var co = require('co');

var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

co(gen);

// co 函数返回一个 Promise 对象，因此可以用 then 方法添加回调函数。
co(gen).then(function (){
  console.log('Generator 函数执行完成');
});
```


