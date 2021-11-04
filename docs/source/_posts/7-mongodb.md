---
title: MongoDB | MongoDB 学习笔记
date: 2020-7-16 11:20:33
tags:  
- MongoDB
photos:
    - /blog/img/mongodb.jpg
categories:
- notes
---
<br>
<!-- more -->

# 安装

```bash
# ubuntu 安装
apt install mongodb


# windows 安装
# 下载 zip 压缩包，解压至自定义目录
# 根目录创建 /data/db  /data/log  mongod.cfg

# mongod.cfg 作为 mongodb 的配置文件
systemLog:
    destination: file
    path: D:\Mongodb\data\log\mongodb.log
storage:
    dbPath: D:\Mongodb\data\db

# 因为我事先将 mongodb 加入了环境变量，可以直接使用 .exe
# 根目录命令行执行
mongod --config "D:\Mongodb\mongod.cfg" --install

# 安装完成，运行 mongodb
net start MongoDB   # 和 mysql 差不多

# 启动客户端
mongo   # 由于添加环境变量
# ===>
# MongoDB shell version v3.6.19
# connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
# Implicit session: session { "id" : UUID("0abec236-a474-4515-aadd-b16594421115") }
# MongoDB server version: 3.6.19
# Welcome to the MongoDB shell.
# For interactive help, type "help".
# For more comprehensive documentation, see
#         http://docs.mongodb.org/

# 随后进入后台管理 Shell，是 MongoDB 自带的交互式 Javascript shell

# 离开 shell
exit

# 关闭 mongodb 服务
net stop MongoDB

# 移除 mongodb
mongod --remove   # 由于添加环境变量
```

# 使用

```js
// mongodb 可以直接在命令行导入 json 文件
mongoimport -d test -c xxx xxx.json
```



# 参考链接

[Mongodb 的下载与安装](https://www.cnblogs.com/wuweixiong/p/12592172.html)
[Mongodb 下载](http://dl.mongodb.org/dl/win32/x86_64)
[Mongodb 教程](https://www.runoob.com/mongodb/nosql.html)