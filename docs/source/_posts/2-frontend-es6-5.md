---
title: 前端 | ES6笔记 | 五、Set 和 Map
date: 2020-5-26 14:45:21
tags: 
- ES6
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->



# Set

> - Set 数据结构类似于数组，但是成员的值都是**唯一**的，没有重复的值。
> - Set 本身是一个**构造函数**，用来生成 Set 数据结构。
> - Set 函数可以接受一个数组（或者具有 iterable 接口的其他数据结构）作为参数，用来初始化。
> - 向 Set 加入值的时候，不会发生类型转换
> - 可以利用 Set 特性进行数组去重、字符串内重复字符去除


```js
// 使用 Set 构造函数生成 set
// 遍历一个数组将每个元素添加到 set 里
// 重复的值不会再次被添加
const s = new Set();

[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));

for (let i of s) {
  console.log(i);
}
// 2 3 5 4


// 参数为数组
const set = new Set([1, 2, 3, 4, 4]);
[...set]    // [1, 2, 3, 4]

// 利用扩展运算符转化为数组
Object.prototype.toString.call(set)  // "[object Set]"
Object.prototype.toString.call([...set])   // "[object Array]"

// 获取 set 成员的个数用属性 size
const items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
items.size  // 5

// 参数为 NodeList
const set = new Set(document.querySelectorAll('div'));
set.size  // 56

// 类似
const set = new Set();
document
 .querySelectorAll('div')
 .forEach(div => set.add(div));
set.size // 56


// 数组去重
[...new Set(array)]   // 生成 set 再转为数组

// 去除字符串里面的重复字符
[...new Set('ababbc')].join('')


// 在 Set 内部，两个 NaN 是相等的
let set = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);
set    // Set {NaN}
set.size // 1

// 在 Set 内部，两个对象总是不相等的
let set4 = new Set();

set4.add({});
set4.size // 1

set4.add({});
set4.size // 2

set4   // Set(2) {{…}, {…}}
```

## Set 的遍历方法

> -  Set 结构的实例有四个遍历方法，可以用于遍历成员。Set 的遍历顺序就是插入顺序。
> -  Set 结构没有键名，只有键值（或者说键名和键值是同一个值），所以 **keys** 方法和 **values** 方法的行为完全一致。**entries** 方法返回的遍历器，同时包括键名和键值，所以每次输出一个数组，它的两个成员完全相等
> - Set 结构的实例默认可遍历，它的默认遍历器生成函数就是它的 values 方法，因此可以直接用 **for...of** 循环遍历 Set。

遍历方法 | 作用
------- | ----
Set.prototype.keys()    | 返回键名的遍历器
Set.prototype.values()  | 返回键值的遍历器
Set.prototype.entries() | 返回键值对的遍历器
Set.prototype.forEach() | 使用回调函数遍历每个成员

```js
let set = new Set(['red', 'green', 'blue']);

for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]


// 直接用 for ... of 遍历
for (let x of set) {
  console.log(x);
}
// red
// green
// blue


// Set 的 forEach方法，用于对每个成员执行某种操作，没有返回值。
// 该方法的第一个参数为一个处理函数，其参数依次为键值、键名、集合本身
// 第二个参数，表示绑定处理函数内部的this对象。
let set = new Set([1, 4, 9]);
set.forEach((value, key) => console.log(key + ' : ' + value))
// 1 : 1
// 4 : 4
// 9 : 9
```



# WeakSet

> - WeakSet 结构与 Set 类似，也是不重复的值的集合。它与 Set 有两个区别：
>   - 首先，WeakSet 的成员只能是**对象**，而不能是其他类型的值。
>   - 其次，WeakSet 中的对象都是**弱引用**，即**垃圾回收机制**不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。


# Map

> - ES6 提供了 Map 数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，**各种类型的值**（包括对象）都可以当作键。也就是说，Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。
> - Map 也可以接受一个数组作为参数。该数组的成员是一个个表示键值对的数组。实际不仅仅是数组，任何具有 Iterator 接口、且每个成员都是一个**双元素的数组**的数据结构（详见《Iterator》一章）都可以当作 Map构造函数的参数。这就是说，Set 和 Map 都可以用来生成新的 Map。
> - 只有对同一个对象的引用，Map 结构才将其视为同一个键（这与 Set 相同，两个相同的具体的对象是不等的）。Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。
> - 如果 Map 的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map 将其视为一个键。undefined 和 null 是两个不同的键，NaN 不严格相等于自身，Map 将其视为同一个键。
> - Map 结构转为数组结构，比较快速的方法是使用扩展运算符

