---
title: 前端 | JS深入 | 一、作用域及执行上下文
date: 2020-5-6 15:45:21
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

# JavaScript的作用域

- 作用域是指程序源代码中定义变量的区域。
- JavaScript 采用**词法作用域**(lexical scoping)，也就是静态作用域。

```js
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();
// foo 函数内无局部变量，则会向上层查找变量，此处上层即是全局变量
// 而动态作用域会从调用函数的作用域查找变量，也就是 bar() 里的
```

# 执行上下文栈

## 执行上下文

>JavaScript 引擎并非一行一行地分析和执行程序，而是一段一段地分析执行。当执行一段代码的时候，会进行一个“准备工作”，称为执行上下文(execution context)

- 
```js
// 此段代码会使 foo 变量提前，但具体的值并未执行到，因此第一个 foo() 返回 foo1
var foo = function () {
    console.log('foo1');
}

foo();  // foo1

var foo = function () {
    console.log('foo2');
}

foo(); // foo2

// 此段代码会使 foo 函数提前，foo2 的函数将 foo1 给覆盖掉，因此两个都返回 foo2
function foo() {
    console.log('foo1');
}

foo();  // foo2

function foo() {
    console.log('foo2');
}

foo(); // foo2
```

## 执行上下文栈

>JavaScript 引擎创建执行上下文栈（Execution context stack，ECS）来管理上下文

```js
// ------------ 过程 --------------
// 那么上方代码的执行过程用 ECS 来解释
// 就可以将 ECS 理解为一个动态数组，执行的每一步当作元素
// JavaScript 可执行有三种：全局代码、函数代码、eval 代码
// 1.那么首先就会执行全局代码，因此 ECS 的数组底部有一个全局执行上下文(globalContext)
// 2.然后每执行一个函数，就会创建一个执行上下文(准备工作)，并压入执行上下文栈(管理)
//   相当于添加一个元素 .push()
// 3.当此函数执行完毕后，再将刚才添加的元素删除 .pop()
// 4.整个应用程序结束后，该数组被清空
// -------------------------------
function fun3() {
    console.log('fun3')
}

function fun2() {
    fun3();
}

function fun1() {
    fun2();
}

fun1();

// 上边这段代码的执行顺序 ECS 的管理过程：(伪代码)
// fun1()
ECS.push(<fun1> functionContext);

// fun1中调用了fun2，创建fun2的执行上下文
ECS.push(<fun2> functionContext);

// fun2还调用了fun3，创建fun3的执行上下文
ECS.push(<fun3> functionContext);

// fun3执行完毕
ECS.pop();

// fun2执行完毕
ECS.pop();

// fun1执行完毕
ECS.pop();

// javascript继续执行下面的代码
// 但是ECStack底层永远有个globalContext
// 只有整个程序执行完，数组被清空
```

## 对一般函数执行顺序和闭包的对比

```js
// 闭包的执行顺序解释：
var scope = 'global scope'          // 全局变量
function checkscope() {
    var scope = 'local scope'       // 局部变量
    function f() { return scope}    // 该嵌套函数中使用的 scope 即为自由变量
    return f
}
checkscope()()
// 闭包的形式是先执行 checkscope()，得到 f() 后继续执行 f()
//  1.添加 checkscope 执行上下文 元素，执行完成后删除该元素
//  2.继续向下执行，添加 f 执行上下文 元素，执行完成后删除该元素

// 一般函数 嵌套函数
var scope = "global scope"
function checkscope(){
    var scope = "local scope"
    function f(){
        return scope
    }
    return f()
}
checkscope()
// 直接调用的方式是执行 checkscope()，在其内继续执行 f()
//  1.添加 checkscope 执行上下文 元素，调用到 f，继续添加 f 的执行上下文 元素
//  2.执行完成后删除 f 元素，接着完成 checkscope 并删除
```

# 执行上下文

## 执行上下文的三个属性

>当 JavaScript 代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)。

