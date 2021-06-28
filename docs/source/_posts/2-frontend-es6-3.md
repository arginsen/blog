---
title: 前端 | ES6笔记 | 三、对各数据类型的扩展
date: 2020-5-23 10:45:21
tags: 
- ES6
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->


# 数值的扩展



# 字符串的扩展



# 字符串的新增方法



# 函数的扩展

## rest 参数

> ES6 引入 rest 参数（形式为...变量名），用于获取函数的**多余参数**，这样就不需要使用 arguments 对象了。rest 参数搭配的变量是一个**数组**，该变量将多余的参数放入数组中。

```js
// 定义一个求和函数，利用 rest 参数，可以向函数传入任意数目的参数
function add(...values) {   // values 变量作为一个数组
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3) // 10


// rest 参数代替 arguments 变量
// arguments 对象是一个类似数组的对象
// 必须用 Array.prototype.slice.call 先将其转为数组再调用 sort()
// arguments变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();  // 将 arguments 转为数组
}

// rest参数的写法
const sortNumbers = (...numbers) => numbers.sort();


// 定义一个函数，将 rest 参数传入的值，作为第一个参数定义的数组的元素
function push(array, ...items) {
  items.forEach(function(item) {
    array.push(item);
    console.log(item);
  });
}

var a = [];
push(a, 1, 2, 3)
```

## name 属性

```js
function foo() {}
foo.name // "foo"

// f 相当于匿名函数
var f = function () {};
f.name   // "f"

const bar = function baz() {};
bar.name // "baz"

(new Function).name // "anonymous"

// bind 返回的函数，name 属性值会加上 bound 前缀
function foo() {};
foo.bind({}).name // "bound foo"

(function(){}).bind({}).name // "bound "
```

## 箭头函数

> 箭头函数表达式的语法比函数表达式更短，并且**不绑定自己的** this，arguments，super或 new.target。这些函数表达式最适合用于非方法函数(non-method functions)(静态)，并且它们不能用作构造函数。

### 基本书写

```js
var f = v => v;

// 等同于
var f = function (v) {
  return v;
};


// 箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分
var f = () => 5;
// 等同于
var f = function () { return 5 };


var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};


// 如果箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错
// 报错
let getTempItem = id => { id: id, name: "Temp" };

// 不报错
let getTempItem = id => ({ id: id, name: "Temp" });


// 箭头函数与变量解构结合使用
const full = ({ first, last }) => first + ' ' + last;

// 等同于
function full(person) {
  return person.first + ' ' + person.last;
}


// 箭头函数简单表达工具函数
const isEven = n => n % 2 === 0;
const square = n => n * n;


// 箭头函数简化回调函数
// 简单定义回调函数平方
[1,2,3].map(x => x * x);

// 等同于
[1,2,3].map(function (x) {
  return x * x;
});

// 简单定义排序后的相减回调函数
var result = values.sort((a, b) => a - b);

// 等同于
var result = values.sort(function (a, b) {
  return a - b;
});


// 箭头函数与 rest 参数结合
// 将传入的参数作为数组返回
const numbers = (...nums) => nums;
numbers(1, 2, 3, 4, 5)   // [1,2,3,4,5]

// 对传入的参数进行划分返回数组，rest 参数返回单独的数组
const headAndTail = (head, ...tail) => [head, tail];
headAndTail(1, 2, 3, 4, 5)   // [1,[2,3,4,5]]
```

### 注意事项

> - 函数体内的 this 对象，就是定义时所在的对象，而不是使用时所在的对象。实际箭头函数**没有** this，所以需要通过查找**作用域链**来确定 this 的值。
> - 不可以当作构造函数，也就是说，不可以使用 **new** 命令调用，否则会抛出一个错误；也没有 new.target 值；也没有构建原型的需求，于是也不存在 prototype 这个属性。
> - 不可以使用 arguments 对象，该对象在函数体内**不存在**。如果要用，可以用 rest 参数代替
> - 不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数

