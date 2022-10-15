---
title: 面试 | 编程题
date: 2020-8-29
tags:
  - interview
  - Frontend
categories: notes
hide: false
photos:
    - /blog/img/interview.jpg
---

<br>
<!--more-->

# 合并两条有序链表

```js
var mergeTwoLists  = function (l1, l2) {
    if (!l1) return l2;
    if (!l2) return l1;
    if (l1.val > l2.val) {
        l2.next = mergeTwoLists(l1, l2.next);
        return l2;
    } else {
        l1.next = mergeTwoLists(l1.next, l2);
        return l1;
    }
};
```

# 写出数组的冒泡排序与快速排序

```js
// 冒泡
var bubbleSort = function(arr) {
  for (let j = 0; j < arr.length; j++) {
    for (let i = 0; i < arr.length - j; i++) {
      if (arr[i] > arr[i + 1]) {
        [arr[i], arr[i + 1]] = [arr[i + 1], arr[i]];
      }
    }
  }
  return arr;
}

// 快排
var quickSort = function(arr) {
  if (arr.length < 2) {
    return arr
  }

  const target = arr[0]
  const left = []
  const right = []

  for (let i = 1; i < arr.length; i++) {
    if (arr[i] < target) {
      left.push(arr[i])
    } else {
      right.push(arr[i])
    }
  }

  return [...quickSort(left), target, ...quickSort(right)]
}
```

# 字符串中的最长字符间隔

```js
/**
 * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
 *
 *
 * @param s string字符串 字符串
 * @return int整型
 */
function maxLengthBetweenEqualCharacters( s ) {
    let n = s.length
    let set = new Set()
    for (let i = 0; i < n - 2; i++) {
        let p = s[i]
        for (let j = n - 1; j > i; j--) {
            let q = s[j]
            if (p === q) {
                set.add(j - i - 1)
            }
        }
    }
    return [...set].length ? Math.max(...set) : -1
}
```

# 求两个数组的中位数

例：
nums1 = [1, 3, 4]
nums2 = [1, 2, 5, 6]

nums1 的中位数 3.0
nums2 的中位数 3.5
nums1 和 nums2 的中位数为 3.3

```js
var findMedianSortedArrays = function(nums1, nums2) {
    const arr = [...nums1, ...nums2].sort((a, b) => a - b);
    const { length } = arr;
    return length % 2 ? arr[Math.floor(length / 2)] : (arr[length / 2] + arr[length / 2 - 1]) / 2;
};

// 双指针排序法，时间复杂度为O(m + n)。
var findMedianSortedArrays = function(nums1, nums2) {
    let reIndex = nums2.length - 1;
    for (let i = nums1.length - 1; i >= 0; i--) {
        while (nums1[i] <= nums2[reIndex] && reIndex > -1) {
            nums1.splice(i + 1, 0, ...(nums2.splice(reIndex, 1)));
            reIndex--;
        }
    }
    const arr = nums2.concat(nums1);
    const { length } = arr;
    return length % 2 ? arr[Math.floor(length / 2)] : (arr[length / 2] + arr[length / 2 - 1]) / 2;
};


// 二分法 时间复杂度O(log(min(m, n)))
var findMedianSortedArrays = function(nums1, nums2) {
    // 保证num1是比较短的数组
    if (nums1.length > nums2.length) {
        [nums1, nums2] = [nums2, nums1];
    }
    const length1 = nums1.length;
    const length2 = nums2.length;
    let min = 0;
    let max = length1;
    let half = Math.floor((length1 + length2 + 1) / 2);
    while (max >= min) {
        const i = Math.floor((max + min) / 2);
        const j = half - i;
        if (i > min && nums1[i - 1] > nums2[j]) {
            max = i - 1;
        } else if (i < max && nums1[i] < nums2[j - 1]) {
            min = i + 1;
        } else {
            let left,right;
            if (i === 0) left = nums2[j - 1];
            else if (j === 0) left = nums1[i - 1];
            else left = Math.max(nums1[i - 1], nums2[j - 1]);

            if (i === length1) right = nums2[j];
            else if (j === length2) right = nums1[i];
            else right = Math.min(nums1[i], nums2[j]);

            return (length1 + length2) % 2 ? left : (left + right) / 2;
        }
    }
    return 0;
};
```


