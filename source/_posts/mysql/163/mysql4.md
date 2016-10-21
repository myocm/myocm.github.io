---
title: SQL语言简介
list_number: false
categories:
  - MySQL
comments: true
description:
  - SQL语言简介
date: 2016-10-13 15:58:43
updated: 2016-10-13 15:58:43
tags:
---
# SQL语言简介

## SQL是什么
- Structured Query Language 的首字母缩写
- 用于关系数据库中的数据存取操作
- 与数据库沟通的钥匙

## SQL语言与数据库
- 用SQL创建表，定义表的字段
- 用SQL向表添加，删除，修改记录
- 用SQL从表中查询出想要的记录
- 用SQL操作数据库的一切......

## SQL语句的分类
SQL语句的分类 | 大致用途
--|--
DDL（Data Definition Language）  | 创建数据库，创建表，修改表，删除表等
DML（Data Manipulation Language）| 向表中增加，修改，删除记录，取出记录
DCL（Data Control Language ）    | 控制数据库的访问权限等设置
TCL（Transaction Control Language ） | 控制事务进展


- DDL语句
CREATE TABLE
ALTER TABLE
DROP TABLE


- DML语句
SELECT FROM TABLE
INSERT INTO TABLE
UPDATE TABLE SET
DELETE FROM TABLE


- DCL语句
GRANT
REVOKE


- TCL语句
COMMIT
ROLLBACK

示例：
DDL:
```sql
CREATE TABLE friend(
id int(11) unsigned not null auto_increment,
name varchar(50),
age int(11),
gender varchar(5),
PRIMARY KEY(id)
);
```

DML:
```sql
INSERT INTO friend(name,age,gender) VALUES('Tom',16,'male');
UPDATE friend SET age=19 WHERE name='Tom';
SELECT * FROM friend;
DELETE FROM friend WHERE name='Tom';
```

## SQL语言在MySQL上的基本操作
### 1. 连接和断开MySQL
```sql
shell> mysql -h host -u user -p

Enter password: ********

QUIT 或 \q
Bye
```

### 2. 输入查询
```sql
SELECT USER(),CURRENT_DATE(),CURRENT_TIME(),CURRENT_TIMESTAMP() ;
+----------------+----------------+----------------+---------------------+
| USER()         | CURRENT_DATE() | CURRENT_TIME() | CURRENT_TIMESTAMP() |
+----------------+----------------+----------------+---------------------+
| root@localhost | 2015-07-13     | 11:12:37       | 2015-07-13 11:12:37 |
+----------------+----------------+----------------+---------------------+
1 row in set (0.00 sec)
```
- 可以当作计算器
```sql
 SELECT SIN(PI()/4), (4+1)*5;
+--------------------+---------+
| SIN(PI()/4)        | (4+1)*5 |
+--------------------+---------+
| 0.7071067811865475 |      25 |
+--------------------+---------+
1 row in set (0.08 sec)
```
- 多个查询同时执行
```sql
 SELECT VERSION(); SELECT NOW();
+-----------+
| VERSION() |
+-----------+
| 5.6.25    |
+-----------+
1 row in set (0.00 sec)

+---------------------+
| NOW()               |
+---------------------+
| 2015-07-13 11:29:54 |
+---------------------+
1 row in set (0.00 sec)
```
- 取消本次查询
```sql
 select user()
    -> \c

```
- 提示符含义

