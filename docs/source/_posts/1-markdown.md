---
title: Markdown 使用指南
tags: 
- markdown
categories: notes
date: 2019-3-25
photos:
    - /blog/img/markdown.jpg
---

<br>
<!-- more -->

## Markdown软件推荐(win)


>在写一些博客、笔记时还是独立的软件在切换时会方便一些，像微信公众平台在网页编辑还是相对麻烦一点。当然，对于所思即所言的人来说，网页版也还好。


1. Markpad

来自微软家的markdown编辑器，主题简约(只有默认的...), 操作还算方便，可以直接将图片粘贴到md文档。

![](http://img.xiumi.us/xmi/ua/1O3nK/i/0928d1d8add88b32c1dc41b89e3a3176-sz_86178.png)

**下载地址**：http://code52.org/DownmarkerWPF/

2. Yu Writer

这一款还在不断测试，但其界面相对其他应该是相当美观了。最新版设了诸多限制，推荐老版。

从网页复制过来的内容会自动转换为 Markdown 格式，支持多种缩放级别的即时预览，还可拖入图片。

有四种主题供选择，页面精简舒适。除即时预览(Instant View)外，还提供成品预览(Read Mode)。 

文档热保存，随时随地打开关闭，不担心内容丢失；自动还原上次工作现场，方便继续书写。

可导出pdf、html、md、image等格式，选择页面大小。 

好了就吹这么多吧.... 


![](http://img.xiumi.us/xmi/ua/1O3nK/i/8743047f97626f79a0a9421c5d7b425f-sz_204674.png)

*Read Mode:*

![](http://img.xiumi.us/xmi/ua/1O3nK/i/cee2d84b05d3614933b3f4bb61ae01fc-sz_132291.png)

**下载地址**：https://jp.archboy.org/yu-writer/yu-writer-beta-0.4.4-windows.zip


3. csdn在线编辑器

在csdn个人博客管理处，方便于用户编辑博客。但实际上功能完整，布局美观，有些功能，样式是其他软件没有的，而且即时预览及其方便，书写使可直接借助上边工具栏。

当然应用选取方面看个人习惯、需求。此款不失为一个好选择。

![](http://img.xiumi.us/xmi/ua/1O3nK/i/39d178b5877c49b4811457587117ac1d-sz_187740.png)

**网址**：https://mp.csdn.net/

4. Visual Studio Code（好吧这个最好用了）

VS Code是一款免费开源的现代化轻量级代码编辑器，而用来写markdown只是很小的一个应用，它的功能及其强大，界面整洁，功能强大，主题、扩展插件丰富。

![](http://img.xiumi.us/xmi/ua/1O3nK/i/d38b9405d1d46ef9c897c303393f173d-sz_278699.png)

下载地址：https://code.visualstudio.com/

## Markdown使用操作

>markdown的编辑各平台大致相同，这里讲下Yu Writer 4.0.4beta版的操作

`=`

文章名可以直接输入，再另起一行输入 `===`

`#`
文内有六级标题，`#` 代表一级标题(注意#后要空格，与文章名同级),依此类推

`[[toc]]`

在由#号构建的文章框架下，在文章标题下输入 `[[toc]]` 自动按标题等级生成目录(注意这里与其他平台不同，csdn是 `@[toc]()`

`**`

文本两端各加两个星号；加粗文本

`*`

文本两端各加一个星号，再外层两端都要有一个空格；斜体

`1. `

数字后加点，再空格；缩进的有序列表

`* `

星号后空格；无序列表

`&nbsp;`

可用作空行

`>`

引用文本

`` `JavaScript` ``

键盘tab上的小点，代码两端添加；表示一行计算机代码

`` ```  ``

键盘tab上的小点，单独占一行，代码群上下行添加；表示计算机代码群

`---`

三个减号独占一行表示分行线，或者使用 `<hr>` 标签

`[Link Text](https://a.com)`

给目标文本添加超链接，text 表示图片未加载出来显示的文字

`![Image Text](https://a.com/b.jpg)`

给文章插入图片，注意前有！号，[]内可不书写；yu writer可以直接粘贴图片

`<!-- more -->`

表示发送的文章在博客首页只显示`<!-- more -->`以上的部分，可添加阅读全文按钮

```
---
title: Markdown 使用指南
tags: setiment
categories: notes
photos:
  - https://images.pexels.com/photos/5834/nature-grass-leaf-green.jpg?auto=compress&cs=tinysrgb&dpr=2&h=650&w=940
---
```
对于markdown标题内的信息进行配置，想要在主页显示图片，在文章信息内添加即可

`|`

使用竖线来实现表格

&nbsp;| col | col
---   | --- | ---
row   | ... | ...
row   | ... | ...

```
xxx   | col | col
---   | --- | ---
row   | ... | ...
row   | ... | ...
```



&nbsp;

大概就这些常用了。