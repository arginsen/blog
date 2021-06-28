---
title: 前端 | JS基础 | JS学习笔记Ⅶ：DOM
date: 2020-4-28 16:32:11
tags: 
- JavaScript
- DOM
- Frontend
categories: notes
photos:
    - /blog/img/javascript.jpg
---

<br>
<!--more-->

# DOM

> - DOM 是 JavaScript 操作网页的接口，全称为“文档对象模型”（Document Object Model）。
> - DOM 的作用是将网页转为一个 JavaScript 对象，从而可以用脚本进行各种操作（比如增删内容）。
> - 浏览器会根据 DOM 模型，将结构化文档（比如 HTML 和 XML）解析成一系列的节点，再由这些节点组成一个树状结构（DOM Tree）。所有的节点和最终的树状结构，都有规范的对外接口。
> - DOM 只是一个接口规范，可以用各种语言实现。所以严格地说，DOM 不是 JavaScript 语法的一部分，但是 DOM 操作是 JavaScript 最常见的任务

## 节点

> - DOM 的最小组成单位叫做节点（node）。文档的树形结构（DOM 树），就是由各种不同类型的节点组成。每个节点可以看作是文档树的一片叶子。
> - 浏览器提供一个**原生的节点对象 Node**，下面这七种节点都继承了 Node，因此具有一些共同的属性和方法。


- **节点的类型有七种：**

类型 | 说明
------ | ------
Document         | 整个文档树的顶层节点
DocumentType     | doctype标签（比如`<!DOCTYPE html>`）
Element          | 网页的各种HTML标签（比如`<body>、<a>`等）
Attr             | 网页元素的属性（比如`class="right"`）
Text             | 标签之间或标签包含的文本
Comment          | 注释
DocumentFragment | 文档的片段

## 节点树

> - 一个文档的所有节点，按照所在的层级，可以抽象成一种树状结构。这种树状结构就是 DOM 树。
> - 它有一个顶层节点，下一层都是顶层节点的子节点，然后子节点又有自己的子节点，就这样层层衍生出一个金字塔结构，又像一棵树。
> - 浏览器原生提供 **document** 节点，代表整个文档。


- 文档的第一层有两个节点，第一个是文档类型节点（`<!doctype html>`），第二个是 HTML 网页的顶层容器标签 `<html>`。后者构成了树结构的根节点（root node），其他 HTML 标签节点都是它的下级节点。
- 除了根节点，其他节点都有三种层级关系。
    - 父节点关系（parentNode）：直接的那个上级节点
    - 子节点关系（childNodes）：直接的下级节点
    - 同级节点关系（sibling）：拥有同一个父节点的节点
- DOM 提供操作接口，用来获取这三种关系的节点。比如，子节点接口包括 firstChild（第一个子节点）和 lastChild（最后一个子节点）等属性，同级节点接口包括 nextSibling（紧邻在后的那个同级节点）和 previousSibling（紧邻在前的那个同级节点）属性。

# Node 接口

> 所有 DOM 节点对象都**继承**了 Node 接口，拥有一些**共同**的**属性**和**方法**

## 属性


属性 | 作用
---- | -----
Node.prototype.nodeType | 节点的类型
Node.prototype.nodeName | 节点的名称
Node.prototype.nodeValue| 当前节点本身的文本值
Node.prototype.textContent  | 当前节点和它的所有后代节点的文本内容
Node.prototype.baseURI  | 当前网页的绝对路径
Node.prototype.nextSibling  | 当前节点后边第一个同级节点
Node.prototype.previousSibling | 当前节点前边最近的一个同级节点
Node.prototype.parentNode   | 当前节点的父节点
Node.prototype.parentElement| 当前节点的父元素节点
Node.prototype.firstChild，Node.prototype.lastChild | 当前节点的第一个子节点/最后一个子节点
Node.prototype.childNodes   | 当前节点的所有子节点
Node.prototype.isConnected  | 检测当前节点是否在文档中


```js
// Node.prototype.nodeType
// -----------------------------------------
// nodeType 属性返回一个整数值，表示节点的类型。
document.nodeType // 9
document.nodeType === Node.DOCUMENT_NODE // true    如下展示 document 的节点类型为 9

// Node 对象定义了几个常量，对应这些类型值：
// 文档节点（document）：            9， 对应常量 Node.DOCUMENT_NODE
// 元素节点（element）：             1， 对应常量 Node.ELEMENT_NODE
// 属性节点（attr）：                2， 对应常量 Node.ATTRIBUTE_NODE
// 文本节点（text）：                3， 对应常量 Node.TEXT_NODE
// 文档片断节点（DocumentFragment）：11，对应常量 Node.DOCUMENT_FRAGMENT_NODE
// 文档类型节点（DocumentType）：    10，对应常量 Node.DOCUMENT_TYPE_NODE
// 注释节点（Comment）：             8， 对应常量 Node.COMMENT_NODE

// 例
var ele = document.querySelector('.title')
ele    // <a href="/" class="navbar-item title has-text-grey has-text-weight-light is-5">网道 / WangDoc.com </a>
ele.nodeType   // 1

var attr = document.querySelector('.title').attributes[0]
attr.nodeType   // 2

var text = document.querySelector('.title').firstChild
text.nodeType   // 3
// -----------------------------------------


// Node.prototype.nodeName
// -----------------------------------------
// nodeName 属性返回节点的名称
document.nodeName // #document

// 不同节点的 nodeName 属性值如下：
// 文档节点（document）：            #document
// 元素节点（element）：             大写的标签名
// 属性节点（attr）：                属性的名称
// 文本节点（text）：                #text
// 文档片断节点（DocumentFragment）：#document-fragment
// 文档类型节点（DocumentType）：    文档的类型
// 注释节点（Comment）：             #comment

// 例
var ele = document.querySelector('.title')
ele.nodeName   // "A"

var attr = document.querySelector('.title').attributes[0]
attr.nodeName   // "href"

var text = document.querySelector('.title').firstChild
text.nodeName   // "#text"
// -----------------------------------------


// Node.prototype.nodeValue
// -----------------------------------------
// nodeValue 属性返回一个字符串，表示当前节点本身的文本值，该属性可读写
// 只有属性节点（attr）、文本节点（text）和注释节点（comment）有文本值
// 此三类节点的 nodeValue 可以返回结果，其他类型的节点一律返回 null

// 例
var ele = document.querySelector('.title')
ele.nodeName   // null

var attr = document.querySelector('.title').attributes[0]
attr.nodeName   // "/"

var text = document.querySelector('.title').firstChild
text.nodeName   // "网道 / WangDoc.com "
// -----------------------------------------


// Node.prototype.textContent
// -----------------------------------------
// textContent 属性返回当前节点和它的所有后代节点的文本内容
// textContent 属性自动忽略当前节点内部的 HTML 标签，返回所有文本内容
// 该属性是可读写的，设置该属性的值，会用一个新的文本节点，替换所有原来的子节点
var ele = document.querySelector('.title')
ele.textContent   // "网道 / WangDoc.com "

document.documentElement.textContent    // 读取整个文档的内容

// 1. 对于属性节点（attr）、文本节点（text）和注释节点（comment），
//    textContent 属性的值与 nodeValue 属性相同
// 2. 对于其他类型的节点，该属性会将每个子节点（不包括注释节点）的内容连接在一起返回
// 3. 如果一个节点没有子节点，则返回空字符串
// 4. 文档节点（document）和文档类型节点（doctype）的 textContent 属性为 null
// -----------------------------------------


// Node.prototype.baseURI 
// -----------------------------------------
// baseURI属性返回一个字符串，表示当前网页的绝对路径。
// 浏览器根据这个属性，计算网页上的相对路径的 URL。
// 该属性为只读，如果无法读取，则返回 null
document.baseURI    // "https://wangdoc.com/javascript/dom/node.html"
// -----------------------------------------


// Node.prototype.nextSibling 
// -----------------------------------------
// nextSibling 属性返回紧跟在当前节点后面的第一个同级节点。
// 如果当前节点后面没有同级节点，则返回 null
// 对于
// <div id="d1">hello</div>
// <div id="d2">world</div>
var d1 = document.getElementById('d1');
var d2 = document.getElementById('d2');

d1.nextSibling === d2   // true

// 使用 nextSibling 遍历所有子节点
var ele = document.getElementById('div1').firstChild;

while (ele !== null) {
  console.log(ele.nodeName);
  ele = ele.nextSibling;
}
// -----------------------------------------


// Node.prototype.previousSibling
// -----------------------------------------
// previousSibling 属性返回当前节点前面的、距离最近的一个同级节点
// 如果当前节点前面没有同级节点，则返回null
// HTML 代码如下
// <div id="d1">hello</div>
// <div id="d2">world</div>
var d1 = document.getElementById('d1');
var d2 = document.getElementById('d2');

d2.previousSibling === d1   // true
// -----------------------------------------


// Node.prototype.parentNode
// -----------------------------------------
// parentNode 属性返回当前节点的父节点
// 对于一个节点来说，它的父节点只可能是三种类型：
// 元素节点（element）、文档节点（document）和文档片段节点（documentfragment）。
if (node.parentNode) {
  node.parentNode.removeChild(node);   // 通过 parentNode 属性将当前节点从文档里面移除
}

// 文档节点（document）和文档片段节点（documentfragment）的父节点都是 null
// 对于那些生成后还没插入 DOM 树的节点，父节点也是 null
// -----------------------------------------


// Node.prototype.parentElement
// -----------------------------------------
// parentElement属性返回当前节点的父元素节点
// 当前节点没有父节点，或者父节点类型不是元素节点，返回 null
if (node.parentElement) {
  node.parentElement.style.color = 'red';   // 给父元素节点的样式设定了红色
}

// 由于父节点只可能是三种类型：元素节点、文档节点（document）和文档片段节点（documentfragment），
// parentElement 属性相当于把后两种父节点都排除了
// -----------------------------------------


// Node.prototype.firstChild
// Node.prototype.lastChild
// -----------------------------------------
// firstChild 属性返回当前节点的第一个子节点，如果当前节点没有子节点，则返回 null
// lastChild 属性返回当前节点的最后一个子节点，如果当前节点没有子节点，则返回 null

// HTML 代码如下
// <p id="p1"><span>First span</span></p>
var p1 = document.getElementById('p1');
p1.firstChild.nodeName  // "SPAN"   
// 返回第一个子节点是元素节点 span

// <p id="p1">
//   <span>First span</span>
//  </p>
var p1 = document.getElementById('p1');
p1.firstChild.nodeName // "#text"   
// p元素与span元素之间有空白字符，因此返回文本节点（标签之间）
// -----------------------------------------
t

// Node.prototype.childNodes
// -----------------------------------------
// childNodes 属性返回一个类似数组的对象（NodeList集合），包括当前节点的所有子节点
// 文档节点（document）就有两个子节点：文档类型节点（docType）和 HTML 根元素节点
document.childNodes     // NodeList(2) [html, html]   
// 返回 NodeList，有两个元素（两个子节点）
document.childNodes.length        // 2   
// 类数组则拥有 length 属性
document.childNodes[0].nodeType   // 10
// 第一个子节点 nodeType 为 10，对应文档类型节点
document.childNodes[1].nodeType   // 1
// 第二个子节点 nodeType 为 1，对应元素节点，此处即是 html 根元素节点
// 此时也就对应了节点树所说的，文档第一层有两个节点，
// 其中第二个 html 作为顶层容器标签构成了 DOM 树的根节点
// -----------------------------------------


// Node.prototype.isConnected
// -----------------------------------------
// isConnected 属性返回一个布尔值，表示当前节点是否在文档之中
var test = document.createElement('p');
test.isConnected  // false

document.body.appendChild(test);    // 该方法将 test 作为子节点，插入到 body 的尾部
test.isConnected  // true
// -----------------------------------------
```


