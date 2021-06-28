---
title: 前端 | ES6笔记 | 二、变量的解构赋值
date: 2020-5-21 15:45:21
tags: 
- ES6
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->

> ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。以前，为变量赋值，只能直接指定值。

# 各数据类型的解构赋值


## 数组的解构赋值

```js
// 从数组中提取值，按照对应位置，对左边变量赋值
let [a, b, c] = [1, 2, 3];

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

// 使用嵌套数组进行解构
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3


// ... 是扩展运算符，与解构赋值相结合用于生成数组
// 更多关于扩展运算符的内容将在【ES6 - 数组的扩展】中详细说明
let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []


// 解构不成功，变量的值就等于undefined
// foo 的返回值为 undefined
let [foo] = []; 
let [bar, foo] = [1];

// 不完全解构，等号左边只匹配了一部分等号右边的数组
let [x, y] = [1, 2, 3];
x // 1
y // 2

let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4


// 等号的右边不是数组（或者严格地说，不是可遍历的结构），那么将会报错
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;   // 以上，转为对象后不具备 Iterator 接口
let [foo] = {};     // 本身不具备 Iterator 接口

// 只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值。
// fibs 是一个 Generator 函数，原生具有 Iterator 接口。解构赋值会依次从这个接口获取值
function* fibs() {
  let a = 0;
  let b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

let [first, second, third, fourth, fifth, sixth] = fibs();
fifth   // 4
sixth   // 5


// 解构赋值允许指定默认值，左边变量在 let 下不可重复声明
let [x, y = 'b'] = ['a'];             // x='a', y='b'   y 对应位置为空
let [x, y = 'b'] = ['a', undefined];  // x='a', y='b'   y 对应位置指定 undefined

// ES6 内部使用严格相等运算符（===），判断一个位置是否有值
// 只有当一个数组成员严格等于 undefined，默认值才会生效
// y 对应位置指定 null，默认值就不会生效，因为 null 不严格等于 undefined
let [x, y = 'b'] = ['a', null];  // x='a', y=null


// 当默认值是一个表达式，这个表达式则是惰性求值的，即只有在用到的时候，才会求值
function f() {
  console.log('aaa');
}

let [x = f()] = [1];    
// x = 1    此时 x 有指定值，因而返回 1

function f() {
  console.log('aaa');
}

let [x = f()] = [];
// aaa      此时并未在右侧指定 x，那么就会执行默认的函数，返回 aaa


// 默认值可以引用解构赋值的其他变量，但该变量必须已经声明
let [x = 1, y = x] = [];     // x=1; y=1    两者均无指定，x 取默认值，y默认取 x 值
let [x = 1, y = x] = [2];    // x=2; y=2    y 无指定，取默认的 x 值
let [x = 1, y = x] = [1, 2]; // x=1; y=2    y 有指定，就等于指定值
let [x = y, y = 1] = [];     // ReferenceError: y is not defined    y 变量声明不提前
```

## 对象的解构赋值

> - 对象的解构与数组有一个重要的不同。数组的元素是**按次序排列**的，变量的取值由它的**位置**决定；而对象的属性**没有次序**，变量必须与属性**同名**，才能取到正确的值
> - 对象的解构赋值，可以很方便地将现有对象的**方法**，赋值到某个变量
> - 解构赋值允许等号左边的模式之中，不放置任何变量名。虽然毫无意义orz

