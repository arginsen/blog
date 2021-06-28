---
title: 配置一言功能并添加至网站
date: 2019-3-30
categories: practice
tags:
 - php
photo: 
    - /blog/img/hitokoto.jpg
---

# 前言

在博客页中想到实现将一些喜欢的句子呈现出来

每刷新一次推送一句，不用自己去更新

于是借用调用php页面来实现在页面任意位置呈现句子
<!-- more -->
# 步骤

+ 先将一言功能部署至网站的下级`https://你的域名/oneword/` 页面呈现
+ 将此功能添加至博客界面 

## 材料准备

**需要php文件+txt文件**

1. 创建一个oneword.php文件，写入：

```php
<?php
// 获取句子文件的绝对路径
// 如果你介意别人可能会拖走这个文本，可以把文件名自定义一下，或者通过Nginx禁止拉取也行。
$path = dirname(__FILE__);
$file = file($path."/oneword.txt");
 
# 随机读取一行
$arr  = mt_rand( 0, count( $file ) - 1 );
$content  = trim($file[$arr]);
 
# 编码判断，用于输出相应的响应头部编码
if (isset($_GET['charset']) && !empty($_GET['charset'])) {
    $charset = $_GET['charset'];
    if (strcasecmp($charset,"gbk") == 0 ) {
        $content = mb_convert_encoding($content,'gbk', 'utf-8');
    }
} else {
    $charset = 'utf-8';
}
header("Content-Type: text/html; charset=$charset");
 
# 格式化判断，输出js或纯文本
if ($_GET['format'] === 'js') {
    echo "function oneword(){document.write('" . $content ."');}";
} else {
    echo $content;
}
```
代码来自[张戈博客]([https://zhang.ge/5127.html](https://zhang.ge/5127.html))

2. 再建立oneword.txt文件，存入你的句子，一行一句

**将txt和php文件放入oneword文件夹**
**将你的onword文件夹压缩并传在一个云/github上** *(为了下载至服务器)*

## 服务器配置

进入服务器
```
cd /home/wwwroot/xxxxx.com
wget https://xxx/xxx/xxx/oneword.zip  //下载你的oneword压缩包
unzip oneword.zip                     //解压oneword.zip
rm -rf oneword.zip                    //删除oneword.zip
```
即在网站中添加成功，此时访问`https://你的域名/oneword/`
就可以看见网页中显示随机从 oneword.txt 中随机抽取的一句话

## 博客集成

将下面两行代码添加到博客你想显示一言的位置即可：
```
<script  type="text/javascript"  src="https://你的域名/oneword/?format=js&charset=utf-8"></script>
<div  id="oneword"  ><script>oneword()</script></div>
```
此时，就可以显示出帅气的句子啦
可查看我自己制作的：https://lixupeng.cn/essay/about/



&nbsp;