## 方法


方法 | 作用
---- | -----
Node.prototype.appendChild()   | 当前节点插入子节点，排到最后
Node.prototype.hasChildNodes() | 检测当前节点是否有子节点
Node.prototype.cloneNode()     | 克隆一个节点
Node.prototype.insertBefore()  | 将某个节点插入父节点内部的指定节点的前边
Node.prototype.removeChild()   | 从当前节点移除该子节点
Node.prototype.replaceChild()  | 将一个新的节点，替换当前节点的某一个子节点
Node.prototype.contains()      | 检测一个节点是否为当前节点，或者是 其子节点，后代节点
Node.prototype.isEqualNode()   | 检测两个节点是否相等
Node.prototype.isSameNode()    | 检测两个节点是否为同一个节点
Node.prototype.normalize()     | 清理当前节点内部的所有文本节点，去除空的，合并相邻的文本节点
Node.prototype.getRootNode()   | 得到当前节点所在的文档根节点


```js
// Node.prototype.appendChild()
// -----------------------------------------
// appendChild() 方法接受一个节点对象作为参数，将其作为最后一个子节点，插入当前节点
// 该方法的返回值就是插入文档的子节点
var p = document.createElement('p');
document.body.appendChild(p);    // <p></p>

// 如果参数节点是 DOM 已经存在的节点，appendChild()方法会将其从原来的位置，移动到新位置
var div = document.getElementById('myDiv');   // 声明一个已经存在的节点
document.body.appendChild(div);   // 插入到 body，实际上是将这个节点移动到 document.body 的尾部
// -----------------------------------------


// Node.prototype.hasChildNodes()
// -----------------------------------------
// hasChildNodes() 方法返回一个布尔值，表示当前节点是否有子节点

// 判断一个节点有没有子节点，有许多种方法，下面是其中的三种：
node.hasChildNodes()
node.firstChild !== null
node.childNodes && node.childNodes.length > 0
// -----------------------------------------


// Node.prototype.cloneNode()
// -----------------------------------------
// cloneNode() 方法用于克隆一个节点
// 该方法接受一个布尔值作为参数，表示是否同时克隆子节点
// 其返回值是一个克隆出来的新节点。

// -----------------------------------------


// Node.prototype.insertBefore()
// -----------------------------------------
// insertBefore() 方法用于将某个节点插入父节点内部的指定位置
// 该方法接受两个参数，
// 第一个参数是所要插入的新节点，第二个参数是父节点内部一个用于定位的子节点。
// 新节点将插在定位的子节点前面
// 返回值是插入的新节点
// 当第二个参数为 null 时，则新节点将插在当前节点内部的最后位置

var p = document.createElement('p');
document.body.insertBefore(p, document.body.firstChild);    // 将新建的元素节点插入到 body 节点的第一个子节点

// 如果要将新节点插在父节点的某个子节点后边，可以结合 nextSibling 实现
parent.insertBefore(s1, s2.nextSibling);    // 将 s1 插入到 s2 节点的后边
// -----------------------------------------


// Node.prototype.removeChild()
// -----------------------------------------
// removeChild() 方法接受一个子节点作为参数，用于从当前节点移除该子节点
// 返回值是移除的子节点
// 被移除的节点依然存在于内存之中，但不再是 DOM 的一部分。
// 所以，一个节点移除以后，依然可以使用它，比如插入到另一个节点下面。

var divA = document.getElementById('A');
divA.parentNode.removeChild(divA);   // 在该节点的父节点来删除该节点
// 如果参数节点不是当前节点的子节点，removeChild方法将报错
// 上边就相当于删除该节点的子节点 divA
// -----------------------------------------


// Node.prototype.replaceChild()
// -----------------------------------------
// replaceChild() 方法用于将一个新的节点，替换当前节点的某一个子节点
// 方法接受两个参数，
// 第一个参数是用来替换的新节点，第二个参数是将要替换走的子节点
// 返回值是替换走的那个节点 oldChild，也就是说替换了 xxx
var replacedNode = parentNode.replaceChild(newChild, oldChild);
// -----------------------------------------


// Node.prototype.contains()
// -----------------------------------------
// contains() 方法返回一个布尔值，表示参数节点是否满足以下三个条件之一
//  1.参数节点为当前节点。
//  2.参数节点为当前节点的子节点。
//  3.参数节点为当前节点的后代节点
node.contains(node)   // true   参数 node 节点是当前节点
document.body.contains(node)   // true   参数 node 节点包含在当前文档中
// -----------------------------------------


// Node.prototype.isEqualNode()
// Node.prototype.isSameNode()
// -----------------------------------------
// isEqualNode() 方法返回一个布尔值，用于检查两个节点是否相等
// 所谓相等的节点，指的是两个节点的类型相同、属性相同、子节点相同
var p1 = document.createElement('p');
var p2 = document.createElement('p');
p1.isEqualNode(p2) // true

// isSameNode() 方法返回一个布尔值，表示两个节点是否为同一个节点
p1.isSameNode(p2) // false  相等但不是同一个
p1.isSameNode(p1) // true
// -----------------------------------------


// Node.prototype.normalize()
// -----------------------------------------
// normalize() 方法用于清理当前节点内部的所有文本节点（text）：
// 操作：去除空的文本节点，并且将毗邻的文本节点合并成一个，
// 也就是说不存在空的文本节点，以及毗邻的文本节点。
var wrapper = document.createElement('div');

wrapper.appendChild(document.createTextNode('Part 1 '));    // 创建文本子节点
wrapper.appendChild(document.createTextNode('Part 2 '));    // 创建文本子节点
wrapper.appendChild(document.createTextNode(''));           // 创建一个空的文本节点

wrapper.childNodes.length   // 3    wrapper 元素节点有两个子节点
wrapper.normalize();
wrapper.childNodes.length   // 1    三个毗邻的文本节点合并
wrapper.childNodes[0].data  // "Part 1 Part 2 "    空的文本节点被去除
// -----------------------------------------


// Node.prototype.getRootNode()
// -----------------------------------------
// getRootNode() 方法返回当前节点所在文档的根节点 document
// 该方法与 ownerDocument 属性的作用相同，但该方法可以用于 document 自身
document.body.firstChild.getRootNode() === document
// true
document.body.firstChild.getRootNode() === document.body.firstChild.ownerDocument
// true

document.getRootNode()  // document
document.ownerDocument  // null
// -----------------------------------------
```

