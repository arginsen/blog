---
title: Linux | 允许使用 root 登录微软/谷歌/亚马逊服务器
date: 2019-8-15 20:43:33
tags: Linux
categories: practice
photo: 
    - /blog/img/linux1.jpeg
---

<br>
<!-- more -->

> login as the user "azureuser" rather than the user "root".

```bash
# 为 root 用户创建密码
sudo passwd root


# 修改 sshd 配置文件
vim /etc/ssh/sshd_config

PasswordAuthentication yes
PermitRootLogin yes
UsePAM no


# 修改 authorized_keys 文件，给公钥换行，并将之前的提示注释
sed -ri 's/^/#/;s/sleep 10"\s+/&\n/' /root/.ssh/authorized_keys


# 重启sshd服务
service sshd restart

# or
systemctl restart sshd.service 
```