---
title: Tomcat | Tomcat启用及配置
date: 2019-7-9 12:20:21
tags: tomcat
photos:
  - /blog/img/tomcat.jpeg
categories:
- notes
---
<br>
<!-- more -->

# 简介

+ Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。

+ Tomcat是Apache服务器的扩展，但运行时它是独立运行的。当一台机器配置好Apache服务器，可利用tomcat响应HTML页面的访问请求。

+ Apache 为HTML页面服务，而Tomcat 实际上运行JSP 页面和Servlet。另外，Tomcat和IIS等Web服务器一样，具有处理HTML页面的功能，另外它还是一个Servlet和JSP容器，独立的Servlet容器是Tomcat的默认模式。不过，Tomcat处理静态HTML的能力不如Apache服务器。


## 安装jdk

首先需要配置java环境，在Oracle网站下载jdk：
[https://www.oracle.com/technetwork/java/javase/downloads/jdk12-downloads-5295953.html](https://www.oracle.com/technetwork/java/javase/downloads/jdk12-downloads-5295953.html)

这里是使用win7 64位测试，选择*jdk-12.0.1_windows-x64_bin.exe*下载

安装(记着安装路径)

## 配置java环境变量

右键计算机-属性-高级系统设置：
![](/blog/img/2019/tomcat/1.png)

打开环境变量:
![](/blog/img/2019/tomcat/2.png)

添加java变量：
```
用户变量下添加：
名称： CLASS_PATH
变量值：.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar 

系统变量下添加：
名称：JAVA_HOME
变量值：之前安装jdk的路径

并在已有的系统变量"path"的变量值加上：
%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;       #将语句至于path变量最前，且整句末不加封号
```

![](/blog/img/2019/tomcat/3.png)

此时java环境配置完成，可在cmd内查看：
![](/blog/img/2019/tomcat/4.png)


## 启动tomcat

目前tomcat最新版为9.0.21，可在下边网站下载：
[https://tomcat.apache.org/download-90.cgi](https://tomcat.apache.org/download-90.cgi)

这里是在win7虚拟机进行测试，选择*64-bit windows zip*下载。

下载解压后，在*bin*文件夹下找到*startup.bat*文件，
直接打开后可见如下弹框：
![](/blog/img/2019/tomcat/5.png)

同时登入http://localhost:8080   可见：
![](/blog/img/2019/tomcat/6.png)

出现如上页面，说明成功。

## tomcat参数配置

### 端口配置

在阅读器中打开conf文件夹下的server.xml

1.
ctrl+f查找<Connector port="8080" protocol="http"  相关内容，
修改connector端口为未被其他程序占用的端口。
![](/blog/img/2019/tomcat/7.png)

2.
寻找<Server port="xxxx" shutdown="xxxx    相关内容
修改server端口：
![](/blog/img/2019/tomcat/8.png)

3.
寻找<Connector port="xxxx" protocol="AJP/1.3" redirectPort="8443" />
修改端口：
![](/blog/img/2019/tomcat/9.png)

### 性能参数配置

#### 内存配置

在tomcat上运行j2ee项目代码时，经常会出现内存溢出的情况，解决办法是在系统参数中增加系统参数。
打开bin文件夹下的*catalina.bat*进行编辑，在第二行输入：
```
set JAVA_OPTS=-Xms512m -Xmx1024m -XX:MaxPermSize=512m
set JPDA_ADDRESS=58000
```
![](/blog/img/2019/tomcat/10.png)

#### 线程池

使用线程池，用较少的线程处理较多的访问，可以提高tomcat处理请求的能力。

打开conf文件夹下server.xml  ,输入以下:
```
<Connector port="8080" protocol="org.apache.coyote.http11.Http11AprProtocol" 
    connectionTimeout="20000"                       #网络连接超时
    URIEncoding="UTF-8"                             #使tomcat可以解析含有中文名的文件的url，避免乱码
    enableLookups="false" 
    maxThreads="500"                                #可创建线程的最大数，一般500
    acceptCount="300"                               #指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理。

    compression="on"                                #打开压缩功能，gzip(HTTP)压缩可以大大提高浏览网站的速度；原理：客户端请求网页后，从服务端将网页文件压缩，再下载到客户端，由客户端的浏览器负责解压并浏览。
    compressionMinSize="2048"                       #启用压缩的输出内容大小，默认2kb
    noCompressionUserAgents="gozilla, traviata"     #对于以下的浏览器，不启用压缩
    compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain"    #压缩类型
    redirectPort="8443" />
```

![](/blog/img/2019/tomcat/11.png)

#### 配置apr模式

修改bin文件夹下的catalina.bat  在第三行添加：

```
set JAVA_OPTS=-XX:PermSize=512M -XX:MaxPermSize=1024m –Xms2048M –Xmx2048M -XX:MaxNewSize=256m -XX:+HeapDumpOnOutOfMemoryError 
```
![](/blog/img/2019/tomcat/12.png)

windows下直接启动即可。

## tomcat发布应用

### 关闭tombat

直接关闭程序

### 释放tomcat占用端口

在bin中运行shutdown.bat

### 添加/修改应用xml文件

在\conf\Catalina\localhost文件夹下新建xml文件，命名与需要发布的文件一致

### 发布应用

xml文件修改配置完成后，在bin文件夹下启动startup.bat，即可发布多个应用

关闭tomcat，并释放窗口。

启动多个tomcat时需要配置tomcat使用的端口，
依旧在/conf/server.xml下修改，之前提到的三个端口

以及MIS端口的修改：
修改文件：eUrbanMIS\ WEB-INF\classes\ config.properties
修改端口：socket.security_xml_server_port=844 为未被占用的端口即可

需要把端口修改为不一样的端口
修改完成后启动tomcat

### 修改tomcat应用显示名

启动多个tomcat之后，tomcat的显示名称相同，在更新tomcat的时候，无法进行区分tomcat发布的服务，因此需要修改tomcat启动之后的显示名称

关闭tomcat
修改文件：tomcat/bin/ catalina.bat
修改内容：找到内容：if "%TITLE%" == "" set TITLE=Tomcat 并把title修改为需要显示的名称
修改后，启动tomcat

## tomcat日志目录说明

tomcat每次启动时，自动在logs目录下生产以下日志文件，按照日期自动备份
```
  localhost.2016-07-05.txt              //经常用到的文件之一 ，程序异常没有被捕获的时候抛出的地方
  catalina.2016-07-05.txt               //经常用到的文件之一，程序的输出，tomcat的日志输出等等
  manager.2016-07-05.txt                //估计是manager项目专有的
  host-manager.2016-07-05.txt           //估计是manager项目专有的
  localhost_access_log.2016-10-01.txt   //tomcat访问日志记录，需要配置
```

### 让所有文件都输出到同一个文件中

打开Tomcat目录conf\logging.properties，修改如下，所有日志输出到tomcat开头的文件中:
```
1catalina.org.apache.juli.FileHandler.level = FINE
1catalina.org.apache.juli.FileHandler.directory = ${catalina.base}/logs
# 1catalina.org.apache.juli.FileHandler.prefix = catalina.   #将#号的语句改为下一句

1catalina.org.apache.juli.FileHandler.prefix = tomcat.

2localhost.org.apache.juli.FileHandler.level = FINE
2localhost.org.apache.juli.FileHandler.directory = ${catalina.base}/logs
# 2localhost.org.apache.juli.FileHandler.prefix = localhost.

2localhost.org.apache.juli.FileHandler.prefix = tomcat.

3manager.org.apache.juli.FileHandler.level = FINE
3manager.org.apache.juli.FileHandler.directory = ${catalina.base}/logs
# 3manager.org.apache.juli.FileHandler.prefix = manager.

3manager.org.apache.juli.FileHandler.prefix = tomcat.

4host-manager.org.apache.juli.FileHandler.level = FINE
4host-manager.org.apache.juli.FileHandler.directory = ${catalina.base}/logs
# 4host-manager.org.apache.juli.FileHandler.prefix = host-manager.

4host-manager.org.apache.juli.FileHandler.prefix = tomcat.
```
![](/blog/img/2019/tomcat/13.png)

### 打开访问日志

默认 tomcat 不记录访问日志，如下方法可以使 tomcat 记录访问日志
编辑 /conf/server.xml 文件. 
把以下的注释 () 去掉即可。

![](/blog/img/2019/tomcat/14.png)

*上面的pattern可以修改格式*

该项值可以为： common 与 combined ，这两个预先设置好的格式对应的日志输出内容如下：

common 的值： %h %l %u %t %r %s %b
combined 的值： %h %l %u %t %r %s %b %{Referer}i %{User-Agent}i

pattern 也可以根据需要自由组合, 例如 pattern="%h %l"

对于各fields字段的含义可参照 :
http://tomcat.apache.org/tomcat-6.0-doc/config/valve.html 中的 Access Log Valve 项

### tomcat日志级别

Tomcat 日志分为下面５类：
catalina 、 localhost 、 manager 、 admin 、 host-manager

每类日志的级别分为如下 7 种：
SEVERE (highest value) > WARNING > INFO > CONFIG > FINE > FINER > FINEST (lowest value)

#### 日志级别的设定方法：

修改 conf/logging.properties 中的内容，设定某类日志的级别

示例：

设置 catalina 日志的级别为： FINE
```
1catalina.org.apache.juli.FileHandler.level = FINE
```
禁用 catalina 日志的输出：
```
1catalina.org.apache.juli.FileHandler.level = OFF
```
输出 catalina 所有的日志消息均输出：
```
1catalina.org.apache.juli.FileHandler.level = ALL
```

# 对于tomcat的补充

以下摘取网上搜集到的一些内容：
http://how2j.cn/k/tomcat/tomcat-tutorial/541.html
https://www.w3cschool.cn/tomcat/

此大佬写的教程很详细，会介绍tomcat的原理结构等：
https://www.cnblogs.com/windy1118/tag/Tomcat/

###备份地址：
https://lixupeng11.github.io/collection/tomcat/tomcat1.html
https://lixupeng11.github.io/collection/tomcat/tomcat2.html
https://lixupeng11.github.io/collection/tomcat/tomcat3.html
https://lixupeng11.github.io/collection/tomcat/tomcat4.html
