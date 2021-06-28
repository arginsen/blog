---
title: MySQL | SQL学习笔记
date: 2019-7-2 18:20:33
tags: 
- SQL 
- MySQL
photos:
    - /blog/img/mysql.jpeg
categories:
- notes
---
<br>
<!-- more -->

# SQL简介

+ sql是用于访问和处理数据库的标准的计算机语言。
+ sql语句用于取回和更新数据库的数据。sql可与数据库程序协同工作。
+ *sql是一门语言，mysql、sql server、oracle是数据库程序*


+ 结构化查询语言
+ 使我们有能力访问数据库
+ 一种ANSI的标准计算机语言

# SQL作用

+ 面向数据库执行查询
+ 从数据库中取回数据
+ 在数据库中插入新的纪录
+ 更新数据库中的数据
+ 从数据库删除记录
+ 创建新数据库
+ 在数据库中创建新表
+ 在数据库中创建存储过程
+ 在数据库创建视图
+ 设置表、存储过程、视图的权限
&nbsp;
+ 搭建动态网站(RDBMS数据库程序+服务器端脚本语言PHP/ASP+SQL+HTML/CSS

# RDBMS

+ 是一种关系型数据库管理系统。是SQL的基础，也是很多数据库系统的基础。

+ RDBMS中的数据存储在表（数据库对象）中，表是相关数据项的集合，由列和行组成。

# 数据库表

一个数据库通常包含一个或多个表。每个表由一个名字标识，表包含带有数据的记录（行）

# SQL语句

SQL对大小写不敏感，有些数据库系统要在每条SQL命令**末端使用分号**。

## SQL分类之DML&DDL

SQL可分为两部分，数据操作语言（DML)和数据定义语言（DDL)

**DML部分**  :查询和更新

+ SELECT  从数据库表中获取数据
+ UPDATE  更新数据库表中的数据
+ DELETE  从数据库表中删除数据
+ INSERT INTO  向数据库表中插入数据

**DDL语句**  :创建、删除表格，定义索引、规定表间链接、表间约束

+ CREATE DATABASE 创建新数据库
+ ALER DATABASE 修改数据库
+ CREATE TABLE 创建新表
+ ALTER TABLE 变更数据库表
+ DROP TABLE 删除表
+ CREATE INDEX 创建索引
+ DROP INDEX 删除索引

## 语句

### SQL SELECT 语句

select语句用于从表中选取数据并将结果存储在一个结果表中。

```
SELECT 列名称 FROM 表名称
```


```
SELECT * FROM 表名称
```
  *意味表内所有列
eg:
```sql
SELECT lastname、firstname FROM Persons;
```


#### DISTINCT 关键词

```sql
SELECT DISTINCT 列名称 FROM 表名称;
```

如果从一列中选取所有的值，直接用SELECT语句；如果只需要提取种类剔除重复的值，则额外使用DISTINCT进行限制。

---

### SQL WHERE 子句

*有条件地*从表中选取数据。

```sql
SELECT 列名称 FROM 表名称 WHERE 列 运算符 值;
```


*运算符*：= 、<>（!=不等于）、<、>、<=、>=、BETWEEN 、LIKE


eg：
```sql
SELECT * FROM Persons WHERE City='Beijing';
```

其中
`值`：文本值用单引号，数值直接填写

---

### SQL AND&OR 运算符

可使用AND/OR在WHERE子语句中把*两个或多个条件*结合起来。

AND:
```sql
SELECT * FROM Persons WHERE firstname='Thomas' AND lastname='Carter';
```

OR:
```sql
SELECT * FROM Persons WHERE firstname='Thomas' OR lastname='Carter';
```

两者结合：
```sql
SELECT * FROM Persons WHERE (FirstName='Thomas' OR FirstName='William') AND LastName='Carter';
```

---

### SQL ORDER BY 子句

对选出的列对结果集进行排序（默认升序）。

