---
title: hexo博客 | 简单搭建hexo博客
date: 2019-3-27
categories: practice
tags: 
 - hexo
 - blog
photos: 
    - /blog/img/hexo.jpg
---

<br>
<!-- more -->

## 前言

之前出的搭梯子教程被封了，于是传在个人网站：
[https://lixupeng.cn/blog](https://lixupeng.cn/blog)
有兴趣可以动手玩玩，
那么这里就对自己搭建网站作以记录，
供大家参考。
个人主页：
[https://lixupeng.cn](https://lixupeng.cn)
[https://www.lixupeng.cn](https://www.lixupeng.cn)



>Hexo是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

详情见 https://hexo.io/zh-cn/docs/

<br>

## 教 程

### 安装Node.js和Git

直接在网上搜索 nodejs 和 git 即可，windows 下载安装包安装，mac/linux 直接命令安装
需要注意的是，node 版本不能过高，大概大版本为 12.xx.xx 即可
如果本地需要别的 node，那么可以使用 nvm 管理 node 版本，windows 有 nvm for windows 版

### 安装hexo-cli


```
 npm install hexo-cli -g 
```

### 创建Github仓库

此次建站采用GitHub托管我们的网站，我们需要新建一个代码仓库。

GitHub目前对免费用户开放了私密仓库，但免费用户的私密仓库不支持GitHub Pages，这里选择public

**1.注册github账号**

前往github.com按步骤注册账号

![](http://img.xiumi.us/xmi/ua/1O3nK/i/0451daa0463e3e4d8f0df9939adfa557-sz_202085.png)

![](http://img.xiumi.us/xmi/ua/1O3nK/i/53c6c7124b20c28d8edad0609cdc29fc-sz_74111.png)

**2.创建仓库** 

<p style="color:#B0B0B0;font-weight:bold;">名字可以用 `xxxxx.github.io` ,方便后续直接生成网页</p>

点选生成 **README.md** 文档，这个文档是自己编写，介绍你这个仓库项目是干嘛的，这里用来初始化仓库

![](http://img.xiumi.us/xmi/ua/1O3nK/i/d86a47c119ac8b9766609ebfee2ee4c7-sz_93596.png)

**3.克隆到本地编辑**

<p style="color:#B0B0B0;font-weight:bold;">下方图片因为没有更新，名字还显示原来的。如果是作为个人主页的博客，建议还是`xxxxx.github.io`这种格式，到时候github pages直接会用`https://xxxxx.github.io`作为个人网址</p>

<p style="color:#B0B0B0;font-weight:bold;">当然你如果想自定义主页，然后将博客作为主页下的一个分支，且在不买域名的情况下，博客的仓库名就可以自定义（如blog）,而自定义主页的仓库名命名为 `xxxxx.github.io`</p>

![](http://img.xiumi.us/xmi/ua/1O3nK/i/d7bd9f8cfc008ee74ce063160abcf17d-sz_26278.png?x-oss-process=image/resize,w_1080/auto-orient,1/crop,x_0,y_0,w_1080,h_479)

![](http://img.xiumi.us/xmi/ua/1O3nK/i/e79302b51950d32249bb41101076bbec-sz_43070.png)

<p style="color:#B0B0B0;font-weight:bold;">这里可以使用github下的atom编译器，也可以使用功能更加强大的VS code，当然也可以选择直接在终端用vim编写文章，不过要是你会vim等操作命令，那就基本不用看我的教程了..</p>

**4.打开保存在本地的仓库（文件夹）**

等下就会用到VS code进行编辑，而且VS code内自带终端，就不用外边打开cmd/poweshell/git bash了

### 安装Hexo

打开你刚才clone到本地的 ``xxxxx.github.io` 文件夹，这时会看到**.git**文件夹，表示git在此文件夹内工作，如果未看到就按下图设置一下

![](http://img.xiumi.us/xmi/ua/1O3nK/i/d94dfe111abf96e32a7e1e76b49e904c-sz_152397.png)

在`xxxxx.github.io`文件夹内创建docs文件夹

进入docs文件夹，鼠标右键打开`git bash `

<font color="#EE6363">输入：</font>

```
hexo init 
```

![](http://img.xiumi.us/xmi/ua/1O3nK/i/582a1004397113e02b4ff8ea5889df4f-sz_427721.png)

可以看到start blogging with hexo,即安装完成

这时我们可以接着<font color="#EE6363">输入：</font>
```
hexo server 
```
即可按提示在 http://localhost:4000 看到博客已生成，上边安装介绍hexo默认安装landscape主题，当然主题可以在hexo官网主题去自己选择

![](http://img.xiumi.us/xmi/ua/1O3nK/i/f625e396ac8881372f739451ad0924a1-sz_231423.png)

### 配置hexo

这时候再打开刚才的VS code，打开的是xxxxx.github.io文件夹，那么在界面的左侧栏就可以看见里边的所有内容，这时hexo文件已经全部生成完成，只需要我们自定义，以及写文章了

![](http://img.xiumi.us/xmi/ua/1O3nK/i/e2a3570566d30c13cd9c3bb487fdf514-sz_130230.png)

**1.主要配置**

打开docs文件夹下_config.yml文件，在右边编辑修改内容，将主题内容修改成自己的内容

<font color="#B0B0B0">这里要注意url
url即为你的主页地地址，xxxxx.github.io 仓库对应生成 `https://xxxxx.github.io`

root这里也默认 /  
#而你若是将hexo博客作为自定义主页的一个分支，那么就需要改为：</font>
```
url：https://xxxxx.github.io/blog
root: /blog/
```
<font color="#B0B0B0">###注意config文加里的设置冒号后必须要有一个空格</font>

**2.装主题**

这里推荐next主题，安装按next讲
next也是目前依旧在不断维护更新的一个主题
你也可以自己去选择主题

链接：https://hexo.io/themes/

安装细节一般主题都会有简介

* 在docs文件夹下打开终端(cmd/git bash)-文件夹内打开

或者直接在vs code内打开 ，不过需要从仓库文件夹下进入docs，<font color="#EE6363">输入</font>指令：

```
 cd docs 
```

![](http://img.xiumi.us/xmi/ua/1O3nK/i/d34b0f28f3415c71b4741d798e6b034f-sz_169019.png)

* 安装next主题<font color="#EE6363">输入：</font>

```
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

更改docs文件夹内的_config.yml文件

找到theme ，改为next

![](http://img.xiumi.us/xmi/ua/1O3nK/i/cbad72862abb136b368b74c5aa2d95b3-sz_6795.png)

保存更改，和之前一样输入命令：

```
hexo server
```

即可预览

![](http://img.xiumi.us/xmi/ua/1O3nK/i/5f126af070cdf1b7e3debc52b98657a4-sz_109667.png)

上图选自https://theme-next.org/ 官网

一般新安装的只能看见hello world字样

对于博客主题的配置参见：

https://github.com/theme-next/hexo-theme-next 

### 博文发布

**1.文章**

打开docs/source/_posts，里边的初始文章是一篇 helloworld.md , 你可以撰写自己的文章，发新文章的话直接在此文件夹下新建xxx.md，书写形式可以网上找寻_markdown文件编辑教程_，也可以参考我的下篇文章：

markdown编辑器使用

![](http://img.xiumi.us/xmi/ua/1O3nK/i/7010edad8d3f4637d46179b13f27b010-sz_38704.png?x-oss-process=image/resize,w_1080/auto-orient,1/crop,x_0,y_0,w_227,h_309)

**2.添加CNAME,提交推送至远端仓库**

<font color="#B0B0B0">如果是要绑定自己的域名，那么就在VS Code左侧找到source文件夹，右键创建名为CNAME的文件(无后缀格式)</font>

<font color="#B0B0B0">如果没买域名也可以直接用github.io给的，不用添加CNAME</font>

![](http://img.xiumi.us/xmi/ua/1O3nK/i/82dab7b5bb722638df185614b6fd08c3-sz_24900.png)

*   在右侧写入你的域名(如`lixupeng.cn`),保存

*   再打开之前的github桌面软件，可以看到左栏有很多新的文件/改动的文件，这是需要在下边输入总结记录

*   再`commit to master`，提交至master分支，这时完成了本地仓库的提交。

*   接着点击`push origin`，推送至远程仓库(新的都会显示push)

![](http://img.xiumi.us/xmi/ua/1O3nK/i/b69184bbb27ffec96b7f0c46d1a002a9-sz_111289.png)

**3.发布文章**

返回vs code界面，

打开/docs/_config.yml,配置最下边的deploy设置

`repo` 后为你仓库的地址，后边接`.git`

![](http://img.xiumi.us/xmi/ua/1O3nK/i/451307d4d2f576b1b4f5510f9ef629a4-sz_12876.png)

在终端下/docs/路径<font color="#EE6363">输入：</font>
```
 npm install hexo-deployer-git --save 
```
安装 `hexo-deployer-git`发送工具

完成后<font color="#EE6363">输入：</font>
```
 hexo clean
 hexo generate 
 hexo deploy 
```
**generate**表示将博客已经生成html页面

**deploy**表示将生成的博客推送到了远程的仓库

所以此时不需要再在github desktop提交推送

而且以后修改发文章只用`hexo g` 与`hexo d` 即可

配置完成。

### 开启网站

打开github上你博客的仓库settings

![](http://img.xiumi.us/xmi/ua/1O3nK/i/60464ae0c8c91d1f2fa7c37e373de428-sz_66991.png)

在下边开启GitHub pages，如果有域名就在下边保存

![](http://img.xiumi.us/xmi/ua/1O3nK/i/57fd45d729452f1251999f71406fb0a4-sz_62893.png)

<div style="color:grey">#没域名的在这里已经完成啦！

你的网站已上线！

#有域名的同学请到域名购买商解析域名到 xxxxx.github,io

那么我这里用的是lixupeng.cn,就用@
</div>
![](http://img.xiumi.us/xmi/ua/1O3nK/i/463d75fbcd2e12c4772cc00e30747898-sz_3908.png)

<div style="font-size:18px;font-weight:bold;">大功告成，博客上线！</div>

btw，以下是一个博主说的...表示很不能理解

我觉得给外行来讲，虚拟主机这个概念更不好理解

在虚拟主机架设也不好操作，如相当简易的typecho博客，号称仅几百k，但是又要配置好环境，需要在虚拟主机安装nginx/apache2+mysql+php，这些里边又分很多类，直接操作也是相当麻烦，理解每一个概念也不容易，一个配置不好就全部凉凉，而且用完一年免费的又得花钱，还是gayhub好
不过上述在服务器端架设博客，如typecho/wordpress其实也都是有环境搭建包的，就像本文的巧克力，一部部署。

总之，看个人需求吧

![](http://img.xiumi.us/xmi/ua/1O3nK/i/cc4fedcd352ddda6c15e988a1a4f4a40-sz_37689.png)