```js
// this 对象的指向在箭头函数里是固定的
// this 指向 setTimeout 函数定义生效时所在的对象，此处是 call 指定的
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

var id = 21;

foo.call({ id: 42 });
// id: 42


// 箭头函数的 this 绑定定义时所在的作用域（Timer 函数）
// 普通函数的 this 则指向运行时所在的作用域
function Timer() {
  this.s1 = 0;
  this.s2 = 0;
  // 箭头函数
  setInterval(() => this.s1++, 1000);  // 每隔 1000 毫秒执行一次，s1 递增
  // 普通函数
  setInterval(function () {
    this.s2++;
  }, 1000);
}

var timer = new Timer();

setTimeout(() => console.log('s1: ', timer.s1), 3100);  // 更新三次
setTimeout(() => console.log('s2: ', timer.s2), 3100);  // this 指向了全局，无变化
// s1: 3
// s2: 0


// 箭头函数封装回调函数
// init 方法中使用箭头函数，使箭头函数的 this 指向定义生效时所在的对象 handler
// 因此可以调用当前对象的 doSomething 方法
// 如果是普通函数，那么此处的 this 会指向 document 对象（元素节点）
var handler = {
  id: '123456',

  init: function() {
    document.addEventListener('click',    // addEventListener 方法的 this 指向触发事件的那个节点
      event => this.doSomething(event.type), false);  // 但是箭头函数会让 this 指向 handler
  },

  doSomething: function(type) {
    console.log('Handling ' + type  + ' for ' + this.id);
  }
};


// this 指向的固定化，并不是因为箭头函数内部有绑定 this 的机制，
// 实际上箭头函数根本没有自己的 this，导致内部的 this 就是外层代码块的this。
// 正是因为它没有 this，所以也就不能用作构造函数。
```

### 不适用

> - 箭头函数使得 this 从“动态”变成“静态”，下面两个场合不应该使用箭头函数
> 1. 第一个场合是定义对象的方法，且该方法内部包括 this;
> 2. 第二个场合是需要动态 this 的时候，也不应使用箭头函数。


```js
// 将对象里的方法用箭头函数写，会使 this 指向全局对象
// 因为对象不构成单独作用域，此时箭头函数定义的作用域是全局作用域
// 普通函数则会指向当前所在的对象 cat
const cat = {
  lives: 9,
  jumps: () => {
    this.lives--;
  }
}


// addEventListener 方法中监听函数为箭头函数
// 这会使 this 指向为全局对象，因为箭头函数定义时的作用域为全局作用域
// 监听函数为普通函数时 this 则会指向 button
var button = document.getElementById('press');
button.addEventListener('click', () => {
  this.classList.toggle('on');    // 此时 this 指向错误，需改为动态
});
```


# 数组的扩展

## 扩展运算符

> - 扩展运算符（spread）是三个点（...）。它好比 rest 参数的逆运算，将**一个数组**转为用逗号分隔的参数序列
> - 该运算符主要用于**函数调用**，也只有函数调用时，扩展运算符才可以放在圆括号中，否则会报错
> - 扩展运算符可以展开数组，因此可以替代函数的 apply 方法。用于复制、合并数组，与解构赋值结合起来生成数组
> - 如果将扩展运算符用于数组赋值，只能放在参数的最后一位