以字母顺序显示公司名称：
```sql
SELECT Company, OrderNumber FROM Orders ORDER BY Company;
```


以字母顺序显示公司名称，以数字顺序显示序号：
```sql
SELECT Company, OrderNumber FROM Orders ORDER BY Company, OrderNumber;
```
  

#### DESC关键词

使指定的列对结果集按降序排列。

以逆字母顺序显示公司名称：
```sql
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC;
```


以逆字母顺序显示公司名称，并以数字顺序显示顺序号：
```
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC, OrderNumber ASC;
```

---

### SQL INSERT INTO 语句

用于向表格插入新的行。
```sql
INSERT INTO 表名称 VALUES（值1，值2，....）;
```


```
INSERT INTO table_name （列1，列2，...） VALUES (值1，值2，...);
```

---

### SQL UPDATE 语句

用于修改表中的数据（更新）

对表内的某行指定的一行的特定列进行修改：
```sql
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 原值/某值
```
#对于指定的值文本/数字加单引/不加

对本表某行中的多列修改，在WHERE后进行定位，在之前指定修改项：
```sql
UPDATE 表名称 SET 列1 = 新值1,...2,... WHERE 列名称 = 其中一原值/某值;
```

---

### SQL 删除语句

1. delete
  + delete是DML，执行delete操作时，每次从表中删除一行，并且同时将该行的的删除操作记录在redo和undo表空间中以便进行回滚（rollback）和重做操作，但要注意表空间要足够大，需要手动提交（commit）操作才能生效，可以通过rollback撤消操作。
  + delete可根据条件删除表中满足条件的数据，如果不指定where子句，那么删除表中所有记录。
  + delete语句不影响表所占用的extent，高水线(high watermark)保持原位置不变。
2. truncate
  + truncate是DDL，会隐式提交，所以，不能回滚，不会触发触发器。
  + truncate会删除表中所有记录，并且将重新设置高水线和所有的索引，缺省情况下将空间释放到minextents个extent，除非使用reuse storage，。不会记录日志，所以执行速度很快，但不能通过rollback撤消操作（如果一不小心把一个表truncate掉，也是可以恢复的，只是不能通过rollback来恢复）。
  + 对于外键（foreignkey ）约束引用的表，不能使用 truncate table，而应使用不带 where 子句的 delete 语句。
  + truncatetable不能用于参与了索引视图的表。
3. drop
  + drop是DDL，会隐式提交，所以，不能回滚，不会触发触发器。
  + drop语句删除表结构及所有数据，并将表所占用的空间全部释放。
  + drop语句将删除表的结构所依赖的约束，触发器，索引，依赖于该表的存储过程/函数将保留,但是变为invalid状态。

```sql
DELETE FROM 表名称 WHERE 列名称 = 值;      #从表中指定某列的值进行删除

DELETE FROM 表名称;                     #直接删除所有行

DELETE * FROM 表名称;          

DROP TABLE 表名称;                      #删除表（表的结构、属性以及索引也会被删除）

DROP DATABASE 数据库名称;               #删除数据库

TRUNCATE TABLE 表名称;                  #仅删除表格中的数据
```

---

### SQL TOP 子句

用于规定要返回的记录的数目

```sql
SELECT column_name(s) FROM table_name LIMIT number;
```

 
---

### SQL LIKE 操作符

用于在WHERE 子句中搜索列中的指定模式。

```sql
SELECT 某列/* FROM 表 WHERE 某列 LIKE '%g'/'%lon%;      # %为通配符，替代一个或多个字符
```
#意为从表中选出以 g结尾/包含lon 的列里的单词

```sql
SELECT * FROM 表 WHERE 某列 NOT LIKE '%lon';
```

---

### SQL 通配符

SQL 通配符可以替代一个或多个字。
SQL通配符必须与LIKE运算符一起使用

**字符**：
```
%                        代替一个或多个字符
_                        仅代替一个字符
[charlist]               字符列中的任何单一字符
[^charlist]/[!charlist]  不在字符列中的任何单一字符
```

