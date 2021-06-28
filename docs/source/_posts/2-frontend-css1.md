---
title: 前端 | css 学习笔记
date: 2019-5-6
tags: 
- CSS
- Frontend
categories: notes
photos:
    - /blog/img/css.jpg
---
<br>
<!--more-->

CSS全称为“层叠样式表 (Cascading Style Sheets)”，它主要是用于定义HTML内容在浏览器内的显示样式，如文字大小、颜色、字体加粗等。

# 注释

`/*注释文字*/`  css代码内插入注释


span

在`<head>`标签里添加css样式 `<style type="text/css"></style>`

用span{}进行规范

再在`<body>`里对指定文本添加此样式，用`<span>`

# css形式

css样式代码插入的形式：内联式、嵌入式、外部式，优先级为 内联式 > 嵌入式 > 外部式，其中嵌入式>外部式有一个前提：嵌入式css样式的位置一定在外部式的后面，即在`<head>`内，`<link>`的css在`<style>`前。

+ 内联式：css样式表就是把css代码直接写在现有的html标签中

`<p style="color:red">这里文字是红色</p>`

+ 嵌入式：把css样式代码写在<style type="text/css"></style>标签之间

+ 外部式：把css代码写入一个单独的外部文件中，这个css样式文件以".css"为扩展名，再在<head>内使用<link>标签将css样式文件链接到html文件内，rel="stylesheet" type="text/css" 是固定写法，不可修改，如：

`<link href="base.css" rel="stylesheet" type="text/css" />`

# css组成

css样式由 **选择符** 和 **声明** 组成

`p{ color : blue；font-size:12px; }`

+ 选择符：指明网页中要应用样式规则的元素

+ 声明：在英文大括号{}中的即为声明,属性和值之间用英文冒号分隔；当有多条声明时，用英文分号分隔



## 选择器

每一条css样式声明（定义）由两部分组成，形式如下：
选择器{
    样式;
}
在{}之前的部分就是“选择器”，“选择器”指明了{}中的“样式”的作用对象，也就是“样式”作用于网页中的哪些元素，可以同时设定多个选择器，用逗号隔开即可

### 选择器分类

#### 标签选择器

标签选择器其实就是html代码中的标签。如`<html>、<body>、<h1>、<p>、<img>`

#### 类选择器

`.类选器名称{css样式代码;}`

使用合适的标签标记要修饰的内容，标签内添加`class`设定类选择器名称，再设置css样式
```css
.stress{color:red;}

<span class="stress">修饰内容</span>
```

##### 注：
1. 类选择器可以在一个HTML文件中多处使用
```css
<span class="stress">小女孩</span>
<span class="stress">小朋友</span>
```
2. 类选择器可以为一个元素同时设置多个样式
```css
.stress{
    color:red;
}
.bigsize{
    font-size:25px;
}

<span class="stress bigsize">小女孩</span>
```

#### id选择器

`#id选择器名称{css样式代码;}`

使用合适的标签标记要修饰的内容，标签内添加`id`设定id选择器名称，再设置css样式
```css
#stress{color:red;}

<span id="stress">修饰内容</span>
```

##### 注：
1. ID选择器只能在文档中使用一次
```css
<span class="stress">小女孩</span>
```
2. ID选择器一个元素只能设定一个样式id
```css
<span id="stress">小女孩</span>
```

#### 子选择器

`.类选器名称>子选择器名称{css样式代码;}`

大于符号(>),用于选择指定标签元素的第一代子元素.仅第一代（不可绕过span指定p，也不可作用与span内的二三代span）
如下div的第一代子选择器为span，所以会在一代span外边加框。
```css
.stress>span {border: 1px solid red;}

<div class="stress">
    <span>
        <span>修饰内容1</span>
        <p>修饰内容2</p>
    </span>
</div>
```

#### 后代选择器

`.类选器名称 后代选择器名称{css样式代码;}`

