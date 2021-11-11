---
title: 前端 | promise 实现
date: 2021-9-26 18:45:00
tags: 
- promise
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->

# 实现

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

/**
 * @description 对 then 中用户写入的 onFulfilled(value) 和 onRejected(reason) 函数执行后的返回值分情况处理
 * @param {Promise} promise 
 * @param {*} x then 方法传入的回调执行后得到的结果
 * @param {*} resolve 将 promise 对象的状态从 pending 转为 fulfilled
 * @param {*} reject 将 promise 对象的状态从 pending 转为 rejected
 * @returns 返回任意值
 */
const resolvePromise = (promise, x, resolve, reject) => {
  // 防止循环引用
  if (x === promise) {
    reject(new TypeError('Chaining cycle detected for promise!'))
  }

  // 如果 x 是一个对象或函数
  if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
    let called
    try { // then = x.then 可能会报错
      let then = x.then
      if (typeof then === 'function') { // 如果 then 是一个函数
        then.call(x, y => { // 为保证 fn1 与 fn2 只调用一个，设置 called 标记位
          if (called) return
          called = true
          resolvePromise(promise, y, resolve, reject)
        }, r => {
          if (called) return
          called = true
          reject(r)
        })
      } else { // 如果 then 不是一个函数
        resolve(x)
      }
    } catch (e) {
      if (called) return
      reject(e)
    }
  } else {
    resolve(x)
  }
}

class Promise {
  constructor (executor) {
    this.status = PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledCallbacks = []
    this.onRejectedCallbacks = []

    const resolve = value => {
      // 如果 value 值是一个 Promise 递归解析
      if (value instanceof Promise) {
        return value.then(resolve, reject)
      }

      // 确保 onFulfilled 和 onRejected 方法异步执行，
      // 在 then 方法被调用的那一轮事件循环之后的新执行栈中执行
      setTimeout(() => {
        if (this.status === PENDING) {
          this.status = FULFILLED
          this.value = value
          this.onFulfilledCallbacks.forEach(cb => cb(value))
        }
      })
    }

    const reject = reason => {
      setTimeout(() => {
        if (this.status === PENDING) {
          this.status = REJECTED
          this.reason = reason
          this.onRejectedCallbacks.forEach(cb => cb(reason))
        }
      })
    }

    try {
      executor(resolve, reject)
    } catch (e) {
      reject(e)
    }
  }

  /**
   * @description then 为 promise 对象收集 onfulfilled 和 onrejected 回调函数，在 promise 状态转变后进行回调的调用
   * @param {*} onFulfilled 接受 promise 的 fulfilledValue 作为入参并在 promise 状态为 fulfilled 时被调用
   * @param {*} onRejected 接受 promise 的 rejectedReason 作为入参并在 promise状态 为 rejected 时被调用
   * @returns 返回一个 promise 实例
   */
  then(onFulfilled, onRejected) {
    // 判断 then 传入的两方法是否为函数
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }

    let newPromise = new Promise((resolve, reject) => {
      if (this.status === FULFILLED) {
        setTimeout(() => { // then 的亮哥哥参数函数需要异步执行
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
        // 将 fulfilled 和 rejected 的函数压入当前 promise 回调收集器里，
        // 这些回调函数在之后会被 resolve 或 reject 调用，也是异步执行
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

  /**
   * @description 用于 promise 方法链式，捕获前面 onFulfilled/onRejected 抛出的异常
   * @param {*} errorCallback 
   * @returns 返回一个 promise 实例
   */
  catch(errorCallback) {
    return this.then(null, errorCallback)
  }

  /**
   * @description finally 传入的函数无论成功和失败都执行
   * @param {*} callback 回调函数
   * @returns 返回一个 promise 实例
   */
  finally(callback) {
    return this.then(value => {
      setTimeout(callback)
      return value
    }, error => {
      setTimeout(callback)
      throw error
    })
  }

  /**
   * @description 当这个数组里的所有 promise 对象全部变为resolve 状态的时候，才会resolve 当有一个promise对象变为reject状态时 直接 reject
   * @param {*} values promise 对象组成的数组作为参数
   * @returns 返回一个 promise 实例
   */
  static all(values) {
    return new Promise((resolve, reject) => {
      let resolvedValues = []
      let resolvedCount = 0

      for (let i = 0; i < promiseNum; i++) {
        (function (i) {
          values[i].then(value => {
            resolvedValues[i] = value
            if (++resolvedCount === values.length) {
              resolve(resolvedValues)
            }
          }, reject)
        })(i)
      }

      // 传入的 values 不一定为数组，也支持 iterater 接口的类数组
      // [...values].forEach((promise, i) => {
      //   promise.then(value => {
      //     resolvedValues[i] = value
      //     if (++resolvedCount === values.length) {
      //       resolve(resolvedValues)
      //     }
      //   }, reject)
      // })
    })
  }

  /**
   * @description 只要有一个 promise 对象进入 FULFILLED 或者 REJECTED 状态的话，就会继续执行后面的处理
   * @param {*} values  接受 promise 对象组成的数组作为参数
   * @returns 返回一个 promise 实例
   */
  static race(values) {
    return new Promise((resolve, reject) => {
      [...values].forEach(promise => {
        promise.then(resolve, reject)
      })
    })
  }

  /**
   * @description 报错会再次尝试，超过一定次数才真正抛出 reject
   * @param {Function} retryCallback 待执行的函数体
   * @param {Number} times 重试的次数
   * @param {Number} delay 重试的时间间隔
   */
  static retry(retryCallback, times, delay) {
    return new Promise((resolve, reject) => {
      function attempt() {
        retryCallback().then(value => {
          resolve(value)
        }).catch(reason => {
          if (times === 0) {
            reject(reason)
          } else {
            times--
            setTimeout(attempt, delay)
          }
        })
      }
    })
  }

  /**
   * @description 将一个 promise 实例状态从 pending 直接转为 fulfilled
   * @param {*} value 任意值
   * @returns 返回一个 promise 实例
   */
  static resolve(value) {
    let promise = new Promise((resolve, reject) => {
      resolvePromise(promise, value, resolve, reject)
    })
    return promise
  }

  /**
   * @description 将一个 promise 实例状态从 pending 直接转为 fulfilled
   * @param {*} value 任意值
   * @returns 返回一个 promise 实例
   */
  static reject(reason) {
    return new Promise((resolve, reject) => {
      reject(reason)
    })
  }
}
```

# 写的 ts 版

```ts
const PENDING: string = 'pending'
const FULFILLED: string = 'fulfilled'
const REJECTED:string = 'rejected'

interface Executor {
  (resolve: VoidFn, reject: VoidFn): void
}

interface VoidFn {
  (arguments: any): void
}

interface AnyFn {
  (arguments: any): any
}

function resolvePromise(promise: myPromise, x: any, resolve: VoidFn, reject: VoidFn) {
  if (x === promise) {
    return new TypeError('chained cycle')
  }

  if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
    let called: boolean
    try {
      let then = x.then
      if (typeof then === 'function') {
        try {
          then.call(x, (y) => {
            if (called) return
            called = true
            resolvePromise(promise, y, resolve, reject)
          }, r => {
            if (called) return
            called = true
            reject(r)
          })
        } catch(e) {
          reject(e)
        }
      } else {
        resolve(x)
      }
    } catch(e) {
      if (called) return
      reject(e)
    }
  } else {
    resolve(x)
  }
}

class myPromise {
  private status: string
  private value: any
  private reason: any
  private onFulfilledCallbacks: VoidFn[]
  private onRejectedCallbacks: VoidFn[]
  constructor(executor: Executor) {
    this.status = PENDING
    this.value = undefined
    this.reason = undefined
    this.onFulfilledCallbacks = []
    this.onRejectedCallbacks = []

    const resolve: VoidFn = (value) => {
      if (this.status === PENDING) {
        setTimeout(() => {
          this.status = FULFILLED
          this.value = value
          this.onFulfilledCallbacks.forEach(cb => cb(value))
        })
      }
    }

    const reject: VoidFn = (reason) => {
      if (this.status === PENDING) {
        setTimeout(() => {
          this.status = REJECTED
          this.reason = reason
          this.onRejectedCallbacks.forEach(cb => cb(reason))
        })
      }
    }

    try {
      executor(resolve, reject)
    } catch(e) {
      reject(e)
    }
  }

  then(onFulFilled: AnyFn, onRejected: AnyFn) {
    let newPromise = new myPromise((resolve, reject) => {
      if (this.status === FULFILLED) {
        setTimeout(() => {
          try {
            let x = onFulFilled(this.value)
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
            let x = onFulFilled(this.value)
            resolvePromise(newPromise, x, resolve, reject)
          } catch(e) {
            reject(e)
          }
        })
        this.onRejectedCallbacks.push(() => {
          try {
            let x = onRejected(this.value)
            resolvePromise(newPromise, x, resolve, reject)
          } catch(e) {
            reject(e)
          }
        })
      }
    })
    return newPromise
  }

  catch(onRejected: AnyFn) {
    return this.then(null, onRejected)
  }

  static all(promises: any[]) {
    return new myPromise((resolve, reject) => {
      let resolvedValues: any[] = []
      let resolvedCount: number = 0
      
      for (let i = 0; i < promises.length; i++) {
        (function (i) {
          promises[i].then((value) => {
            resolvedValues[i] = value
            if (++resolvedCount === promises.length) {
              resolve(resolvedValues)
            }
          }, reject)
        })(i)
      }
    })
  }
}
```

# 参考链接

https://github.com/lgwebdream/FE-Interview/issues/29

https://github.com/xieranmaya/Promise3/blob/master/Promise3.js

https://mp.weixin.qq.com/s?__biz=MzkxMTIzODU3Mg==&mid=2247483674&idx=1&sn=f75eb751848293cf69b31f8bbf46bce8&chksm=c11e75d9f669fccff3fafa85ce2dc5d76d3aeedc2c06612aaf075f05923785e05c939d05cc1e&scene=178&cur_album_id=2089002932997324800#rd

https://juejin.cn/post/6844903989058748429