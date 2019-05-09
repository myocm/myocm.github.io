---
title: MySQL存储引擎概述
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL存储引擎概述
date: 2016-10-25 10:36:44
updated: 2016-10-25 10:36:44
tags:
---
# MySQL存储引擎概述

## 存储引擎的概念
![MySQL体系架构图](/img/markdown-img-paste-20161025105054209.png)

在MySQL架构体系中，服务层相当于书记员的大脑，决定写什么，而存储引擎层相当于书记员的手，决定怎么写，写于何处。
![](/img/markdown-img-paste-20161025105411825.png)


- MySQL 有多种存储引擎，可插拔，可修改存储引擎
```sql
+--------------------+
| Engine             |
+--------------------+
| InnoDB             |
| FEDERATED          |
| MRG_MYISAM         |
| BLACKHOLE          |
| MyISAM             |
| MEMORY             |
| CSV                |
| ARCHIVE            |
| PERFORMANCE_SCHEMA |
+--------------------+
```
- 基于表选择使用何种存储引擎
```sql
show create table t \G
*************************** 1. row ***************************
       Table: t
Create Table: CREATE TABLE `t` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(100) COLLATE utf8_bin DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
1 row in set (0.00 sec)
```

每个表可以指定不同的存储引擎，在建表语句后加上 `engine = 存储引擎名` 即可。  
可用的存储引擎，可以用`show engines;` 查看。
```sql
show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```



## MySQL主要存储引擎的特点及适用场景
- 主要存储引擎及特点
![](/img/markdown-img-paste-2016102511234671.png)

- 改变表的存储引擎  
```sql
ALTER TABLE t1 ENGINE = InnoDB;

-- 例：
create table e1 (id int ) engine = myisam;
Query OK, 0 rows affected (0.06 sec)

show create table e1 \G
*************************** 1. row ***************************
       Table: e1
Create Table: CREATE TABLE `e1` (
  `id` int(11) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COLLATE=utf8_bin
1 row in set (0.00 sec)

alter table e1 engine = innodb;
Query OK, 0 rows affected (0.11 sec)
Records: 0  Duplicates: 0  Warnings: 0

show create table e1 \G
*************************** 1. row ***************************
       Table: e1
Create Table: CREATE TABLE `e1` (
  `id` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
1 row in set (0.00 sec)
```

### InnoDB存储引擎
- 索引组织表
- 支持事务
- 支持行级锁
- 数据块缓存
- 日志持久化
- 稳定可靠，性能好，生产环境尽量使用InnoDB

### MyISAM存储引擎
- 堆表
- 不支持事务
- 只维护索引缓存池，表数据缓存交给操作系统
- 锁粒度较大
- 数据文件可以直接拷贝，偶尔可能会用上
- 不建议线上业务数据使用

### MEMORY存储引擎
- 数据全内存存放，无法持久化
- 性能较高
- 不支持事务
- 适合偶尔作为临时表使用
- create temporary table tmp(id int) engine=memory;

### BLACKHOLE存储引擎
- 数据不作任何存储
- 利用MySQL Replicate，充当日志服务器
- 在MySQL Replicate环境中充当代理主库
![](/img/markdown-img-paste-20161025114127991.png)

### TokuDB存储引擎
- 分形树存储结构
- 支持事务
- 行锁
- 压缩效率较高
- 适合大批量insert 的场景
[TokuDB官方网站](https://www.percona.com/software/mysql-database/percona-tokudb)

### MySQL Cluster (MySQL 官方集群)
- 多主分布式集群
- 数据节点间冗余，高可用
- 支持事务
- 设计上易于扩展
- 面向未来，线上慎用
![MySQL Cluster Components](/img/markdown-img-paste-20161025142838857.png)
----
Good luck!  
冬日暖阳
