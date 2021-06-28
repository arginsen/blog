---
title: Linux | 服务器ping不通域名可ping服务器地址
date: 2019-7-30 10:28:49
tags: 
- dns
- Linux
categories: practice
photo: 
    - /blog/img/linux1.jpeg
---

<br>
<!-- more -->

问题:服务器端ping不通域名但可ping服务器地址


1. 修改 `/etc/NetworkManager/NetworkManager.conf` 文件，在main部分添加 “dns=none” 选项(不操作这一步，直接修改/etc/resolv.conf，重启网卡又会变回改之前的配置）,去掉注释的#号即可。

![](/blog/img/2019/dns-ping/1.jpg)

2. 修改 `/etc/resolv.conf` 加上DNS服务器 `114.114.114.114` (仅保留此项)

![](/blog/img/2019/dns-ping/2.jpg)

3. 重启

```
systemctl restart NetworkManager.service
```
即可ping通。