```js
console.log(...[1, 2, 3])
// 1 2 3

console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5

[...document.querySelectorAll('div')]
// [<div>, <div>, <div>]


// 利用扩展运算符，将数组转为参数传入 add 函数中
function add(x, y) {
  return x + y;
}

const numbers = [4, 38];
add(...numbers) // 42


// 用来替代函数的 apply 方法
// 例 1
// ES5 的写法
function f(x, y, z) {
  // ...
}
var args = [0, 1, 2];
f.apply(null, args);

// ES6的写法
function f(x, y, z) {
  // ...
}
let args = [0, 1, 2];   // 将数组拆分
f(...args);


// 例 2
// 将一个数组添加到前一个数组尾部
// ES5 的写法
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
Array.prototype.push.apply(arr1, arr2);   // 6  返回 push 后的元素个数
arr1    // [0, 1, 2, 3, 4, 5]

// ES6 的写法
let arr1 = [0, 1, 2];
let arr2 = [3, 4, 5];
arr1.push(...arr2);   // 6  返回 push 后的元素个数
arr1    // [0, 1, 2, 3, 4, 5]


// 复制数组
// a2 称为 a1 的克隆，修改 a2 并不会改变 a1，它俩并不指向同一份数据
const a1 = [1, 2];
const a2 = [...a1];  


// 合并数组 浅拷贝
// 这两种方法生成的数组成员均是对原数组成员的引用
const arr1 = ['a', 'b'];
const arr2 = ['c'];
const arr3 = ['d', 'e'];

// ES5 的合并数组
arr1.concat(arr2, arr3);
// [ 'a', 'b', 'c', 'd', 'e' ]

// ES6 的合并数组
[...arr1, ...arr2, ...arr3]
// [ 'a', 'b', 'c', 'd', 'e' ]


// 扩展运算符可以与解构赋值结合起来，用于生成数组
[a, ...rest] = list

const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest  // [2, 3, 4, 5]

const [first, ...rest] = [];
first // undefined
rest  // []

const [first, ...rest] = ["foo"];
first  // "foo"
rest   // []

// 如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错
const [...butLast, last] = [1, 2, 3, 4, 5];
// 报错

const [first, ...middle, last] = [1, 2, 3, 4, 5];
// 报错


// 扩展运算符还可以将字符串转为真正的数组
[...'hello']
// [ "h", "e", "l", "l", "o" ]
```


# 对象的扩展

## 对象属性

### 属性表示简写

> 大括号里面，直接写入变量和函数，作为对象的属性和方法。但简写的对象方法不能用作构造函数，会报错。

```js
// 直接写入变量
const foo = 'bar';
const baz = {foo}; // 属性名是变量名，属性值是变量值
baz // {foo: "bar"}

// 等同于
const baz = {foo: foo};


// 将传入的参数变量直接写进对象
function f(x, y) {
  return {x, y};
}

// 等同于
function f(x, y) {
  return {x: x, y: y};
}

f(1, 2) // Object {x: 1, y: 2}


// 直接写入函数
// method () {} 函数直接作为对象的一个方法，而不必拆成键值对
const o = {
  method() {
    return "Hello!";
  }
};

// 等同于
const o = {
  method: function() {
    return "Hello!";
  }
};


// 将外部变量直接传进，如 Vue 实例中直接写 router, store
let birth = '2000/01/01';

const Person = {
  name: '张三',
  //等同于birth: birth
  birth,
  // 等同于hello: function ()...
  hello() { console.log('我的名字是', this.name); }
};


// 打印对象
let user = {
  name: 'test'
};

let foo = {
  bar: 'baz'
};

console.log(user, foo)
// {name: "test"} {bar: "baz"}
console.log({user, foo})
// {user: {name: "test"}, foo: {bar: "baz"}}
```

### 属性名表达式

> 字面量定义对象时，可用**表达式**作为对象的**属性名**，即把表达式放在方括号内。属性名表达式如果是一个对象，默认情况下会自动将对象转为字符串 [object Object]

```js
let propKey = 'foo';

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123
};


// 用方括号加变量表示属性
// 属性名表达式与简洁表示法，不能同时使用，会报错。
// 不能直接写 [lastWord],
let lastWord = 'last word';

const a = {
  'first word': 'hello',
  lastWord,
  [lastWord]: 'world',
};

a['first word'] // "hello"
a["lastWord"] // "last word"
a[lastWord] // "world"
a['last word'] // "world"


// 当表达式为一个对象
const keyA = {a: 1};
const keyB = {b: 2};

const myObject = {
  [keyA]: 'valueA',
  [keyB]: 'valueB'
};

myObject // Object {[object Object]: "valueB"}
```

## 对象的扩展运算符

### 解构赋值

> - 对象的解构赋值用于从一个**对象**取值，相当于将目标对象自身的所有可遍历的（enumerable）、但尚未被读取的属性，分配到指定的对象上面。所有的键和它们的值，都会拷贝到新对象上面。
> - 解构赋值要求等号右边是一个对象，所以如果等号右边是 undefined 或 null，就会报错，因为它们无法转为对象。
> - 解构赋值必须是**最后一个参数**，否则会报错。
> - 解构赋值的拷贝是浅拷贝，即如果一个键的值是复合类型的值（数组、对象、函数）、那么解构赋值拷贝的是这个值的引用，而不是这个值的副本。所以会跟随原值改变而改变
> - 扩展运算符的解构赋值，不能复制继承自原型对象的属性。

