---
title: 前端 | JS深入 | 三、原型和原型链
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

# 原型

> - 每一个 JavaScript 对象(null除外) 都和另一个对象(原型)相关联，每一个对象都从原型继承属性。
> - 所有通过对象直接量创建的对象都具有同一个原型对象，并通过 Object.prototype 获取对原型对象的引用通过。

# 原型对象之 prototype 属性（对象→原型）

> - 原型对象的作用，就是定义所有实例对象共享的属性和方法。这也是它被称为原型对象的原因，而实例对象可以视作从原型对象衍生出来的子对象。
> - 每一个JavaScript对象(除了 null )都具有的一个属性，叫__proto__，这个属性会指向该对象的原型。它并不存在于对象的原型中，它来自 Object.prototype 

```js
// JavaScript 规定，每个函数都有一个 prototype 属性，指向一个对象
// 函数 f 默认具有 prototype 属性，它指向一个对象
function f() {}
typeof f.prototype  // "object"

// 构造函数在生成实例的时，prototype 属性会自动成为实例对象的原型
function Animal(name) {
  this.name = name;
}
Animal.prototype.color = 'white'  // 给 Animal 的 prototype 属性添加属性

var cat1 = new Animal('英短')     // 此时实例 cat1 的原型对象就是 Animal 的 prototype 属性
var cat2 = new Animal('美短')

cat1.color  // 'white'           // 实例继承了其原型对象的属性，color 并不作为实例本身的属性
cat2.color  // 'white'           // 所有实例均会继承

// 实例对象自身有某个属性或方法，它就不会再去原型对象寻找这个属性或方法。
cat1.color = 'black';

cat1.color  // 'black'
cat2.color  // 'yellow'
Animal.prototype.color // 'yellow'  原型对象的属性依旧不变

// __proto__ 是暴漏在外边的对象的原型，等同于 prototype
function Person() {
    //
}
var person = new Person();
console.log(person.__proto__ === Person.prototype);  // true

// 实例原型的原型 Object.prototype，实例原型就是通过 Object 构造函数生成的
console.log(Person.prototype.__proto__ === Object.prototype);  // true

//  Object.prototype 的原型为 null，null 即表示它没有原型
console.log(Object.prototype.__proto__ === null)  // true

// __proto__ 来自于 Object.prototype
// 与其说是 Object 的属性，不如说是它的 getter/setter 方法
person.__proto__ === Object.getPrototypeOf(person)
```

# 原型链

> - JavaScript 规定，所有对象都有自己的原型对象（prototype）。一方面，任何一个对象，都可以充当其他对象的原型；另一方面，由于原型对象也是对象，所以它也有自己的原型。因此，就会形成一个“原型链”（prototype chain）
> - 所有对象的原型最终都可以上溯到 Object.prototype，它也是原型链的顶端，因此所有对象都继承了Object.prototype 的属性。Object.prototype 对象的原型是 null，表示原型链到此为止
> - 读取对象属性，对象本身的属性找不到，就会在其原型查找，再到原型的原型，直到最顶层的 Object.prototype 还是找不到，则返回 undefined。对象自身和它的原型，都定义了一个同名属性，那么优先读取**对象自身**的属性，这叫做“覆盖”（overriding）

```js
// 使用 Object.getPrototypeOf() 方法返回参数对象的原型
Object.getPrototypeOf(Object.prototype)     // null

// 如果让构造函数的prototype属性指向一个数组，那么实例对象可以调用数组方法。
var MyArray = function () {};

MyArray.prototype = new Array();
MyArray.prototype.constructor = MyArray;   // MyArray 原型的构造器指向构造函数 MyArray

var mine = new MyArray();
mine.push(1, 2, 3);     // mine 函数使用数组的方法 .push()
mine.length             // 3
mine instanceof Array   // true
// instanceof 表达式，用来比较一个对象是否为某个构造函数的实例
// 结果就是证明 mine 为 Array 的实例
```

# 原型对象之 constructor 属性（原型→对象）

> - prototype 对象有一个 constructor 属性，默认**指向** prototype 对象所在的**构造函数**。
> - constructor 属性定义在 prototype 对象上面，意味着可以被所有实例对象继承

```js
// 构造函数 P 的原型对象的 constructor 属性指向 P 本身
function P() {}
P.prototype.constructor === P   // true

// 实例对象 p 继承了 构造函数 P 的原型的 constructor 属性
function P() {}
var p = new P();

p.constructor === P                         // true
p.constructor === P.prototype.constructor   // true
p.hasOwnProperty('constructor')             // false  继承来的
```

## constructor 属性的作用

> - 得知某个实例对象，到底是哪一个构造函数产生的。
> - 可以从一个实例对象新建另一个实例。
> - constructor属性表示原型对象与构造函数之间的关联关系，如果修改了原型对象，一般会同时修改constructor属性，防止引用的时候出错。

