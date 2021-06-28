---
title: MySQL | win10下安装MySQL8
date: 2019-11-21 9:30:21
tags: MySQL
photos:
    - /blog/img/mysql.jpeg
categories:
- practice
---
<br>
<!-- more -->

# 下载

下载地址：https://dev.mysql.com/downloads/mysql/

# 安装

1. 解压至 D:\Mysql\mysql-8.0.18-winx64\

2. 文件夹下新建my.ini文件和data文件夹
<!--more-->
```
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=D:\Mysql\mysql-8.0.18-winx64
# 设置mysql数据库的数据的存放目录
datadir=D:\Mysql\mysql-8.0.18-winx64\data
# 设置登陆加密的规则
default_authentication_plugin=mysql_native_password
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
```

3. 添加环境变量

用户变量的Path添加： %MYSQL_HOME%\bin
系统变量中添加： 变量 MYSQL_HOME  值 D:\Mysql\mysql-8.0.18-winx64\

4. 初始化

在D:\Mysql\mysql-8.0.18-winx64\bin路径下，以管理员身份打开powershell
```sql
mysqld --initialize --user=mysql --console    #后边会给root用户生成一个随机密码
```

#此处若是忘记了随机密码，执行：
```sql
net stop mysql       #关闭服务
mysqld --console --skip-grant-tables --shared-memory    #无密码启动mysql服务
mysql.exe -uroot     #无密码登陆，重新打开窗口执行

mysql>update mysql.user set authentication_string='' where user='root';      #更新密码为空
mysql>flush privileges;
mysql>ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
mysql>flush privileges;
```

5. 安装

```sql
mysqld -install     #安装
net start mysql     #启动
```

6. 配置

```sql
mysql -u root -p    #登陆，使用之前生成的随机密码

mysql>ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';    #更改密码，密码加密方式最好选择mysql_native_password
mysql>use mysql;        #使用mysql数据库
mysql>update user set host = '%' where user ='root';     #更改root可连接的host为任意ip
mysql>flush privileges;
mysql>select user,host from user;      #查看人员
mysql>exit

net stop mysql    #配置后重启mysql
net start mysql  
```

参考链接：
安装：https://blog.csdn.net/qq_41848006/article/details/88295973