# NodeList 接口

> NodeList 实例是一个类似数组的对象，其成员是节点对象。通过以下方法可以得到 NodeList 实例：
> - Node.childNodes
> - document.querySelectorAll() 等节点搜索方法
>
> NodeList 实例可能是动态集合，也可能是静态集合。所谓动态集合就是一个活的集合，DOM 删除或新增一个相关节点，都会立刻反映在 NodeList 实例。目前，只有 **Node.childNodes** 返回的是一个**动态集合**，其他的 NodeList 都是静态集合


属性/方法 | 作用
---- | -----
NodeList.prototype.length | NodeList 实例包含的节点数量
NodeList.prototype.forEach() | 遍历 NodeList 的所有成员
NodeList.prototype.item()    | 提取参数指定序号位置的节点
NodeList.prototype.keys()    | 返回键名的遍历器
NodeList.prototype.values()  | 返回键值的遍历器
NodeList.prototype.entries() | 返回的遍历器同时包含键名和键值的信息

```js
// document 文档下 body 节点的子节点对象集就是 Nodelist 构造函数的实例
document.body.childNodes instanceof NodeList  // true

// NodeList 实例很像数组，可以使用 length 属性和 forEach 方法，当然也可以使用 for 循环
var children = document.body.childNodes;
Array.isArray(children)  // false   调用 Array 的方法检测 children 不是数组

children.length // 34
children.forEach(console.log)   // 将 34 个子节点注意打印出来

// 可以将 NodeList 实例转为真正的数组
var nodeArr = Array.prototype.slice.call(children);


// NodeList.prototype.length
// -----------------------------------------
// length 属性返回 NodeList 实例包含的节点数量
var wrapper = document.createElement('div');

wrapper.appendChild(document.createTextNode('Part 1 '));    // 创建文本子节点 1
wrapper.appendChild(document.createTextNode('Part 2 '));    // 创建文本子节点 2
wrapper.appendChild(document.createTextNode(''));           // 创建空文本子节点

wrapper.childNodes.length   // 3

// -----------------------------------------


// NodeList.prototype.forEach()
// -----------------------------------------
// forEach() 方法用于遍历 NodeList 的所有成员
wrapper.childNodes.forEach(console.log)  // console.log 作为回调函数，使遍历一个节点就打印出来

// 该方法的第一个参数为引入的回调函数，每一轮遍历就执行一次这个回调函数
// 该方法的第二个参数，用于绑定回调函数内部的this，该参数可省略
// 其中回调函数 f 有三个参数，依次是当前成员、位置和当前 NodeList 实例
wrapper.childNodes.forEach(function f(item, i, list) {
  // ...
}, this);
// -----------------------------------------


// NodeList.prototype.item()
// -----------------------------------------
// item() 方法接受一个整数值作为参数，表示成员的位置，返回该位置上的成员
wrapper.childNodes.item(0)  // "Part 1 "
// 等同于方括号提取，一般情况下也使用方括号
wrapper.childNodes[0]       // "Part 1 "
// -----------------------------------------


// NodeList.prototype.keys()
// NodeList.prototype.values()
// NodeList.prototype.entries()
// -----------------------------------------
// 这三个方法都返回一个 ES6 的遍历器对象
// 可以通过 for...of 循环遍历获取每一个成员的信息
// keys() 返回键名的遍历器
// values() 返回键值的遍历器
// entries() 返回的遍历器同时包含键名和键值的信息

var childwrapper = wrapper.childNodes

for (var key of childwrapper.keys()) {
    console.log(key)
}
// 0
// 1
// 2

for (var value of childwrapper.values()) {
    console.log(value)
}
// "Part 1 "
// "Part 2 "
// #text    之前插入的空文本节点

for (var entry of childwrapper.entries()) {
    console.log(entry)
}
// [0, text]
// [1, text]
// [2, text]
// -----------------------------------------

```

# HTMLCollection 接口

> - HTMLCollection 是一个节点对象的**集合**，只能包含**元素节点**（element），不能包含其他类型的节点。它的返回值是一个类似数组的对象
> - HTMLCollection 没有 forEach 方法，只能使用 for 循环遍历
> - 返回 HTMLCollection 实例的，主要是一些 Document 对象的集合属性，比如document.links、document.forms、document.images等
> - HTMLCollection实例都是**动态集合**，节点的变化会实时反映在集合中
> - 当元素节点有 id 或 name 属性，


属性/方法 | 作用
---- | -----
HTMLCollection.prototype.length | HTMLCollection 实例包含的成员数量
HTMLCollection.prototype.item() | 提取参数指定位置的 HTMLCollection 实例的值
HTMLCollection.prototype.namedItem() |  提取指定 id 属性或 name 属性的参数对应的元素节点


```js
// document 文档下 links 对象就是 HTMLCollection 构造函数的实例
document.links instanceof HTMLCollection  // true

// 使用 id 或 name 属性引用该节点元素
// HTML 代码如下
// <img id="pic" src="http://example.com/foo.jpg">
document.images.pic === document.getElementById('pic')  // true
// document.images 从实例通过元素的 id 属性提取到对应的元素


// HTMLCollection.prototype.length
// -----------------------------------------
// length属性返回 HTMLCollection 实例包含的成员数量
document.links.length   // 145
// -----------------------------------------


// HTMLCollection.prototype.item()
// -----------------------------------------
// item() 方法接受一个整数值作为参数，表示成员的位置，返回该位置上的成员
document.links.item(3)  // <a href=​"../​dom/​index.html">​DOM​</a>​

// 等同于 方括号
document.links[3]   // <a href=​"../​dom/​index.html">​DOM​</a>​
// -----------------------------------------


// HTMLCollection.prototype.namedItem()
// -----------------------------------------
// namedItem() 方法的参数是一个字符串，
// 表示 id 属性或 name 属性的值，返回对应的元素节点
// 如果没有对应的节点，则返回 null

// 该方法与使用元素的 id 或 name 属性提取值一样
// HTML 代码如下
// <img id="pic" src="http://example.com/foo.jpg">
document.images.namedItem('pic') === document.images.pic // true
document.images.pic === document.getElementById('pic')   // true
// -----------------------------------------

```

# ParentNode 接口

> - 如果当前节点是父节点，就会混入了（mixin）ParentNode 接口
> - 由于只有元素节点（element）、文档节点（document）和文档片段节点（documentFragment）拥有子节点，因此只有这三类节点会拥有 ParentNode 接口。且多半属性都是处理子元素节点


属性/方法 | 作用
---- | -----
ParentNode.children     | 当前节点的所有元素子节点
ParentNode.firstElementChild | 当前节点的第一个元素子节点
ParentNode.lastElementChild  | 当前节点的最后一个元素子节点
ParentNode.childElementCount | 当前节点下所有元素子节点的数目
ParentNode.append()     | 给当前节点在最后一个元素子节点后添加一个子节点
ParentNode.prepend()    | 给当前节点在第一个元素子节点的前边添加一个子节点



```js
// ParentNode.children
// -----------------------------------------
// children 属性返回一个 HTMLCollection 实例
// 成员是当前节点的所有元素子节点。该属性只读

// 遍历 body 元素节点下的所有元素子节点
var bdchild = document.body.children
for (var i = 0; i < bdchild.length; i++) {
    console.log(bdchild[i])
}
// -----------------------------------------


// ParentNode.firstElementChild
// -----------------------------------------
// firstElementChild 属性返回当前节点的第一个元素子节点
// 如果没有任何元素子节点，则返回 null
document.firstElementChild.nodeName     // "HTML"  返回大写的元素标签
// -----------------------------------------


// ParentNode.lastElementChild
// -----------------------------------------
// lastElementChild 属性返回当前节点的最后一个元素子节点
// 如果不存在任何元素子节点，则返回 null
document.body.lastElementChild.nodeName  // "DIV"
// -----------------------------------------


// ParentNode.childElementCount
// -----------------------------------------
// childElementCount 属性返回一个整数，
// 表示当前节点的所有元素子节点的数目
// 如果不包含任何元素子节点，则返回 0
document.body.childElementCount     // 11
// -----------------------------------------


// ParentNode.append()
// ParentNode.prepend()
// -----------------------------------------
// append() 方法为当前节点追加一个或多个子节点，
// 位置是最后一个元素子节点的后面（prepend 是第一个元素的前边）
// 该方法不仅可以添加元素子节点，还可以添加文本子节点
// 该方法没有返回值，可对比 Node.prototype.appendChild()

var parent = document.body;

// 添加元素子节点
var p = document.createElement('p');
parent.append(p);

// 添加文本子节点
parent.append('Hello');

// 添加多个元素子节点
var p1 = document.createElement('p');
var p2 = document.createElement('p');
parent.append(p1, p2);

// 添加元素子节点和文本子节点
var p = document.createElement('p');
parent.append('Hello', p);
// -----------------------------------------
```

# ChildNode 接口