```js
let { foo, bar } = { foo: 'aaa', bar: 'bbb' };
foo // "aaa"
bar // "bbb"

let { bar, foo } = { foo: 'aaa', bar: 'bbb' };  // 等号左边的变量与次序无关，只在右边对象中匹配对应的属性
foo // "aaa"
bar // "bbb"

// 如果解构失败，变量的值等于 undefined
let {foo} = {bar: 'baz'};
foo // undefined


// 将现有对象的方法，赋值到某个变量
// Math 作为已有的内置对象
// 变量 log、sin、cos 在 Math 中可找到对应的属性（方法），功能分别是 对数、正弦、余弦
let { log, sin, cos } = Math;  

// log 作为 console 对象的一个属性（方法），被赋值到同名变量 log，
// 因此变量 log 就有了 console.log 的功用
const { log } = console;
log('hello') // hello


// 当变量名与属性名不一致，令变量与值对应，那么就相当于同名的属性的值被赋值到变量
// 此时变量也作为值，这也是位置对应
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'

// 因此可以理解，对象的解构赋值是下面形式的简写（参见《对象的扩展》一章）。
// 左侧对象只指定变量，则默认将变量解释成：属性与值都是该变量名，再在右侧匹配
let { foo: foo, bar: bar } = { foo: 'aaa', bar: 'bbb' };
let { foo, bar } = { foo: 'aaa', bar: 'bbb' };

// 对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者。
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"
foo // error: foo is not defined


// 解构用于嵌套结构的对象
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p: [x, { y }] } = obj;    // 嵌套结构，属性名与右侧对象一致
p // ReferenceError: p is not defined   
x // "Hello"
y // "World"
// 此时 p 是属性，不是变量，实际被赋值的是其值里的变量
// 要使 p 也作为变量，需要单独指定变量 p，那么此时默认为 let {p: p}，就会输出右侧 p 属性对应的值
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p } = obj;
p // ["Hello", {y: "World"}]

// 三次解构赋值，分别是对 loc、start、line 三个属性的解构赋值
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: { line }} } = node;
line // 1
loc  // {start: {line: 1, column: 5}}
start // {line: 1, column: 5}

// 进行组合嵌套赋值
let obj = {};
let arr = [];

({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true }); 
 // 给 obj 的属性 prop 赋值 123，给 arr 第一个元素赋值 true

obj // {prop:123}
arr // [true]


// 如果解构模式是嵌套的对象，而且子对象所在的父属性不存在，那么将会报错
// 此时 bar 所在的子对象所在的父属性 foo 在右侧对象中没有对应的属性，那么 foo 就 undefined，再取子对象就会报错
let {foo: {bar}} = {baz: 'baz'};   // Uncaught TypeError: Cannot read property 'bar' of undefined


// 对象的解构赋值可以取到继承的属性
const obj1 = {};
const obj2 = { foo: 'bar' };
Object.setPrototypeOf(obj1, obj2);  // 设置 obj1 的原型对象是 obj2

const { foo } = obj1;   // obj1 继承了 obj2 的属性 foo
foo // "bar"


// 对象的解构也可以指定默认值
var {x = 3} = {};
x // 3

var {x, y = 5} = {x: 1};
x // 1
y // 5

var {x: y = 3} = {};    // 值作为变量
y // 3

var {x: y = 3} = {x: 5};
y // 5

var { message: msg = 'Something went wrong' } = {};
msg // "Something went wrong"

// 同数组解构赋值一样，对象默认值生效的条件是，对象的属性值严格等于 undefined
var {x = 3} = {x: undefined};
x // 3

var {x = 3} = {x: null};    // null与undefined不严格相等，所以是个有效的赋值
x // null   此时就不能获取默认值


// 对已经声明的变量，后续进行解构赋值，需要用圆括号包起来。直接行首用花括号会被认为是代码块，导致报错
let x;
{x} = {x: 1};
// SyntaxError: syntax error

// 正确的写法
let x;
({x} = {x: 1});


// 由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last  // 3
// 数组 arr 的 0 键对应的值是 1，[arr.length - 1] 就是 2 键，对应的值是 3。
// 此时可以将数组看作索引为属性，元素为值的对象
```

## 字符串的解构赋值

> 包装对象：存取字符串、数字或布尔值的属性时创建的临时对象

```js
// 字符串不是对象，但也可以解构赋值。
// 这是因为此时字符串被转换成了一个类似数组的对象
// 字符串值通过 new String() 的方式转化成对象，
// 这个临时对象继承了字符串原型 String.prototype 的方法
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"

// 此时引用字符串的 length 属性，右侧字符串转为对象，来处理属性的引用 
let {length : len} = 'hello';
len // 5
```

