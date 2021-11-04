---
title: ubuntu下搭建typecho博客
date: 2019-3-29
categories: practice
tags:
 - typecho
 - blog
photo: 
 - /blog/img/typecho.jpg
---

<br>

# 前言

之前注册过亚马逊云送了12个月服务器，大概9月份过期，
于是打算在服务器上搭个博客

不比hexo、jekyll这类静态博客生成工具，有个github账号就能有自己的博客
typecho是基于php+数据库的一个动态博客平台，需要在服务器搭建相应的环境
<!-- more -->
一旦建好，管理很是方便，切换主题、发布文章、调节个性化
确实相比hexo这方面要好操作很多，但就是环境架设，配置服务器端不好操作

## 工具

我们这里采用LNMP一键安装包进行环境架设，虽说是一步到位
不用你去思考选什么，版本匹配之类的问题，但大致背景还是要心里有底

> Linux作为操作系统，Apache 或 Nginx 作为 Web 服务器，MySQL 作为数据库，PHP/Perl/Python作为服务器端脚本解释器。由于这四个软件都是免费或开放源码软件（FLOSS)，因此使用这种方式不用花一分钱（除开人工成本）就可以建立起一个稳定、免费的网站系统，被业界称为“LAMP“或“LNMP”组合。

我们使用的一键安装包中Web服务器是Nginx，所以称为LNMP

## 背景

> - 系统选择Ubuntu 18.04.2 LTS
> - typecho环境需要Nginx/Apache2+MySQL/SQLite+PHP+PHP系列组件(curl/mbstring/dev/pear/mysql/gd/pdo/sqlite3/...)

>Nginx：Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，特点是占有内存少，并发能力强。Nginx作为负载均衡服务：Nginx 既可以在内部直接支持 Rails 和 PHP 程序对外进行服务，也可以支持作为 HTTP代理服务对外进行服务。

>Apache2：Apache是世界使用排名第一的Web服务器软件。它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的Web服务器端软件之一。它快速、可靠并且可通过简单的API扩充，将Perl/Python等解释器编译到服务器中。

>MySQL：MySQL是一种关系数据库管理系统，是指包括相互联系的逻辑组织和存取这些数据的一套程序 (数据库管理系统软件)。关系数据库管理系统就是管理关系数据库，并将数据逻辑组织的系统。关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。

>SQLite：SQLite是一款轻型的数据库，它的设计目标是嵌入式的，它占用资源非常的低，在嵌入式设备中，可能只需要几百K的内存就够了。SQLite引擎不是个程序与之通信的独立进程，而是连接到程序中成为它的一个主要部分。所以主要的通信协议是在编程语言内的直接API调用。这在消耗总量、延迟时间和整体简单性上有积极的作用。整个数据库(定义、表、索引和数据本身)都在宿主主机上存储在一个单一的文件中。

>PHP：PHP（外文名:PHP: Hypertext Preprocessor，中文名：“超文本预处理器”）是一种通用开源脚本语言。语法吸收了C语言、Java和Perl的特点，利于学习，使用广泛，主要适用于Web开发领域。用PHP做出的动态页面与其他的编程语言相比，PHP是将程序嵌入到HTML（标准通用标记语言下的一个应用）文档中去执行，执行效率比完全生成HTML标记的CGI要高许多；PHP还可以执行编译后代码，编译可以达到加密和优化代码运行，使代码运行更快。


# 教程

## LNMP环境部署

### win下用putty连接虚拟主机

与上次搭vpn时的方法一致，下载puttygen+putty

用puttygen将你实例生成时的xxx.pem密钥生成xxx.ppk
再在putty上用连接你实例的ssh和xxx.ppk文件登陆服务器

登陆显示：
```shell
Using username "ubuntu".
Authenticating with public key "imported-openssh-key"
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-1032-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Apr  4 22:36:40 CST 2019

  System load:  0.08              Processes:           105
  Usage of /:   47.8% of 7.69GB   Users logged in:     1
  Memory usage: 34%               IP address for eth0: x.x.x.x
  Swap usage:   0%

 * Ubuntu's Kubernetes 1.14 distributions can bypass Docker and use containerd
   directly, see https://bit.ly/ubuntu-containerd or try it now with

     snap install microk8s --classic

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

45 packages can be updated.
6 updates are security updates.


*** System restart required ***
Last login: Thu Apr  4 20:21:28 2019 from x.x.x.x
```

### LNMP一键安装包安装

