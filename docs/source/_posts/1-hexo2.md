---
title: hexo博客 | Hexo主题细节配置
date: 2019-3-27
categories: practice
tags:
 - hexo
photo: 
 - /blog/img/hexo.jpg
---

## 前言 

在建站的过程中，可谓是碰到了很多坑，但是作为一个初学者，这些坑也是能让我更好的消化所接触的知识，理解一些基础的东西。所以说实战还是养人，实际去做，提出问题，后边系统再学时就会有一种很好的代入感。
<!-- more -->
其实在初次尝试中，即便跟着教程走，也不一定能解决遇到的问题，那么这时就要多思考，多尝试。教程是死的，而人是活的。



### 更改代码颜色

代码颜色属于css里，所以我们首先要想到在主题的css里修改参数。

定位到*themes\主题名\source\css*，此外了解到所选主题不同，文件都可能是对不上的。而我这里是在*css\_partial\articles.styl*文件内修改。

**思路**

在浏览器中先打开本地网页，在代码区右键游览器检查元素

![2](/blog/img/2019/hexo-set/2.png)

可以看到右侧style栏的网页选中区的css样式，显示article，因此我们打开articles.styl

![3](/blog/img/2019/hexo-set/3.png)
输入自定义样式(附我的)：
```css
.articles
  .article-entry pre, .article-entry .highlight
    background dark
    padding 1rem
    color shenju           /*字体颜色，可参考https://www.114la.com/other/rgb.htm*/
  .article-entry code
    padding 1rem
    background dark       /*背景颜色*/
```

### 更改链接显示字体颜色

对于css文件，找到对页面链接进行配置的文件，

```css
/* Links */
a {
  color: #999;
}

a:hover {
  color: #f33;
}
```
而styl文件，

```styl
a {
  color: #1fa0ae;
  &:hover {
    color #333
	}
}
```
### 对主页标题&描述进行配置

```css
/* HEADER   #位于fashion.css文件夹下，对主页的标题&描述进行配置
----------------------------------------------- */
.site-header {
	padding: 40px 0 20px;
	}
.site-branding {
	float: left;
	width: 100%;
	}
.site-title {
	margin: 0;
	font-size: 48px;             /*定义字体大小*/
	font-family: Oswald, "Microsoft Yahei", Arial, Helvetica, sans-serif;
	text-transform: uppercase;   /*英文转为大写字母*/
	line-height: 1.2em;          /*表示行高*/
	}
.site-title a {
	color: #333;
	text-decoration: none;
	}
.site-description {
	margin-bottom: 20px;
	color: #a6a6a6;             /*字体颜色*/
	font-weight: 300;
	font-family: Oswald, "Microsoft Yahei", Arial, Helvetica, sans-serif;
	font-size: 14px;
	letter-spacing: 2px;        /*字体间距*/
	}

```
<br>