# 用两个栈来描述一个队列

只可以用 push/pop

```ts
class Queue {
  constructor() {
    this.arr1 = [] // 主栈
    this.arr2 = [] // 辅栈
    this.length = 0
    return this.arr1
  }
  push(inEl) {
    let temp = null
    for (let i = 0; i < this.length; i++) {
      temp = this.arr1.pop()
      this.arr2.push(temp)
    }
    this.arr1 = this.arr2
    this.arr1.push(inEl)
    this.arr2 = []
    this.length++
    return inEl
  }
  pop() {
    let outEl = this.arr1.pop()
    this.length--
    return outEl
  }
  count() {
    return this.length
  }
}

let arr = new Queue()
```


# 实现深拷贝

```js
// 使用 map 处理循环引用问题
function deepClone(target) {
  const map = new Map()
  function clone (target) {
    if (isObject(target)) {
      let cloneTarget = isArray(target) ? [] : {};
      if (map.get(target)) {
        return map.get(target)
      }
      map.set(target,cloneTarget)
      for (const key in target) {
        cloneTarget[key] = clone(target[key]);
      }
      return cloneTarget;
    } else {
      return target;
    }
  }
  return clone(target)
}

// 直接递归
function deepClone(target, map = new Map()) {
  if (typeof target === 'object' && target !== null) {
    let cloneObj = Array.isArray(target) ? [] : {}
    if (map.get(target)) {
      return map.get(target)
    }
    map.set(target, cloneObj)
    for (let i in target) {
      cloneObj[i] = deepClone(target[i], map)
    }
    return cloneObj
  } else {
    return target
  }
}

// 先浅拷贝， 再处理对象数组
function cloneDeep(target, map = new Map()) {
  let res
  let isObject = val => typeof val === 'object' && val !== null
  if (isObject(target)) {
    res = Array.isArray(target) ? [...target] : {...target}
  } else {
    return target
  }

  // 完成浅拷贝后 得到对象所有属性 处理对象
  Reflect.ownKeys(res).forEach(key => {
    if (isObject(res[key])) {
      if (map.get(res[key])) {
        res[key] = map.get(res[key])
      } else {
        map.set(res[key], res[key])
        res[key] = cloneDeep(res[key], map)
      }
    }
  })

  return res
}
```

# 使用js实现斐波那契数列

```js
// generate 函数运行时会生成 Iterator 对象，不可以用 async
function* fibonacci() {
  let [prev, curr] = [0, 1]; // 赋值
  for (;;) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

for (let n of fibonacci()) {
  if (n > 1000) break; // 1000 之前 ，也可以写个计数器
  console.log(n);
}

// 数组
function fibonacci(n) {
  let arr = [0, 1]
  for (let i = 2; i < n; i++) {
    arr[i] = arr[i - 2] + arr[i - 1]
  }
  return n <= 2 ? arr.slice(0, n) : arr
}
```

# 输出斐波那契数列的第n项

```js
function Fibonacci(n)
{
    // write code here
    let [prev, curr] = [0, 1];
    for (let i = 1; i <= n; i++) {
        [prev, curr] = [curr, prev + curr];
    }
    return prev;
}
```

# 输出1-10000所有对称数字

```js
function getNumber(num) {
  const arr = [];
  for (let i = 10; i < num; i++) {
    let a = i.toString();
    let b = a.split("").reverse().join(""); // 核心反转
    if (a === b) {
      arr.push(b)
    }
  }return arr;
}

console.log(getNumber(10000)) ;
```

