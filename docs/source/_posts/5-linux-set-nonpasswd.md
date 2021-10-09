---
title: Linux | 服务器配置免密登录
date: 2019-8-13 20:43:33
tags: Linux
categories: practice
photo: 
    - /blog/img/linux1.jpeg
---

<br>

使从主控机通过ssh登录远程受控服务器时免密登录且使用自定义默认端口

<!--more-->

## 生成密钥

### 使用root

有两种情况，现场可能有root用户密码，或者为提供仅有sudo权限的账号

1. 直接使用root

2. 通过sudo进入root

```
sudo su -
```

### 生成密钥

```
ssh-keygen -t rsa
```
生成当前主机的`id_rsa、id_rsa_pub`密钥文件：
![](/blog/img/2019/nonpasswd/1.png)

### 创建authorized_keys文件

```
touch authorized_keys                   #创建authorized_keys文件
cat id_rsa.pub > ./authorized_keys      #将生成密钥文件写入authorized_keys
```

![](/blog/img/2019/nonpasswd/2.png)


### 传输authorized_keys文件至受控机

```
scp -r authorized_keys  root@42.48.99.2:/root/.ssh/
```

进入受控机，检查tmp下是否有authorized_keys文件，并且authorized_keys的内容要和主控机的authorized_keys内容一致

//要是远程传输不了，也可在受控机建立authorized_keys，把密钥复制粘贴进去.
//传输过去因为还没有配置密钥，所以需要输一次密码，之后远程服务器，通过远程改下配置文件即可

### 配置权限

.ssh和authorized_keys的权限，前者需要为700，后者需要为600

```
chmod 700 .ssh
chmod 600 authorized_keys
```

### 修改ssh配置文件

1. sshd_config文件--服务端

进入`/etc/ssh/sshd_config`文件，修改如下两个位置的配置信息（主、受都需要更改）
```
vi /etc/ssh/sshd_config

#########修改为如下
PermitRootLogin yes
StrictModes no
```
![](/blog/img/2019/nonpasswd/3.png)

2. ssh_config文件--客户端

使用ssh命令进行远程登录时，不使用-p，使ssh命令默认端口为22022：
```
vi /etc/ssh/ssh_config

########修改为如下
port 22022
```
![](/blog/img/2019/nonpasswd/4.png)

### 重启sshd服务

```
service sshd restart
```

### 验证

在主控机上测试是能通过ssh免密登录受控机
```
ssh root@10.0.13.32
```
![](/blog/img/2019/nonpasswd/5.png)