> 如果一个节点有父节点，那么该节点就拥有了 ChildNode 接口


属性/方法 | 作用
---- | -----
ChildNode.remove()  | 从父节点移除当前节点
ChildNode.before()  | 在当前节点的前面，插入一个或多个同级节点
ChildNode.after()   | 在当前节点的后面，插入一个或多个同级节点
ChildNode.replaceWith() | 使用参数节点，替换当前节点


```js
// ChildNode.remove()
// -----------------------------------------
// remove() 方法用于从父节点移除当前节点
document.body.remove()    // body 有父节点，直接移除 body 节点
// -----------------------------------------


// ChildNode.before()
// ChildNode.after()
// -----------------------------------------
// before() 方法用于在当前节点的前面，插入一个或多个同级节点
// after() 方法用于在当前节点的后面，插入一个或多个同级节点
// 新节点与当前节点拥有相同的父节点
// 该方法不仅可以插入元素节点，还可以插入文本节点
var p = document.createElement('p');
var p1 = document.createElement('p');

// 插入元素节点
el.before(p);

// 插入文本节点
el.before('Hello');

// 插入多个元素节点
el.before(p, p1);

// 插入元素节点和文本节点
el.before(p, 'Hello');
// -----------------------------------------


// ChildNode.replaceWith()
// -----------------------------------------
// replaceWith() 方法使用参数节点，替换当前节点
// 参数可以是元素节点，也可以是文本节点
var span = document.createElement('span');
el.replaceWith(span);   // 将 el 节点用 span 节点替换
// -----------------------------------------
```

# Document 节点

> document 节点对象代表整个文档，每张网页都有自己的 document 对象

## 属性

### 快捷方式属性

> **指向**文档内部的某个节点的快捷方式

属性 | 作用
---- | -----
document.defaultView        | document 对象所属的 window 对象
document.doctype            | `<DOCTYPE>`节点
document.documentElement    | 当前文档的根元素节点
document.body，document.head| 指向`<body>`节点与`<head>`节点
document.scrollingElement   | 文档的滚动元素
document.activeElement      | 当前焦点（focus）的 DOM 元素
document.fullscreenElement  | 当前以全屏状态展示的 DOM 元素


```js
// document.defaultView
// -----------------------------------------
// document.defaultView 属性指向 document 对象所属的 window 对象
// 如果当前文档不属于 window 对象，该属性返回 null
document.defaultView === window // true
// -----------------------------------------


// document.doctype
// -----------------------------------------
// 对于 HTML 文档来说，document 对象一般有两个子节点
// 第一个子节点是 document.doctype，指向 <DOCTYPE> 节点
// 即文档类型（Document Type Declaration，简写 DTD）节点
document.doctype      // "<!DOCTYPE html>"
document.doctype.name // "html"
// -----------------------------------------


// document.documentElement
// -----------------------------------------
// document.documentElement 属性指向当前文档的根元素节点（root）
document.documentElement    // 返回整个 html 节点
// -----------------------------------------


// document.body
// document.head
// -----------------------------------------
// document.body 属性指向 <body> 节点
// document.head 属性指向 <head> 节点
// 这两个属性总是存在的，如果网页源码里面省略了 <head> 或 <body>，浏览器会自动创建
// 这两个属性是可写的，如果改写它们的值，相当于移除所有子节点
// -----------------------------------------


// document.scrollingElement
// -----------------------------------------
// document.scrollingElement 属性返回文档的滚动元素
// 当文档整体滚动时，到底是哪个元素在滚动
// -----------------------------------------


// document.activeElement
// -----------------------------------------
// document.activeElement 属性返回获得当前焦点（focus）的 DOM 元素
// 通常，这个属性返回的是 <input>、<textarea>、<select> 等表单元素
// 如果当前没有焦点元素，返回 <body> 元素或 null
// -----------------------------------------


// document.fullscreenElement
// -----------------------------------------
// document.fullscreenElement 属性返回当前以全屏状态展示的 DOM 元素
// 如果不是全屏状态，该属性返回 null
// -----------------------------------------
```

### 节点集合属性

> - 返回一个 **HTMLCollection 实例**，表示文档内部特定的**一类元素**集合,如 links、form、images、embeds、scripts
> - 这些集合都是动态的，原节点有任何变化，立刻会反映在集合中

属性 | 作用
---- | -----
document.links                    | 所有设定了href属性的`<a>`及`<area>`节点
document.forms                    | 所有`<form>`表单节点
document.images                   | 所有`<img>`图片节点
document.embeds，document.plugins | 所有`<embed>`节点
document.scripts                  | 所有`<script>`节点
document.styleSheets              | 文档内嵌或引入的样式表集合


```js
document.links instanceof HTMLCollection   // true
document.images instanceof HTMLCollection  // true
document.forms instanceof HTMLCollection   // true
document.embeds instanceof HTMLCollection  // true
document.scripts instanceof HTMLCollection // true
// document 的这些属性都是包含对应节点的对象
// 这些对象都是 HTMLCollection 构造函数的实例


// document.links
// -----------------------------------------
// document.links 属性返回当前文档所有设定了 href 属性的 <a> 及 <area> 节点
// -----------------------------------------


// document.forms
// -----------------------------------------
// document.forms 属性返回所有 <form> 表单节点
// 除了使用方括号，id 属性和 name 属性也可以用来引用表单
// HTML 代码如下
//   <form name="foo" id="bar"></form>

document.forms[0] === document.forms.foo   // true
document.forms.bar === document.forms.foo  // true
// -----------------------------------------


// document.images
// -----------------------------------------
// document.images 属性返回所有<form>表单节点
// -----------------------------------------


// document.embeds
// document.plugins
// -----------------------------------------
// document.embeds 属性和 document.plugins 属性，都返回所有 <embed> 节点
// -----------------------------------------


// document.scripts
// -----------------------------------------
// document.scripts 属性返回所有 <script> 节点
// -----------------------------------------
```

### 文档静态信息属性

> 返回文档信息

属性 | 作用
---- | -----
document.documentURI，document.URL  | 当前文档的网址
document.domain                     | 当前文档的域名
document.location                   | 提供 URL 相关的信息和操作方法
document.lastModified               | 当前文档最后修改的时间
document.title                      | 当前文档的标题
document.characterSet               | 当前文档的编码
document.referrer                   | 访问者从哪来
document.dir                        | 文字方向


```js
// document.documentURI
// document.URL
// -----------------------------------------
// document.documentURI 属性和 document.URL 属性都返回一个字符串，表示当前文档的网址
// documentURI 继承自 Document 接口，可用于所有文档
// URL 继承自 HTMLDocument 接口，只能用于 HTML 文档
// -----------------------------------------


// document.domain
// -----------------------------------------
// document.domain 属性返回当前文档的域名，不包含协议和端口
// -----------------------------------------


// document.location
// -----------------------------------------
// Location 对象是浏览器提供的原生对象，提供 URL 相关的信息和操作方法
// 通过 window.location 和 document.location 属性，可以拿到这个对象
// -----------------------------------------


// document.lastModified
// -----------------------------------------
// document.lastModified 属性返回一个字符串，表示当前文档最后修改的时间
// 不同浏览器的返回值，日期格式是不一样的
// -----------------------------------------


// document.title
// -----------------------------------------
// document.title 属性返回当前文档的标题
// 默认情况下，返回 <title> 节点的值
// 该属性是可写的，一旦被修改，就返回修改后的值
// -----------------------------------------


// document.characterSet
// -----------------------------------------
// document.characterSet 属性返回当前文档的编码
document.characterSet   // "UTF-8"
// -----------------------------------------


// document.referrer
// -----------------------------------------
// document.referrer 属性返回一个字符串，表示当前文档的访问者来自哪里
// 如果无法获取来源，或者用户直接键入网址而不是从其他网页点击进入，
// document.referrer 返回一个空字符串
document.referrer
// "https://wangdoc.com/javascript/dom/parentnode.html"
// 此处表明我刚才看完 parentNode 接口，然后进入当前文档页面
// -----------------------------------------


// document.dir
// -----------------------------------------
// document.dir 返回一个字符串，表示文字方向
// 此属性只有两个可能的值：
//   rtl 表示文字从右到左，阿拉伯文是这种方式；
//   ltr 表示文字从左到右，包括英语和汉语在内的大多数文字采用这种方式
// -----------------------------------------


// document.compatMode
// -----------------------------------------
// compatMode 属性返回浏览器处理文档的模式，
// 可能的值为 BackCompat（向后兼容模式）和 CSS1Compat（严格模式）
// -----------------------------------------
```

### 文档状态属性


属性 | 作用
---- | -----
document.hidden          | 当前页面是否可见
document.visibilityState | 返回文档的可见状态 visible/hidden/prerender/unloaded
document.readyState      | 当前文档的状态 loading/interactive/complete