后代选择器，用空格隔开，接入指定标签元素下的后代元素。作用于该标签元素下的所有后代元素。
如下div下的span共有两层，那么这两层都会加框
```css
.stress span {border: 1px solid red;}

<div class="stress">
    <span>
        <span>修饰内容1</p>
        <p>修饰内容2</p>
    </span>
</div>
```

#### 通用选择器

`* {css样式代码;}`

通用选择器用一个 * 指定，定义html中的所有标签样式

#### 伪类选择器

`a:hover {css样式代码;}`

伪类选择符允许给html不存在的标签（标签的某种状态）设置样式，比如说我们给html中一个标签元素的鼠标滑过的状态来设置字体颜色.
目前常用的就是a:hover,浏览器对于p:hover等兼容不好

### 选择器优先级

内联样式 > id选择器 > 类选择器 > 标签选择器 > 通配符选择器

### css继承

继承是一种规则，它允许样式不仅应用于某个特定html标签元素，而且应用于其后代。

### css权值

+ 浏览器是根据权值来判断使用哪种css样式的，权值高的就使用哪种css样式
+ 标签的权值为1，类选择符的权值为10，ID选择符的权值最高为100
+ 用!important来为某些样式设置最高权值
```css
p{color:red;}                   /*权值为1*/
p span{color:green;}            /*权值为1+1=2*/
.warning{color:white;}          /*权值为10*/
p span.warning{color:purple;}   /*权值为1+1+10=12*/
#footer .note p{color:yellow;}  /*权值为100+10+1=111*/
p{color:red!important;}         /*最高权值*/
```

## 样式

### 字体

```css
/* 字体样式 */
body{font-family:"Microsoft Yahei";}

/* 字体大小 */
body{font-size:"12px";}

/* 字体粗细 */
body{font-weight:"bold";}

/* 字体样式：normal 正常, italic 斜体, oblique 倾斜 */
body{font-style:"12px";}

/* 字体颜色：英文命令, RGB颜色, 十六进制颜色 */
body{color:red;}
body{color:rgb(133,45,200);}    /*RGB可为0~255的整数*/
body{color:rgb(20%,45%,20%);}   /*RGB可为0~100%的百分数*/
body{color:#00ffff;}

/* 
对字体样式进行简写
至少指定 font-size 和 font-family 属性
缩写时 font-size 与 line-height 中间要加入"/"斜扛 
*/
body{font:italic bold 16px/1.5em "times new roman";}
```

### 文本

```css
/* 文本装饰-线：overline 上方线, underline 下方线, line-through 贯穿线 */
p{text-decoration: line-through;}

/* 首行缩进 */
p{text-indent: 2em;}

/* 行间距 */
p{line-height: 2em;}

/* 字符间距：letter-spacing 文字/字母间距, word-spacing 英文单词间距 */
p{letter-spacing: 50px}

/* 排列方式-居中, 针对于块状元素 
元素为文本、图片等行内元素时，通过给父元素设置来实现水平居中 */
div{text-align: center;}

/* 文字添加阴影效果 */
text-shadow: h-shadow v-shadow blur color;
/* 共四个属性：
h-shadow	必需。水平阴影的位置。允许负值。
v-shadow	必需。垂直阴影的位置。允许负值。
blur	    可选。模糊的距离。
color	    可选。阴影的颜色。 */
```
#### 注：长度值

1. 像素px
像素指的是显示器上的小点（CSS规范中假设“90像素=1英寸”，是一个相对值

2. em
    - 本元素给定字体的 font-size 值，如果元素的 font-size 为 14px ，那么 1em = 14px；如果 font-size 为 18px，那么 1em = 18px
    - 但当给 font-size 设置单位为 em 时，此时计算的标准以其**父元素**的 font-size 为基础
```css
<p>以这个<span>例子</span>为例。</p>

p{font-size:14px}
span{font-size:0.8em;}

/* span 中的字体“例子”字体大小就为 11.2px（14 * 0.8 = 11.2px） */
```
3. 百分比
针对文本，百分比也是建立在字体大小的基础上
```css
p{font-size:12px;line-height:130%}
/* 设置行高（行间距）为字体的130%（12 * 1.3 = 15.6px） */
```

### 块