# 树状数据转换

origin 存储同级的对象，用pid和id标记树的关系；转成target就是嵌套的结构

```js
origin = [
  {
    id: 1,
    pid: null
  },
  {
    id: 2,
    pid: 1
  },
  {
    id: 3,
    pid: 1
  },
  {
    id: 4,
    pid: 2
  },
  {
    id: 5,
    pid: 2
  }
]

target = [{
  id: 1,
  ch: [
    {
      id: 2,
      ch: [
        {
          id: 4,
          ch: []
        },
        {
          id: 5,
          ch: []
        }
      ]
    },
    {
      id: 3,
      ch: []
    }
  ]
}]
```

实现：

```js
function toTree(arr) {
  let rootId = arr.find((v) => v.pid === null)["id"]
  const loop = (parentId) => {
    let res = []
    for(let i = 0; i < arr.length; i++) {
      let item = arr[i]
      if (item.pid !== parentId) continue
      let newItem = { id: item.id }
      newItem.ch = loop(item.id)
      res.push(newItem)
    }
    return res
  }

  return loop(rootId)
}

const toTree = (arr) => {
  let rootId = arr.find((item) => item["pid"] === null)["id"] // 找到根节点id
  const loop (parentId) => {
    let res = []
    for (let i of arr) {
      let item = arr[i]
      if (item["pid"] !== parentId) continue
      let newItem = { id: item["id"], ch: loop(item["id"]) }
      res.push(newItem)
    }
    return res
  }
  return loop(rootId)
}
```

# 贪心算法 - 找零钱

```js
// 例 商店老板有 1, 2, 5, 10 面值, 买了 80 的东西给了 100, 如何找零最佳

function minCoinChange(coins) {
  coins = coins.sort((a, b) => a - b)
  return function(amount) {
    let total = 0, change = []
    for (let i = coins.length; i >= 0; i--) { // 倒着来找
      let coin = coins[i]
      while (total + coin <= amount) { // 取出最大面值, 看需要几张
        change.push(coin)
        total += coin
      }
    }
    return change
  }
}

minCoinChange([1,2,5,10])(20)
```

# 大数相加 - 浮点数

按小数点剪开，给后边位数少的补 0 ，两者位数一样再一位一位进行相加，采用一个标记来记录是否进 1，如果是相减就需要退 1

```js
// 例
a = 12345.12123, b = 987654.92

function sum(a, b) {
  let arr1 = a.split(''),
      arr2 = b.split('')
  let a1 = a.split('.')[1], b1 = b.split('.')[1]
  let len = a1.length - b1.length
  if (len > 0) arr2.push('0'.repeat(len))
  if (len < 0) arr1.push('0'.repeat(Math.abs(len)))

  let count = 0 // 标记是否进一
  let arr = [] // 按个位存放相加后的数

  while(arr1.length || arr2.length) {
    let m = arr1.pop() || 0,
        n = arr2.pop() || 0
    if (m !== '.') {
      let num = m + n + count
      if (num > 9) {
        count = 1
        num -= 10
      } else {
        count = 0
      }
      arr.unshift(num)
    } else {
      arr.unshift('.')
    }
  }

  if (count > 0) arr.unshift(count)
  let res = arr.join('')
  console.log(res)
}

function sub(a, b) {
  // 处理位数
  let a1 = a.split('.')[1], b1 = b.split('.')[1]
  let differ = a1.length - b1.length
  differ > 0 ? b + '0'.repeat(differ) : a + 'o'.repeat(Math.abs(differ))

  // 按个加减
  let arr1 = a.split(''), arr2 = b.split('')
  let count = 0, arr = []
  while(arr1.length || arr2.length) {
    let m = arr1.pop() || 0, n = arr2.pop() || 0
    if (m !== '.') {
      let res = m - n - count
      if (res >= 0) {
        count = 0
      } else {
        res += 10
        count = 1
      }
      arr.unshift(res)
    } else {
      arr.unshift('.')
    }
  }

  // 各位相加完后
  return count > 0 ? -arr : arr
}
```