```js
// 使用 new 创建一个 map
const m = new Map();
const o = {p: 'Hello World'};

// 使用 set() 方法添加键值
m.set(o, 'content')

// 使用 get() 方法读取键
m.get(o) // "content"

// 使用 has() 方法查询是否有这个键
m.has(o) // true

// 使用 delete() 方法删除这个键
m.delete(o) // true
m.has(o) // false


// 数组作为 Map 构造函数的参数
const map = new Map([
  ['name', '张三'],
  ['title', 'Author']
]);

map.size // 2
map.has('name') // true
map.get('name') // "张三"
map.has('title') // true
map.get('title') // "Author"


// Set 对象作为 Map 构造函数的参数
const set = new Set([
  ['foo', 1],
  ['bar', 2]
]);
const m1 = new Map(set);
m1.get('foo') // 1


// Map 对象作为 Map 构造函数的参数
const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);
m3.get('baz') // 3


// 如果对同一个键多次赋值，后面的值将覆盖前面的值
const map = new Map();

map
.set(1, 'aaa')
.set(1, 'bbb');

map.get(1) // "bbb"


//读取一个未知的键，则返回 undefined
new Map().get('asfddfsasadf')   // undefined


// 只有对同一个对象的引用，才算是一个键，
// 即指向同个对象的变量是一个键，而对象本身则不是
const map = new Map();

map.set(['a'], 555);
map.get(['a'])   // undefined

// 因此可以令一个变量等于 ['a']
const map = new Map();
let a = ['a']
map.set(a, 555);
map.get(a)   // 555

// 两个不同的变量指向相同的对象，是两个键
const map = new Map();

const k1 = ['a'];
const k2 = ['a'];

map
.set(k1, 111)
.set(k2, 222);

map   // Map(2) {Array(1) => 111, Array(1) => 222}


// null undefined NaN 都为一个键，null 与 undefined 不同
let map = new Map();

map.set(undefined, 3);
map.set(null, 4);
map.get(undefined) // 3
map.get(null) // 4

map.set(NaN, 123);
map.get(NaN) // 123
```

## Map 的属性方法

属性/方法 | 作用
-------- | ----
Map.prototype.set(key, value) | 设置键名key对应的键值为value，返回整个 Map 结构，可以采用链式写法
Map.prototype.get(key) | get 方法读取 key 对应的键值，如果找不到key，返回undefined
Map.prototype.has(key) | 回一个布尔值，表示某个键是否在当前 Map 对象中
Map.prototype.delete(key)| 删除某个键，返回 true。如果删除失败，返回 false
Map.prototype.clear() | clear方法清除所有成员，没有返回值

## Map 的遍历方法

方法 | 作用
-------- | ----
Map.prototype.keys()    | 返回键名的遍历器。
Map.prototype.values()  | 返回键值的遍历器。
Map.prototype.entries() | 返回所有成员的遍历器。
Map.prototype.forEach() | 遍历 Map 的所有成员。

```js
// Map 的遍历顺序就是插入顺序
const map = new Map([
  ['F', 'no'],
  ['T',  'yes'],
]);

for (let key of map.keys()) {
  console.log(key);
}
// "F"
// "T"

for (let value of map.values()) {
  console.log(value);
}
// "no"
// "yes"

for (let item of map.entries()) {
  console.log(item[0], item[1]);
}
// "F" "no"
// "T" "yes"

// 或者利用解构赋值
for (let [key, value] of map.entries()) {
  console.log(key, value);
}
// "F" "no"
// "T" "yes"

// 等同于使用 map.entries()
for (let [key, value] of map) {
  console.log(key, value);
}
// "F" "no"
// "T" "yes"


// forEach() 方法
map.forEach(function(value, key, map) {
  console.log("Key: %s, Value: %s", key, value);
});
```

## Map 与其他数据结构转换

### Map 转为数组

```js
// 运用扩展运算符
const myMap = new Map()
  .set(true, 7)
  .set({foo: 3}, ['abc']);

[...myMap]   // [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
```

### 数组转为 Map

```js
// 将数组传入 Map 构造函数
new Map([
  [true, 7],
  [{foo: 3}, ['abc']]
])
// Map {
//   true => 7,
//   Object {foo: 3} => ['abc']
// }
```

### Map 转对象

```js
// 如果所有 Map 的键都是字符串，它可以无损地转为对象
// 如果有非字符串的键名，此键名会被转成字符串，再作为对象的键名。

// 创建一个空对象，遍历 Map，将键值对插入对象
function strMapToObj(strMap) {
  let obj = Object.create(null);
  for (let [k,v] of strMap) {
    obj[k] = v;
  }
  return obj;
}

const myMap = new Map()
  .set('yes', true)
  .set('no', false);
strMapToObj(myMap)
// { yes: true, no: false }

```

### 对象转为 Map

```js
// 对象转为 Map 可以通过 Object.entries()
let obj = {"a":1, "b":2};
let map = new Map(Object.entries(obj));
```

# WeakMap

> - WeakMap结构与Map结构类似，也是用于生成键值对的集合。
> - WeakMap 与 Map 的区别有两点。
>   - 首先，WeakMap 只接受**对象**作为键名（null除外），不接受其他类型的值作为键名。
>   - 其次，WeakMap 的键名所指向的对象，不计入**垃圾回收机制**。


# 参考链接

[网道 - ES6教程 - Set 和 Map数据结构](https://wangdoc.com/es6/set-map.html)


