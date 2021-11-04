---
title: hexo博客 | substring定义路径读取与配置目录可滚动
date: 2020-4-10
categories: practice
tags:
 - hexo
photo: 
 - /blog/img/hexo.jpg
---

<br>

<!-- more -->

# substring

>在 hipaper 主题时，文章的封面图读取的是在 post 页 photo 填写的地址，我是 `/blog/img/xxx.jpg` ，而 next 主题下，会在 `url_for` 前边自动添加根目录，也就是我的 `/blog/` ，那么此时为了不改变我的文章页 photo 的相对路径，使两个主题都可读取到图片，那么就需要用到 substring()

- substring() 方法用于提取字符串中介于两个指定下标之间的字符。

```js
stringObject.substring(start,stop)
// start 必需。一个非负的整数，规定要提取的子串的第一个字符在 stringObject 中的位置。
// stop 可选。一个非负的整数，比要提取的子串的最后一个字符在 stringObject 中的位置多 1。
// 如果省略stop参数，那么返回的子串会一直到字符串的结尾。

// next 主题的 post.swig 配置：
{% for photo in post.photos %}
    {% if loop.index0 % COLUMN_NUMBER === 0 %}<div class="post-gallery-row">{% endif %}
        // 文章页指定的图片路径适用于hipaper主题，next主题获取路径时多了blog
        // 因此需要读取'/blog'后的路径，采用 JavaScript substring() 方法
        // substring(start, stop) 读取路径指定开始的位置，stop可选，不填写读取到字符串结束
        <img src="{{ url_for(photo).substring(5) }}" itemprop="contentUrl"/>
    {% if loop.index0 % COLUMN_NUMBER === 2 %}</div>{% endif %}
{% endfor %}
```

# 设置目录可滚动

>当前主题的设定是目录所在的div是固定的 (`position: fixed`) , 且 `overflow: hidden` , 因此对于篇幅较大的文章，这时涉及的标题就会很多，生成的目录较多，因此初始的目录配置在页面中容纳不下，且到下方会穿模。

```css
/* 改变以下对于目录 div 的 toc 选择器的几个参数, 增加滑条和滑块的样式 */
/* 修改文件路径 hipaper/source/css/_partial/article.styl */
/* 原有 #toc 修改以下样式 */
/*  搜索，修改#toc参数，参数都缩进两位 */
#toc
  height: 87%

/*  #toc 后增加以下两个样式, 用于修改滑块、滑条样式 */
/* 也可用单独的 ::-webkit-scrollbar 改变滑块、滑条样式 */
#toc::-webkit-scrollbar
  background-color: #ece9e9
  -webkit-border-radius: 2em
  -moz-border-radius: 2em
  border-radius: 2px
  width: 5px

#toc::-webkit-scrollbar-thumb
  border-radius: 4px
  background: rgba(0,0,0,0.3)
  -webkit-box-shadow: inset 0 0 6px RGB(243, 243, 244)
```
