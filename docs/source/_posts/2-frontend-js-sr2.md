---
title: 前端 | JS深入 | 二、闭包
date: 2020-5-8
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

- 函数对象可以通过**作用域链**相互关联起来, 函数体内部的变量都可以保存在**函数作用域**内, 这种特性称为 闭包
- 闭包指那些能够访问自由变量的函数（MDN）
    - 自由变量是指在函数中使用的，但既不是函数参数也不是函数的局部变量的变量。

# 引入

```js
// 函数调用
function 加 (a, b) {
    return a + b
}

var x = 加(1, 1)
console.log("x:", x)


// 示例引入
var user = {
    name: '王花花',
    age: 20,
    sex: '女'
}

user.age = 21   // 更改其中一个属性 不好管理
console.log("user:", user)


// 通过 闭包 的方法实现宏观管理
// 从母函数获取子函数的函数作用域, 继而 存/取 里边的值
function user (name) {
    var age, sex  // 自由变量
    return {      // 返回一个对象 包含不同的方法 方法：定义在对象里的函数
        getName: function() {
            return name
        },
        setName: function(newName) {
            name = newName;
        },
        getAge: function() {
            return age
        },
        setAge: function(newAge) {
            age = newAge
        },
        getSex: function() {
            return sex
        },
        setSex: function(newSex) {
            sex = newSex
        }
    }
}

var whh = user('王花花')
whh.setSex('女')            // 赋值
whh.setAge(22)
var name = whh.getName()
var sex = whh.getSex()
var age = whh.getAge()
console.log(name, sex, age) // 调用
```

## getter 和 setter

```js
// 上边也可以使用 getter 和 setter 替换属性, 分别实现 读/写 的功能
// getter 和 setter 定义的属性称为存取器属性
// 存取器属性定义一个或两个和属性同名的函数
// 
// 此时函数定义并未使用 function, 而是使用 get 和 set
// 此处在对象内也未使用 冒号 将属性名和函数体分隔开
// 但函数体的结束和下一个 方法/数据属性 之间用逗号隔开 
function user (name) {
    var age, sex
    return {    // 返回一个对象 包含不同的方法 方法：定义在对象里的函数
        get Name() {
            return name
        },
        set Name(newName) {
            name = newName;
        },
        get Age() {
            return age
        },
        set Age(newAge) {
            age = newAge
        },
        get Sex() {
            return sex
        },
        set Sex(newSex) {
            sex = newSex
        }
    }
}

var whh = user('王花花')
whh.Sex = '女'          // 赋值, 调用 setter
whh.Age = 22
var name = whh.Name     // 查询, 调用 getter
var sex = whh.Sex
var age = whh.Age
console.log(name, sex, age)
```

# 闭包的变量

```js
// 闭包可以捕捉到嵌套函数调用的局部变量, 并将局部变量用作私有状态
var scope = 'global scope'          // 全局变量
function checkscope() {
    var scope = 'local scope'       // 局部变量
    function f() { return scope}    // 该嵌套函数中使用的 scope 即为自由变量
    return f
}
checkscope()()      // checkscope() 返回函数对象 f, 嵌套的 f() 获取的是局部变量


// 多个闭包不共享变量
function constfuncs(v) {
    return function() { return v}   
} 
var funcs = []
for (var i = 0; i < 10; i++) {  // 此时生成的闭包都是各自独立的
    funcs[i] = constfuncs(i)
}

funcs //        // [ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ]
funcs[5]        // ƒ () { return i }  获取的 i 是外部给定的参数
funcs[5]()      // 5  此时 funcs[5] , funcs 数组中的第 5 个函数为 constfuncs(5)


// 多个闭包共享变量
function constfuncs() {
    var funcs = []
    for (var i = 0; i < 10; i++) {      // 共享一个变量 i
        funcs[i] = function() { return i }
    }
    return funcs
}

var funcs = constfuncs()
funcs           // [ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ, ƒ]
funcs[5]        // ƒ () { return i }   获取的 i 是函数内的局部变量
funcs[5]()      // 10  此时 i++ 后的 i = 10, 不满足条件, 循环停止, i = 10 也因此保留下来
                // funcs 数组中的第 5 个函数为 f() { return i } 获取动态的 i , 即 i = 10
                // funcs 数组中其它的 "值" (函数) 也都是 调用的当前的共享变量 i = 10

// ------------------------
// 了解 EC 后用执行上下文解释
// 1.当执行到 funcs[5] 函数之前，此时全局上下文的 VO 为：
globalContext = {
    VO: {
        funcs: [...],
        i: 10
    }
}

// 2.当执行 data[0] 函数的时候，data[0] 函数的作用域链为：
funcs[5]Context = {
    Scope: [AO, globalContext.VO]
}

// 3.data[0]Context 的 AO 并没有 i 值
//  所以会从 globalContext.VO 中查找，i 为 10，所以打印的结果就是 10。
// ------------------------

// 可以通过 let 解决，let 使 循环块内的变量仅在循环内有效，不泄露为全局变量
var funcs = [];
for (let i = 0; i < 10; i++) {
    funcs[i] = function () {
        console.log(i);
    };
}
funcs[5]();     // 5
```

