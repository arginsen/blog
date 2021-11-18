---
title: 面试 | HTML & CSS
date: 2020-8-26 17:20:33
tags: 
  - interview
  - Frontend
categories: notes
hide: true
photos:
    - /blog/img/interview.jpg
---


<br>
<!--more-->

# HTML

## html块元素、行内元素有哪些？区别？

- 块元素：`div、h1-6、p、ul、ol、li、hr、dl、dt、dd、nav、section、article、audio、pre`等
- 行内元素：`input、span、img、video、a、i(em)、b(strong)、sub、sup、ins、del、br、textarea、code`等

块元素：
1）独占一行；
2）能定义宽高
3）`margin`、`padding` 上下左右有效
4）能包含块元素和行内元素

行内元素：
1）横向排列
2）不能改变宽高
3）`margin`、`padding` 左右有效，上下无效
4）不能包含其他元素

## src和href的区别是什么

`href` 是Hypertext Reference的简写，表示超文本引用，指向网络资源所在位置。
`src` 是source的简写，目的是要把文件下载到html页面中去。

href 用于在当前文档和引用资源之间确立联系
src 用于替换当前内容

当浏览器遇到href会并行下载资源并且不会停止对当前文档的处理。(同时也是为什么建议使用 link 方式加载 CSS，而不是使用 @import 方式)
当浏览器解析到src ，会暂停其他资源的下载和处理，直到将该资源加载或执行完毕。(这也是script标签为什么放在底部而不是头部的原因)

## script 标签中的 defer 和 async 的区别

浏览器默认会立即加载并执行相应的脚本，在此期间会阻塞后续文档的加载。

defer 和 async 属性都是异步加载外部的 js 脚本，不会阻塞页面的解析；
两者区别：
  1) 执行顺序： 多个带 async 属性的标签不能保证加载的顺序；多个带 defer 属性的标签，会按照顺序执行
  2) 脚本并行执行：async 属性表示后续文档、js 脚本的加载执行是并行的；defer 则是 js 脚本需要等到文档所有元素解析完成后才执行

## iframe 有哪些优缺点

iframe 元素创建包含另外一个文档的内联框架（行内）

优点：
- 用来加载速度较慢的内容（广告）
- 使脚本可以并行下载
- 可以实现跨子域通信

缺点：
- iframe 会阻塞主页面 onload 事件
- 无法被一些搜索引擎识别
- 会产生很多页面，不容易管理

## canvas 和 svg 的区别

1. svg：

svg 是可缩放矢量图形 Scalable Vector Graphics 是基于可扩展标记语言 xml 描述的 2D 图形的语言，svg 基于 xml 所以 svg dom 中的每个元素都是可用的，也就可为某个元素附加 JavaScript 事件处理器。在 svg 中，每个被绘制的图形均被视为对象。如果 svg 对象的属性发生变化，浏览器可以自动重现图形。

- 不依赖分辨率
- 支持事件处理器
- 适合带有大型渲染区域的应用程序（谷歌地图）
- 复杂度高会减慢渲染速度（频繁操作dom）
- 不适合游戏应用


2. canvas：

canvas 是画布，通过 JavaScript 来绘制 2D 图形，是逐像素进行渲染的。其位置发生改变，就会重新进行绘制。

- 依赖分辨率
- 不支持事件处理器
- 弱的文本渲染能力
- 能以 .png/.jpg 格式保存结果图像
- 最适合图像密集型游戏

## 浏览器乱码的原因是什么？如何解决？

原因：

- 网页源代码是 gbk 编码，而内容中的中文字是 utf-8 编码
- html 网页编码是 gbk，而程序从数据库中调出呈现是 utf-8 编码的内容
- 浏览器不能自动检测网页编码

解决方法：

- 使用软件编辑 html 网页内容
- 对于网页设置编码是 gbk ，而数据库传过来的内容是 utf-8，需要在程序查询数据库数据显示数据前进行转码
- 如果浏览器浏览时候出现网页乱码，在浏览器中找到转换编码的菜单进行转换。

## html5 新特性

1. 语义标签 header footer nav section article aside main
2. 增强型表单
input 输入新特性：color date email search
新增表单元素：datalist progress meter keygen output
新增表单属性：placeholder height width 
3. 音频和视频 audio video
4. canvas
5. svg
6. 地理位置 getCurrentPosition()
7. 拖放 API
ondragstart ondrag ondranenter ondragover ondragleave ondragend
8. Web Worker 加载一个脚本文件，进而创建一个在主线程之外独立工作的线程
```js
var worker = new Worker("worker.js"); //创建一个Worker对象并向它传递将在新线程中执行的脚本的URL
worker.postMessage("hello world");     //向worker发送数据
worker.onmessage = function(evt){     //接收worker传过来的数据函数
   console.log(evt.data);              //输出worker发送来的数据
}
```
9. web storage 有 localStorage 和 sessionStorage
10. websocket


# CSS

## CSS3 有哪些新特性

1. RGBA 和透明度(opacity)
2. background-image background-origin background-size background-repeat
3. word-wrap
4. text-shadow
5. font-face
6. border-radius
7. border-image
8. box-shadow
9. @media screen and (min-width: 800px) and (max-width: 1200px)

## 元素不定宽高

> https://vue3js.cn/interview/css/center.html

1. 父级相对定位，子级绝对定位，top left 等都为 0，margin 设置 auto

2. 父级相对定位，子级绝对定位，需要知道子元素宽高，top 和 left 设置为 50%，margin-left 和 margin-right 设置为负的宽高的一半