# 数组去重

```js
// 例
arr = [1,1,1,1,1,1,3,3,3,3,3,3,5,5]

function delRepeat(arr) {
  arr = arr.sort()
  for(let i = 0; i < arr.length; i++) {
    if (arr[i] === arr[i+1]) {
      arr.splice(i, 1) // 删掉当前位
      i-- // 少了一位, 往前回退, 等下 +1 又会加回来
    }
  }
  return arr
}
```

# 数组扁平化的方法

```js
// 例
let arr = [1, 2, [3, 4], [[5, 6], 7, [8, [9]]]]

// forEach
function flatten(arr) {
  let res = []
  arr.forEach(item => {
    res = res.concat(Array.isArray(item) ? flatten(item) : item)
  })
  return res
}

function flatten(arr) {
  while(arr.some(item => Array.isArray(item)) {
    arr = [].concat(...arr) // 元素存在数组则解构一次拼成数组，接着再检查
  }
}

function flatten(arr) {
  return arr.toString().split(',').map(item => Number(item))
}

function  flatten(arr) {
  return arr.flat(Infinity)
}
```

# 函数柯里化实现 sum(2, 3) 和 sum(2)(3)

```js
function sum(){
    var num = arguments[0];
    if (arguments.length === 1) {
        return function (sec){
            return num + sec;
        }
    } else {
        return [...arguments].reduce((acc, curr) => {
          return acc + curr
        })
    }
}

sum(2,3);
sum(2)(3);
```

指定问题函数：

```js
function curry (fn, currArgs) {
  return function() {
    // arguments 数组化
    let args = [].slice.call(arguments); // args = [...arguments]

    // 首次调用时，若未提供最后一个参数currArgs，则不用进行args的拼接
    // 拼接的是递归过来的剩余参数，等参数够数了就一起交由 fn 执行
    if (currArgs !== undefined) {
      args = args.concat(currArgs);
    }

    // 递归调用
    if (args.length < fn.length) {
      return curry(fn, args);
    }

    // 递归出口
    return fn.apply(null, args);
  }
}

// 三数和
function sum(a, b, c) {
    console.log(a + b + c);
}

const fn = curry(sum);
fn(1, 2, 3); // 6
fn(1, 2)(3); // 6
```

不限定 sum 函数柯里化, 参考版

```js
function curry(...args){
  let parmas = args

  function sum() {
    parmas = [...parmas,...arguments]
    return sum
  }

  sum.res = function() {
    return parmas.reduce((acc, item) => { // 累积变量 当前变量
      return acc + item
    })
  }

  return sum
}

curry(1)(2)(3)(10)(10,20).res()
```

本人优化改良版, 在最后再立即执行

```js
function curry(...args) {
  let params = args

  return function sum () {
    params = [...params, ...arguments]
    return arguments.length ? sum : params.reduce((acc, item) => {
      return acc + item
    })
  }
}

curry(1,2)() // 3
curry(1,2)(3)(4,5)() // 15
```

# 实现 Promise

