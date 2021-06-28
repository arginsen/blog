---
title: 前端 | ES6笔记 | 十二、类
date: 2020-5-30 11:45:21
tags: 
- ES6
- Frontend
categories: notes
photos:
    - /blog/img/es6.jpg
---

<br>
<!--more-->


# 类的概念


> - JavaScript 语言中，生成实例对象的传统方法是通过构造函数。ES6 提供了更接近传统语言的写法，引入了 Class（类）这个概念，作为对象的**模板**，通过 class 关键字定义类，通过 new 生成实例对象。
> - 基本上，ES6 的 class 可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的 class 写法**只是**让对象原型的写法更加清晰、更像面向对象编程的语法而已。
> - 类的数据类型就是**函数**，类本身就指向**构造函数**。类的所有方法都定义在类的 **prototype** 属性上面。在类的实例上面调用方法，其实就是调用原型上的方法。类内部定义的方法，是**不可枚举**的。


```js
// ES5 通过定义一个构造函数,来 new 一个实例对象
// 同时将方法追加在构造函数的原型上，实例通过继承可直接调用
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};

var p = new Point(1, 2);

p   //  {x: 1, y: 2}
p.toString()    // "(1, 2)"


// 用 ES6 书写
class Point {
  constructor(x, y) {   // 构造方法
    this.x = x;
    this.y = y;
  }

  toString() {   // 方法
    return '(' + this.x + ', ' + this.y + ')';
  }
}

var p = new Point(1, 2);

p   // {x: 1, y: 2}
p.toString()   // "(1, 2)"

// 定义类里边方法的时候，不需要 function 声明，其本身就为函数，指向构造函数
typeof Point   // "function"
Point === Point.prototype.constructor   // true

 
// 在 类的实例 上面调用方法，等于从原型上调用方法
p.constructor === Point.prototype.constructor // true
p.toString === Point.prototype.toString   // true


// 类的新方法可以添加在 prototype 对象上面
// Object.assign 方法可以一次向类添加多个方法
Object.assign(Point.prototype, {
  ValueOf(){},
  toValue(){}
});


// 类内部定义的方法，是不可枚举的
Object.keys(Point.prototype)   // []
```

## constructor 方法

> constructor 方法是类的**默认方法**，通过 new 命令生成对象实例时，自动调用该方法。一个类必须有 constructor 方法，如果没有显式定义，一个空的 constructor 方法会被默认添加。

## 类的实例

> - 使用 new 命令生成类的实例。如果忘记加上 new，像函数那样调用 Class，将会报错
> - 实例的属性除非显式定义在其本身（即定义在 this 对象-实例），否则都是定义在**原型**上（即定义在 class 上）

```js
class Point {
  constructor(x, y) {
    this.x = x;    // 显式定义属性 x    this 指向类的实例
    this.y = y;    // 显式定义属性 y
  }

  toString() {    // 定义在原型上
    return '(' + this.x + ', ' + this.y + ')';
  }
}

var point = new Point(2, 3);
point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true


// 类的所有实例共享一个原型对象
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__ === p2.__proto__   // true


// 通过实例的原型给类添加方法
// 生产环境中可以使用 Object.getPrototypeOf 方法来获取实例对象的原型
// 给实例对象添加 printName 方法，同时也会应用到其他实例
Object.getPrototypeOf(p1).printName = function () { console.log(this.constructor.name) }

p1.printName() // Point
p2.printName() // Point
```

## 取值函数（getter）和存值函数（setter）


```js
// Val 属性有对应的存值函数和取值函数，因此赋值和读取行为都被自定义了
class myClass {
    constructor(value) {
        this.value = value
    }
    get Val() {
        return this.value
    }
    set Val(value) {
        this.value = value
    }
}

let c = new myClass()

c.Val(5)    // 5
c.Val()     // 5
```

## 属性表达式

```js
// 类的属性名，可以采用表达式
let methodName = 'getArea';

// 定义一个类
class Square {
  constructor(length) {
    // ...
  }

  // 方法名从外部变量获得
  [methodName]() {
    // ...
  }
}
```

## Class 表达式