```js
// document.hidden
// -----------------------------------------
// document.hidden 属性返回一个布尔值，表示当前页面是否可见
// 窗口正常打开
document.hidden     // false
// 如果窗口最小化、浏览器切换了 Tab
document.hidden     // true
// -----------------------------------------


// document.visibilityState
// -----------------------------------------
// document.visibilityState 返回文档的可见状态
// 四种状态：
// visible：页面可见。页面也可能是部分可见，即不是焦点窗口，前面被其他窗口部分挡住了。
// hidden：页面不可见，有可能窗口最小化，或者浏览器切换到了另一个 Tab。
// prerender：页面处于正在渲染状态，对于用户来说，该页面不可见。
// unloaded：页面从内存里面卸载了
// -----------------------------------------


// document.readyState
// -----------------------------------------
// document.readyState 属性返回当前文档的状态
// 三种状态：
// loading：加载 HTML 代码阶段（尚未完成解析）
// interactive：加载外部资源阶段
// complete：加载完成

// readyState 状态在整个页面加载过程中的变化：
// 1.浏览器开始解析 HTML 文档，document.readyState 属性为 loading
// 2.浏览器遇到 HTML 文档中的 <script> 元素，并且没有 async 或 defer 属性，
//   就暂停解析，开始执行脚本，这时document.readyState属性还是等于 loading
// 3.HTML 文档解析完成，document.readyState属性变成interactive
// 4.浏览器等待图片、样式表、字体文件等外部资源加载完成，
//   一旦全部加载完成，document.readyState 属性变成 complete
// -----------------------------------------
```

### 其他 

属性 | 作用
---- | -----
document.cookie         | 
document.designMode     | 控制当前文档是否可编辑
document.currentScript  | 在`<script>`中，返回当前脚本所在的 DOM 节点
document.implementation | 创建独立于当前文档的新的 Document 对象


```js
// document.cookie
// -----------------------------------------
// document.cookie 属性用来操作浏览器 Cookie
// -----------------------------------------


// document.designMode
// -----------------------------------------
// document.designMode 属性控制当前文档是否可编辑
// 该属性只有两个值 on 和 off，默认值为 off
// -----------------------------------------


// document.currentScript
// -----------------------------------------
// document.currentScript 属性只用在 <script> 元素的内嵌脚本或加载的外部脚本之中，
// 返回当前脚本所在的那个 DOM 节点，即 <script> 元素的 DOM 节点
`<script id="foo">
  console.log(
    document.currentScript === document.getElementById('foo')
  );    // true
</script>`
// -----------------------------------------


// document.implementation
// -----------------------------------------
// document.implementation 属性返回一个 DOMImplementation 对象
// 该对象有三个方法，主要用于创建独立于当前文档的新的 Document 对象
DOMImplementation.createDocument()        // 创建一个 XML 文档
DOMImplementation.createHTMLDocument()    // 创建一个 HTML 文档
DOMImplementation.createDocumentType()    // 创建一个 DocumentType 对象

// 例：创建一个 HTML 文档
var doc = document.implementation.createHTMLDocument('Title');  // 生成一个新的 HTML 文档 doc
var p = doc.createElement('p');   // 创建一个元素标签 p
p.innerHTML = 'hello world';      // p 标签里插入字符串
doc.body.appendChild(p);          // 将 p 添加到 doc 文档的 body 元素的子元素的最后一个

document.replaceChild(            // 用新创建的 doc 替换掉旧的 html 文档的元素
  doc.documentElement,
  document.documentElement
);
// -----------------------------------------
```

## 方法

方法 | 作用
---- | -----
document.open()，document.close()   | 清除当前文档所有内容，使文档处于可写状态/关闭打开的文档
document.write()，document.writeln()| 向当前文档写入内容
document.querySelector()，document.querySelectorAll() | 匹配符和参数指定的 css 选择器的元素节点
document.getElementsByTagName()     | 匹配符和参数指定的 html 标签元素
document.getElementsByClassName()   | 匹配符和参数指定的 css 类选择器的元素
document.getElementsByName()        | 匹配符和参数指定的 name 属性的元素
document.getElementById()           | 匹配指定 id 属性的元素节点
document.elementFromPoint()，document.elementsFromPoint() | 返回位于页面指定位置最上层的元素节点
document.createElement()            | 生成元素节点，并返回该节点
document.createTextNode()           | 生成文本节点，并返回该节点
document.createAttribute()          | 生成一个新的属性节点，并返回该节点
document.createComment()            | 生成一个新的注释节点，并返回该节点
document.createDocumentFragment()   | 生成一个空的文档片段对象
document.createEvent()              | 生成一个事件对象
document.addEventListener()，document.removeEventListener()，document.dispatchEvent()                     | 添加事件监听函数/移除事件监听函数/触发事件
document.hasFocus()                 | 当前文档之中是否有元素被激活或获得焦点
document.adoptNode()，document.importNode() | 将某个节点及其子节点，从原来的文档或文档片段里移除，归属当前 document 对象，返回插入后的新节点/从原来所在的文档或文档片段里，拷贝某个节点及其子节点，让它们归属当前 document 对象
document.createNodeIterator()       | 返回一个子节点遍历器
document.createTreeWalker()         | 返回一个 DOM 的子树遍历器
document.execCommand()              | 改变内容的样式
document.queryCommandSupported()    | 检测浏览器是否支持 document.execCommand() 的某个命令
document.queryCommandEnabled()      | 检测当前是否可用 document.execCommand() 的某个命令。
document.getSelection()             | 指向 window.getSelection()


