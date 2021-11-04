---
title: MySQL | MySQL更改系统部门的执法主体与所属区域
date: 2019-12-18 17:24:21
tags: MySQL
photos:
    - /blog/img/mysql.jpeg
categories:
- practice
---
<br>
<!-- more -->

分层级是初步摸索的sql语句，整体是对于分层sql的总结与封装

# 分层级更新

层级说明：
- 衡阳市
	- 衡阳市直各部门-衡阳市水利局-局安全信息科
	- 衡山县-衡山县直各部门-衡山县统计局

## 对市级的下下级进行改动

1. 对市下下级部门region_id的改动
 
```
-- 查找senior_id为1的各部门的unit_id和region_id
SELECT unit_id,unit_name from tc_unit where  senior_id = 1 AND delete_flag != 1 

-- 筛选出senior_id为37（衡阳市）的下下级
SELECT  unit_id,unit_name,region_id 
FROM    tc_unit a
WHERE   a.senior_id 
IN (SELECT unit_id 
	FROM   (SELECT a.unit_id
			FROM tc_unit a
			WHERE a.senior_id = 37 
			AND a.delete_flag != 1 ) t)

-- 更新语句 1.123s
UPDATE  tc_unit a 
set     region_id = 5 
WHERE   a.senior_id 
IN (SELECT unit_id 
	FROM   (SELECT a.unit_id
			FROM tc_unit a
			WHERE a.senior_id = 37 
			AND a.delete_flag != 1 ) t)

-- 优化更新语句 
UPDATE  tc_unit a
JOIN   (SELECT a.unit_id
		FROM tc_unit a 
		WHERE a.senior_id = 37 
		AND a.delete_flag != 1) t
ON a.senior_id = t.unit_id
SET     region_id = 5
```
<!--more-->
2. 对市下下级部门law_subject_id的改动

```
UPDATE tc_unit a 
set law_subject_id = 
WHERE a.senior_id 
IN (SELECT unit_id 
	FROM tc_unit 
	WHERE senior_id = 37 
	AND delete_flag != 1 )
```

## 对市级的下下下级进行改动

1. 对市下下下级部门region_id的改动
```sql
-- 筛选出senior_id为37（衡阳市）的下下下级
SELECT  unit_id,unit_name,region_id 
FROM    tc_unit a
WHERE   a.senior_id 
IN (SELECT unit_id
	FROM   (SELECT a.unit_id 
			FROM tc_unit a 
			WHERE a.senior_id 
			IN (SELECT b.unit_id
				FROM tc_unit b
				WHERE b.senior_id = 37 
				AND b.delete_flag != 1 ) ) y)

-- -- 更新语句 1.285s
UPDATE  tc_unit a 
set 	region_id = 5 
WHERE 	a.senior_id 
IN (SELECT unit_id
	FROM   (SELECT a.unit_id 
			FROM tc_unit a 
			WHERE a.senior_id 
			IN (SELECT b.unit_id
				FROM tc_unit b
				WHERE b.senior_id = 37 
				AND b.delete_flag != 1 ) ) y)
```


# 整体层级更新

1. 所属区域的改动
```sql
-- 查询
SELECT unit_id,unit_name,region_id 
FROM tc_unit a
WHERE a.senior_id = 37 
AND a.delete_flag != 1 
OR	  a.senior_id 
IN (SELECT unit_id 
	FROM   (SELECT a.unit_id
			FROM tc_unit a
			WHERE a.senior_id = 37 
			AND a.delete_flag != 1 ) t)
OR	  a.senior_id 
IN (SELECT unit_id
	FROM   (SELECT a.unit_id 
			FROM tc_unit a 
			WHERE a.senior_id 
			IN (SELECT b.unit_id
				FROM tc_unit b
				WHERE b.senior_id = 37 
				AND b.delete_flag != 1 ) ) y)

-- 更新
UPDATE  tc_unit a 
set 	region_id = 5 
WHERE a.senior_id = 37 
AND a.delete_flag != 1 
OR	  a.senior_id 
IN (SELECT unit_id 
	FROM   (SELECT a.unit_id
			FROM tc_unit a
			WHERE a.senior_id = 37 
			AND a.delete_flag != 1 ) t)
OR	  a.senior_id 
IN (SELECT unit_id
	FROM   (SELECT a.unit_id 
			FROM tc_unit a 
			WHERE a.senior_id 
			IN (SELECT b.unit_id
				FROM tc_unit b
				WHERE b.senior_id = 37 
				AND b.delete_flag != 1 ) ) y)

```

2. 执法主体的改动
```sql
-- 查询
SELECT unit_name,region_id,unit_id,law_subject_id
FROM tc_unit a
WHERE a.senior_id = 1 
AND a.delete_flag != 1 
OR	  a.senior_id 
IN (SELECT unit_id 
	FROM   (SELECT a.unit_id
			FROM tc_unit a
			WHERE a.senior_id = 1 
			AND a.delete_flag != 1 ) t)
OR	  a.senior_id 
IN (SELECT unit_id
	FROM   (SELECT a.unit_id 
			FROM tc_unit a 
			WHERE a.senior_id 
			IN (SELECT b.unit_id
				FROM tc_unit b
				WHERE b.senior_id = 1 
				AND b.delete_flag != 1 ) ) y)
				
-- 更新
UPDATE  tc_unit a 
set law_subject_id = unit_id
WHERE a.senior_id = 1 
AND a.delete_flag != 1 
OR	  a.senior_id 
IN (SELECT unit_id 
	FROM   (SELECT a.unit_id
			FROM tc_unit a
			WHERE a.senior_id = 1 
			AND a.delete_flag != 1 ) t)
OR	  a.senior_id 
IN (SELECT unit_id
	FROM   (SELECT a.unit_id 
			FROM tc_unit a 
			WHERE a.senior_id 
			IN (SELECT b.unit_id
				FROM tc_unit b
				WHERE b.senior_id = 1 
				AND b.delete_flag != 1 ) ) y)
```