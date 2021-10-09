---
title: Linux | centos7将centos-home磁盘容量分一部分给centos-root
date: 2019-12-18 17:24:21
tags: 
- centos
- Linux
categories: practice
photo: 
    - /blog/img/linux1.jpeg
---

<br>
<!-- more -->


>centos-root也就是服务器文件系统的根目录

1. 查看分区

```           
[root@localhost ~]# df -h      #centos-home和centos-root每人的名字可能不一样
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  500G  3.4G  497G   1% /            #根目录只有500G大小
devtmpfs                 126G     0  126G   0% /dev
tmpfs                    126G     0  126G   0% /dev/shm
tmpfs                    126G   35M  126G   1% /run
tmpfs                    126G     0  126G   0% /sys/fs/cgroup
/dev/sda2                100G  142M  100G   1% /boot
/dev/sda1                200M   12M  189M   6% /boot/efi
/dev/mapper/centos-home  2.7T   34M  2.7T   1% /home        #这里显示大小为2.7T
tmpfs                     26G     0   26G   0% /run/user/0


vgdisplay        #查看空闲磁盘大小,计划分多少出去，到时候又补多少
```

2. 备份home分区文件

```
tar cvf /tmp/home.tar home     #进入根目录打包
```

3. 卸载/home，如果无法卸载，先终止使用/home文件系统的进程

```
umount /home                   #卸载
fuser -km /home                #终止/home文件系统的进程

lsof | grep /home              #查找使用/home的进程，再kill掉
```

4. 删除/home所在的lv

```
lvremove /dev/mapper/centos-home     #
```

5. 扩展/root所在的lv

```
lvextend -L +2000G /dev/mapper/centos-root
```

6. 扩展/root文件系统

```
xfs_growfs /dev/mapper/centos-root
```

7. 重新创建home lv 

这里需要计算一下，Free PE乘以单个的PE Size就是目前空闲的磁盘(Free PE旁边显示的)，接着在下边创建lv

```
[root@localhost ~]# vgdisplay           #查看空闲磁盘
  --- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  6
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <3.18 TiB
  PE Size               4.00 MiB
  Total PE              832445
  Alloc PE / Size       772096 / <2.95 TiB
  Free  PE / Size       60349 / <235.74 GiB
  VG UUID               lSHABr-yKHy-vUzC-RIyV-rQMx-V4ej-hyCx1r

lvcreate -L 235.7G -n /dev/mapper/centos-home   #计算有235.7G，创建home lv
```

8. 创建文件系统

```
mkfs.xfs /dev/mapper/centos-home
```

9. 挂载home

```
mount /dev/mapper/centos-home
```

10. home文件恢复

```
tar xvf /tmp/home.tar -C /home
```