```ts
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

const resolvePromise = (promise, x, resolve, reject) => {
  if (x === promise) {
    return new TypeError('chained cycle')
  }

  if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
    let called
    try {
      let then = x.then
      if (typeof then === 'function') {
        then.call(x, y => {
          if (called) return
          called = true
          resolvePromise(promise, y, resolve, reject)
        }, r => {
          if (called) return
          called = true
          reject(r)
        })
      } else {
        resolve(x)
      }
    } catch(r) {
      if (called) return
      reject(r)
    }
  } else {
    resolve(x)
  }
}

class Promise {
  constructor(executor) {
    // 定义内部状态
    this.status = PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledCallbacks = []
    this.onRejectedCallbacks = []

    const resolve = value => {
      if (value instanceof Promise) {
        this.then(resolve, reject)
      }

      if (this.status === PENDING) {
        setTimeout(() => {
          this.status = FULFILLED
          this.value = value
          this.onFulfilledCallbacks.forEach(cb => cb(value))
        }
      }
    }

    const reject = reason => {
      if (this.status === PENDING) {
        setTimeout(() => {
          this.status = REJECTED
          this.reason = reason
          this.onRejectedCallbacks.forEach(cb => cb(reason))
        }
      }
    }

    try {
      executor(resolve, reject)
    } catch(e) {
      reject(e)
    }
  }

  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }

    let newPromise = new Promise((resolve, reject) => {
      if (this.status === FULFILLED) {
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value)
            resolvePromise(newPromise, x, resolve, reject)
          } catch(e) {
            reject(e)
          }
        })
      }

      if (this.status === REJECTED) {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason)
            resolvePromise(newPromise, x, resolve, reject)
          } catch(e) {
            reject(e)
          }
        })
      }

      if (this.status === PENDING) {
        this.onFulfilledCallbacks.push(() => {
          try {
            let x = onFulfilled(this.value)
            resolvePromise(newPromise, x, resolve, reject)
          } catch(e) {
            reject(e)
          }
        })
        this.onRejectedCallbacks.push(() => {
          try {
            let x = onRejected(this.reason)
            resolvePromise(newPromise, x, resolve, reject)
          } catch(e) {
            reject(e)
          }
        })
      }
    })

    return newPromise
  }

  catch(errorCallback) {
    return this.then(null, errorCallback)
  }
}
```

# 实现 Promise.all

```js
Promise.all = function (promises) {
  return new Promise(function(resolve, reject) {
    var resolvedCounter = 0
    var promiseNum = promises.length
    var resolvedValues = new Array(promiseNum)
    for (var i = 0; i < promiseNum; i++) {
      (function(i) {
        promises[i].then(function(value) {
          resolvedCounter++
          resolvedValues[i] = value
          if (resolvedCounter == promiseNum) {
            return resolve(resolvedValues)
          }
        }, reject)
      })(i)
    }
  })
}
```

# 实现 new 操作符

```js
function mynew (fn, ...args) {
  if (typeof fn !== 'function') {
    throw 'mynew function the first param must be a function'
  }

  const obj = {}
  obj.__proto__ = fn.prototype
  const result = fn.apply(obj, args)
  return result instanceof Object ? result : obj // 忽略原始值
}
```

# 实现 bind

```js
// 方式一：只在bind中传递函数参数
fn.bind(obj,1,2)()

// 方式二：在bind中传递函数参数，也在返回函数中传递参数
fn.bind(obj,1)(2)
// 也有可能是
let wait = fn.bind(obj, 1) // 返回函数等待调用
let res = new wait(2)

// 实现
Function.prototype.bind = function (context) {
  // 判断调用的对象是否为函数
  if (!(this instanceof Function)) {
    throw new TypeError("Error")
  }

  // 获取后续参数
  const args = [...arguments].slice(1),
        fn = this

  return function Fn() {
    // 用来判断直接调用还是 new 调用
    // 如果是 new fn() ，那么生成的函数对象即是 fn 的实例
    return fn.apply(this instanceof Fn ? new fn(...arguments) : context, args.concat(...arguments))
  }
}
```

# 实现 instanceof

用来判断对象的类型: 函数、数组、对象
意为 xxx 是某个构造函数的实例

```js
function instanceof (left, right) {
  if (left !== 'object' || left === null) return false
  let proto = Object.getPrototypeOf(left)
  while(true) {
    if (proto === null) return false // 到顶了
    if (proto === right.prototype) return true // 原型相同返回
    proto = Object.getPrototypeOf(proto)
  }
}
```

# 判断两棵二叉树是否相同

