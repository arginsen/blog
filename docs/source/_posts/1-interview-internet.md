---
title: 面试 | web 性能
date: 2020-8-30
tags: 
  - interview
  - Frontend
categories: notes
photos:
    - /blog/img/interview.jpg
---

<br>
<!--more-->

# TCP/IP 协议

```
OSI
应用层   \            HTTP/SMTP/FTP/SSH 协议
表示层   - 应用程序   数据压缩，数据加密以及数据描述
会话层   /            数据交换的定界和同步功能
传输层   TCP/UDP 传输协议
网络层   IP 协议
链路层   以太网/WiFi/电缆接入网 等
物理层   光纤/电缆 等
```

# TCP 和 UDP 的区别

TCP：
面向连接
一对一通信
面向字节流
全双工通信
可靠传输，使用流量控制和拥塞控制
报头最小20字节，最大60字节

UDP：
无连接
支持一对一，一对多，多对一和多对多的通信
面向报文
不可靠传输，不使用流量控制和拥塞控制
报头开销小，仅8字节

应用：
流媒体应用
用于 DNS 和 SNMP


# TCP 的三次握手、四次挥手

- SYN: Synchronize Sequence Numbers 同步序列编号
- ACK：Acknowledgement 确认
- seq：sequence number 序列号
- ack：acknowledge number 应答号


三次握手：
在客户端和服务器之间建立正常的 TCP 网络连接时，客户端首先会发出一个 SYN 消息，服务器使用 SYN+ACK 应答

# http 状态码（字节）

301——请求的网页已永久移动到新位置
302——临时性重定向
304——自从上次请求后，请求的网页未修改过
504——关口过载，服务器使用另一个关口或服务来响应用户，等待时间设定值较长

# http 缓存

强缓存和协商缓存

强缓存在浏览器加载资源时，先从缓存中查找结果，如果不存在则向服务端发起请求

协商缓存的更新策略是不再指定缓存的有效时间，而是浏览器直接发送请求到服务端进行确认缓存是否更新，如果请求响应返回的 http 状态为 304，则表示缓存仍然有效

