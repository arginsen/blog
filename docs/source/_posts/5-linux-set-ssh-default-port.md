---
title: Linux | 配置ssh使远程不用指定-p
date: 2019-8-14 13:23:33
tags: 
- ssh
- Linux
categories: practice
photo: 
    - /blog/img/linux1.jpeg
---

<br>
<!-- more -->



修改ssh客户端配置文件使ssh不需要指定`-p`来登录远程


## ssh_config文件

+ ssh_config文件是ssh客户端的配置文件
+ ssh 可以从用户级配置文件和系统级配置文件中获取更多的配置数据，这样我们可以在使用ssh时省掉很多繁杂的命令选项。例如使用ssh命令进行远程登录时，实际上可以不使用-p选项来指明端口，我们可以通过配置文件的方式来设置ssh命令默认端口。
+ ssh获取配置数据会以下面的顺序进行：
    1. command-line options
    2. user's configuration file (~/.ssh/config)
    3. system-wide configuration file (/etc/ssh/ssh_config)

## 修改ssh_config文件
```
vi /etc/ssh/ssh_config

########修改为如下
port 22022
```

![](/blog/img/2019/nonpasswd/4.png)