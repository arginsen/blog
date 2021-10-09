---
title: 前端 | JS深入 | 四、继承
date: 2020-5-9 17:45:21
tags: 
- JavaScript
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->


# 原型链继承

> 1.引用类型的属性被所有实例共享 
> 2.在创建 Child 的实例时，不能向 Parent 传参

```js
// 从原型链上继承方法
function Parent () {
    this.name = 'kevin';
}

Parent.prototype.getName = function () {
    console.log(this.name);
}

function Child () {

}

Child.prototype = new Parent();

var child1 = new Child();

console.log(child1.getName()) // kevin


// child1 改动属性后也会影响其他实例
function Parent () {
    this.names = ['kevin', 'daisy'];
}

function Child () {

}

Child.prototype = new Parent();   

var child1 = new Child();  // 实例会动态继承（引用）原型链上的属性、方法
child1.names.push('yayu'); // 在原型链上定义该属性的地方进行修改
console.log(child1.names); // ["kevin", "daisy", "yayu"]

var child2 = new Child();
console.log(child2.names); // ["kevin", "daisy", "yayu"]
```


# 借用构造函数（经典继承）

> 1.避免了引用类型的属性被所有实例共享
> 2.可以在 Child 中向 Parent 传参
> 3.**方法都在构造函数中定义，每次创建实例都会创建一遍方法**

```js
// 每次用 Child 创建实例都会执行一遍 Parent()，此处给 Parent 绑定了 this 指向
// 那么除了 Parent 构造函数定义的属性，其余的属性增删减改都将作为当前新实例独有的操作
而且将
function Parent () {
    this.names = ['kevin', 'daisy'];
}

function Child () {
    Parent.call(this);  // 给 Parent 函数通过 call 绑定 this 指向（运行环境）
}

var child1 = new Child();
child1.names.push('yayu');
console.log(child1.names); // ["kevin", "daisy", "yayu"]

var child2 = new Child();  // 再次新建的实例又会拥有一个新的运行环境
console.log(child2.names); // ["kevin", "daisy"]


// 将 Child 传入的参数作为从 Parent 继承来的 name 属性的值
// 但每个实例在通过构造函数创建的时候都绑定了自己的 this 指向
function Parent (name) {
    this.name = name;
}

function Child (name) {
    Parent.call(this, name); // 此时的 name 作为 Parent 的参数传入函数内
}

var child1 = new Child('kevin');
console.log(child1.name); // kevin

var child2 = new Child('daisy');
console.log(child2.name); // daisy
```

# 组合继承

> 融合原型链继承和构造函数的优点，是 JavaScript 中最常用的继承模式

```js
// 定义 Parent 函数
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

// 定义一个 Parent 原型上的方法
Parent.prototype.getName = function () {
    console.log(this.name)
}

// 定义 Child 函数
function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}

Child.prototype = new Parent();  // 指定 Child 原型为 Parent 实例
Child.prototype.constructor = Child;  // 指定 Child 原型构造器为 Child 构造函数

// 创建 Child 函数的实例 child1
var child1 = new Child('kevin', '18');

child1.colors.push('black');

console.log(child1.name); // kevin
console.log(child1.age); // 18
console.log(child1.colors); // ["red", "blue", "green", "black"]

// 创建 Child 函数的实例 child2，各实例独立，又可传参
var child2 = new Child('daisy', '20');

console.log(child2.name); // daisy
console.log(child2.age); // 20
console.log(child2.colors); // ["red", "blue", "green"]
```

# 原型式继承

```js
// 传入的对象作为创建的对象的原型
// 这也会使包含引用类型的属性值始终都会共享相应的值
function createObj(o) {
    function F(){}
    F.prototype = o;
    return new F();
}

var person = {
    name: 'kevin',
    friends: ['daisy', 'kelly']
}

var person1 = createObj(person);
var person2 = createObj(person);

// 给 person1 添加了 name 值，并非修改了原型上的 name 值
person1.name = 'person1';
console.log(person2.name); // kevin

person1.firends.push('taylor');
console.log(person2.friends); // ["daisy", "kelly", "taylor"]
```

# 寄生式继承

> 创建一个仅用于封装继承过程的函数，该函数在内部以某种形式来做增强对象，最后返回对象。缺点：跟借用构造函数模式一样，每次创建对象都会创建一遍方法。

```js
// 定义一个函数，将参数作为函数内新生成的对象的原型，再给新对象添加方法
function createObj (o) {
    var clone = Object.create(o);  // 创建并指定 clone 的原型为 o
    clone.sayName = function () {  // 给 clone 添加方法
        console.log('hi');
    }
    return clone;
}
```

# 寄生组合式继承

```js
// 该方式高效率体现在只调用了一次 Parent 构造函数，
// 并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。
// 与此同时，原型链还能保持不变；因此，能够正常使用 instanceof 和 isPrototypeOf

// 定义 Parent 函数，添加属性
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

// 在 Parent 原型链上创建一个方法
Parent.prototype.getName = function () {
    console.log(this.name)
}

// 定义 Child 函数，继承 Parent 的两个属性，并新建一个属性
function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}

// 关键的三步

// 新定义一个中间函数 F，F 的原型等于 Parent 的原型
// 此时 F 仅使用了原型获取 Parent 的原型，而 F 本身并没有多余属性
var F = function () {};
F.prototype = Parent.prototype;

// 令 Child 的原型等于 F 实例
// 以此 Child 的原型就只有从 Parent 原型上继承到的方法
// Child 本身才会有从 Parent 本身继承的属性
Child.prototype = new F();

var child1 = new Child('kevin', '18');
console.log(child1);


// 对上边的代码进行封装
// ------------------
// 定义一个函数，使参数成为返回空函数的原型
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}

// 定义一个函数，使 prototype 变量等于一个原型为 parent 的空实例
// 使 prototype 变量（返回的实例）的构造器指向 child 构造函数函数
// 再令 child 的原型等于 prototype 变量（parent 的实例）
function prototype(child, parent) {
    var prototype = object(parent.prototype);
    prototype.constructor = child;
    child.prototype = prototype;
}

// 调用时
prototype(Child, Parent);
// ------------------
```