3. 父级相对定位，子级绝对定位，top 和 left 设置为 50%，再转换：`transform: translate(-50%,-50%)`

4. flex 实现

5. grid 布局，实现同 flex



## 如何隐藏一个元素

```css
/* 隐藏的元素不占任何空间 */
display: none;

/* 透明度 */
opacity: 0;

/* 无交互 */
visibility: hidden;
```

## display:none 与 visibility: hidden 的区别

1) 渲染树中，display:none 会让元素完全从渲染树中消失，渲染时不会占据任何空间；而 visibility: hidden 不会让元素从渲染树中消失，渲染的元素还会占据相应空间，只是内容不可见

2) display:none 是非继承属性，子孙节点会随父节点从渲染树消失；而 visibility: hidden 是继承属性，子孙节点消失是因为继承了该属性

3) 修改常规文档流中元素的 display 属性会造成文档的重排（回流），而修改 visibility 只会造成本元素重绘

4) 使用读屏器对 display:none 的内容不可读取，而设置 visibility: hidden 的内容会被读取

## overflow 的用法

overflow 属性规定当内容溢出元素框时发生的事情。

```
visible	默认值。内容不会被修剪，会呈现在元素框之外。
hidden	内容会被修剪，并且其余内容是不可见的。
scroll	内容会被修剪，但是浏览器会显示滚动条以便查看其余的内容。
auto	  如果内容被修剪，则浏览器会显示滚动条以便查看其余的内容。
inherit	规定应该从父元素继承 overflow 属性的值。
```

## 如何对超出元素边界的文字进行处理

```css
/* 超出部分直接剪掉 */
text-overflow: clip;
overflow: hidden;
white-space: nowrap;

/* 超出部分文本显示为省略号 */
text-overflow: elipsis;
overflow: hidden;
white-space: nowrap;
```

## 清除浮动

1. 使用 clear 属性的空元素 `<br class="clear">`，同时 `.clear {clear: both;}`
2. `overflow: hidden;` 或 `overflow: auto;`
3. 使用 `:after` 伪元素，给浮动元素 class 添加 `clearfix`，给该 class 添加伪元素

## CSS 优化、提高性能

1. 将样式写为单独的 css 文件，通过 link 引用，尽量不用内联，或 `style` 标签
2. 不使用 `@import`
3. 避免使用复杂的选择器，层级越少越好
4. 利用 css 继承减少重复
5. 避免 `!important`

## 什么是响应式设计，基本原理

响应式网站设计是一个网站能够兼容多个终端，而不是为每个终端做一个特定的版本。
基本原理是通过媒体查询检测不同的设备屏幕尺寸做处理

## 有哪些盒模型（字节）

盒子模型分为两种 第一种是W3c标准的盒子模型（标准盒模型） 、第二种IE标准的盒子模型（怪异盒模型）

（IE6.IE7.IE8）的版本触发 IE 盒模型，其他浏览器中会默认为W3c盒模型。

标准盒模型中 width 指的是内容区域content的宽度；height指的是内容区域content的高度。
怪异盒模型中的 width 指的是内容、边框、内边距总的宽度（content + border + padding）；height指的是内容、边框、内边距总的高度。

```css
/* 切换盒模型, css3的box-sizing属性 */
box-sizing: content-box /* 是 W3C 盒子模型 默认 */
box-sizing: border-box /* 是 IE 盒子模型 */
```

## 什么是BFC机制

BFC（Block Formatting Context），块级格式化上下文，是一个独立的渲染区域，让处于 BFC 内部的元素与外部的元素相互隔离，使内外元素的定位不会相互影响。

触发条件：
1. 根元素()
2. 浮动元素（元素的 float 不是 none）
3. 绝对定位元素（元素的 position 为 absolute 或 fixed）
4. 行内块元素（元素的 display 为 inline-block）
5. 表格单元格（元素的 display为 table-cell，HTML表格单元格默认为该值）
6. 表格标题（元素的 display 为 table-caption，HTML表格标题默认为该值）
7. 匿名表格单元格元素（元素的 display为 table、table-row、 table-row-group、table-header-group、table-footer-group（分别是HTML table、row、tbody、thead、tfoot的默认属性）或 inline-table）
8. overflow 值不为 visible 的块元素 -弹性元素（display为 flex 或 inline-flex元素的直接子元素）
9. 网格元素（display为 grid 或 inline-grid 元素的直接子元素） 等等

BFC 布局规则：
1. 浮动的元素会被父级计算高度
2. 非浮动元素不会覆盖浮动元素的位置（非浮动元素触发了 BFC）
3. margin 不会传递给父级（父级触发 BFC）
4. 属于同一个 BFC 的两个相邻元素上下 margin 会重叠
5. 普通文档流布局：浮动的元素是不会被父级计算高度
6. 非浮动元素会覆盖浮动元素的位置
7. margin 会传递给父级元素
8. 两个相邻元素上下的 margin 会重叠

开发中的应用：
1. 阻止 margin 重叠
2. 可以包含浮动元素 —— 清除内部浮动（清除浮动的原理是两个 div 都位于同一个 BFC 区域之中）
3. 自适应两栏布局
4. 可以阻止元素被浮动元素覆盖

## 伪元素和伪类的区别和作用？

伪元素：在内容元素的前后插入额外的元素或样式，但是这些元素实际上并不在文档中生成。它们只在外部显示可见，但不会在文档代码中找到它们

伪类：将特殊的效果添加到特定选择器上，它是对已有的元素添加类别，不会产生新元素

## 如何画一条 1px 的线