```js
// document.open()
// document.close()
// -----------------------------------------
// document.open 方法清除当前文档所有内容，使得文档处于可写状态，
// 供 document.write 方法写入内容
// document.close 方法用来关闭 document.open() 打开的文档
document.open();
document.write('hello world');
document.close();
// -----------------------------------------


// document.write()
// document.writeln()
// -----------------------------------------
// document.write 方法用于向当前文档写入内容
// 1.在网页的首次渲染阶段，只要页面没有关闭写入（即没有执行document.close()），
//   document.write 写入的内容就会追加在已有内容的后面
// 2.如果页面已经解析完成（DOMContentLoaded事件发生之后），再调用 write 方法，
//   它会先调用 open 方法，擦除当前文档所有内容，然后再写入

// document.write 会当作 HTML 代码解析，不会转义。

document.write('<p>hello world</p>');  // <p> 标签解释

// document.writeln 方法与 write 方法完全一致，除了会在输出内容的尾部添加换行符
document.write(1);
document.write(2);
// 12

document.writeln(1);
document.writeln(2);
// 1
// 2
// -----------------------------------------


// document.querySelector()
// document.querySelectorAll()
// -----------------------------------------
// document.querySelector 方法返回匹配指定 css 选择器的元素节点
// 如果有多个节点满足匹配条件，则返回第一个匹配的节点
// 如果没有发现匹配的节点，则返回 null
// 这个方法不仅可以在 document 对象上调用，也可以在任何元素节点上调用
var el1 = document.querySelector('.myclass');
var el2 = document.querySelector('#myParent > [ng-click]');

// document.querySelectorAll 方法返回一个 NodeList 对象，
// 包含所有匹配给定选择器的节点

// 这两个方法的参数，可以是逗号分隔的多个 CSS 选择器，
//  返回匹配其中一个选择器的元素节点，这与 CSS 选择器的规则是一致的。
// 它们不支持 CSS 伪元素的选择器（比如:first-line和:first-letter）
//  和伪类的选择器（比如:link和:visited）

// querySelectorAll 方法的参数是字符串 *，则会返回文档中的所有元素节点
// querySelectorAll 的返回结果不是动态集合，不会实时反映元素节点的变化
// -----------------------------------------


// document.getElementsByTagName()
// -----------------------------------------
// 该方法搜索 HTML 标签名，返回符合条件的元素
// 它的返回值是一个类似数组对象（HTMLCollection实例），实时反映 HTML 文档的变化
// 如果没有任何匹配的元素，就返回一个空集
// 如果传入 *，就可以返回文档中所有 HTML 元素
// 这个方法不仅可以在document对象上调用，也可以在任何元素节点上调用
// -----------------------------------------


// document.getElementsByClassName()
// -----------------------------------------
// 该方法返回匹配指定 class 属性的 HTML 元素集（HTMLCollection实例）
// 元素的变化实时反映在返回结果中
// 这个方法不仅可以在 document 对象上调用，
// 也可以在任何元素节点上调用多个 class，它们之间使用空格分隔
var elements = document.getElementsByClassName('foo bar');  // 类选择器为 foo bar
// -----------------------------------------


// document.getElementsByName()
// -----------------------------------------
// 该方法返回匹配指定 name 属性的 HTML 元素集（NodeList实例）
// （比如<form>、<radio>、<img>、<frame>、<embed>和<object>等），
// 表单为 <form name="x"></form>
var forms = document.getElementsByName('x');
forms[0].tagName // "FORM"
// -----------------------------------------


// document.getElementById()
// -----------------------------------------
// 该方法返回匹配指定 id 属性的元素节点
// 如果没有发现匹配的节点，则返回 null
// 只能在 document 对象上使用，不能在其他元素节点上使用
// 与 document.querySelector 相比
document.getElementById('myElement')
document.querySelector('#myElement')    // 描述为 css 选择器语法
// -----------------------------------------


// document.elementFromPoint()
// document.elementsFromPoint()
// -----------------------------------------
// 该方法返回位于页面指定位置最上层的元素节点
// -----------------------------------------


// document.createElement()
// -----------------------------------------
// 该方法用来生成元素节点，并返回该节点
// 其参数可以是自定义的标签名
var newDiv = document.createElement('div');  // 创建 div 标签
// -----------------------------------------


// document.createTextNode()
// -----------------------------------------
// 该方法用来生成文本节点（Text实例），并返回该节点
// 其参数是文本节点的内容
// 此方法不对单引号和双引号转义，所以不能用来对 HTML 属性赋值
var newDiv = document.createElement('div');  // 创建 div 标签
var newContent = document.createTextNode('Hello');  // 创建新的文本节点
newDiv.appendChild(newContent);    // 将文本节点插入 div 节点
// -----------------------------------------


// document.createAttribute()
// -----------------------------------------
// 该方法生成一个新的属性节点（Attr实例），并返回它
var attribute = document.createAttribute(name);   // 参数 name 是属性的名称
// -----------------------------------------


// document.createComment()
// -----------------------------------------
// 方法生成一个新的注释节点，并返回该节点
var CommentNode = document.createComment(data);  // 参数 data 是注释的内容
// -----------------------------------------


// document.createDocumentFragment()
// -----------------------------------------
// 该方法生成一个空的文档片段对象（DocumentFragment实例）
// DocumentFragment 是一个存在于内存的 DOM 片段，不属于当前文档，
// 常用来生成一段较复杂的 DOM 结构，然后再插入当前文档
// -----------------------------------------


// document.createEvent()
// -----------------------------------------
// 方法生成一个事件对象（Event实例）
// 该对象可以被 element.dispatchEvent 方法使用，触发指定事件
var event = document.createEvent(type);
// 参数是事件类型，比如 UIEvents、MouseEvents、MutationEvents、HTMLEvents
// -----------------------------------------


// document.addEventListener()
// document.removeEventListener()
// document.dispatchEvent()
// -----------------------------------------
// 三个方法的作用分别为：
// 添加事件监听函数
document.addEventListener('click', listener, false);

// 移除事件监听函数
document.removeEventListener('click', listener, false);

// 触发事件
var event = new Event('click');
document.dispatchEvent(event);
// -----------------------------------------


// document.hasFocus()
// -----------------------------------------
// 该方法返回一个布尔值，表示当前文档之中是否有元素被激活或获得焦点
// -----------------------------------------


// document.adoptNode()
// document.importNode()
// -----------------------------------------
方法将某个节点及其子节点，从原来所在的文档或DocumentFragment里面移除，归属当前document对象，返回插入后的新节点。插入的节点对象的ownerDocument属性，会变成当前的document对象，而parentNode属性是null

// -----------------------------------------


// document.createNodeIterator()
// -----------------------------------------
// 该方法返回一个子节点遍历器（NodeFilter实例）
// 第一个参数为所要遍历的根节点，
// 第二个参数为所要遍历的节点类型

//  几种主要的节点类型写法如下：
//  所有节点：NodeFilter.SHOW_ALL
//  元素节点：NodeFilter.SHOW_ELEMENT
//  文本节点：NodeFilter.SHOW_TEXT
//  评论节点：NodeFilter.SHOW_COMMENT

var nodeIterator = document.createNodeIterator(
  document.body,            // 返回 <body> 元素子节点的遍历器
  NodeFilter.SHOW_ELEMENT   // 这里指定为元素节点
);
// -----------------------------------------


// document.createTreeWalker()
// -----------------------------------------
// 方法返回一个 DOM 的子树遍历器（TreeWalker实例）
// 第一个参数是所要遍历的根节点，
// 第二个参数指定所要遍历的节点类型

var treeWalker = document.createTreeWalker(
  document.body,          // 遍历 <body> 节点下属的所有元素节点
  NodeFilter.SHOW_ELEMENT
);

var nodeList = [];        // 设定一个空数组

while(treeWalker.nextNode()) {
  nodeList.push(treeWalker.currentNode);   // 将得到的所有元素节点插入上边数组
}
// -----------------------------------------


// document.execCommand()
// -----------------------------------------
// 1.如果 document.designMode 属性设为 on，那么整个文档用户可编辑；
// 2.如果元素的contenteditable属性设为true，那么该元素可编辑。
// 这两种情况下，可以使用该方法改变内容的样式

document.execCommand(command, showDefaultUI, input)
// 该方法接受三个参数：
// command：字符串，表示所要实施的样式。
// showDefaultUI：布尔值，表示是否要使用默认的用户界面，建议总是设为false。
// input：字符串，表示该样式的辅助内容
//        比如生成超级链接时，这个参数就是所要链接的网址
//        如果第二个参数设为 true，浏览器会弹出提示框，要求用户输入该参数
// -----------------------------------------


// document.queryCommandSupported()
// -----------------------------------------
// 该方法用于检测浏览器是否支持 document.execCommand() 的某个命令
// -----------------------------------------


// document.queryCommandEnabled() 
// -----------------------------------------
// 该方法返用于检测当前是否可用 document.execCommand() 的某个命令
// -----------------------------------------
```


# Element 节点

> - Element 节点对象对应网页的 HTML 元素。每一个 HTML 元素，在 DOM 树上都会转化成一个 Element 节点对象
> - Element 对象继承了 Node 接口，因此 Node 的属性和方法在 Element 对象都存在

## 实例属性

### 元素特性的相关属性


属性 | 作用
---- | -----
Element.id          | 指定元素的id属性
Element.tagName     | 指定元素的大写标签名
Element.dir         | 当前元素的文字方向
Element.accessKey   | 用于读写分配给当前元素的快捷键
Element.draggable   | 检测当前元素是否可拖动
Element.lang        | 当前元素的语言设置
Element.tabIndex    | 当前元素在 Tab 键遍历时的顺序
Element.title       | 读写当前元素的 HTML 属性title


### 元素状态的相关属性


属性 | 作用
---- | -----
Element.hidden      | 控制当前元素是否可见
Element.contentEditable，Element.isContentEditable | 元素的内容可以编辑/是否设置前一属性


### 其他


属性 | 作用
---- | -----
Element.attributes  | 当前元素节点的所有属性节点
Element.className   | 读写当前元素节点的 class 属性
Element.classList   | 将当前元素所有的 class 属性放进一个对象
Element.dataset     | 读写自定义的 data- 属性
Element.innerHTML   | 当前元素包含的所有 HTML 代码，设置其内容
Element.outerHTML   | 当前元素节点的所有 HTML 代码，可读写
Element.clientHeight，Element.clientWidth | 设置块级元素本身+padding高度、宽度
Element.clientLeft，Element.clientTop     | 元素节点左边框（left border）的宽度/元素顶部边框的宽度
Element.scrollHeight，Element.scrollWidth | 当前元素的总高度，包括 padding/当前元素的总宽度
Element.scrollLeft，Element.scrollTop     | 前元素的水平滚动条向右侧滚动/垂直滚动条向下滚动的像素数量
Element.offsetParent| 返回最靠近当前元素的、并且 CSS 的position不为static的上层元素
Element.offsetHeight，Element.offsetWidth | 元素的 CSS 垂直高度（本身的高度、padding 和 border）/水平
Element.offsetLeft，Element.offsetTop     | 相对于父节点的位移
Element.style       | 读写该元素的行内样式信息
Element.children，Element.childElementCount | 当前元素节点的所有子元素节点/个数
Element.firstElementChild，Element.lastElementChild | 当前元素的第一个/最后一个元素子节点
Element.nextElementSibling，Element.previousElementSibling | 当前元素节点的后/前一个同级元素节点


## 实例方法

### 属性相关方法


方法 | 作用
---- | -----
Element.getAttribute()      | 读取某个属性的值
Element.getAttributeNames() | 返回当前元素的所有属性名
Element.setAttribute()      | 写入属性值
Element.hasAttribute()      | 某个属性是否存在
Element.hasAttributes()     | 当前元素是否有属性
Element.removeAttribute()   | 删除属性