```js
// 尚未被读取的属性 a b，被解构赋值到对象 z
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };

x // 1
y // 2
z // { a: 3, b: 4 }


// x 解构赋值拷贝了对象 obj
// a 属性引用了一个对象，修改这个对象的值，会影响到解构赋值对它的引用。
let obj = { a: { b: 1 } };
let { ...x } = obj;

obj.a.b = 2; // 重新设置 b 为 1
x.a.b // 2


// 扩展运算符的解构赋值，不能复制继承自原型对象的属性。
let o1 = { a: 1 };
let o2 = { b: 2 };
o2.__proto__ = o1;
let { ...o3 } = o2;

o3 // { b: 2 }
o3.a // undefined

// 解构赋值的 newObj 实际只得到了实例对象 o 的值，
// 并未得到 create 参数原型的值
const o = Object.create({ x: 1, y: 2 });
o.z = 3;

let { x, ...newObj } = o;
let { y, z } = newObj;
x // 1
y // undefined
z // 3
```

### 扩展运算符

> - 对象的扩展运算符（...）用于取出参数对象的所有可遍历属性，拷贝到当前对象之中。
> - 如果扩展运算符后面不是对象，则会自动将其转为对象
> - 如果用户自定义的属性，放在扩展运算符后面，则扩展运算符内部的同名属性会被覆盖掉。
> - 扩展运算符可以用于合并两个对象
> - 如果把自定义属性，放在扩展运算符后面，则扩展运算符内部的**同名属性会被覆盖掉**。
> - 如果把自定义属性，放在扩展运算符前面，就变成了**设置新对象的默认属性值**。
> - 对象的扩展运算符后面可以跟表达式

```js
// 展开对象
let z = { a: 3, b: 4 };
let n = { ...z };
n // { a: 3, b: 4 }

// 展开数组，拷贝到当前
let foo = { ...['a', 'b', 'c'] };
foo
// {0: "a", 1: "b", 2: "c"}


// 如果扩展运算符后面不是对象，则会自动将其转为对象
// 扩展运算符后面是整数 1，会自动转为数值的包装对象 Number{1}。
// 由于该对象没有自身属性，所以返回一个空对象。
// 等同于 {...Object(1)}
{...1} // {}

// 等同于 {...Object(true)}
{...true} // {}

// 等同于 {...Object(undefined)}
{...undefined} // {}

// 等同于 {...Object(null)}
{...null} // {}


// 扩展运算符可以用于合并两个对象
let ab = { ...a, ...b };
// 等同于
let ab = Object.assign({}, a, b);


// 自定义属性，放在扩展运算符后面
// 修改现有对象的部分属性
let newVersion = {
  ...previousVersion,
  name: 'New Name' // Override the name property
};

//自定义属性，放在扩展运算符前面
// 作为 a 的默认属性可被替换
let aWithDefaults = { x: 1, y: 2, ...a };

// 对象的扩展运算符后面跟表达式
const obj = {
  ...(x > 1 ? {a: 1} : {}),
  b: 2,
};
```

# 对象的新增方法

# 参考资料

[网道- ES6教程 - 字符串的扩展](https://wangdoc.com/es6/string.html)
[网道- ES6教程 - 数值的扩展](https://wangdoc.com/es6/number.html)
[网道- ES6教程 - 函数的扩展](https://wangdoc.com/es6/function.html)
[网道- ES6教程 - 数组的扩展](https://wangdoc.com/es6/array.html)
[网道- ES6教程 - 对象的扩展](https://wangdoc.com/es6/object.html)
[网道- ES6教程 - 正则的扩展](https://wangdoc.com/es6/regex.html)

[冴羽的博客 - ES6 系列之箭头函数](https://github.com/mqyqingfeng/Blog/issues/85)
[冴羽的博客 - ES6 系列之模板字符串](https://github.com/mqyqingfeng/Blog/issues/84)




