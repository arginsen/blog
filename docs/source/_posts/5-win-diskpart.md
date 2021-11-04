---
title: win10 | 硬盘重插后 recovery 盘驱动器号改动
date: 2020-10-13 13:23:33
tags: 
- win10
categories: practice
photo: 
    - /blog/img/win10.jpg
---

<br>
<!-- more -->

> 笔记本拆机之后，测试了下另一块硬盘，发现那块硬盘是好的，是它的接口坏掉了，于是又买了个新的托盘；但是我原来的硬盘装回去后，原本 recovery(E:) 变成了 F 盘，F 盘变成了 E，而我的软件在 D 盘，且我大部分文件保存路径都是 F 盘，看了下普通盘可以随便换驱动器号，但是 recovery 不行。找到了惠普工作人员发的贴子

CMD 下使用命令 diskpart 替换：

```
C:\Users\xx>diskpart

Microsoft DiskPart 版本 10.0.18362.1

Copyright (C) Microsoft Corporation.
在计算机上: ARGINSEN

DISKPART> list vol

  卷 ###      LTR  标签         FS     类型        大小     状态       信息
  ----------  ---  -----------  -----  ----------  -------  ---------  --------
  卷     0     D   应用         NTFS   磁盘分区      319 GB  正常
  卷     1     G   个人         NTFS   磁盘分区      299 GB  正常
  卷     2     H   文件         NTFS   磁盘分区      299 GB  正常
  卷     3     F   RECOVERY     NTFS   磁盘分区       11 GB  正常
  卷     4     C   Windows      NTFS   磁盘分区      117 GB  正常         启动
  卷     5                      FAT32  磁盘分区      260 MB  正常         系统

DISKPART> sel vol f
卷 3 是所选卷。

DISKPART> remove letter=f
DiskPart 成功地删除了驱动器号或装载点。

DISKPART> assign letter=e
DiskPart 成功地分配了驱动器号或装载点。

DISKPART>
```
