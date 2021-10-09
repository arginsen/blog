---
title: 前端 | ES6笔记 | 十三、模块
date: 2020-5-30 20:45:21
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

> - 在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。
> - ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。
> - ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西
> - ES6 模块通过export命令显式指定输出的代码，再通过import命令输入。

# ES6 的严格模式

ES6 的模块自动采用严格模式，具有以下限制：

- 变量必须声明后再使用
- 函数的参数不能有同名属性，否则报错
- 不能使用 with 语句
- 不能对只读属性赋值，否则报错
- 不能使用前缀 0 表示八进制数，否则报错
- 不能删除不可删除的属性，否则报错
- 不能删除变量 `delete prop`，会报错，只能删除属性 `delete global[prop]`
- eval 不会在它的外层作用域引入变量
- eval 和 arguments 不能被重新赋值
- arguments 不会自动反映函数参数的变化
- 不能使用 arguments.callee
- 不能使用 arguments.caller
- 禁止 this 指向全局对象，ES6 模块之中，顶层的 this 指向 undefined
- 不能使用 fn.caller 和 fn.arguments 获取函数调用的堆栈
- 增加了保留字（比如 protected、static 和 interface）

# export

> 模块功能主要由两个命令构成：export 和 import。export 命令用于规定模块的对外接口，import 命令用于输入其他模块提供的功能。

> 一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。外部能够读取模块内部的某个变量，就必须使用 export 关键字输出该变量

```js
// 从 profile.js 输出变量
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;

// 或者写为
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export { firstName, lastName, year };


// 输出函数
// 直接输出,也可用大括号写函数名输出
export function multiply(x, y) {
  return x * y;
};

// 输出多个函数并用 as 重命名
function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,   // 将 v1 重命名为 streamV1
  v2 as streamV2,
  v2 as streamLatestVersion
};


// export命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系
// 要想输出声明的变量名、函数名、类名，都得加括号
// 错误示例
var m = 1;
export m;  // 没加括号

// 正确示例
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};
```

# import

> - 使用 import 导入模块， form 后接模块的相对位置，用单引号表示，若只写模块名，则应配置。
> - import命令具有提升效果，会提升到整个模块的头部，首先执行
> - import 命令输入的变量都是只读的，因为它的本质是输入接口，如果该输入变量指向一个对象，那么可以改写其属性。但最后仅仅用于使用

```js
// 在其他 js 文件里 profile.js 输入变量
import { firstName, lastName, year } from './profile.js';

function setName(element) {
  element.textContent = firstName + ' ' + lastName;
}


// 执行所加载的模块，但不输入任何值
import 'lodash';
```

# 模块的整体加载

> 除了指定加载某个输出值，还可以使用整体加载，即用星号（*）指定一个对象，所有输出值都加载在这个对象上面。

```js
// 输出 circle 模块
// circle.js
export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}

// 在 main.js 加载 circle 模块
// main.js
import * as circle from './circle';  // 将所有输出的变量标记为 circle 对象

// 通过新标记的对象 circle 调用该模块的方法
console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
```

# export default

> 为了方便用户调用模块，使用该命令为模块指定默认输出。
> export default 命令用于指定模块的默认输出。因此一个模块只能有一个默认输出，所以，import 命令后面不用加大括号，因为它对应唯一的 export default 命令。


```js
// 该模块默认输出一个函数
// export-default.js
export default function () {
  console.log('foo');
}

// 加载上边模块时可以为该匿名函数指定任意名字
// import-default.js
import customName from './export-default';
customName(); // 'foo'


// 默认输出  不用 {} 直接写函数名 crc32
export default function crc32() { // 输出
  // ...
}

import crc32 from 'crc32'; // 输入

// 正常输出
export function crc32() { // 输出
  // ...
};

import {crc32} from 'crc32'; // 输入


// export default 的本质，就是预加了一个中转的变量 default，使得一对一成立
// 默认输出
export default add;
//
import foo from 'modules';

// 相当于正常的：
export {add as default};
//
import { default as foo } from 'modules';


// 相应的，一些表述
// 正确
export var a = 1;

// 正确
var a = 1;
export default a;

// 错误
export default var a = 1;

// 同时输入默认方法和其他接口
import a, { each, forEach } from 'lodash';

// export default也可以用来输出类
// MyClass.js
export default class { ... }

// main.js
import MyClass from 'MyClass';
let o = new MyClass();
```

# export 与 import 的复合写法

> 如果在一个模块之中，先输入后输出同一个模块，import 语句可以与 export 语句写在一起

```js
// 同一模块中
import { foo, bar } from 'my_module';
export { foo, bar };

// 可以写为
export { foo, bar } from 'my_module';
// 相当于对外转发了这两个接口，导致当前模块不能直接使用 foo 和 bar


// 默认接口写为
export { default } from 'foo';


// 具名接口改为默认接口
export { es6 as default } from './someModule';

// 等同于输入 es6，又默认输出 es6
import { es6 } from './someModule';
export default es6;


// 默认接口改为具名接口
export { default as es6 } from './someModule';

import _ as es6 from './someModule';
export { es6 };
```

# 模块的继承

```js
// circleplus.js 继承 circle.js 模块
// circleplus.js
// -------------
// export * 命令会忽略 circle 模块的 default 方法
export * from 'circle';  

// 自定义输出变量
export var e = 2.71828182846;

// 自定义输出默认方法
export default function(x) {   
  return Math.exp(x);
}

// 将 circle 的属性或方法改名后再输出
// 只输出 circle 模块的 area 方法并改名为 circleArea
export { area as circleArea } from 'circle';
// -------------

// main.js
// -------------
// 输入 circleplus 模块所有变量，放进对象 math 里
import * as math from 'circleplus';   // import * 不会输入默认

// 输入 circleplus 模块默认输出，取名为 exp
import exp from 'circleplus';

// 得到该模块后，就可以调用其方法
console.log(exp(math.e));
// -------------
```

# 跨模块常量

> const 声明的常量只在当前代码块有效，但有时一些常量需要跨模块使用

```js
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3


// 如果要使用的常量非常多，可以建一个专门的 constants 目录，
// 将各种常量写在不同的文件里面，保存在该目录下。
// 然后，将这些文件输出的常量，合并在常量索引文件 index.js 里面。
// 使用的时候，直接加载 index.js 就可以了。
```

# import()

待补充...

# 参考资料

[网道- ES6教程 - Module 的语法](https://wangdoc.com/es6/module.html)