```js
// Element.getAttribute()
// -----------------------------------------
// 返回当前元素节点的指定属性。如果指定属性不存在，则返回 null

// <div id="div1" align="left">
var div = document.getElementById('div1');
div.getAttribute('align') // "left"     得到 align 属性的值
// -----------------------------------------


// Element.getAttributeNames()
// -----------------------------------------
// 返回一个数组，成员是当前元素的所有属性的名字
// 如果当前元素没有任何属性，则返回一个空数组

// 遍历 mydiv 节点的所有属性
var mydiv = document.getElementById('mydiv');

mydiv.getAttributeNames().forEach(function (key) {
  var value = mydiv.getAttribute(key);
  console.log(key, value);
})
// -----------------------------------------


// Element.setAttribute()
// -----------------------------------------
// 用于为当前元素节点新增属性。如果同名属性已存在，则相当于编辑已存在的属性。该方法没有返回值

// <button>Hello World</button>
var b = document.querySelector('button');   // 属性值总是字符串
b.setAttribute('name', 'myButton');     // name 属性设置为 myButton
b.setAttribute('disabled', true);       // disabled 属性设置为 true
// -----------------------------------------


// Element.hasAttribute()
// -----------------------------------------
// 返回一个布尔值，表示当前元素节点是否包含指定属性

// 检查div节点是否含有align属性。如果有，则设置为居中对齐
var d = document.getElementById('div1');

if (d.hasAttribute('align')) {
  d.setAttribute('align', 'center');
}
// -----------------------------------------


// Element.hasAttributes()
// -----------------------------------------
// 返回一个布尔值，表示当前元素是否有属性
// 如果没有任何属性，就返回 false，否则返回 true

var foo = document.getElementById('foo');
foo.hasAttributes() // true
// -----------------------------------------


// Element.removeAttribute()
// -----------------------------------------
// 移除指定属性。该方法没有返回值

document.getElementById('div1').removeAttribute('align');
// -----------------------------------------
```

### 其他


方法 | 作用
---- | -----
Element.querySelector()         | 继承自 document
Element.querySelectorAll()      | 继承自 document
Element.getElementsByClassName()| 继承自 document
Element.getElementsByTagName()  | 继承自 document
Element.closest()               | 匹配参数指定的css选择器对应最接近当前节点的一个祖先节点
Element.matches()               | 检测当前元素是否匹配给定的 CSS 选择器
Element.addEventListener()      | 继承自 document
Element.removeEventListener()   | 继承自 document
Element.dispatchEvent()         | 继承自 document
Element.scrollIntoView()        | 滚动当前元素，进入浏览器的可见区域
Element.getBoundingClientRect() | 提供当前元素节点的 css 盒装模型信息
Element.getClientRects()        | 当前元素在页面上形成的所有矩形
Element.insertAdjacentElement() | 相对于当前元素的指定位置，插入一个新的节点
Element.insertAdjacentHTML()    | 将一个 HTML 字符串，解析生成 DOM 结构，插入相对于当前节点的指定位置
Element.insertAdjacentText()    | 相对于当前节点的指定位置，插入一个文本节点
Element.remove()                | 继承自 ChildNode 接口
Element.focus()，Element.blur() | 将当前页面的焦点，转移到指定元素上/将焦点从当前元素移除
Element.click()                 | 在当前元素上模拟一次鼠标点击，相当于触发了click事件


# 对 CSS 的操作

## HTML 元素的 style 属性

### 使用 Attribute 操作 style 属性

> 使用网页元素节点的 getAttribute() 方法、setAttribute() 方法和 removeAttribute() 方法，直接读写或删除网页元素的 style 属性

```js
div.setAttribute(
  'style',
  'background-color:red;' + 'border:1px solid black;'
);

// 等同于
<div style="background-color:red; border:1px solid black;" />
```

### 使用 CSSStyleDeclaration 接口操作元素样式

> CSSStyleDeclaration 接口可以**直接**读写 CSS 的样式属性，但是名字需要改写，比如 background-color 写成 backgroundColor，如果 CSS 属性名是 JavaScript 保留字，则规则名之前需要加上字符串 css，比如 float 写成 cssFloat，采用这种骆驼拼写法
>
> 三个地方存在该接口：元素节点的style属性（Element.style）、CSSStyle实例的style属性、window.getComputedStyle() 的返回值
>
> 该对象的属性值都是**字符串**，设置时必须包括单位，但是不含规则结尾的分号。比如，divStyle.width 不能写为 100，而要写为 100px。另外，Element.style 返回的只是**行内样式**，并不是该元素的全部样式，元素的全部样式要通过 window.getComputedStyle() 得到


```js
var divStyle = document.querySelector('div').style;

divStyle.backgroundColor = 'red';
divStyle.border = '1px solid black';
divStyle.width = '100px';
divStyle.height = '100px';
divStyle.fontSize = '10em';

divStyle.backgroundColor // red
divStyle.border // 1px solid black
divStyle.height // 100px
divStyle.width // 100px
```

#### CSSStyleDeclaration 实例属性

```js
// CSSStyleDeclaration.cssText
// -----------------------------------------
// 用来读写当前规则的所有样式声明文本
// cssText 的属性值不需要改写 CSS 属性名

var divStyle = document.querySelector('div').style;

divStyle.cssText = 'background-color: red;'
  + 'border: 1px solid black;'      // 和 setAttribute() 差不多
  + 'height: 100px;'
  + 'width: 100px;';

// 删除一个元素的所有行内样式，最简便的方法就是设置 cssText 为空字符串
divStyle.cssText = '';
// -----------------------------------------


// CSSStyleDeclaration.length
// -----------------------------------------
// 返回一个整数值，表示当前规则包含多少条样式声明

// <div id="myDiv" style="height: 1px;width: 100%;background-color: #CA1;"></div>
var myDiv = document.getElementById('myDiv');
var divStyle = myDiv.style;
divStyle.length   // 3
// -----------------------------------------


// CSSStyleDeclaration.parentRule
// -----------------------------------------
// 返回当前规则所属的那个样式块（CSSRule 实例）
// 如果不存在所属的样式块，该属性返回 null
// 该属性只读，且只在使用 CSSRule 接口时有意义

var declaration = document.styleSheets[0].rules[0].style;
declaration.parentRule === document.styleSheets[0].rules[0]  // true
// -----------------------------------------
```

#### CSSStyleDeclaration 实例方法

```js
// CSSStyleDeclaration.getPropertyPriority()
// -----------------------------------------
// 接受 CSS 样式的属性名作为参数，返回一个字符串，表示有没有设置 important 优先级
// 如果有就返回 important，否则返回空字符串

// <div id="myDiv" style="margin: 10px!important; color: red;"/>
var style = document.getElementById('myDiv').style;
style.margin // "10px"
style.getPropertyPriority('margin') // "important"   设置了优先级
style.getPropertyPriority('color')  // ""            没有设置
// -----------------------------------------


// CSSStyleDeclaration.getPropertyValue()
// -----------------------------------------
// 接受 CSS 样式属性名作为参数，返回一个字符串，表示该属性的属性值

// <div id="myDiv" style="margin: 10px!important; color: red;"/>
var style = document.getElementById('myDiv').style;
style.margin                     // "10px"  直接调用
style.getPropertyValue("margin") // "10px"

// -----------------------------------------


// CSSStyleDeclaration.item()
// -----------------------------------------
// 接受一个整数值作为参数，返回该位置的 CSS 属性名

// <div id="myDiv" style="color: red; background-color: white;"/>
var style = document.getElementById('myDiv').style;
style.item(0) // "color"
style.item(1) // "background-color"
style[1]      // "background-color"
// -----------------------------------------


// CSSStyleDeclaration.removeProperty()
// -----------------------------------------
// 接受一个属性名作为参数，在 CSS 规则里面移除这个属性，返回这个属性原来的值

// <div id="myDiv" style="color: red; background-color: white;"> </div>
var style = document.getElementById('myDiv').style;
style.removeProperty('color')   // 'red'  意为删除 color，确认删除 color: red

// HTML 代码变为
// <div id="myDiv" style="background-color: white;">
// -----------------------------------------


// CSSStyleDeclaration.setProperty()
// -----------------------------------------
// 用来设置新的 CSS 属性。该方法没有返回值
// 该方法可以接受三个参数:
//  参数1：属性名，该参数是必需的
//  参数2：属性值，该参数可选。如果省略，则参数值默认为空字符串
//  参数3：优先级，该参数可选。如果设置，唯一的合法值是 important，表示 CSS 规则里面的 !important

// <div id="myDiv" style="color: red; background-color: white;"></div>
var style = document.getElementById('myDiv').style;
style.setProperty('border', '1px solid blue');
// -----------------------------------------
```


### 使用 window.getComputedStyle() 

> 行内样式（inline style）具有最高的优先级，改变行内样式，通常会立即反映出来。最终表现出的样式由由各种规则组合形成，除行内样式，还有多种规则设置的地方，window.getComputedStyle 方法，就用来返回浏览器计算后得到的**最终规则**。它接受一个**节点对象**作为参数，返回一个 CSSStyleDeclaration **实例**，包含了指定节点的最终样式信息。

