---
title: 前端 | JS基础 | JS学习笔记Ⅷ：事件
date: 2020-4-30 10:32:11
tags: 
- JavaScript
- Event
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

> 事件的本质是程序各个组成部分之间的一种通信方式，也是异步编程的一种实现。

# EventTarget 接口

> DOM 的事件操作（监听和触发），都定义在 EventTarget 接口。所有节点对象都部署了这个接口，其他一些需要事件通信的浏览器内置对象（比如，XMLHttpRequest、AudioNode、AudioContext）也部署了这个接口.

**该接口主要提供三个实例方法**：
- addEventListener：**绑定**事件的监听函数
- removeEventListener：**移除**事件的监听函数
- dispatchEvent：**触发**事件
 
## EventTarget.addEventListener()

> - EventTarget.addEventListener() 用于在**当前节点或对象**上，定义一个特定事件的监听函数。一旦这个事件发生，就会执行监听函数。该方法没有返回值。
> - addEventListener 方法可以为针对**当前对象**的同一个事件，添加多个不同的监听函数。这些函数按照**添加顺序**触发，即先添加先触发。如果为同一个事件多次添加**同一个**监听函数，该函数只会执行一次，多余的添加将自动被去除。

```js
target.addEventListener(type, listener[, useCapture]);

// 该方法接受三个参数：
// type：事件名称，大小写敏感。
// listener：监听函数。事件发生时，会调用该监听函数。
// useCapture：布尔值，表示监听函数是否在捕获阶段（capture）触发，
//      默认为 false（监听函数只在冒泡阶段被触发）。该参数可选。


// button 节点绑定 click 事件的监听函数 hello
function hello() {
  console.log('Hello world');
}

var button = document.getElementById('btn');
button.addEventListener('click', hello, false);
```

### 补充：

```js
// 第二个参数除了监听函数，还可以是一个具有 handleEvent 方法的对象
buttonElement.addEventListener('click', {
  handleEvent: function (event) {
    console.log('click');
  }
});

// 第三个参数除了布尔值 useCapture，还可以是一个属性配置对象。
//    该对象有以下属性：
// capture：布尔值，表示该事件是否在捕获阶段触发监听函数。
// once：布尔值，表示监听函数是否只触发一次，然后就自动移除。
// passive：布尔值，表示监听函数不会调用事件的 preventDefault 方法。
//  如果监听函数调用了，浏览器将忽略这个要求，并在监控台输出一行警告。
element.addEventListener('click', function (event) {
  // 只执行一次的代码
}, {once: true});


// 监听函数内部的 this，指向当前事件所在的那个对象
// HTML 代码如下
// <p id="para">Hello</p>
var para = document.getElementById('para');
para.addEventListener('click', function (e) {
  console.log(this.nodeName); // "P"  this 指向对象 para
}, false);
```

## EventTarget.removeEventListener()

> - EventTarget.removeEventListener 方法用来移除 addEventListener 方法添加的事件监听函数。该方法没有返回值。
> - removeEventListener 方法的参数，与 addEventListener 方法完全一致。
> - removeEventListener 方法移除的监听函数，必须是 addEventListener 方法添加的那个监听函数，而且必须在同一个元素节点，否则无效。

```js
div.addEventListener('click', listener, false);
div.removeEventListener('click', listener, false);
```

## EventTarget.dispatchEvent()

> - EventTarget.dispatchEvent 方法在当前节点上**触发**指定事件，从而触发监听函数的执行。该方法返回一个布尔值，只要有一个监听函数调用了 Event.preventDefault()，则返回值为 false，否则为 true
> - 如果dispatchEvent方法的参数为空，或者不是一个有效的事件对象，将报错

```js
para.addEventListener('click', hello, false);
var event = new Event('click');
para.dispatchEvent(event);   // 参数为事件名
```


# 事件模型

## 监听函数

> 浏览器的事件模型，就是通过监听函数（listener）对事件做出反应。事件发生后，浏览器监听到了这个事件，就会执行对应的监听函数。JavaScript 有三种方法，可以为事件绑定监听函数。

### HTML 的 on- 属性

> - HTML 语言允许在元素的**属性**中，**直接定义**某些事件的监听代码，一旦事件发生，就会执行这段代码
> - 元素的事件监听属性，都是 **on 加上事件名**，比如 onload 就是 on + load，表示 load 事件的监听代码，属性的值是将会**执行**的代码，而不是一个函数
> - 使用此方法指定的监听代码，只会在**冒泡阶段**触发，事件会从子元素开始冒泡到父元素

```html
// load 事件一旦发生，执行 doSomething()
<body onload="doSomething()">


<!-- <button> 是 <div> 的子元素
     <button> 的 click 事件，也会触发 <div> 的 click 事件
     冒泡阶段触发，因此先输出 1，再输出 2 -->
<div onclick="console.log(2)">
  <button onclick="console.log(1)">点击</button>
</div>
```