```css
/* 盒子设置阴影 */
box-shadow: h-shadow v-shadow blur spread color inset;
/* 给div标签设定盒子外阴影 */
box-shadow: 2px 2.5px 7px 4px rgba(14, 13, 13, 0.12); 
/* 共有六个属性：
h-shadow	必需。水平阴影的位置。允许负值。
v-shadow	必需。垂直阴影的位置。允许负值。
blur	    可选。模糊距离。	
spread	    可选。阴影的尺寸。	
color	    可选。阴影的颜色。
inset	    可选。将外部阴影 (outset) 改为内部阴影。 */


/* 水平居中-定宽块状元素 */
/* 设置“左右margin”值为“auto” */
div{ 
margin-left:auto;
margin-right:auto;
}
/* 也可简写 */
div{  
    width:200px;       /*定宽*/
    margin:20px auto;  /* margin-left 与 margin-right 设置为 auto */
}

/* 水平居中-定宽高块状元素 */
/* 利用父元素设置相对定位,子元素设置绝对定位,那么子元素就是相对于父元素定位的特性 */
/* 子元素设置上和左偏移的值都为50%，是元素的左上角在父元素中心点的位置 */
/* margin给上和左都给负的自身宽高的一半 */
 .box {
        border: 1px solid #00ee00;
        height: 300px;
        position: relative;
    }
    .box1 {
        width: 200px;
        height: 200px;
        border: 1px solid red;
        position: absolute;
        top: 50%;
        left: 50%;
        margin: -100px 0 0 100px;
    }

/* 水平居中-不定宽高块状元素 */
/* 利用父元素设置相对定位,子元素设置绝对定位,那么子元素就是相对于父元素定位的特性 */
/* 子元素设置上和左偏移的值都为 50% */
/* translate位移,给 上 和 左 都位移 -50% 距离 */
 .box {
        border: 1px solid #00ee00;
        height: 300px;
        position: relative;
    }
    .box1 {
        border: 1px solid red;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
    }
```

# css盒模型

## 标签元素分类

在CSS中，html中的标签元素大体被分为三种不同的类型：块状元素、内联元素(又叫行内元素)和内联块状元素。

常用的块状元素有：

`<div>、<p>、<h1>...<h6>、<ol>、<ul>、<dl>、<table>、<address>、<blockquote> 、<form>`

常用的内联元素有：

`<a>、<span>、<br>、<i>、<em>、<strong>、<label>、<q>、<var>、<cite>、<code>`

常用的内联块状元素有：

`<img>、<input>`

### 块级元素

- html中`<div>、<p>、<h1>、<form>、<ul>、<li>`就是块级元素。
- 设置 `display:block` 可将元素显示为块级元素。
    - 如将内联元素a转换为块状元素：`a{display:block;}`
- **块状元素**特点:
    - 每个块级元素都从新的一行开始，并且其后的元素也另起一行。（真霸道，一个块级元素独占一行）
    - 元素的高度、宽度、行高以及顶和底边距都可设置。
    - 元素宽度在不设置的情况下，是它本身父容器的100%（和父元素的宽度一致），除非设定一个宽度

### 内联元素

- html中`<span>、<a>、<label>、<strong>、<em>`就是典型的内联元素（行内元素）（inline）元素。
- 设置 `display: inline` 将元素显示为内联元素
    - 如将块状元素div转换为内联元素：`div{display:inline}`
- **内联元素**特点:
    - 和其他元素都在一行上；
    - 元素的高度、宽度及顶部和底部边距不可设置；
    - 元素的宽度就是它包含的文字或图片的宽度，不可改变。

### 内联块状元素

- 内联块状元素 (inline-block) 就是同时具备内联元素、块状元素的特点
- `<img>、<input>`标签就是这种内联块状标签
- 设置 `display:inline-block` 就是将元素设置为内联块状元素
- **内联块状元素**特点:
    - 和其它元素都在一行
    - 元素的高度、宽度、行高以及顶和底边距都可设置

## 盒模型组成