```js
// 使用表达式定义一个类，
// Me 只在类内部可用，指代当前类，而外部只能用 MyClass 引用这个类
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};

let inst = new MyClass();
inst.getClassName() // Me

MyClass.name  // "Me"
Me.name // ReferenceError: Me is not defined


// 采用 Class 表达式，可以写出立即执行的 Class
let person = new class {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}('张三');

person.sayName(); // "张三"
```

# 类的注意事项

> - 默认严格模式
> - 不存在变量提升
> - Generator 方法：某个方法之前加上星号（*），就表示该方法是一个 Generator 函数。
> - this 的指向：默认指向**类的实例**，如将用到 this 的方法（xxx）提取出来单独使用，则可通过以下方法解决：
>   1. 在构造方法中绑定 this  `this.xxx = this.xxx.bind(this)`
>   2. 使用箭头函数 `this.getThis = () => this`
>   3. 使用 Proxy，获取方法的时候，自动绑定 this


# 静态方法

> - 类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上 static 关键字，就表示**该方法不会被实例继承**，而是**直接通过类来调用**，这就称为“静态方法”
> - 静态方法里包含 this，这个 this 指的是**类**，而不是实例
> - 父类的静态方法，可以被子类继承
> - 静态方法可以从 super 对象上调用的


```js
// 直接通过类来调用静态方法
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'
Foo.hasOwnProperty('classMethod')  // true

// 在实例上调用静态方法则会报错
var foo = new Foo();
foo.classMethod()
// TypeError: foo.classMethod is not a function


// 静态方法包含 this，这个 this 指的是类，不是实例
class Foo {
  static bar() {
    this.baz();
  }
  static baz() {
    console.log('hello');
  }
  baz() {
    console.log('world');
  }
}

Foo.bar() // hello


// 父类的静态方法，可以被子类继承
class Foo {
  static classMethod() {
    return 'hello';
  }
}

// 定义子类
class Bar extends Foo {
}

Bar.classMethod() // 'hello'


// 静态方法也是可以从 super 对象上调用的
class Foo {
  static classMethod() {
    return 'hello';
  }
}

// 通过 super 在子类内调用父类的该静态方法
class Bar extends Foo {
  static classMethod() {
    return super.classMethod() + ', too';  
  }
}

Bar.classMethod() // "hello, too"
```

# 实例属性

> 实例对象的属性，实例属性除了定义在 constructor() 方法里面的 this 上面，也可以定义在类的最顶层

```js
// 定义在 constructor() 方法里面的 this 上
class IncreasingCounter {
  constructor() {
    this._count = 0;
  }
  get value() {
    console.log('Getting the current value!');
    return this._count;
  }
  increment() {
    this._count++;
  }
}

// 实例属性与方法定义在同一层级，不需要加 this
class IncreasingCounter {
  _count = 0;
  get value() {
    console.log('Getting the current value!');
    return this._count;
  }
  increment() {
    this._count++;
  }
}
```

# 静态属性

> 静态属性指的是 Class 本身的属性，即 Class.propName，而不是定义在实例对象（this）上的属性
> 因为 ES6 明确规定，Class 内部只有静态方法，没有静态属性。现在有一个提案提供了类的静态属性，写法是在实例属性的前面，加上 static 关键字。

```js
class Foo {
}

Foo.prop = 1;   // 给类本身设置 prop 属性
Foo.prop // 1


// 定义 myStaticProp 为静态属性
class MyClass {
  static myStaticProp = 42;

  constructor() {
    console.log(MyClass.myStaticProp); // 42
  }
}
```

# 类的继承

> - Class 可以通过 **extends** 关键字实现继承
> - 子类必须在 constructor 方法中调用 **super** 方法，super 方法表示**父类的构造函数**，用来新建父类的 **this 对象**。子类自己的 this 对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法（将父类实例整个进 this），然后再修改、加上子类自己的实例属性和方法。如果不调用 super 方法，子类就得不到 this 对象。
> - 在子类的构造函数中，只有调用 super 之后，才可以使用 this 关键字，否则会报错。这是因为子类实例的构建，基于父类实例，只有 super 方法才能调用父类实例。
> - 父类的静态方法，也会被子类继承