## 数值和布尔值的解构赋值

```js
// 解构赋值时，如果等号右边是数值和布尔值，则会先转为对象
let {toString: s} = 123;
s === Number.prototype.toString  // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
// 数值和布尔值的包装对象都有从原型继承下来的 toString 属性
// 因此变量 s 就被等号右边数值和布尔值的包装对象赋予 toString 方法
// 而包装对象是临时的，不存在名称，因此只能通过原型的方法进行对比


// 解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。
// 由于 undefined 和 null 无法转为对象，所以对它们进行解构赋值，都会报错。
let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```

## 函数参数的解构赋值

```js
// 参数将数组解构成变量 x 和 y
function add ([x, y]) {
  return x + y;
}

add([1, 2]); // 3

// 使用箭头函数，将数组解构成变量 a 和 b
[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]


// 函数参数的解构也可以使用默认值
// 函数 move 的参数是一个对象，通过对这个对象进行解构，得到变量 x 和 y 的值
// 此时函数的参数已指定默认值，需给两个变量指定值
function move({x = 0, y = 0} = {}) {    // 等号左边已初始化参数
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]   传入指定值
move({x: 3});       // [3, 0]   y 未传入指定值，因此返回默认值
move({});           // [0, 0]   未指定值，返回默认值
move();             // [0, 0]   为传入参数，返回默认值

// 为函数 move 的参数指定默认值，而不是为变量 x 和 y 指定默认值
function move({x, y} = { x: 0, y: 0 }) { // 等号左边未初始化参数，没有参数就返回undefined
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3});       // [3, undefined]   参数 y 未指定，返回 undefined
move({});           // [undefined, undefined]
move();             // [0, 0]

// undefined 会触发函数参数的默认值
[1, undefined, 3].map((x = 'yes') => x);
// [ 1, 'yes', 3 ]
```


# 解构赋值的用途

1. 交换变量的值

```js
let x = 1
let y = 2
[x, y] = [y, x]
```

2. 从函数返回多个值

```js
// 返回一个数组
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();  // 从函数提取返回值


// 返回一个对象
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();   // 从函数提取返回值
```

3. 函数参数的定义

```js
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});    // 直接解构赋值传参
```

4. 提取 JSON 数据

```js
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

5. 函数参数的默认值

```js
jQuery.ajax = function (url, {
  async = true,                 // 结构赋值时直接指定函数参数的默认值
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
} = {}) {
  // ... do stuff
};
```

6. 遍历 Map 结构

```js
// 任何部署了 Iterator 接口的对象，都可以用for...of循环遍历
// Map 结构原生支持 Iterator 接口，配合变量的解构赋值获取键名和键值
const map = new Map();
map.set('first', 'hello');      // 给 map 对象添加键值对 "first" => "hello"
map.set('second', 'world');

for (let [key, value] of map) {     // 从对象中获取两个值，解构赋值给两个有顺序的变量
  console.log(key + " is " + value);    // 遍历一次便输出两个变量的值，接着提取下一对变量
}
// first is hello
// second is world

// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}
```

7. 输入模块的指定方法

```js
const { SourceMapConsumer, SourceNode } = require("source-map");
```


# 参考资料

[网道 - ES6教程 - 变量的解构赋值](https://wangdoc.com/es6/destructuring.html)
[网道 - ES6教程 - 数组的扩展](https://wangdoc.com/es6/array.html)
[网道 - ES6教程 - Set 和 Map 数据结构](https://wangdoc.com/es6/set-map.html)
[网道 - ES6教程 - Iterator 和 for...of 循环](https://wangdoc.com/es6/iterator.html)
[冴羽的博客 - ES6 系列之迭代器与 for of](https://github.com/mqyqingfeng/Blog/issues/90)
[冴羽的博客 - ES6 系列之模拟实现一个 Set 数据结构](https://github.com/mqyqingfeng/Blog/issues/91)