盒模型由本身的 内容 + padding填充 + border边框 + margin边界 组成。
一个元素实际宽度（盒子的宽度）=左边界+左边框+左填充+内容宽度+右填充+右边框+右边界。

### 盒模型的边框

盒子模型的边框就是围绕着内容及补白（padding）的线，这条线你可以设置它的粗细、样式和颜色(边框三个属性) 

```css
/* 
border-width（边框宽度）中的宽度也可以设置为：
thin | medium | thick（但不是很常用），最常还是用像素（px）

border-style（边框样式）常见样式有：
dashed（虚线）| dotted（点线）| solid（实线）

border-color（边框颜色）中的颜色可设置为十六进制颜色，如:
border-color:#888; 
*/
div{
    border-width:2px;
    border-style:solid;
    border-color:red;
}
/* 可以简写为以下格式 */
div{border:2px solid red;}
/* 可以将边框拆开 上右下左 分别设定 */
div{
    border-bottom:1px solid red;
    border-top:1px solid red;
    border-right:2px dashed green; 
    border-left:2px dashed green;
}


/* 边框设置圆角 */
/* 圆角可分为左上、右上、右下、左下 */
div{border-radius: 20px 10px 15px 30px;}
/* 可将圆角拆开四个分别写 */
div{
    border-top-left-radius: 20px;
   border-top-right-radius: 10px;
   border-bottom-right-radius: 15px;
   border-bottom-left-radius: 30px;
}
/* 四个圆角都为10px */
div{ border-radius:10px;}
/* 左上角和右下角圆角效果一样为10px，右上角和左下角圆角一样为20px */
div{ border-radius:10px 20px;}
/* 一个正方形，当设置圆角效果值为元素宽度一半（或50%）时，显示效果为圆形 */
div {
    width: 200px;
    height: 200px;
    border: 5px solid red;
    border-radius: 100px;
}
```

### 盒模型的内边距（填充）

元素内容与边框之间是可以设置距离的，称之为“内边距（填充）”。

```css
/* 填充设定的顺序分为上、右、下、左(顺时针) */
div{padding:20px 10px 15px 30px;}
/* 可将四边拆开分别定义大小 */
div{
   padding-top:20px;
   padding-right:10px;
   padding-bottom:15px;
   padding-left:30px;
}
/* 上、右、下、左的填充都为10px */
div{padding:10px;}
/* 如果上下填充一样为10px，左右一样为20px */
div{padding:10px 20px;}
/* 如果上边界为10px, 下边界为15px, 左右一样为20px */
div{padding:10px 20px 15px;}
```

### 盒模型的外边距（边界）

元素与其它元素之间的距离可以使用边界（margin）来设置。

```css
/* 边界也是可分为上、右、下、左 */
div{margin:20px 10px 15px 30px;}
/* 可将四边拆开分别定义大小 */
div{
   margin-top:20px;
   margin-right:10px;
   margin-bottom:15px;
   margin-left:30px;
}
/* 如果上右下左的边界都为10px */
div{margin:10px;}
/* 如果上下边界一样为10px，左右一样为20px */
div{margin:10px 20px;}
/* 如果上边界为10px, 下边界为15px, 左右一样为20px */
div{margin:10px 20px 15px;}
```

## 弹性盒模型

### flex属性

- 通过 `display: flex` , 可以使本身独占一行的盒模型显示在一排(一条轴上)
- flex需要添加在父元素上，改变子元素的排列顺序
- 默认为从左往右依次排列,且和父元素左边没有间隙

### 各块在flex主轴上的对齐方式