```js
// 定义父类 Point
class Point {
  constructor(x, y) {
    this.x = x;    // 显式定义属性 x
    this.y = y;    // 显式定义属性 y
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}

// 定义子类 ColorPoint，继承自 Point
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的 constructor(x, y)
    this.color = color;  // 调用 super 之后才可使用 this
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的 toString()
  }
}

// 实例对象 cp 同时是 ColorPoint 和 Point 两个类的实例
let cp = new ColorPoint(25, 8, 'green');

cp   // {x: 25, y: 8, color: "green"}
cp.toString()   // "green (25, 8)"

cp instanceof ColorPoint // true
cp instanceof Point // true
```

## Object.getPrototypeOf()

> 可用来判断一个类是否继承了另一个类

```js
// 从子类上获取父类
Object.getPrototypeOf(ColorPoint) === Point  // true
```

## super 关键字

> super 这个关键字，既可以当作**函数**使用，也可以当作**对象**使用。
> super 作为函数调用时，代表**父类的构造函数**，但是返回的是子类的实例;
> super 作为对象时，在普通方法中，指向**父类的原型对象**；在静态方法中，指向**父类**。但定义在父类实例（this）上的方法或属性，是无法通过 super 调用的。

```js
// super 作为函数调用时，代表父类的构造函数
// 但是返回的是子类的实例，即 super 内部的 this 指的是子类的实例
// 子类的构造函数必须执行一次 super 函数
class A {}

class B extends A {
  constructor() {
    super();   // super 做函数只能用在子类的构造函数中
  }
}

// super() 就相当于 A 构造函数的 this 指向切换到参数指明的对象 B
// 即父类构造函数在子类环境中调用
A.prototype.constructor.call(this)


// super 作为父类的原型对象，因而可以调用父类的方法
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();


// p 非显式定义在 A 本身（this 上）
// 并不是定义在原型上，子类不能继承到
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m   // undefined


// 在子类普通方法中通过 super 调用父类的方法时，方法内部的 this 指向当前的子类实例
class A {
  constructor() {
    this.x = 1;
  }
  print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.print();  // 父类原型上的 print 方法
  }
}

let b = new B();

// 此时 m 方法中 super 调用父类 print 方法的 this 为 子类的 this
b.m() // 2


// 子类中的 this 指向为子类实例，
// 通过 super 对某个属性赋值，那么 super 也就相当于 this
// 而读取时 super 依然为父类的原型
class A {
  constructor() {
    this.x = 1;
  }
}

// super.x 赋值为 3，这时等同于对 this.x 赋值为 3
// 而当读取 super.x 的时候，读的是 A.prototype.x，所以返回 undefined
class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x);  // undefined
    console.log(this.x);  // 3
  }
}

let b = new B();


// super 作为对象，在静态方法之中，super 将指向父类，而不是父类的原型对象
class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }

  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }

  myMethod(msg) {
    super.myMethod(msg);
  }
}

// 调用子类本身的方法，即静态方法，而此时 super 指向父类
Child.myMethod(1); // static 1

// 此时实例 child 调用子类的方法，即子类原型上的方法，super 也指向父类的原型
var child = new Child();
child.myMethod(2); // instance 2


// 在子类的静态方法中通过 super 调用父类的方法时，
// 方法内部的 this 指向当前的子类，而不是子类的实例
class A {
  constructor() {
    this.x = 1;
  }
  static print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  static m() {
    super.print();  // 调用父类的方向
  }
}

// 只有类的实例才可以读取到 this.x，B 是类本身
B.x  // undefined
B.x = 3;
B.m() // 3


// 使用super的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super); // 报错
  }
}
```

# 类的 prototype 属性和__proto__属性


> 大多数浏览器的 ES5 实现之中，每一个对象都有__proto__属性，指向对应的构造函数的prototype属性。Class 作为构造函数的语法糖，同时有 **prototype 属性和 `__proto__` 属性**，因此同时存在两条继承链。
>（1）子类的 `__proto__` 属性，表示构造函数的继承，总是指向父类。
>（2）子类 prototype 属性的 `__proto__` 属性，表示方法的继承，总是指向父类的 prototype 属性。

> 作为一个**对象**，子类（B）的原型（__proto__属性）是父类（A）；
> 作为一个**构造函数**，子类（B）的原型对象（prototype属性）是父类的原型对象（prototype属性）的实例。

```js
class A {
}

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true

```




# 参考资料

[网道- ES6教程 - Class 的基本语法](https://wangdoc.com/es6/class.html)
[网道- ES6教程 - Class 的继承](https://wangdoc.com/es6/class-extends.html)