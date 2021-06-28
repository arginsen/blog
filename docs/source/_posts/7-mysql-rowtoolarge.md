---
title: MySQL | 导数据库报错"Row size too large (> 8126)"
date: 2019-12-04 17:30:21
tags: MySQL
photos:
    - /blog/img/mysql.jpeg
categories:
- practice
---
<br>
<!-- more -->

在导库过程中出现"Row size too large (> 8126). Changing some columns to TEXT or BLOB may help."

检查发现其中的to_law_rec表的建表语句插入的列(这里的row)过多，有237行字段，包含大量varchar。
查看网上说可能是引擎的缘故

**解决办法**：
将表引擎从IoonDB换成MyISAM,完美解决.
```
ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```