```js
function isSameTree(t1, t2) {
  if (t1 === null && t2 === null) return true
  if (t1 === null || t2 === null) return false
  if (t1.val !== t2.val) return false
  return isSameTree(t1.left, t2.left) && isSameTree(t1.right, t2.right)
}
```

# 两棵二叉树相加


# 实现一个实现一个带并发限制的异步调度器，保证同时最多运行2个任务

> `scheduler.add(() => timeout(time).then(() => console.log(order)))`

```js
class Scheduler {
  constructor(limit) {
    this.limit = limit
    this.count = 0
    this.tasks = []
  }

  add(fn) {
    this.tasks.push(fn)
  }

  taskStart() {
    while (this.count < this.limit && this.tasks.length) {
      let fn = this.tasks.shift()
      this.count++
      fn().then(() => {
        this.count--
        this.taskStart()
      })
    }
  }
}

const timeout = (time) => new Promise(resolve => {
  setTimeout(resolve, time);
})

const scheduler = new Scheduler(2);
const addTask = (time, order) => {
  scheduler.add(() => timeout(time).then(() => console.log(order)))
}

addTask(1000, '1');
addTask(500, '2');
addTask(300, '3');
addTask(400, '4');

scheduler.taskStart()
// 2
// 3
// 1
// 4
```

> `scheduler.add(() => timeout(time)).then(() => console.log(order))`

此种 add 为一个 promise 对象, 但是此方法不好用 taskStart 做总开关

```js
class Scheduler {
  constructor(limit) {
    this.limit = limit
    this.count = 0
    this.tasks = []
  }

  add(fn) {
    return new Promise(resolve => {
      fn.resolve = resolve
      this.tasks.push(fn)
    })
  }

  taskStart() {
    while (this.count < this.limit && this.tasks.length) {
      let fn = this.tasks.shift()
      this.count++
      fn().then(() => {
        fn.resolve()
        this.count--
        this.taskStart()
      })
    }
  }
}

const timeout = (time) => new Promise(resolve => {
  setTimeout(resolve,time);
})

const scheduler = new Scheduler(2);
const addTask = (time, order) => {
  scheduler.add(() => timeout(time)).then(() => console.log(order))
}

addTask(1000, '1');
addTask(500, '2');
addTask(300, '3');
addTask(400, '4');

scheduler.taskStart()
```

# 实现 JSON.stringify

首先肯定要用到递归，对于普通类型直接返回，对于

```js
function stringify(obj) {
  let type = typeof obj
  // 基础类型
  if (type !== 'object') {
    if (/string|undefined|function/.test(type)) {
      obj = `${obj}`
    }
    return String(obj)
  } else {
    // 数组 对象
    let isArray = Array.isArray(obj)
    let json = []
    for (let key in obj) {
      let val = obj[key]
      let type = typeof val
      if (/string|undefined|function/.test(type)) {
        val = `${val}`
      } else if (type === 'object') {
        val = stringify(val)
      }
      json.push((isArray ? `` : `${key}:`) + String(val))
    }
    return isArray ? `[${String(json)}]` : `{${String(json)}}`
  }
}
```

# 实现发布订阅模式

```js
class EventEmitter {
  constructor() {
    this.events = []
  }

  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [callback]
    } else {
      this.events[event].push(callback)
    }
  }

  off(event, callback) {
    if (this.events[event]) return
    this.events[event].filter(cb => {
      return cb !== callback
    })
  }

  once(event, callback) {
    function fn() {
      callback()
      this.off(event, callback)
    }
    this.on(event, fn)
  }

  emit(event, ...args) {
    this.events[event] && this.events[event].forEach(cb => cb.apply(this, args))
  }
}

let event = new EventEmitter()
let cb = (...args) => {
  console.log(args)
}

event.on('click', cb)
event.emit('click', 1, 2, 3)
event.once('dbClick', () => { console.log(123) })
event.emit('dbClick')
event.off('click', cb)
```