**语法**：
```sql
SELECT * FROM 表 WHERE 某列 LIKE '%xxx';     #从某表的该列选出以xxx结尾的所有单词

SELECT * FROM 表 WHERE 某列 LIKE '_xxx';     #从某表的该列选出第一字符后为xxx的所有单词

SELECT * FROM 表 WHERE 某列 LIKE '[ALN]%';   #从某表的该列选出以A、L、N开头的单词

SELECT * FROM 表 WHERE 某列 LIKE '[!ALN]%';  #从某表的该列选出不以A、L、N开头的单词
```

---

### SQL IN 操作符

可在WHERE子句中规定多个值

```sql
SELECT 列 FROM 表 WHERE 列 IN (值1、值2,...);   #值为非数字时要加单引号
```
意为从某表的某列选出有该两值的一列或全部信息（具体定位）

---

### SQL BETWEEEN 操作符

选取两值之间的数据范围

```sql
SELECT 列 FROM 表 WHERE 列(独立于前一个列选取) BETWEEN 值1 AND 值2; 
```
其中值可以是数值，也可以是文本、日期。
当为文本时，有的数据库会选取包含值1但不包括值2、有些则都不包括，视情况。其实就是开闭区间的设定

当需要显示范围之外的值，在BETWEEN前加NOT

---

### SQL UNION 和 UNION ALL 操作符

用于合并两个或多个SELECT语句的结果集
！UNION内的SELECT 语句必须拥有相同数量、相似数据类型的列，语句中的列的顺序也必须相同

UNION默认选取不同的值，若几项有重复的值则使用UNION ALL

**SQL UNION**
```sql
SELECT  FROM 表1 UNION 列 FROM 表2;
```

**SQL UNION ALL**
```sql
SELECT  FROM 表1 UNION ALL 列 FROM 表2;
```

---

### SQL SELECT INTO 语句

+ 从一个表中选取数据，再把数据插入到另一表中
+ 常用于创建表的备份件，或对记录进行存档

```sql
SELECT */某列 INTO 新表 IN '外部数据库' FROM 旧表;
```

+ 备份表：

```sql
SELECT * INTO table_backup FROM table;
```

+ 向另一个数据库拷贝表：

```sql
SELECT * INTO 新表 IN '数据库名.mdb';
```

---

### SQL CREATE DATABASE 语句

```sql
CREATE DATABASE 库名;
```

---

### SQL CREATE TABLE 语句

```sql
CREATE TABLE 表名
(
  列1  数据类型,      #int/varchar/char/tinyint等都表示数据类型
  列2  数据类型,
  ...
);
```

#### 数据类型

integer(size)
int(size)              #仅容纳整数。在括号内规定数字的最大位数。
smallint(size)

tinyint(size)          #容纳带有小数的数字。

decimal(size,d)
numeric(size,d)        #"size" 规定数字的最大位数。"d" 规定小数点右侧的最大位数。

char(size)	           #容纳固定长度的字符串（可容纳字母、数字以及特殊字符）。在括号中规定字符串的长度。

varchar(size)	         #容纳可变长度的字符串（可容纳字母、数字以及特殊的字符）。在括号中规定字符串的最大长度。

date(yyyymmdd)	       #容纳日期。

---

### SQL 约束

约束用于限制加入表的数据的类型。
**可在创建表时规定约束，也可在之后用ALTER TABLE进行约束**

#### 约束种类：
NOT NULL
UNIQUE
PRIMARY KEY
FOREIGN KEY
CHECK
DEFAULT

<br>

##### SQL NOT NULL约束

约束强制列不接受NULL值，约束强制字段始终包含值。
```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255)
);
```

---

##### SQL UNIQUE约束

+ UNIQUE 约束唯一标识数据库表中的每条记录。
+ UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。
+ PRIMARY KEY 拥有自动定义的 UNIQUE 约束。
+ 每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