此方法返回一个 CSSStyleDeclaration 实例，需注意几点：
- CSSStyleDeclaration 实例返回的 CSS 值都是绝对单位。比如，长度都是像素单位（返回值包括px后缀），颜色是rgb(#, #, #)或rgba(#, #, #, #)格式。
- CSS 规则的简写形式无效。比如，想读取 margin 属性的值，不能直接读，只能读 marginLeft、marginTop 等属性；再比如，font属性也是不能直接读的，只能读font-size等单个属性。
- 如果读取 CSS 原始的属性名，要用方括号运算符，比如`styleObj['z-index']`；如果读取骆驼拼写法的 CSS 属性名，可以直接读取 `styleObj.zIndex`。
- 该方法返回的 CSSStyleDeclaration 实例的 cssText 属性**无效**，返回undefined。


```js
var div = document.querySelector('div');
var styleObj = window.getComputedStyle(div);
styleObj.backgroundColor    // 此时得到的背景色就是 div 元素最终的背景色

// getComputedStyle 方法还可以接受第二个参数，
// 表示当前元素的伪元素（比如 :before、:after、:first-line、:first-letter 等）


// 获取元素的高度
var elem = document.getElementById('elem-container');
var styleObj = window.getComputedStyle(elem, null)
var height = styleObj.height;
// 等同于
var height = styleObj['height'];
var height = styleObj.getPropertyValue('height');
```

## CSS 对象的两个方法

```js
// CSS.escape()
// -----------------------------------------
// 用于转义 CSS 选择器里面的特殊字符

// <div id="foo#bar">
// 该元素的id属性包含一个#号，该字符在 CSS 选择器里面有特殊含义。
// 不能直接写成 document.querySelector('#foo#bar')，只能写成 document.querySelector('#foo\\#bar')
// 这里必须使用双斜杠的原因是，单引号字符串本身会转义一次斜杠。

// CSS.escape 方法就用来转义那些特殊字符
document.querySelector('#' + CSS.escape('foo#bar'))
// -----------------------------------------


// CSS.supports()
// -----------------------------------------
// 返回一个布尔值，表示当前环境是否支持某一句 CSS 规则

// 参数有两种写法:
// 一种是第一个参数是属性名，第二个参数是属性值；
// 另一种是整个参数就是一行完整的 CSS 语句，其后不可带封号

// 第一种写法
CSS.supports('transform-origin', '5px') // true

// 第二种写法
CSS.supports('display: table-cell')     // true
// -----------------------------------------
```

## CSS 伪元素

> CSS 伪元素是通过 CSS 向 DOM 添加的元素，主要是通过:before和:after选择器生成，然后用content属性指定伪元素的内容
>
> 节点元素的 style 对象无法读写伪元素的样式，这时就要用到 window.getComputedStyle()

```js
// <div id="test">Test content</div>

// #test:before {
//   content: 'Before ';
//   color: #FF0;
// }


var test = document.querySelector('#test');

var result = window.getComputedStyle(test, ':before').content;  // 得到内容
var color = window.getComputedStyle(test, ':before').color;     // 得到 color 样式

// 同样也可以使用 getPropertyValue() 方法获取伪元素的属性
// 因为 getComputedStyle() 返回一个 CSSStyleDeclaration 实例，因而可以使用该实例的方法
var result = window.getComputedStyle(test, ':before').getPropertyValue('content');
var color = window.getComputedStyle(test, ':before').getPropertyValue('color');
```

## StyleSheet 接口

> - StyleSheet 接口代表网页的一张样式表，包括`<link>`元素加载的样式表和`<style>`元素内嵌的样式表
> - document 对象的 styleSheets 属性，可以返回当前页面的所有 StyleSheet 实例（即所有样式表），它是一个类似数组的对象
> - 严格地说，StyleSheet 接口不仅包括网页样式表，还包括 XML 文档的样式表。所以，它有一个子类CSSStyleSheet表示网页的 CSS 样式表。我们在网页里面拿到的样式表实例，实际上是 CSSStyleSheet 的实例。这个子接口继承了 StyleSheet 的所有属性和方法，并且定义了几个自己的属性

```js
// sheet 是 StyleSheet 构造函数的实例
var sheets = document.styleSheets;
var sheet = document.styleSheets[0];
sheet instanceof StyleSheet // true
```

### StyleSheet 实例属性

属性 | 作用
---- | -----
StyleSheet.disabled         | 返回一个布尔值，表示该样式表是否处于禁用状态（只能在脚本中设置）
Stylesheet.href             | 只读，返回样式表的网址；内嵌样式表返回null
StyleSheet.media            | 只读，返回一个类似数组的对象（MediaList实例），包含适用媒介的字符串 screen/print/handheld
StyleSheet.title            | 返回样式表的title属性
StyleSheet.type             | 返回样式表的type属性，通常是 text/css
StyleSheet.parentStyleSheet | 返回包含了当前样式表的那张样式表，如为顶层样式表返回 null
StyleSheet.ownerNode        | 返回 StyleSheet 对象所在的 DOM 节点
CSSStyleSheet.cssRules      | 指向一个类似数组的对象（CSSRuleList实例），包含当前样式表的每一条 CSS 规则
CSSStyleSheet.ownerRule     | 返回一个 CSSRule 实例，代表那行 @import 规则

### StyleSheet 实例方法

方法 | 作用
---- | ----- 
CSSStyleSheet.insertRule() | 用于在当前样式表的插入一个新的 CSS 规则，(CSS 规则的字符串, 插入样式表的位置)
CSSStyleSheet.deleteRule() | 用来在样式表里面移除一条规则，参数是该条规则在 cssRules 对象中的位置。无返回值

## 添加样式表

> 网页添加样式表有两种方式。一种是添加一张内置样式表，即在文档中添加一个`<style>`节点。另一种是添加外部样式表，即在文档中添加一个`<link>`节点，然后将href 属性指向外部样式表的 URL

```js
// 第一种
// 写法一
var style = document.createElement('style');
style.setAttribute('media', 'screen');
style.innerHTML = 'body{color:red}';
document.head.appendChild(style);   
// 在文档头部添加 <style media="screen">body{color:red}</style>

// 用以下方法将刚添加的 style 删除
document.head.querySelectorAll('style').forEach(function f(item, i) {
    if (item.hasAttribute('media')) {
        console.log(item, i);   // item 表示当前节点，i 表示所处的位置
        item.remove();  // item 有父节点 head，所有用 remove 直接删除自己
    }
})


// 写法二
var style = (function () {
  var style = document.createElement('style');
  document.head.appendChild(style);
  return style;
})();
style.sheet.insertRule('.foo{color:red;}', 0);

// 第二种
var linkElm = document.createElement('link');
linkElm.setAttribute('rel', 'stylesheet');
linkElm.setAttribute('type', 'text/css');
linkElm.setAttribute('href', 'reset-min.css');

document.head.appendChild(linkElm);
```

## CSSRuleList 接口

> CSSRuleList 接口是一个类似数组的对象，表示一组 CSS 规则，成员都是 CSSRule 实例。
> 一般是通过StyleSheet.cssRules属性，获取 CSSRuleList 实例

```js
// <style id="myStyle">
//   h1 { color: red; }
//   p { color: blue; }
// </style>
var myStyleSheet = document.getElementById('myStyle').sheet;
var crl = myStyleSheet.cssRules;
crl instanceof CSSRuleList  // true

// CSSRuleList 实例里面，每一条规则（CSSRule 实例）
// 可以通过 rules.item(index) 或者 rules[index] 拿到
// CSS 规则的条数通过 rules.length 拿到

// 添加规则和删除规则不能在 CSSRuleList 实例操作，而要在它的父元素 StyleSheet 实例上
// 通过 StyleSheet.insertRule() 和 StyleSheet.deleteRule() 操作
```

## CSSRule 接口

> - 一条 CSS 规则包括两个部分：CSS 选择器和样式声明
> - 通过 CSSRuleList 接口（StyleSheet.cssRules）获取 CSSRule 实例

```js
// <style id="myStyle">
//   .myClass {
//     color: red;
//     background-color: yellow;
//   }
// </style>
var myStyleSheet = document.getElementById('myStyle').sheet;
var ruleList = myStyleSheet.cssRules;
var rule = ruleList[0];
rule instanceof CSSRule  // true  rule 作为实例
```

### CSSRule 实例的属性


属性 | 作用
---- | -----
CSSRule.cssText           | 返回当前规则的文本
CSSRule.parentStyleSheet  | 返回当前规则所在的样式表对象（StyleSheet 实例）
CSSRule.parentRule        | 返回包含当前规则的父规则，如果不存在父规则（即当前规则是顶层规则），则返回null
CSSRule.type              | 返回一个整数值，表示当前规则的类型（1：普通样式规则（CSSStyleRule 实例）/3：@import规则/4：@media规则（CSSMediaRule 实例）/5：@font-face规则）

## CSSStyleRule 接口

> 如果一条 CSS 规则是普通的样式规则（不含特殊的 CSS 命令），那么除了 CSSRule 接口，它还部署了 CSSStyleRule 接口。

属性 | 作用
---- | -----
CSSStyleRule.selectorText | 返回当前规则的选择器
CSSStyleRule.style        | 返回一个对象（CSSStyleDeclaration 实例），代表当前规则的样式声明


# CSSMediaRule 接口

> 如果一条 CSS 规则是 @media 代码块，那么它除了 CSSRule 接口，还部署了 CSSMediaRule 接口。

属性 | 作用
---- | -----
CSSMediaRule.media         | 返回代表 @media 规则的一个对象（MediaList 实例）
CSSMediaRule.conditionText | 返回 @media 规则的生效条件




# 参考资料

[网道 - JavaScript教程 - DOM](https://wangdoc.com/javascript/dom/index.html)