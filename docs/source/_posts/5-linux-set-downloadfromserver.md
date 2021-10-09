---
title: Linux | 将app放在服务器供人下载
date: 2019-10-12 12:23:33
tags: 
  - download
  - tomcat
  - Linux
categories: practice
photo: 
    - /blog/img/linux1.jpeg
---

<br>
<!-- more -->

## 一、服务器存放文件用于局域网内下载-使用FTP服务器(可直接用软件)

1. 确定ftp已安装

```
find / -name ftp    #查找ftp位置，一般在/var/ftp
yum install vsftpd
```

2. 修改配置文件

```
vi /etc/vsftpd.conf

---------
write_enable=YES              #开启
local_umask=022
chroot_local_user=YES

allow_writeable_chroot=YES    #文件末尾添加此五行
local_root=/var/ftp           #ftp的文件根目录
pasv_enable=Yes
pasv_min_port=40000
pasv_max_port=40100
```

3. 重启ftp服务器

```
service vsftpd restart
```

**将文件放在/var/ftp**
**打开ftp://ip/文件名**

备注：默认装了iis，大约会有一个文件夹/home/wwwroot，将文件放入
http://你的IP/文件名 

## 二、服务器存放文件用于互联网用户下载-使用tomcat服务器(web服务器)

1. 配置文件tomcat/conf/web.xml

```
        <init-param>
            <param-name>listings</param-name>
            <param-value>true</param-value>        #将false改为true

        </init-param>
```

2. 在webapps里创建目录/download，在里边可以放供人下载的文件

此时进 http://localhost:port/download/文件  #tomcat端口，即可访问

3. 此外需要将文件路径映射到磁盘的某个路径下时，还需要配置文件conf/server.xml

```
<Host name="localhost"  appBase="webapps"  unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
		<Context path="/app/" docBase="/download/"></Context>       #在此句中设置path为虚拟路径，docBase为真实路径
</Host>
```
此时进 http://localhost:port/app/文件  即可访问