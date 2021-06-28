---
title: Nginx | 配置conf文件使网站http自动跳转https
date: 2019-5-8
tags: nginx
categories: practice
photos:
    - /blog/img/nginx.jpeg
---
<br>
<!-- more -->

### 前言

不久前用typecho搭建的动态网站生成后，添加了ssl证书，也开启了https

但是出现一个问题：直接输入域名会默认访问http，而且我进去还是乱码的

这时就想着怎么实现默认https访问

大致思路就是输入域名（最简单的进入方法），默认http，再使http重定向到https

### 操作


* 进入vhost目录

```bash
 cd /usr/local/nginx/conf/vhost
```

* 编辑 **.conf** 文件

```bash
 sudo vim www.xxxxx.com.conf    # 在vhost可以输入ls查看下目录
```

* 添加rewrite这句

```conf
server
    {
        listen 80; //80端口是为 http 开放的
        #listen [::]:80;
        server_name www.xxxxx.com;
        index index.html index.htm index.php default.html default.htm default.php;
        root  /home/wwwroot/www.xxxxx.com;
        rewrite ^(.*)$ https://www.xxxxx.com$1 permanent; //将http重定向至https

        ssl_certificate /home/wwwroot/ssl/xxx.crt; // ssl 证书路径
        ssl_certificate_key /home/wwwroot/ssl/xxx.key;
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
    }

```

### 备注

* vim编辑的快捷键：

    写入：`i`
    保存并退出：`:wq!`  (按**esc**键后)
    退出：`:q!`

* 关于**vhost**文件夹下 `www.xxxxx.com.conf`文件的配置

文件内容为两个`server{}`组成，一个server是http(80)的配置，一个server是https(443)的配置

要确保文件内仅有这两段信息，且确保`server_name www.xxxxx.com`这样的域名信息在文件中在每个`server{}`内出现不得重复，否则就是提示 0.0.0.0 443 ignore 或者 80 ignore