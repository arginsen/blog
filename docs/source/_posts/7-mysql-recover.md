---
title: MySQL | MySQL配置binlog恢复库以及定期生成binlog
date: 2020-1-20 17:24:21
tags: MySQL
photos:
    - /blog/img/mysql.jpeg
categories:
- practice
---
<br>
<!-- more -->

增量备份的 binlog 是一个二进制格式的文件,用于记录用户对数据库更新的 SQL 语句信息,例如更改数据库表和更改内容的 SQL 语句都会记录到 binlog 里,但是对库表等内容的查询不会记录。

# 使用binlog恢复库

```sql
-- 查看log_bin是否开启
mysql> show variables like '%log_bin%';  
+---------------------------------+------------------------+
| Variable_name                   | Value                  |
+---------------------------------+------------------------+
| log_bin                         | ON                     |
| log_bin_basename                | /egova/db/binlog       |
| log_bin_index                   | /egova/db/binlog.index |
| log_bin_trust_function_creators | ON                     |
| log_bin_use_v1_row_events       | OFF                    |
| sql_log_bin                     | ON                     |
+---------------------------------+------------------------+

-- 查看目前的binlog
mysql> show binary logs;
+---------------+-----------+
| Log_name      | File_size |
+---------------+-----------+
| binlog.000003 | 346226822 |
| binlog.000004 |       199 |
| binlog.000005 |       178 |
| binlog.000006 |       178 |
| binlog.000007 |    847112 |
+---------------+-----------+

-- 查看其中的一个log
mysql> show binlog events in 'binlog.000015';
+---------------+-----+----------------+-----------+-------------+-----------------------------------+
| Log_name      | Pos | Event_type     | Server_id | End_log_pos | Info                              |
+---------------+-----+----------------+-----------+-------------+-----------------------------------+
| binlog.000015 |   4 | Format_desc    |         1 |         124 | Server ver: 8.0.18, Binlog ver: 4 |
| binlog.000015 | 124 | Previous_gtids |         1 |         155 |                                   |
+---------------+-----+----------------+-----------+-------------+-----------------------------------+

-- (插入测试数据)
mysql> create table student ( id int(11), name varchar(15), gender varchar(5), date varchar(5), major varchar(15), address varchar(20) );
INSERT INTO student VALUES (901, '张老大', '男', 1985, '计算机系', '北京市海淀区');
INSERT INTO student VALUES (902, '张老二', '男', 1986, '中文系', '北京市昌平区');
INSERT INTO student VALUES (903, '张三', '女', 1990, '中文系', '湖南省永州市');
INSERT INTO student VALUES (904, '李四', '男', 1990, '英语系', '辽宁省阜新市');
INSERT INTO student VALUES (905, '王五', '女', 1991, '英语系', '福建省厦门市');
INSERT INTO student VALUES (906, '王六', '男', 1988, '计算机系', '湖南省衡阳市');

mysql> select * from student;
+-----+--------+--------+------+----------+--------------+
| id  | name   | gender | date | major    | address      |
+-----+--------+--------+------+----------+--------------+
| 901 | 张老大 | 男     | 1985 | 计算机系 | 北京市海淀区 |
| 902 | 张老二 | 男     | 1986 | 中文系   | 北京市昌平区 |
| 903 | 张三   | 女     | 1990 | 中文系   | 湖南省永州市 |
| 904 | 李四   | 男     | 1990 | 英语系   | 辽宁省阜新市 |
| 905 | 王五   | 女     | 1991 | 英语系   | 福建省厦门市 |
| 906 | 王六   | 男     | 1988 | 计算机系 | 湖南省衡阳市 |
+-----+--------+--------+------+----------+--------------+

-- (误删数据)
mysql> delete from student where id = 905;
mysql> set binlog_format = statement;     -- 或者在my.ini/my.cnf中加入binlog_format = statement

-- 查看binlog记录
mysql> show binlog events in 'binlog.000017';
+---------------+-----+----------------+-----------+-------------+------------------------------------------------+
| Log_name      | Pos | Event_type     | Server_id | End_log_pos | Info                                           |
+---------------+-----+----------------+-----------+-------------+------------------------------------------------+
| binlog.000017 |   4 | Format_desc    |         1 |         124 | Server ver: 8.0.18, Binlog ver: 4              |
| binlog.000017 | 124 | Previous_gtids |         1 |         155 |                                                |
| binlog.000017 | 155 | Anonymous_Gtid |         1 |         234 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'           |
| binlog.000017 | 234 | Query          |         1 |         316 | BEGIN                                          |
| binlog.000017 | 316 | Query          |         1 |         427 | use `zfdb`; delete from student where id = 906 |
| binlog.000017 | 427 | Xid            |         1 |         458 | COMMIT /* xid=6 */                             |
| binlog.000017 | 458 | Anonymous_Gtid |         1 |         537 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'           |
| binlog.000017 | 537 | Query          |         1 |         619 | BEGIN                                          |
| binlog.000017 | 619 | Query          |         1 |         730 | use `zfdb`; delete from student where id = 905 |
| binlog.000017 | 730 | Xid            |         1 |         761 | COMMIT /* xid=12 */                            |
+---------------+-----+----------------+-----------+-------------+------------------------------------------------+

-- 将数据恢复到删除id 905前的一部，起点Pos234，终点427，终端下输入
mysqlbinlog --start-position=234 --stop-position=427 D:\Mysql\mysql-8.0.18-winx64\data\binlog.000017 | mysql -u root -p;


```

# binlog配置

修改配置文件my.ini（window系统环境下，若为Linux环境，则修改my.cnf文件）文件：
在`[mysqld]`标签内增加如下内容
expire_logs_days=7       #每7天生成一个log
max_binlog_size=500M     #binlog最大500M

除超出缓存限制还可手动执行`flush logs`
作用：重新启动时(MySQL将会new一个新文件用于记录binlog)

如果binlog非常多，不要轻易设置该参数，有可能导致io争用，这时候可以使用purge命令予以清除:

将bin.000055之前的binlog清掉:
```
mysql>purge binary logs to 'bin.000055';
```

参考链接：
https://www.jxtxzzw.com/archives/4496
https://blog.csdn.net/qq_40195432/article/details/84934804
https://www.cnblogs.com/linjiqin/p/11520052.html
https://www.cnblogs.com/kevingrace/p/6065088.html