1. 创建表时

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
UNIQUE (Id_P)
);
```

2. 表已创建时

```sql
ALTER TABLE 该表 ADD UNIQUE (约束列);              #给某列添加UNIQUE约束

ALTER TABLE Persons ADD CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName);         #命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，CONSTRAINT后为添加外键的表的列

ALTER TABLE 表 DROP INDEX uc_PersonID;             # 撤销UNIQUE约束
```

---

##### SQL PRIMARY KEY约束

+ PRIMARY KEY 约束唯一标识数据库中的每条记录
+ 主键必须包含唯一的值
+ 主键列不能包含NULL值
+ 每个表都应该有一个主键，只有一个

1. 创建表时添加主键

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (Id_P)               #将Id_P作为主键
);

###########

CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)        #命名主键约束，为多个列定义主键约束，CONSTRAINT后为对应外键的表的列
);
```

2. 表已存在的情况下

```sql
ALTER TABLE Persons ADD PRIMARY KEY (Id_P);        #给Id_P创建主键约束

ALTER TABLE Persons ADD CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName);       #命名主键约束，为多个列定义主键约束

ALTER TABLE Persons DROP PRIMARY KEY;               #撤销主键约束
```

---

##### SQL FOREIGN KEY约束

+ 一个表中的FOREIGN KEY指向另一表的PRIMARY KEY.
+ FOREIGN KEY 约束用于预防破坏表之间连接的动作。
+ FOREIGN KEY 约束也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一(一个主键可以对应多个外键)。

1. 创建表时添加外键

```sql
CREATE TABLE Orders
(
Id_O int NOT NULL,
OrderNo int NOT NULL,
Id_P int,
PRIMARY KEY (Id_O),
FOREIGN KEY (Id_P) REFERENCES Persons(Id_P)    在Orders表给创建外键对应 Persons的列Id_P
);

#############

CREATE TABLE Orders
(
Id_O int NOT NULL,
OrderNo int NOT NULL,
Id_P int,
PRIMARY KEY (Id_O),
CONSTRAINT fk_PerOrders FOREIGN KEY (Id_P)
REFERENCES Persons(Id_P)               #命名 FOREIGN KEY 约束，以及为多个列定义 FOREIGN KEY 约束
);
```
2. 表已创建时添加

```sql
ALTER TABLE Orders ADD FOREIGN KEY (Id_P) REFERENCES Persons(Id_P);           #为 "Id_P" 列创建 FOREIGN KEY 约束

ALTER TABLE Orders ADD CONSTRAINT fk_PerOrders FOREIGN KEY (Id_P) REFERENCES Persons(Id_P);       #命名 FOREIGN KEY 约束，以及为多个列定义 FOREIGN KEY 约束

ALTER TABLE Orders DROP FOREIGN KEY fk_PerOrders;            #撤销外键约束
```

---

##### SQL CHECK 约束

CHECK约束用于限制列中的**值的范围**
+ 如果对单个列定义 CHECK 约束，那么该列只允许特定的值。
+ 如果对一个表定义 CHECK 约束，那么此约束会在特定的列中对值进行限制。

1. 创建表时添加check约束

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CHECK (Id_P>0)          #如果是sql server/oracle/ms access，则直接将约束写在目标列后
);

#################

CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),                                           #此类方式适用于mysql/sql server/oracle/
CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes')      #命名check约束以及定义多个列check约束
);
```

2. 表已存在

```sql
ALTER TABLE Persons ADD CHECK (Id_P>0);        #给Id_P创建check约束，规定该列值的范围

ALTER TABLE Persons ADD CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes');     #命名 CHECK 约束，以及为多个列定义 CHECK 约束

ALTER TABLE Persons DROP CHECK chk_Person;     #撤销check约束
```

---

##### SQL DEFAULT 约束

DEFAULT约束用于向列中插入默认值，如果没有规定其他值则添加默认值

```sql
创建表：
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255) DEFAULT 'Sandnes'      #默认在城市一栏填入悉尼
);