```css
/* justify-content属性可设定各块在横轴上的排列方式 */
/* 先设置 flex 陈列, 再设置justify-content属性 */
div {
display: flex;
justify-content: flex-start | flex-end | center | space-between | space-around;
}

/* justify-content有五个属性：
flex-start：    交叉轴的起点对齐
flex-end：      右对齐
center：        居中
space-between： 两端对齐，项目之间的间隔都相等
space-around：  每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍 */


/* align-items属性可设定各块在纵轴上的排列方式 */
/* 是在多个块在一排的基础上设置纵轴的位置 */
/* 先设置 flex 陈列, 再设置align-items属性 */
div {
display: flex;
align-items: flex-start | flex-end | center | baseline | stretch;
}

/* align-items有五个属性：
flex-start：        默认值，左对齐，占满整个文字外框大小的范围
flex-end：          交叉轴的终点对齐
center：            交叉轴的中点对齐
baseline:           项目的第一行文字的基线对齐
stretch(默认值):    如果项目未设置高度或设为auto，将占满整个父容器的高度。 */


/* 设置flex子元素相对于父元素的占比 */
/* flex属性的值只能是正整数,表示占比多少 */
/* 给子元素设置了flex之后,其宽度属性会失效 */
    .box {
        height: 300px;
        background: blue;
        display: flex;
    }
    .box div {
        width: 200px;
        height: 200px;
    }
    .box1 {
        flex: 1;
        background: red;
    }
    .box2 {
        flex: 1;
        background: orange;
    }
```

# css布局

## 布局分类

CSS3包含3种基本的布局模型：
1. 流动模型（Flow）
2. 浮动模型 (Float)
3. 层模型（Layer）

### 流动模型

- 流动（Flow）是默认的网页布局模式。也就是说网页在默认状态下的 HTML 网页元素都是根据流动模型来分布网页内容的。
- 流动布局模型具有2个比较典型的特征：
    - 块状元素都会在所处的包含元素内自上而下按顺序垂直延伸分布，因为在默认状态下，块状元素的宽度都为100%。实际上，块状元素都会以行的形式占据位置。
    - 在流动模型下，内联元素都会在所处的包含元素内从左到右水平分布显示, 作用于**同一级**的多个元素。

### 浮动模型

- 任何元素在默认情况下是不能浮动的，但可以用 CSS 定义为浮动，如 div、p、table、img 等元素都可以被定义为浮动。
- 设置元素浮动就可以实现让两个块状元素并排显示: `div{float: left}`

### 层模型

- 如何让html元素在网页中精确定位，就像PhotoShop中的图层一样可以对每个图层能够精确定位操作。CSS定义了一组定位（positioning）属性来支持层布局模型。

- 层模型形式：
    - 绝对定位(`position: absolute`)
    - 相对定位(`position: relative`)
    - 固定定位(`position: fixed`)

#### 层模型之绝对定位absolute

- 设置 `position:absolute` (表示绝对定位)，这条语句的作用将元素从文档流中拖出来。
- 同时使用left、right、top、bottom属性**相对于其最接近的一个具有定位属性的父包含块**进行绝对定位。
```css
/* div 相对于其父级左边隔开 100px , 上边相隔 50px */
div{
    width:200px;
    height:200px;
    border:2px red solid;
    position:absolute;
    left:100px;
    top:50px;
}
```
- 如果不存在这样的包含块，则相对于body元素，即相对于浏览器窗口。

#### 层模型之相对定位relative

- 设置position:relative（表示相对定位）
- 通过left、right、top、bottom属性确定元素在正常文档流中的偏移位置。
- 相对定位完成的过程是首先按static(float)方式生成一个元素(并且元素像层一样浮动了起来)，然后相对于以前的位置移动，移动的方向和幅度由left、right、top、bottom属性确定，偏移前的位置保留不动。
- 其他标签依旧按原来的排布进行排布，变的只是设定相对定位的元素标签。

#### 层模型之固定定位fixed

- fixed 定位与 absolute 的区别是 fixed 不是相对于父包含块定位，而是相对于视图（屏幕内的网页窗口）本身
- 固定定位的元素会始终位于浏览器窗口内视图的某个位置，不会受文档流动影响，这与 `background-attachment:fixed;` 属性功能相同。

#### 用法：相对元素定位

- 除相对父包含块/网页窗口定位，也可实现元素相对于其他元素定位：
    - 参照定位的元素必须是相对定位元素的前辈元素
    - 参照定位的元素必须加入 `position:relative;`
    - 定位元素加入 `position:absolute;` ,即可使用top、bottom、left、right定位