### 元素节点的事件属性

> - 元素节点对象的事件属性，同样可以指定监听函数。此方法监听函数也只会在冒泡阶段触发
> - 缺点是同一个事件只能定义一个监听函数

```js
window.onload = doSomething;  // 监听函数名，而不是监听代码

div.onclick = function (event) {
  console.log('触发事件');
};
```

### EventTarget.addEventListener()

> 所有 DOM 节点实例都有 [addEventListener 方法](#EventTarget-addEventListener)，用来为该节点定义事件的监听函数。

## 监听函数内部的 this 指向

> 监听函数内部的 this 指向触发事件的那个**元素节点**。

```js
// 执行下面代码，点击后会输出 btn
<button id="btn" onclick="console.log(this.id)">点击</button>


// 其他两种监听函数的写法，this 的指向也是如此
// HTML 代码如下
// <button id="btn">点击</button>
var btn = document.getElementById('btn');

// 写法一
btn.onclick = function () {
  console.log(this.id);
};

// 写法二
btn.addEventListener(
  'click',
  function (e) {
    console.log(this.id);
  },
  false
);
```

## 事件的传播

> 一个事件发生后，会在子元素和父元素之间**传播**（propagation）。

**这种传播分成三个阶段**:
- 第一阶段：从 window 对象传导到目标节点（上层传到底层），称为**捕获阶段**（capture phase）。
- 第二阶段：在目标节点上触发，称为**目标阶段**（target phase）。
- 第三阶段：从目标节点传导回 window 对象（从底层传回上层），称为**冒泡阶段**（bubbling phase）。


```js
<div>
  <p>点击</p>
</div>

// 上面代码中，<div> 节点之中有一个 <p> 节点
// 如果对这两个节点，都设置click事件的监听函数
// （每个节点的捕获阶段和冒泡阶段，各设置一个监听函数）
// 共计设置四个监听函数。
// 然后，对 <p> 点击，click 事件会触发四次。

var phases = {
  1: 'capture',
  2: 'target',
  3: 'bubble'
};

var div = document.querySelector('div');
var p = document.querySelector('p');

div.addEventListener('click', callback, true);
p.addEventListener('click', callback, true);
div.addEventListener('click', callback, false);
p.addEventListener('click', callback, false);

function callback(event) {
  var tag = event.currentTarget.tagName;
  var phase = phases[event.eventPhase];
  console.log("Tag: '" + tag + "'. EventPhase: '" + phase + "'");
}

// 点击以后的结果:
// Tag: 'DIV'. EventPhase: 'capture'
// Tag: 'P'. EventPhase: 'target'
// Tag: 'P'. EventPhase: 'target'
// Tag: 'DIV'. EventPhase: 'bubble'

// click 事件被触发了四次：
// <div> 节点的捕获阶段和冒泡阶段各 1 次，
// <p> 节点的目标阶段触发了 2 次
// 浏览器假定 click 事件的目标节点是点击位置嵌套最深的节点（<div>里的<p>）
// 所以，<p> 节点的捕获阶段和冒泡阶段，都会显示为 target 阶段

// 1.捕获阶段：事件从<div>向<p>传播时，触发<div>的click事件；
// 2.目标阶段：事件从<div>到达<p>时，触发<p>的click事件；
// 3.冒泡阶段：事件从<p>传回<div>时，再次触发<div>的click事件。

// 事件传播顺序，在捕获阶段依次为 window、document、html、body、div、p，
// 在冒泡阶段依次为 p、div、body、html、document、window
```

## 事件的代理

> 由于事件会在**冒泡阶段**向上传播到父节点，因此可以把子节点的监听函数定义在**父节点**上，由父节点的监听函数统一处理多个子元素的事件。这种方法叫做事件的**代理**（delegation）。
> 
> 优点：只要定义一个监听函数，就能处理多个子节点的事件，而不用在每个子节点上定义监听函数。而且以后再添加子节点，监听函数依然有效。

```js
// click事件的监听函数定义在 <ul> 节点，
// 实际上，它处理的是子节点 <li> 的 click 事件
var ul = document.querySelector('ul');

ul.addEventListener('click', function (event) {
  if (event.target.tagName.toLowerCase() === 'li') {
    // some code
  }
});
```

## 事件终止传播

### stopPropagation()

```js
// 捕获阶段阻止了事件的传播
p.addEventListener('click', function (event) {
  event.stopPropagation();  // 事件传播到 p 元素后，就不再向下传播了
}, true);

// 冒泡阶段阻止了事件的传播
p.addEventListener('click', function (event) {
  event.stopPropagation();  // 事件冒泡到 p 元素后，就不再向上冒泡了
}, false);


// stopPropagation 方法只会阻止事件的传播，
// 不会阻止该事件触发 <p> 节点的其他 click 事件的监听函数。
// 也就是说，不是彻底取消 click 事件
p.addEventListener('click', function (event) {
  event.stopPropagation();
  console.log(1);
});

p.addEventListener('click', function(event) {
  // 上一个事件被阻止传播，但此事件依旧会触发
  console.log(2);
});
// 1
// 2
```

### stopImmediatePropagation()

```js
// 彻底取消该事件，不再触发后面所有 click 的监听函数，
// 可以使用 stopImmediatePropagation 方法
p.addEventListener('click', function (event) {
  event.stopImmediatePropagation();
  console.log(1);
});

p.addEventListener('click', function(event) {
  // 用了这个方法，这个事件及后边所有的监听函数都不会被触发
  console.log(2);
});
// 1
```


# Event 对象

> 事件发生以后，会产生一个**事件对象**，作为参数传给监听函数。浏览器原生提供一个 Event 对象，所有的事件都是这个对象的实例，或者说继承了 Event.prototype 对象

## Event()

> Event对象本身就是一个构造函数，可以用来生成新的实例。

```js
event = new Event(type, options);

// Event 构造函数接受两个参数：
// 第一个参数 type 是字符串，表示事件的名称；
// 第二个参数 options 是一个对象，表示事件对象的配置。该对象主要有下面两个属性。

// bubbles：布尔值，可选，默认为 false，表示事件对象是否冒泡。
// cancelable：布尔值，可选，默认为 false，表示事件是否可以被取消，
//    即能否用 Event.preventDefault() 取消这个事件。
//    一旦事件被取消，就好像从来没有发生过，不会触发浏览器对该事件的默认行为。


// 新建一个 look 事件实例，然后使用 dispatchEvent 方法触发该事件
// 如果不是显式指定 bubbles 属性为 true，生成的事件就只能在“捕获阶段”触发监听函数
var ev = new Event(
  'look',
  {
    'bubbles': true,    // 冒泡阶段被触发
    'cancelable': false
  }
);
document.dispatchEvent(ev);   // 触发


// 给父节点绑定一个冒泡阶段触发的事件
// 给子节点绑定一个捕获阶段触发的事件并手动触发
var div = document.querySelector('div');
var p = document.querySelector('p');

function callback(event) {
  var tag = event.currentTarget.tagName;
  console.log('Tag: ' + tag);  // 没有任何输出
}

div.addEventListener('click', callback, false);   // 冒泡阶段监听

var click = new Event('click');   // 无显式指定 bubbles 为 true，默认捕获阶段触发
p.dispatchEvent(click);    // 无法触发监听函数 callback，因为是 div 在冒泡阶段触发

// 如果这个事件在 div 元素上触发，div 作为事件的目标，会接收事件导致监听函数生效
div.dispatchEvent(click);
```

## Event 对象的实例属性

属性 | 作用
---- | ----
Event.bubbles      | 返回一个布尔值，表示当前事件是否会冒泡，只读
Event.eventPhase   | 返回一个整数常量，表示事件目前所处的阶段，只读
Event.cancelable   | 返回一个布尔值，表示事件是否可以取消，只读
Event.cancelBubble | 布尔值，如果设为 true，相当于执行 Event.stopPropagation()，可以阻止事件的传播
event.defaultPrevented | 返回一个布尔值，表示该事件是否调用过 Event.preventDefault 方法，只读
Event.currentTarget| 返回事件当前所在的节点，即事件当前正在通过的节点
Event.target       | 返回原始触发事件的那个节点，即事件最初发生的节点
Event.type         | 返回一个字符串，表示事件类型，生成事件时指定的，只读
Event.timeStamp    | 返回一个毫秒时间戳，表示事件发生的时间
Event.isTrusted    | 返回一个布尔值，表示该事件是否由真实的用户行为产生
Event.detail       | 返回一个数值，表示事件的某种信息。具体含义与事件类型相关


## Event 对象的实例方法

方法 | 作用
---- | ----
Event.preventDefault() | 取消浏览器对当前事件的默认行为，但不会阻止事件的传播
Event.stopPropagation()| 阻止事件在 DOM 中继续传播，防止再触发定义在**别的节点**上的监听函数，但是不包括在当前节点上其他的事件监听函数
Event.stopImmediatePropagation() | 阻止同一个事件的其他监听函数被调用，不管监听函数定义在当前节点还是其他节点
Event.composedPath()   | 返回一个数组，包含事件的最底层节点和依次冒泡经过的所有上层节点


# 鼠标事件

# 键盘事件

# 进度事件

# 表单事件

# 触摸事件

# 拖拉事件

# 参考链接

[网道- JavaScript教程 - 事件](https://wangdoc.com/javascript/events/index.html)