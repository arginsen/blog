---
title: Linux | Linux学习笔记Ⅱ：文件基本属性
date: 2019-8-20 10:30:02
categories: notes
tags:
 - Linux
photo: 
    - /blog/img/linux1.jpeg
---
<br>
<!--more-->

# Linux文件基本属性

Linux系统是一种典型的多用户系统，不同的用户处于不同的地位，拥有不同的权限。为了保护系统的安全性，Linux系统对不同的用户访问同一文件（包括目录文件）的权限做了不同的规定。

```
[root@cg2 www.eyebrows.ink]# ll             #ll用于查询当前目录上各个文件的属性及所属
total 76
drwxr-xr-x. 5 www  www   4096 Jul 26 14:12 admin
-rw-r--r--. 1 www  www   1550 Jul 26 21:07 config.inc.php
drwxr-xr-x. 2 root root    43 Aug  1 20:38 hitokoto
-rw-r--r--. 1 www  www    721 Oct 30  2017 index.php
drwxr-xr-x. 2 www  www    110 Jul 26 14:12 install
-rw-r--r--. 1 www  www  48403 Oct 30  2017 install.php
-rw-r--r--. 1 www  www  14974 Oct 30  2017 LICENSE.txt
drwxr-xr-x. 5 root root    50 Jul 24 19:36 usr
drwxr-xr-x. 5 www  www    164 Oct 30  2017 var
```

## 文件属性

其中前边的一串字母规定了该文件的权限，共有**10**位，其中第一个字符代表了该文件是目录、文件或是链接文件等。

一般有五类：
+ `d`表示目录
+ `-`表示文件
+ `l`表示链接文档
+ `b`表示为装置文件里面的可供储存的接口设备(可随机存取装置)
+ `c`表示装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)。

## 文件权限

后边**9**位以『rwx』三个参数位一组，共有三个组，分别代表**属主**(所有者)、**属组**(所有者同组用户)、**其他用户**对该文件的权限。

『rwx』中，[ r ]代表可读(read)、[ w ]代表可写(write)、[ x ]代表可执行(execute)。 要注意的是，这三个权限的位置不会改变，如果没有权限，就会出现减号[ - ]而已。

[](/blog/img/2019/linux-file/1.png)

## Linux文件属主和属组

```
[root@cg2 www.eyebrows.ink]# ll             #ll用于查询当前目录上各个文件的属性及所属
total 76
drwxr-xr-x. 5 www  www   4096 Jul 26 14:12 admin
-rw-r--r--. 1 www  www   1550 Jul 26 21:07 config.inc.php
drwxr-xr-x. 2 root root    43 Aug  1 20:38 hitokoto
-rw-r--r--. 1 www  www    721 Oct 30  2017 index.php
drwxr-xr-x. 2 www  www    110 Jul 26 14:12 install
-rw-r--r--. 1 www  www  48403 Oct 30  2017 install.php
-rw-r--r--. 1 www  www  14974 Oct 30  2017 LICENSE.txt
drwxr-xr-x. 5 root root    50 Jul 24 19:36 usr
drwxr-xr-x. 5 www  www    164 Oct 30  2017 var
```

### 注意事项

+ 每个文件都有一个特定的所有者
+ Linux系统中用户按组分类，以对一个文件的权限划分为3个组
+ 一般来说，对root用户，文件的权限对其不起作用

### 更改文件属性

```
chgrp 属组名 文件名             # 更改文件属组
chgrp -r 属组名 文件名          # -r 表示递归更改文件属组
```

#### 更改文件属主

```
chown 属主名 文件名             # 更改文件属主
chown 属主:属组名 文件名         # 同时更改文件属组
```
#### 更改文件权限

##### 数值法更改

```
chmod [-r] xyz 文件或目录       # -r表示递归变更，此目录下的所有文件都会变更
```

Linux文件的基本权限就有九个，分别是owner/group/others三种身份各有自己的read/write/execute权限。

文件的权限字符为：『-rwxrwxrwx』， 这九个权限是三个三个一组的！其中，我们可以使用数字来代表各个权限，各权限的分数对照表如下：
+ `r`-4
+ `w`-2
+ `x`-1

每种身份(owner/group/others)各自的三个权限(r/w/x)分数是需要累加的，例如当权限为 [-rwxrwx---] 分数则是：
+ owner = rwx = 4+2+1 = 7
+ group = rwx = 4+2+1 = 7
+ others= --- = 0+0+0 = 0

如果要将权限变成 -rwxr-xr-- 呢？那么权限的分数就成为 [4+2+1][4+0+1][4+0+0]=754。
```
chmod 754 文件
```

##### 符号法更改

可以通过权限对象的英文首字母分类：
+ user - u
+ group - g
+ others - o
+ all - a

如果我们需要将文件权限设置为 -rwxr-xr-- ，可以执行：
```
chmod u=rwx,g=rx,o=r 文件       # =为设定，+为增加权限，-为去掉权限
chmod a-x 文件                  # 给全部人都去掉执行的权限
```

# Linux目录管理