###############
已有表：
ALTER TABLE Persons ALTER City SET DEFAULT 'SANDNES';    #Persons表City栏默认写入SANDNES

ALTER TABLE Persons ALTER City DROP DEFAULT;             #撤销DEFAULT值
```

---

### SQL CREATE INDEX

CREATE INDEX 语句用于在表中创建索引。
+ 在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据。

```sql
CREATE INDEX index_name ON table_name (column_name);          #创建多个索引时，列之间用逗号隔开

CREATE UNIQUE INDEX index_name ON table_name (column_name);   #创建一个唯一的索引(两个行不能拥有相同的索引值)

CREATE INDEX index_name ON table_name (column_name DESC);     #DESC表示降序索引列的值

ALTER TABLE table_name DROP INDEX index_name;                  #撤销索引
```

---

### SQL 视图

在 SQL 中，视图是基于 SQL 语句的结果集的可视化表。
+ 视图包含行和列，就像真正的表一样。视图中的字段是一个或多个数据库中真实表中的字段。
+ 可以添加 SQL 函数，在哪里添加，并将语句连接到视图
+ 可以用于呈现数据，就像数据来自单个表一样。

```sql
CREATE VIEW view_name AS                    #视图用[]包裹
SELECT column_name(s)                
FROM table_name                
WHERE 定位;                                 #xx='xx'

SELECT * FROM [xxxxxx];      #查询视图

CREATE OR REPLACE VIEW view_name AS                
SELECT column_name(s)                
FROM table_name                
WHERE 定位;    

DROP VIEW view_name;         #删除视图
```

---

### SQL ALTER TABLE 语句

用于在**已有的表**中添加、修改、删除列

```sql
ALTER TABLE table_name ADD column_name datatype;            #在表中添加列，指定数据类型

ALTER TABLE table_name DROP column_name;                    #删除表中的列

ALTER TABLE table_name ALTER COLUMN column_name datatype;    #改变表中的数据类型
```

---

### SQL AUTO INCREMENT 语句

插入新记录时主动创建主键字段的值

```sql
CREATE TABLE Persons
(
P_Id int NOT NULL AUTO_INCREMENT,            //将"P_Id" 列定义为auto-increment主键
LastName varchar(255) NOT NULL,              //mysql用AUTO_INCREMENT来执行auto-increment任务
FirstName varchar(255),                      //可以使用sql语法改变AUTO_INCREMENT的起始值：
Address varchar(255),                        //ALTER TABLE Persons AUTO_INCREMENT=100
City varchar(255),                           //在该表中新插入记录时，不用给P_ID列添加，它会自动添加一个唯一的值
PRIMARY KEY (P_Id)                           
);
```

---

### SQL Date 函数

MySQL 内建日期函数：
```sql
函数	          描述
NOW()	         返回当前的日期和时间
CURDATE()	     返回当前的日期
CURTIME()	     返回当前的时间
DATE()	       提取日期或日期/时间表达式的日期部分
EXTRACT()	     返回日期/时间按的单独部分
DATE_ADD()	   给日期添加指定的时间间隔
DATE_SUB()	   从日期减去指定的时间间隔
DATEDIFF()	   返回两个日期之间的天数
DATE_FORMAT()	 用不同的格式显示日期/时间
```

日期查找：
```sql
SELECT * FROM 表名称 WHERE Date='2019-08-13';       #针对不含具体时间
```

---

### SQL NULL 值

+ NULL值是遗漏的未知数据。默认表的列可以存放NULL值。
+ NULL用作未知的或不适用的值的占位符。
+ 无法比较NULL和0，两者不等价。

```sql
使用is null选取NULL的记录：
SELECT LastName,FirstName,Address FROM Persons WHERE Address IS NULL;   

使用is not null来选取不带有null值的记录：
SELECT LastName,FirstName,Address FROM Persons WHERE Address IS NOT NULL;
```

<br>