| Prompt   | Meaning     |
-------------|-------------
|       | Ready for new command.     |
| ->  |   Waiting for next line of multiple-line command.      |
| '>     |  Waiting for next line, waiting for completion of a string that began with a single quote (“'”).    |
| ">     |  Waiting for next line, waiting for completion of a string that began with a double quote (“"”).    |
| `>     |  Waiting for next line, waiting for completion of an identifier that began with a backtick (“`”).    |
|  /*>    | Waiting for next line, waiting for completion of a comment that began with /*.     |

### 3. 创建和使用数据库
- 显示数据库
```sql
 show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
5 rows in set (0.00 sec)
```
- 创建和选择一个数据库
```sql
--选择一个数据库，设置为当前要使用的数据库
 use asiainfo
Database changed


--从shell直接登录到某个数据库
shell> mysql -u root -p test
 select database();
+------------+
| database() |
+------------+
| test       |
+------------+
1 row in set (0.00 sec)

--此处的test 是你要直接登录的数据库名，而不是密码，如果要直接输入密码，需要直接跟在-p后面，不能有空格
shell> mysql -u root -p123456 test
--但是不推荐这种方式，这样会暴露密码。
```
- 建表
```sql
 show tables;
Empty set (0.00 sec)

 CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20),
    -> species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);
Query OK, 0 rows affected (0.03 sec)

 show tables;
+--------------------+
| Tables_in_asiainfo |
+--------------------+
| pet                |
+--------------------+
1 row in set (0.00 sec)
```
- 查看表结构
```sql
 describe pet; 或 desc pet;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name    | varchar(20) | YES  |     | NULL    |       |
| owner   | varchar(20) | YES  |     | NULL    |       |
| species | varchar(20) | YES  |     | NULL    |       |
| sex     | char(1)     | YES  |     | NULL    |       |
| birth   | date        | YES  |     | NULL    |       |
| death   | date        | YES  |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+
6 rows in set (0.01 sec)
```
- 显示建表语句
```sql
 show create table pet\G
*************************** 1. row ***************************
       Table: pet
Create Table: CREATE TABLE `pet` (
  `name` varchar(20) DEFAULT NULL,
  `owner` varchar(20) DEFAULT NULL,
  `species` varchar(20) DEFAULT NULL,
  `sex` char(1) DEFAULT NULL,
  `birth` date DEFAULT NULL,
  `death` date DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
```
- 用insert 语句插入新的记录
```sql
 insert into pet
    -> values ('Puffball','Diane','hamster','f','1999-03-30',null);
Query OK, 1 row affected (0.01 sec)
```
- 从表中检索信息
```sql
 select * from pet;
+----------+--------+---------+------+------------+------------+
| name     | owner  | species | sex  | birth      | death      |
+----------+--------+---------+------+------------+------------+
| Fluffy   | Harold | cat     | f    | 1993-02-04 | NULL       |
| Claws    | Gwen   | cat     | m    | 1994-03-17 | NULL       |
| Buffy    | Harold | dog     | f    | 1989-05-13 | NULL       |
| Fang     | Benny  | dog     | m    | 1990-08-27 | NULL       |
| Bowser   | Diane  | dog     | m    | 1979-08-31 | 1995-07-29 |
| Chirpy   | Gwen   | bird    | f    | 1998-09-11 | NULL       |
| Whistler | Gwen   | bird    | NULL | 1997-12-09 | NULL       |
| Slim     | Benny  | snake   | m    | 1996-04-29 | NULL       |
| Puffball | Diane  | hamster | f    | 1999-03-30 | NULL       |
+----------+--------+---------+------+------------+------------+
9 rows in set (0.00 sec)
```
- 更新记录
```sql
 UPDATE pet SET birth = '1989-08-31' WHERE name = 'Bowser';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

 select * from pet;
+----------+--------+---------+------+------------+------------+
| name     | owner  | species | sex  | birth      | death      |
+----------+--------+---------+------+------------+------------+
| Fluffy   | Harold | cat     | f    | 1993-02-04 | NULL       |
| Claws    | Gwen   | cat     | m    | 1994-03-17 | NULL       |
| Buffy    | Harold | dog     | f    | 1989-05-13 | NULL       |
| Fang     | Benny  | dog     | m    | 1990-08-27 | NULL       |
| Bowser   | Diane  | dog     | m    | 1989-08-31 | 1995-07-29 |
| Chirpy   | Gwen   | bird    | f    | 1998-09-11 | NULL       |
| Whistler | Gwen   | bird    | NULL | 1997-12-09 | NULL       |
| Slim     | Benny  | snake   | m    | 1996-04-29 | NULL       |
| Puffball | Diane  | hamster | f    | 1999-03-30 | NULL       |
+----------+--------+---------+------+------------+------------+
9 rows in set (0.00 sec)
```
- 查询部分行
```sql
 SELECT * FROM pet WHERE name = 'Bowser';
+--------+-------+---------+------+------------+------------+
| name   | owner | species | sex  | birth      | death      |
+--------+-------+---------+------+------------+------------+
| Bowser | Diane | dog     | m    | 1989-08-31 | 1995-07-29 |
+--------+-------+---------+------+------------+------------+
1 row in set (0.00 sec)
```
- 查询部分列
```sql
 SELECT name, birth FROM pet;
+----------+------------+
| name     | birth      |
+----------+------------+
| Fluffy   | 1993-02-04 |
| Claws    | 1994-03-17 |
| Buffy    | 1989-05-13 |
| Fang     | 1990-08-27 |
| Bowser   | 1989-08-31 |
| Chirpy   | 1998-09-11 |
| Whistler | 1997-12-09 |
| Slim     | 1996-04-29 |
| Puffball | 1999-03-30 |
+----------+------------+
9 rows in set (0.00 sec)

 SELECT DISTINCT owner FROM pet;
+--------+
| owner  |
+--------+
| Harold |
| Gwen   |
| Benny  |
| Diane  |
+--------+
4 rows in set (0.00 sec)
```
- 用where条件来混合列选择和行选择
```sql
 SELECT name, species, birth FROM pet
    -> WHERE species = 'dog' OR species = 'cat';
+--------+---------+------------+
| name   | species | birth      |
+--------+---------+------------+
| Fluffy | cat     | 1993-02-04 |
| Claws  | cat     | 1994-03-17 |
| Buffy  | dog     | 1989-05-13 |
| Fang   | dog     | 1990-08-27 |
| Bowser | dog     | 1989-08-31 |
+--------+---------+------------+
5 rows in set (0.00 sec)
```
- 行排序
```sql
 SELECT name, birth FROM pet ORDER BY birth;
+----------+------------+
| name     | birth      |
+----------+------------+
| Buffy    | 1989-05-13 |
| Bowser   | 1989-08-31 |
| Fang     | 1990-08-27 |
| Fluffy   | 1993-02-04 |
| Claws    | 1994-03-17 |
| Slim     | 1996-04-29 |
| Whistler | 1997-12-09 |
| Chirpy   | 1998-09-11 |
| Puffball | 1999-03-30 |
+----------+------------+
9 rows in set (0.00 sec)
```












----
Good luck!  
冬日暖阳