# 深入分析闭包变量的保存

> - 函数整个执行过程参见：[深入一之执行上下文执行过程](../../06/2-frontend-js-sr1/#具体执行过程)
> - 函数作用域链的创建四步骤参见：[深入一之作用域链的执行过程](../../06/2-frontend-js-sr1/#作用域链的执行过程)
> - 闭包变量的获取实际依赖当时保存并创建的作用域链，该作用域链提供函数的父作用域、父父作用域的变量对象

```js
// 沿用上边第一个例子
var scope = 'global scope'
function checkscope() {
    var scope = 'local scope'
    function f() { return scope}
    return f
}
checkscope()()

// 为什么函数可以捕捉到单个（嵌套）函数调用的局部变量
// 用 执行上下文(EC) 的知识进行解释：

// 【执行过程】
// 1.进入全局代码，创建全局执行上下文，全局执行上下文压入执行上下文栈
// 2.全局执行上下文初始化
// 3.执行 checkscope 函数，创建 checkscope 函数执行上下文，执行上下文被压入执行上下文栈
// 4. checkscope 执行上下文初始化，创建变量对象、作用域链、this等
// 5. checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出
// 6.执行 f 函数，创建 f 函数执行上下文，f 执行上下文被压入执行上下文栈
// 7. f 执行上下文初始化，创建变量对象、作用域链、this等
// 8. f 函数执行完毕，f 函数上下文从执行上下文栈中弹出


// 【解 释】
// f 函数在创建的过程中，保存并创建了作用域链，f 会沿着作用域链查找变量

// ------------ 简 要 ------------
//  (1) f 函数被创建，保存作用域链到 f 的内部 // fscope.[[scope]]
//  (2)创建 f 函数执行上下文，将其压入执行上下文栈 // ECS = [三个 EC] 
//  (3)执行上下文初始化，复制函数 [[scope]] 属性创建作用域链(创建仅含有 Scope 属性的执行上下文对象) // fContext.Scope
//  (4)用 arguments 创建 f 的活动对象(给上一步的执行上下文对象 EC 加属性 AO) // fContext.AO
//  (5)初始化 AO
//  (6)将 f 的 AO 压入作用域链顶端 // Scope : [AO].concat([[Scope]])
//  (7)沿着作用域链查找 scope 值，返回 scope 值，赋值，修改 AO 的属性值 // checkscopeContext.AO
//  (8)完成函数，checkscope 执行上下文退出 ECS

// ------------ 详 细 ------------
// * --- 在执行第 6 步的时候
//  1. f 函数被创建，保存作用域链到 f 函数的内部属性[[scope]]
fscope.[[scope]] = [
    checkscopeContext.AO,
    globalContext.VO
]

//  2. 执行 f 函数，创建 f 函数执行上下文，f 函数执行上下文被压入执行上下文栈
ECStack = [
    fContext,
    checkscopeContext,
    globalContext
]

// * --- 在执行第 7 步时
//  3. f 函数执行上下文初始化，进行以下四步操作：
//   1)复制函数 [[scope]] 属性创建作用域链
//   2)用 arguments 创建活动对象
//   3)初始化活动对象，即加入形参、函数声明、变量声明
//   4)将活动对象压入 f 作用域链顶端
fContext = {
    AO: {               // 创建活动对象，并初始化活动对象
        arguments: {
            length: 0
        }
    },
    Scope: [AO, checkscopeContext.AO, globalContext.VO],  // 作用域链创建后压入活动对象 
    this: undefined
}
```

# 参考资料

《JavaScript权威指南》
[冴羽的博客- JavaScript深入之执行上下文栈](https://github.com/mqyqingfeng/Blog/issues/4)
[冴羽的博客- JavaScript深入之闭包](https://github.com/mqyqingfeng/Blog/issues/9)