```js
// 通过判断实例 f 的 constructor 属性指向哪个构造函数
function F() {};
var f = new F();

f.constructor === F     // true
f.constructor === R     // false  对应的构造函数不是 R

// 从实例对象 x 新键实例对象 y
function Constr() {}            // 定义构造函数 Constr
var x = new Constr();           // 创建实例对象 x

var y = new x.constructor();    // x 的构造器继承自它的原型，指向其构造函数 Constr，即 x.constructor() === Constr()
y instanceof Constr             // true  说明 y 为构造函数 Constr 的实例

// 使用 createCopy 方法调用构造函数，新建另一个实例
Constr.prototype.createCopy = function () {
  return new this.constructor();    // this 指向函数对象本身，函数对象的构造器又指向函数本身
};

// Person 的原型对象修改后，需要修改 construtor 属性，否则构造器会不再指向 Person
function Person(name) {
  this.name = name;
}

Person.prototype.constructor === Person // true
Person.prototype = {
  method: function () {}
};

Person.prototype.constructor === Person // false
Person.prototype.constructor === Object // true

// 修改原型对象时，同时修改 constructor 属性的指向函数
Person.prototype = {
  constructor: C,           // 原型重新指向对象 = {}，需要添加构造器对指向原来的构造函数
  method1: function (...) { ... },
  // ...
};
// 
Person.prototype.method = function (...) { ... };   // 仅修改原型指定的属性

// 通过 name 属性来确定 construtor 属性指向哪个函数
function Foo() {}
var f = new Foo();
f.constructor.name // "Foo"
```

## instanceof 运算符

> - instanceof 运算符返回一个布尔值，表示对象是否为某个构造函数的实例
> - instanceof 的原理是检查右边构造函数的 prototype 属性，是否在左边对象的原型链上
> - 有一种特殊情况，左边对象的原型链上只有 null 对象,这时 instanceof 判断会失真
> - instanceof 运算符可以判断值的类型
> - instanceof 运算符只能用于对象，不适用原始类型的值
> - undefined 和 null，instanceof 运算符总是返回false

```js
// 对象 v 是构造函数 Vehicle 的实例
var v = new Vehicle();
v instanceof Vehicle    // true

// instanceof 检查 构造函数 Vehicle 的原型对象是否在 实例 v 的原型链上
// 相当于：
Vehicle.prototype.isPrototypeOf(v)   // true   Vehicle 的原型在 v 的原型链上
// isPrototypeOf() 函数用于指示对象是否存在于另一个对象(参数)的原型链中。

// instanceof 检查整个原型链，因此同一个实例对象，可能会对多个构造函数都返回 true
var d = new Date();
d instanceof Date      // true
d instanceof Object    // true
// d 同时是 Date 和 Object 的实例，因此对这两个构造函数都返回 true

// 任意对象（除null）都是 Object 的实例，因此 instanceof 可以判断一个值是否为非 null 对象
var obj = { foo: 123 };
obj instanceof Object  // true

null instanceof Object // false

// 当左边对象的原型链上只有 null 对象时，instanceof 判断会失真
var obj = Object.create(null);
typeof obj             // "object"  从 typeof 判断 obj 的数据类型

obj instanceof Object  // false
// 此时判断 Object 的原型是否在实例 obj 上，而 obj 的原型是 null

// instanceof 运算符判断值的类型
var x = [1, 2, 3];
var y = {};
x instanceof Array     // true
y instanceof Object    // true

// instanceof 运算符不适用原始类型的值
var s = 'hello';       // 字符串并不是对象
s instanceof String    // false  字符串并不是 String 对象的实例

// undefined 与 null 
undefined instanceof Object   // false
null instanceof Object        // false

// 利用 instanceof 运算符解决调用构造函数时，忘了加 new 命令的问题
function Fubar (foo, bar) {
  if (this instanceof Fubar) {  // 判断 this 是否为 Fubar 的实例
    this._foo = foo;
    this._bar = bar;
  } else {
    return new Fubar(foo, bar);
  }
}
```

# 构造函数的继承

> - 构造函数继承另一个构造函数，在子类的构造函数中，调用父类的构造函数，让子类的原型指向父类的原型，这样子类就可以继承父类原型
> - 子类的原型等于一个父类实例（不推荐），会使字类具有父类实例的方法

```js
// 采用第一种继承办法
// 定义一个构造函数 Shape
function Shape() {
  this.x = 0;
  this.y = 0;
}

Shape.prototype.move = function (x, y) {
  this.x += x;
  this.y += y;
  console.info('Shape moved.');
};

// 第一步，子类继承父类的实例
// 定义子类构造函数 Rectangle
function Rectangle() {
  Shape.call(this);     // 调用父类构造函数   this 是子类的实例 
}
// 另一种写法
function Rectangle() {
  this.base = Shape;
  this.base();
}

// 第二步，子类继承父类的原型
// Rectangle 的原型指向 Shape 的原型，使 Rectangle 的原型继承 Shape 原型的属性
// 在改变 Rectangle 的原型后，也重新指定它的构造器
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;          

var rect = new Rectangle();

rect instanceof Rectangle   // true
rect instanceof Shape       // true

```

# 参考资料

[网道- JavaScript教程 - 对象的继承](https://wangdoc.com/javascript/oop/prototype.html)

[冴羽的博客- JavaScirpt深入之从原型到原型链](https://github.com/mqyqingfeng/Blog/issues/2)












