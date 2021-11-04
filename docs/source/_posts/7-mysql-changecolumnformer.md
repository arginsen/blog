---
title: MySQL | MySQL更改某列值的前几位
date: 2019-12-04 10:30:21
tags: MySQL
photos:
    - /blog/img/mysql.jpeg
categories:
- practice
---
<br>
<!-- more -->

例：在导入人员时，个别人员的监督证号为"湘4004900003",本应该为"湘04004900003"。
原因是在添加前边"湘"字时将后边的0给复制没了，但录入系统后，显示湘后少0。
所以需要在数据库对tc_human表操作，令其中湘后没0的补充0。
语句：
```sql
select work_cert_code from tc_human;         #粗略查看监督证号

SELECT unit_name,duty_name,work_cert_code FROM tc_human 
WHERE work_cert_code LIKE '湘4%';            #用通配符查看湘4%的人员有多少个

UPDATE tc_human 
SET work_cert_code = concat( '湘04', mid( work_cert_code, 3, 9 ) ) 
WHERE work_cert_code LIKE '湘4%';            #将监督证号'湘4%'的第3-9位保持原值，湘后加0
```