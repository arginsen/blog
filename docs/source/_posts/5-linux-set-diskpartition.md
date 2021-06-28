---
title: Linux | Centos7下磁盘分区、挂载
date: 2019-11-19 9:30:21
tags: Linux
categories: practice
photo: 
    - /blog/img/linux1.jpeg
---

<br>
<!-- more -->

# 安装硬盘到物理机

# 检查磁盘

```
fdisk -l             #检查新增的硬盘标识、大小
```

# 磁盘分区

1. 当硬盘小于等于2T时，可以用fdisk。
```
fdisk /dev/vdc       #vdc是新磁盘的名称
    n                #新建分区
    p                #创建逻辑分区
    1                #分区号，一般有1-4
    2047             #分区空间起始点
    +1000G           #分区末尾/回车表示默认使用完余下空间
    p                #检查分区情况
    w                #保存配置
```

2. 当硬盘大于2T时，用parted命令。
```
parted /dev/vdc1      #用part命令对3T硬盘进行分区处理
mklabel gpt          #用gpt格式可以将3TB弄在一个分区里
unit TB              #设置单位为TB
mkpart primary 0 3   #设置为一个主分区,大小为3TB，开始是0，结束是3
print                #显示设置的分区大小
quit                 #退出parted程序
```

# 格式化分区

```
mkfs.ext4 /dev/vdc1     #也可用xfs
```

# 磁盘挂载

```
mkdir /egova
mount /dev/vdc1 /egova

blkid                #查看磁盘UUID
vim /etc/fstab       #编辑开机自动挂载

    #UUID="磁盘vdc1对应的UUID"    /egova    ext4    defaults    1 1 

mount -a             #不必重起 执行该命令也可自动挂载
```

参考链接：https://www.cnblogs.com/witz/p/10919686.html