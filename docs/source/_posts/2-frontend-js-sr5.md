---
title: 前端 | JS深入 | 五、模块
date: 2020-5-11 16:12:21
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

> 模块是实现特定功能的一组属性和方法的封装。


# 基本实现方法

> 把模块写成一个对象，所有的模块成员都放到这个对象里面。

```js
// 将一个个特定功能的方法封装进一个对象
var module1 = new Object({
  _count : 0,
  m1 : function (){
    //...
  },
  m2 : function (){
    //...
  }
});

module1.m1();   // 作为方法调用

// 但是外部代码可以直接改变内部属性的值
module1._count = 5;
```

# 封装私有变量

## 利用构造函数

```js
// 利用构造函数，封装私有变量
function StringBuilder() {
  var buffer = [];      // buffer 作为该模块的私有变量

  this.add = function (str) {   // this 指向构造函数生成的对象
     buffer.push(str);
  };

  this.toString = function () {
    return buffer.join('');
  };
}

// 这种方法将私有变量封装在构造函数中，导致构造函数与实例对象是一体的
// 总是存在于内存之中，无法在使用完成后清除
// 这时构造函数有双重作用，既用来塑造实例对象，又用来保存实例对象的数据
// 违背了构造函数与实例对象在数据上相分离的原则

```

## 利用立即执行函数

> 将相关的属性和方法封装在一个函数作用域里面，立即执行仅返回方法，其内部的变量外界不可访问。此种方法也是 JavaScript 模块的基本写法。

```js
var module1 = (function () {
  var _count = 0;
  var m1 = function () {
    //...
  };
  var m2 = function () {
    //...
  };
  return {
    m1 : m1,    // 执行后返回两个方法
    m2 : m2
  };
})();

console.info(module1._count);  //undefined
// 外部代码无法读取内部的_count变量

console.log(module1)    // {m1: ƒ, m2: ƒ}
// 此时 module1 也就成了包含两个方法的对象

module1.m1  //调用方法
```

# 模块的放大模式

> 当一个模块很大，必须分成几个部分，或者一个模块需要继承另一个模块，这时就有必要采用“放大模式”

```js
// 在上边 module1 的基础上，添加一个 mod 类型下边的两个方法
var module1 = (function (mod){
  mod.m3 = function () {
    //...
  };
  mod.m4 = function () {
    //...
  };
  return mod;
})(module1);

// 给 module1 传入 mod 参数，mod 作为两个方法的父级
// 最后返回 mod，而立即执行设定的参数为 module1 本身
// 那么两个方法就直接挂钩在 module1 对象下了

module1.m3  // 调用方法

// 在浏览器环境中，模块的各个部分通常都是从网上获取的，
// 有时无法知道哪个部分会先加载。
// 如果采用上面的写法，第一个执行的部分有可能加载一个不存在空对象，
// 这时就要采用"宽放大模式"（Loose augmentation）
var module1 = (function (mod) {
　//...
　return mod;
})(window.module1 || {});   // 参数可以设为空对象
```

# 模块输入全局变量

> 独立性是模块的重要特点，模块内部最好不与程序的其他部分直接交互。为了在模块内部调用全局变量，必须显式地将其他变量输入模块。

```js
var module1 = (function ($, YAHOO) {
　//...
})(jQuery, YAHOO);

// 上面的module1模块需要使用 jQuery 库和 YUI 库，
// 就把这两个库（其实是两个模块）当作参数输入 module1
// 这样做除了保证模块的独立性，还使得模块之间的依赖关系变得明显。

// 立即执行函数还可以起到命名空间的作用,内部对函数的命名可以与外部相同
(function($, window, document) {
  function go(num) {
  }

  function handleEvents() {
  }

  function initialize() {
  }

  function dieCarouselDie() {
  }

  // attach to the global scope
  window.finalCarousel = {
    init : initialize,
    destroy : dieCarouselDie
  }
})(jQuery, window, document);

// finalCarousel 对象输出到全局，对外暴露 init 和 destroy 接口
// 而内部方法 go、handleEvents、initialize、dieCarouselDie 都是外部无法调用的。
```

# 参考资料

[网道- JavaScript 的模块](https://wangdoc.com/javascript/oop/prototype.html#%E6%A8%A1%E5%9D%97)