1. 下载稳定安装包
```shell
wget http://soft.vpser.net/lnmp/lnmp1.5.tar.gz -cO lnmp1.5.tar.gz && tar zxf lnmp1.5.tar.gz && cd lnmp1.5 && ./install.sh lnmp
```

2. 选择安装默认版本即可

MySQL 5.5.60
PHP 5.6.36
Apache 2.4.33

期间需要设置MySQL的root密码
一般按默认安装，等一会就安装完了

完成后可输入 `sudo lnmp -v` 查询
```shell
ubuntu@ipx-x-x-x:~$ sudo lnmp -v
+-------------------------------------------+
|    Manager for LNMP, Written by Licess    |
+-------------------------------------------+
|              https://lnmp.org             |
+-------------------------------------------+
Usage: lnmp {start|stop|reload|restart|kill|status}
Usage: lnmp {nginx|mysql|mariadb|php-fpm|pureftpd} {start|stop|reload|restart           |kill|status}
Usage: lnmp vhost {add|list|del}
Usage: lnmp database {add|list|edit|del}
Usage: lnmp ftp {add|list|edit|del|show}
Usage: lnmp ssl add
Usage: lnmp {dnsssl|dns} {cx|ali|cf|dp|he|gd|aws}
```
### 绑定域名+创建数据库

这步前先将你要绑定的域名在域名注册商那里先解析到你的服务器ip
不然就没得申请ssl证书

```shell
+-------------------------------------------+
|    Manager for LNMP, Written by Licess    |
+-------------------------------------------+
|              https://lnmp.org             |
+-------------------------------------------+
Please enter domain(example: www.lnmp.org): xxxxx.com
 Your domain: xxxxx.com
Enter more domain name(example: lnmp.org *.lnmp.org):
Please enter the directory for the domain: xxxxx.com
Default directory: /home/wwwroot/xxxxx.com:
Virtual Host Directory: /home/wwwroot/xxxxx.com
Allow Rewrite rule? (y/n) y
Please enter the rewrite of programme,
wordpress,discuzx,typecho,thinkphp,laravel,codeigniter,yii2 rewrite was                  exist.
(Default rewrite: other): typecho
You choose rewrite: typecho
Enable PHP Pathinfo? (y/n) y
Enable pathinfo.
Allow access log? (y/n) y
Enter access log filename(Default:xxxxx.com.log):
You access log filename: xxxxx.com.log
Create database and MySQL user with same name (y/n) y
Enter current root password of Database (Password will not shown):
OK, MySQL root password correct.
Enter database name: xxxxx
Your will create a database and MySQL user with same name: xxxxx  //数据库的名字也会作为user的名字
Please enter password for mysql user myweb: xxxxx
Your password: xxxxx
Add SSL Certificate (y/n) y
1: Use your own SSL Certificate and Key
2: Use Let's Encrypt to create SSL Certificate and Key
Enter 1 or 2: 2
It will be processed automatically.
```

接下来就会看到：
```shell
================================================
Virtualhost infomation:
Your domain: xxxxx.com
Home Directory: /home/wwwroot/xxxxx.com
Rewrite: typecho
Enable log: yes
Database username: xxxxx
Database userpassword: xxxxx
Database Name: xxxxx
Create ftp account: no
Enable SSL: yes
  =>Let's Encrypt
================================================
```

## 安装typecho

之前建立的网站位置在/home/wwwroot/xxxxx.com

所以进入网站位置下载typecho，解压，将build内的文件放在xxxxx.com文件夹下
```shell
wget http://typecho.org/downloads/1.1-17.10.30-release.tar.gz && tar xvzf 1.1-17.10.30-release.tar.gz && cd build && mv * .. && cd .. && rm -rf 1.1-17.10.30-release.tar.gz build && chown -R www:www *

# 一些命令
wget：下载至当前目录
tar xvzf：解压tar.gz文件于当前目录
cd build：进入biuld文件夹
cd .. ：返回上一文件夹
rm -rf ：删除文件/文件夹

# 网站文件夹内的目录：
/admin/
/install/
/usr/
/var/
/license.txt  
/index.php
/install.php
```

> 大致逻辑："下载typecho、解压、进入解压的文件夹build、把build所有东西移出、删除build和压缩包"


此时typecho就已经配置好了

在网页地址栏输入 `https://你的域名/install.php` 即可进入安装界面

输入你之前创建的数据库用户名与数据库名(同名)，密码等

typecho博客的搭建就完成啦，个性化的配置进入控制台尽情折腾吧

补：换主题时，将主题下载解压至 `/home/wwwroot/xxxxx.com/usr/themes` 内即可，更换在控制台外观设置


<br>

