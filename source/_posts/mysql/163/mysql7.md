---
title: MySQL数据库对象
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL数据库对象
date: 2016-10-19 10:22:49
updated: 2016-10-19 10:22:49
tags:
---
# MySQL数据库对象

## 常见数据库对象
- DataBase/Schema
- Table
- Index
- View
- Trigger
- Function
- Procedure

### DataBase/Schema
- 库、表、行、层级关系
![](/img/markdown-img-paste-20161019102844684.png)
- 一个DataBase对应一个Schema
- 一个Schema包含一个或多个表
- 一个表包含一个或多个字段
- 一个表包含一条或多条记录
- 一个表包含一个或多个索引


- 多Database的用途
  - 业务隔离
  - 资源隔离

### Table
CREATE TABLE语法[官方参考文档](http://dev.mysql.com/doc/refman/8.0/en/create-table.html)
- 表上的常用数据对象包括
  - 索引
  - 约束
  - 视图
  - 触发器

### Index
- 索引是数据库中数据的目录
- 索引和数据是两个对象
- 索引主要用来提高数据库的查询效率
- 数据库中数据变更同样需要同步索引数据的变更

CREATE INDEX语法[官方参考文档](http://dev.mysql.com/doc/refman/8.0/en/create-index.html)

- 索引种类
  - 普通索引（CREATE INDEX index_name）
  - 主键索引（PRIMARY KEY）
  - 唯一索引（CREATE UNIQUE INDEX index_name）
  - 全文索引（CREATE FULLTEXT INDEX index_name）
  - 空间索引（CREATE SPATIAL INDEX index_name）

- 唯一约束
  - 唯一约束是一种特殊的索引
  - 唯一约束可以一个或者多个字段
  - 唯一约束可以在建表的时候建好，也可以以后再补上
  - 主键也是一种唯一约束

例如：
```sql
create table t ( id  int(11) unsigned not null primary key auto_increment,
    ->  orderid int(10) unsigned not null,
    ->  bookid int(10) unsigned not null default 0,
    ->  userid int(10) unsigned not null default 0,
    ->  orderdate datetime not null default now(),
    ->  status tinyint(3) unsigned zerofill default '000',
    ->  unique key idx_orderid(orderid),
    ->  unique key idx_uid_orderid(userid,orderid),
    ->  key bookid(bookid)
    ->  );
Query OK, 0 rows affected (0.11 sec)


show create table t \G
*************************** 1. row ***************************
       Table: t
Create Table: CREATE TABLE `t` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `orderid` int(10) unsigned NOT NULL,
  `bookid` int(10) unsigned NOT NULL DEFAULT '0',
  `userid` int(10) unsigned NOT NULL DEFAULT '0',
  `orderdate` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `status` tinyint(3) unsigned zerofill DEFAULT '000',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_orderid` (`orderid`),
  UNIQUE KEY `idx_uid_orderid` (`userid`,`orderid`),
  KEY `bookid` (`bookid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
1 row in set (0.00 sec)
```
本例中的索引有：  
主键索引  id  
单键索引  orderid  
单键索引  bookid  
组合索引  （userid+orderid)  

唯一索引有：  
主键约束      id  
单键唯一索引  orderid  
组合唯一索引  （userid+orderid)  

  - 添加唯一约束
    - 添加主键
    ```sql
    alter table t add primary key (id);
    ```
    - 添加唯一索引
    ```sql
    alter table t add unique key idx_uk_orderid(orderid);
    ```

- 外键约束  
外键指两张表的数据通过某种条件关联起来    
如果没有外键约束很难保证数据一致性  
  - 创建外键约束  
  将用户表和订单表通过外键关联起来
```sql
alter table order add constraint cnst_uid  foreign key (userid) references  user(userid);

create table user (userid int(11) unsigned not null primary key auto_increment, username varchar(50));
Query OK, 0 rows affected (0.04 sec)

alter table t add constraint cnst_uid  foreign key (userid)  references user(userid);
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0


alter table t rename to `order`;
Query OK, 0 rows affected (0.04 sec)

show create table `order` \G
*************************** 1. row ***************************
       Table: order
Create Table: CREATE TABLE `order` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `orderid` int(10) unsigned NOT NULL,
  `bookid` int(10) unsigned NOT NULL DEFAULT '0',
  `userid` int(10) unsigned NOT NULL DEFAULT '0',
  `orderdate` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `status` tinyint(3) unsigned zerofill DEFAULT '000',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_orderid` (`orderid`),
  UNIQUE KEY `idx_uid_orderid` (`userid`,`orderid`),
  KEY `bookid` (`bookid`),
  CONSTRAINT `cnst_uid` FOREIGN KEY (`userid`) REFERENCES `user` (`userid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
1 row in set (0.00 sec)
  ```
  - 使用外键的注意事项
    - 必须是Innodb 表 ，其他存储引擎(如 MyISAM)不支持外键  
    - 相互约束的字段类型必须要一样
    - 主表的约束字段要求有索引
    - 约束名称必须要唯一，即便不在一张表上

### View
- 视图是一组查询语句构成的结果集，是一种虚拟结构，并不是实际数据
- 视图能简化数据库的访问，能够将多个查询语句结构化为一个虚拟结构
- 视图可以隐藏数据库后端表结构，提高数据库的安全性
- 视图也是一种权限管理，只对用户提供部分数据


- 创建视图
```sql
-- 已完成订单视图
create view order_view as select * from `order` where status=1;
Query OK, 0 rows affected (0.10 sec)
```

### Trigger
触发器，可以在数据写入表之前或之后做一些其他的动作。  
需求：随着客户个人等级的提升，系统需要自动更新用户的积分，涉及用户表和积分表  
使用trigger 在每次更新用户表的时候触发更新积分表。

### Function
```sql
delimiter //
create function addone(inparam int)
    -> returns int DETERMINISTIC
    -> return inparam +1 ; //
Query OK, 0 rows affected (0.00 sec)

delimiter ;
select addone(1);
+-----------+
| addone(1) |
+-----------+
|         2 |
+-----------+
1 row in set (0.00 sec)
```
### Procedure
```sql
delimiter //

CREATE PROCEDURE simpleproc (OUT param1 INT)
    -> BEGIN
    ->   SELECT COUNT(*) INTO param1 FROM t
    -> END //
Query OK, 0 rows affected (0.00 sec)

delimiter ;

CALL simpleproc(@a);
Query OK, 0 rows affected (0.00 sec)

SELECT @a ;
+------+
| @a   |
+------+
| 3    |
+------+
1 row in set (0.00 sec)
```





----
Good luck!  
冬日暖阳
