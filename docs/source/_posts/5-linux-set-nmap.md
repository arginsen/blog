---
title: Linux | nmap测远程ip和端口是否通
date: 2019-12-13 9:24:21
tags: 
- nmap
- Linux
categories: practice
photo: 
    - /blog/img/linux1.jpeg
---

<br>
<!-- more -->



一般会遇到这种问题：服务器间做了网络策略需要检测，比如两台服务器间做了指定端口的互通，服务商提供了一个网闸，这时服务器用ping、telnet就行不通，不过还真没见过linux上用telnet的。

这时nmap就是一个很好的选择，可以直接探测ip+port

网址：https://nmap.org/dist/nmap-7.80-1.x86_64.rpm

下载后传入服务器（这里一般指不通互联网的服务器，如政务网）,互联网就直接wget

```
rpm -ivh nmap-7.80-1.x86_64.rpm    #安装nmap
nmap 34.84.25.17 -p 80             #命令使用,如果不指定端口，会测出目标ip的所有开放端口

效果：
Starting Nmap 7.80 ( https://nmap.org ) at 2019-12-13 10:22 CST
Nmap scan report for 100.82.64.193
Host is up (0.00018s latency).

PORT   STATE SERVICE
80/tcp open  linuxconf             #如果不通，STATE会显示closed

Nmap done: 1 IP address (1 host up) scanned in 13.08 seconds
```