- 对于每个执行上下文，都有三个重要属性：
    - [变量对象(Variable object，VO)](#变量对象)
    - [作用域链(Scope chain)](#作用域链)
    - [this](#this)


### 变量对象

>变量对象是与执行上下文相关的**数据作用域**，存储了在上下文中定义的变量和函数声明。

- 全局上下文的变量对象初始化是全局对象
- 函数上下文的变量对象初始化只包括 Arguments 对象
- 在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值
- 在代码执行阶段，会**再次修改**变量对象的属性值

#### 全局上下文

> 全局上下文中的变量对象是全局对象
>
> 全局对象是预定义的对象，作为 JavaScript 的全局函数和全局属性的**占位符**。通过使用全局对象，可以访问所有其他所有预定义的对象、函数和属性。
>
> 在顶层 JavaScript 代码中，可以用关键字 this 引用全局对象。因为全局对象是作用域链的头，这意味着所有非限定性的变量和函数名都会作为该对象的属性来查询。

#### 函数上下文

> - 在函数上下文中，我们用活动对象(Activation object, AO)来表示变量对象。
> - 活动对象和变量对象其实是一个东西，只是变量对象是规范上的或者说是引擎实现上的，不可在 JavaScript 环境中访问。
> - 只有当进入一个执行上下文中，这个执行上下文的变量对象才会被激活，所以才叫 activation object ，也只有被激活的变量对象(活动对象)上的各种属性才能被访问。
> - 活动对象是在进入函数上下文时刻被创建的，它通过函数的 arguments 属性初始化。arguments 属性值是 Arguments 对象。

#### 执行过程

> 执行上下文的代码会分成两个阶段进行处理：分析和执行

##### 分析上下文

**变量对象会包括：**

- **函数的所有形参** (如果是函数上下文)
    - 由名称和对应值组成的一个变量对象的属性被创建
    - 没有实参，属性值设为 undefined
- **函数声明**(其次处理)
    - 由名称和对应值（函数对象(function-object)）组成一个变量对象的属性被创建
    - 如果变量对象已经存在相同名称的属性，则完全替换这个属性
- **变量声明**(最后处理)
    - 由名称和对应值（undefined）组成一个变量对象的属性被创建；
    - 如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性

```js
// ---- 例子 ----
function foo(a) {
  var b = 2;
  function c() {}
  var d = function() {};
  b = 3;
}

foo(1);

// 此时并未执行代码
// 上边的函数在进入执行上下文后，这时候的 AO 是：
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,                               
    b: undefined,
    c: reference to function c(){},
    d: undefined
}
```

##### 执行上下文

```js
// 在代码执行阶段，会顺序执行代码，根据代码，修改变量对象的值
// 上边执行完后，AO 变成：
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: 3,
    c: reference to function c(){},
    d: reference to FunctionExpression "d"
}
```

#### 关于变量对象的例子

```js
// ---- 例 1 ----
function foo() {
    console.log(a);
    a = 1;  // 全局变量
}

foo();  // Uncaught ReferenceError: a is not defined
// 函数中 a 没有通过 var 声明，所以不会被存放在 AO 中
// 此时的 AO：
AO = {
    arguments: {
        length: 0
    }
}

// ---- 例 2 ----
function foo() {
    a = 1;  
    console.log(a);
}

foo();  // 1
// 此时 a 被设定为全局变量，可以在全局中找到 a


// ---- 例 3 ----
console.log(foo);
function foo(){
    console.log("foo");
}
var foo = 1;
// ƒ foo(){
//     console.log("foo");
// }

// 返回 函数
// 进入执行上下文时，首先会处理函数声明，其次会处理变量声明
// 如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性
// 将变量声明放在函数声明前，同样输出函数：
console.log(foo);
var foo = 1;
function foo(){
    console.log("foo");
}


// ---- 例 4 ----
var foo = 1;
console.log(foo);
function foo(){
    console.log("foo");
};

// 返回 1
// -------------------- 过程 --------------------
// 1.执行进入上下文，首先处理函数声明，其次处理变量声明
// 如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性
// VO = {
//     foo: reference to function foo(){}
//     foo: undefined  // 变量
// }
// 2.进入代码执行阶段，先执行 console.log(foo) ，再执行函数 foo
// 再执行 foo = 1，因此输出结果为 1
// VO = {
//     foo: reference to function foo(){}
//     foo = 1
// }
```

### 作用域链

> 当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级(词法层面上的父级)执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就叫做作用域链。

#### 函数创建

> 函数的作用域在函数定义的时候就决定了，这是因为函数有一个内部属性 `[[scope]]`，当函数创建的时候，就会保存所有**父变量对象**到其中，`[[scope]]` 就是所有父变量对象的层级链，但`[[scope]]` 并不代表完整的作用域链。

```js
function foo() {
    function bar() {
        ...
    }
}

// 函数创建时，各自的[[scope]]为：
foo.[[scope]] = [
  globalContext.VO
];

bar.[[scope]] = [
    fooContext.AO,      // 父变量对象
    globalContext.VO    // 父父变量对象
];
```

#### 函数激活

> - 当函数激活时，进入函数上下文，创建 VO/AO 后，就会将活动对象添加到作用链的前端。
> - 这时候执行上下文的作用域链，我们命名为 Scope：
> - `Scope = [AO].concat([[Scope]]);`  // 添加到该数组最前，接着是父变量对象，父父变量对象
> - 至此，作用域链创建完毕。

```js
Scope = [AO, fooContext.AO, globalContext.VO]  // 当前活动对象，父活动对象，全局对象
```

#### 作用域链的执行过程

```js
var scope = "global scope";
function checkscope(){
    var scope2 = 'local scope';
    return scope2;
}
checkscope();

// 【执行过程】
// -------------- 简 要 --------------
// 1. 给 checkscope 函数保存作用域到内部属性 // checkscope.[[scope]]
// 2. 创建函数执行上下文并压入执行上下文栈(ECS) // ECS = [两个 EC] 
// 3. 创建仅含有 Scope 属性的作用域链(checkscopeContext) // checkscopeContext.Scope
// 4. 创建活动对象(AO)并初始化 // checkscopeContext.AO
// 5. 将 checkscope 的 AO 压入作用域链顶端 // Scope : [AO].concat([[Scope]])
// 6. 执行 checkscope 函数，修改 AO  // checkscopeContext.AO
// 7. 完成函数，checkscope 执行上下文退出 ECS

// -------------- 详 细 --------------
// 1.checkscope 函数被创建，保存作用域链到 内部属性[[scope]]
checkscope.[[scope]] = [
    globalContext.VO
];

// 2.执行 checkscope 函数，创建 checkscope 函数执行上下文，
// checkscope 函数执行上下文被压入执行上下文栈
ECStack = [
    checkscopeContext,
    globalContext
];

// 3.checkscope 函数并不立刻执行，开始做准备工作，
// 第一步：复制函数[[scope]]属性创建作用域链
checkscopeContext = {
    Scope: checkscope.[[scope]],
}

// 第二步：用 arguments 创建活动对象，随后初始化活动对象，加入形参、函数声明、变量声明
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope2: undefined
    }，
    Scope: checkscope.[[scope]],
}

// 第三步：将活动对象压入 checkscope 作用域链顶端，作用域链创建完毕
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope2: undefined
    },
    Scope: [AO, [[Scope]]]
}

// 4.准备工作做完，开始执行函数，随着函数的执行，修改 AO 的属性值
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope2: 'local scope'
    },
    Scope: [AO, [[Scope]]]
}

// 5.查找到 scope2 的值，返回后函数执行完毕，函数上下文从执行上下文栈中弹出
ECStack = [
    globalContext   // 随后整个程序执行完，数组被清空
];
```

### this

> 待总结


## 具体执行过程

```js
// -------- 示例一 --------
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();   // "local scope"
// ------------------------

// 执行全过程：
// 1.执行全局代码，创建全局执行上下文，全局上下文被压入执行上下文栈
ECStack = [
    globalContext
];

// 2.全局上下文初始化
// 初始化的同时，checkscope 函数被创建，保存作用域链到函数的内部属性[[scope]]
globalContext = {
    VO: [global],
    Scope: [globalContext.VO],
    this: globalContext.VO
}

checkscope.[[scope]] = [
    globalContext.VO
];

// 3.执行 checkscope 函数，创建 checkscope 函数执行上下文，
// checkscope 函数执行上下文被压入执行上下文栈
ECStack = [
    checkscopeContext,
    globalContext
];

// 4. checkscope 函数执行上下文初始化：
//  1)复制函数 [[scope]] 属性创建作用域链，
//  2)用 arguments 创建活动对象，
//  3)初始化活动对象，即加入形参、函数声明、变量声明，
//  4)将活动对象压入 checkscope 作用域链顶端。
// 同时 f 函数被创建，保存作用域链到 f 函数的内部属性[[scope]]
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope: undefined,
        f: reference to function f(){}
    },
    Scope: [AO, globalContext.VO],  // checkscope 的活动对象被压入 checkscope 作用域链顶端
    this: undefined
}

fscope.[[scope]] = [
    checkscopeContext.AO,
    globalContext.VO
]

// 5.执行 f 函数，创建 f 函数执行上下文，f 函数执行上下文被压入执行上下文栈
ECStack = [
    fContext,
    checkscopeContext,
    globalContext
];

// 6. f 函数执行上下文初始化, 以下跟第 4 步相同：
//  1)复制函数 [[scope]] 属性创建作用域链
//  2)用 arguments 创建活动对象
//  3)初始化活动对象，即加入形参、函数声明、变量声明
//  4)将活动对象压入 f 作用域链顶端
fContext = {
    AO: {
        arguments: {
            length: 0
        }
    },
    Scope: [AO, checkscopeContext.AO, globalContext.VO],
    this: undefined
}

// 7. f 函数执行，沿着作用域链查找 scope 值，返回 scope 值
//  在 checkscope 作用域查找到 scope 值，执行赋值，修改 AO 的属性值
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope: "local scope",
        f: reference to function f(){}
    },
    Scope: [AO, globalContext.VO], 
    this: undefined
}

// 8. f 函数执行完毕，f 函数上下文从执行上下文栈中弹出
ECStack = [
    checkscopeContext,
    globalContext
];

// 9.checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出
ECStack = [
    globalContext   // 整个程序执行完后，清空该数组
];


// -------- 示例二 --------
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();

// -------- 执行过程 ---------
// 1.进入全局代码，创建全局执行上下文，全局执行上下文压入执行上下文栈
// 2.全局执行上下文初始化
// 3.执行 checkscope 函数，创建 checkscope 函数执行上下文，执行上下文被压入执行上下文栈
// 4. checkscope 执行上下文初始化，创建变量对象、作用域链、this等
// 5. checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出
// 6.执行 f 函数，创建 f 函数执行上下文，f 执行上下文被压入执行上下文栈
// 7. f 执行上下文初始化，创建变量对象、作用域链、this等
// 8. f 函数执行完毕，f 函数上下文从执行上下文栈中弹出
// --------------------------
```

# 参考资料

[JavaScript深入之词法作用域和动态作用域](https://github.com/mqyqingfeng/Blog/issues/3)
[JavaScript深入之执行上下文栈](https://github.com/mqyqingfeng/Blog/issues/4)
[JavaScript深入之变量对象](https://github.com/mqyqingfeng/Blog/issues/5)
[JavaScript深入之作用域链](https://github.com/mqyqingfeng/Blog/issues/6)
[JavaScript深入之从ECMAScript规范解读this](https://github.com/mqyqingfeng/Blog/issues/7)
[JavaScript深入之执行上下文](https://github.com/mqyqingfeng/Blog/issues/8)















