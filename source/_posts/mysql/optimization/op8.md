---
title: MySQL优化 -- 8 数据库的约束规则与语义优化
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL优化
date: 2019-05-20 14:12:19
updated: 2019-05-20 14:12:19
tags:
---
## 1. Data Integrity Constraint (数据完整性约束)

### 1.1  What is Data Integrity Constraint ?

数据完整性（Data Integrity）:
是指数据的精确性（Accuracy） 和可靠性（Reliability）。

作用:  
1．防止用户向数据库中添加不合语义的数据。   
2．利用基于DBMS的完整性控制机制来实现业务规则，易于定义，容易理解，而且可以降低应用程序的复杂性，提高应用程序的运行效率。同时，基于DBMS的完整性控制机制是集中管理的，因此比应用程序更容易实现数据库的完整性。



### 1.2  The type of Data Integrity Constraint

数据完整性分为四类：
1. 实体完整性（Entity Integrity）:自己
> 一个关系对应现实世界中一个实体集。--ER模型  
现实世界中的实体具有某种惟一性标识。--主键  
主关键字是多个属性的组合，则所有主属性均不得取空值。--隐含的索引

2. 域完整性（Domain Integrity）: 自己的局部
>保证数据库字段取值的合理性。  
属性值应是域中的值，这是关系模式规定了的。  
包括：  
1 检查（CHECK）  
2 默认值（DEFAULT）  
3 不为空（NOT NULL）  
4 可为空（NULL）等  


3. 参照完整性（Referential Integrity）: 自己与其他"实体”的关系
>参照完整性是定义建立关系之间联系的主关键字与外部关键字引用的约束条件。  
1 关系数据库中通常都包含多个存在相互联系的关系，关系与关系之间的联系是通过公共属性来实现的。  
2 所谓公共属性，它是一个关系R(称为被参照关系或目标关系)的主关键字，同时又是另一关系K(称为参照关系)的外部关键字。如果参照关系K中外部关键字的取值，要么与被参照关系R中某元组主关键字的值相同，要么取空值，那么，在这两个关系间建立关联的主关键字和外部关键字引用，符合参照完整性规则要求。如果参照关系K的外部关键字也是其主关键字，根据实体完整性要求，主关键字不得取空值，因此，参照关系K外部关键字的取值实际上只能取相应被参照关系R中已经存在的主关键字值。


4. 用户自定义完整性（User-definedIntegrity）:用户增加的限制
>用户定义完整性则是根据应用环境的要求和实际的需要，对某一具体应用所涉及的数据提出约束性条件。  
这一约束机制一般不应由应用程序提供，而应有由关系模型提供定义并检验。  
用户定义完整性主要包括字段有效性约束和记录有效性。



## 2. Sematic Optimization

### 2.1 What is Semantic Optimization ?
因为语义的原因，使得SQL可以被优化。包括两个基本概念：

1. 语义转换。因为完整性限制等的原因使得一个转换成立的情况称为语义转换。

2. 语义优化。因为语义转换形成的优化称为语义优化。

    语义转换其实是根据完整性约束等信息对“某特定语义”进行推理，进而得到的一种查询效率不同但结果相同的查询。

    语义优化是从语义的角度对SQL进行优化，不是一种形式上的优化，所以其优化的范围，可能覆盖其他类型的优化范围。

语义优化常见的方式：

1. 连接消除（Join Elimination）：

    对一些连接操作先不必评估代价，根据已知信息（主要依据完整性约束等，但不全是依据完整性约束）能推知结果或得到一个简化的操作。

    例如：
      利用A、B两个基表做自然连接，创建一个视图V，如果在视图V上执行查询只涉及其中一个基表的信息，则对视图的查询完全可以转化为对某个基表的查询。

2. 连接引入（Join Introduction）。

      增加连接有助于原始关系变小或原关系的选择率降低。

3. 谓词引入（Predicate Introduction）：

      根据完整性约束等信息，引入新谓词，如引入基于索引的列，可能使得查询更快；

      例如：
      一个表上，有“c1<c2”的列约束，c2列上存在一个索引，查询语句中的WHERE条件有“c1>200”，则可以推知“c2>200”，

      WHER条件变更为“c1>200 AND c2>200 AND c1<c2”，

      由此可以利用c2列上的索引，对查询语句进行优化；如果c2列上的索引的选择率很低，则优化效果会更高。
4. 检测空回答集（Detecting the Empty Answer Set）：

    查询语句中的谓词与约束相悖，可以推知条件结果为FALSE，也许最终的结果集能为空；

    例如：  
    CHECK约束限定“score”列的范围是60到100，而一个查询条件是“score<60”，则能立刻推知条件不成立。

5. 排序优化（Order Optimizer）：

    ORDERBY操作通常由索引或排序（sort）完成；如果能够利用索引，则排序操作可省略；另外，结合分组等操作，考虑ORDERBY操作的优化。

6. 唯一性使用（Exploiting Uniqueness）：

    利用唯一性、索引等特点，检查是否存在不必要的DISTINCT操作

    例如：  
    在主键上执行DISTINCT操作，若有则可以把DISTINCT消除掉。


### 2.2 How to do Semantic Optimization for MySQL ?

示例1   
语义优化中的检测空回答集技术，MySQL支持。
```
创建表，name列非空，对name列进行非空判断，并插入一些数据，具体命令如下：

CREATE TABLE student (name VARCHAR(30) NOT NULL, age INT);

insert into student values ('tom',19);
insert into student values ('Marry',17);
insert into student values ('Jack',19);

mysql> explain EXTENDED SELECT * FROM student WHERE name IS NULL AND age>18;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra            |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Impossible WHERE |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
1 row in set, 2 warnings (0.00 sec)


mysql> mysql warnings \G
*************************** 1. row ***************************
  Level: Warning
   Code: 1681
Message:
*************************** 2. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`student`.`name` AS `name`,`op`.`student`.`age` AS `age` from `op`.`student` where 0
2 rows in set (0.00 sec)

创建表，增加CHECK约束，MySQL不能利用CHECK约束做优化。创建表：
CREATE TABLE SC (score INT, CHECK(score<100 AND score>60));

mysql> EXPLAIN  SELECT * FROM SC WHERE score<60;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | SC    | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings\G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`sc`.`score` AS `score` from `op`.`sc` where (`op`.`sc`.`score` < 60)
1 row in set (0.00 sec)
```

示例2   
语义优化中的谓词引入技术，MySQL不支持。
```
创建表如下，列c2有唯一索引存在，并创建CHECK约束：
CREATE TABLE C (c1 INT, c2 INT UNIQUE, CHECK(c1<c2));

在c1列上进行条件查询，查询执行计划:
mysql> EXPLAIN SELECT * FROM C WHERE c1>60;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | C     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`c`.`c1` AS `c1`,`op`.`c`.`c2` AS `c2` from `op`.`c` where (`op`.`c`.`c1` > 60)
1 row in set (0.00 sec)

创建表如下，列c2有唯一索引存在，并创建CHECK约束：
CREATE TABLE C (c1 INT, c2 INT UNIQUE, CHECK(c1<c2));
mysql> EXPLAIN SELECT * FROM C WHERE c1>60 and c2>60;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+------------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                              |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+------------------------------------+
|  1 | SIMPLE      | C     | NULL       | range | c2            | c2   | 5       | NULL |    1 |   100.00 | Using index condition; Using where |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+------------------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`c`.`c1` AS `c1`,`op`.`c`.`c2` AS `c2` from `op`.`c` where ((`op`.`c`.`c1` > 60) and (`op`.`c`.`c2` > 60))
1 row in set (0.00 sec)
```

示例3  
语义优化中的排序优化，MySQL支持，但条件较为苛刻。
```
创建表如下：
CREATE TABLE D (d1 INT, d2 INT UNIQUE);
对D进行自连接，连接条件使用有唯一索引的列，且连接条件的列与排序列相同。

查询执行计划：
mysql> EXPLAIN SELECT * FROM D F1, D F2 WHERE F1.d2=F2.d2  ORDER BY F1.d2;
+----+-------------+-------+------------+------+---------------+------+---------+----------+------+----------+-----------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref      | rows | filtered | Extra                       |
+----+-------------+-------+------------+------+---------------+------+---------+----------+------+----------+-----------------------------+
|  1 | SIMPLE      | F1    | NULL       | ALL  | d2            | NULL | NULL    | NULL     |    1 |   100.00 | Using where; Using filesort |
|  1 | SIMPLE      | F2    | NULL       | ref  | d2            | d2   | 5       | op.F1.d2 |    1 |   100.00 | NULL                        |
+----+-------------+-------+------------+------+---------------+------+---------+----------+------+----------+-----------------------------+
2 rows in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`f1`.`d1` AS `d1`,`op`.`f1`.`d2` AS `d2`,`op`.`f2`.`d1` AS `d1`,`op`.`f2`.`d2` AS `d2` from `op`.`d` `f1` join `op`.`d` `f2` where (`op`.`f2`.`d2` = `op`.`f1`.`d2`) order by `op`.`f1`.`d2`
1 row in set (0.00 sec)

mysql> EXPLAIN SELECT d2 FROM D ORDER BY d2;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | D     | NULL       | index | NULL          | d2   | 5       | NULL |    1 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`d`.`d2` AS `d2` from `op`.`d` order by `op`.`d`.`d2`
1 row in set (0.00 sec)
```

示例4  
语义优化技术中的唯一性使用，MySQL支持。
```
创建表：
CREATE TABLE E (e1 INT, e2 INT UNIQUE,e3 INT, PRIMARY KEY(e1));

插入数据：
INSERT INTO E VALUES(1,1,1);
INSERT INTO E VALUES(2,NULL,NULL);
INSERT INTO E VALUES(3,3,3);
INSERT INTO E VALUES(4,NULL,NULL);
INSERT INTO E VALUES(5,5,5);


在有主键的e1列上执行DISTINCT，查询执行计划如下：

mysql> EXPLAIN SELECT DISTINCT e1 FROM E;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | E     | NULL       | index | PRIMARY,e2    | e2   | 5       | NULL |    5 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select distinct `op`.`e`.`e1` AS `e1` from `op`.`e`
1 row in set (0.00 sec)

在有唯一索引的e2列上执行DISTINCT，查询执行计划如下：

mysql> EXPLAIN SELECT DISTINCT e2 FROM E;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | E     | NULL       | index | e2            | e2   | 5       | NULL |    5 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select distinct `op`.`e`.`e2` AS `e2` from `op`.`e`
1 row in set (0.00 sec)

在普通的e3列上执行DISTINCT，查询执行计划如下：

mysql> EXPLAIN SELECT DISTINCT e3 FROM E;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | SIMPLE      | E     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    5 |   100.00 | Using temporary |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select distinct `op`.`e`.`e3` AS `e3` from `op`.`e`
1 row in set (0.00 sec)

DISTINCT e1:  Using index
DISTINCT e2:  Using index
DISTINCT e3:  Using temporary   
```

示例5  
语义优化中的连接消除技术，MySQL不支持。
```
创建表和视图：

CREATE TABLE A (a1 INT, a2 INT);
CREATE TABLE B (b1 INT, b2 INT);
CREATE VIEW V AS SELECT * FROM A, B;

INSERT INTO A VALUES(1,1);
INSERT INTO A VALUES(2,2);
INSERT INTO A VALUES(3,3);

对视图进行查询，只查询视图的某个基表的列，查询执行计划如下：
mysql> EXPLAIN SELECT a1, a2 FROM V WHERE a1>2;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | b     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | NULL                                               |
|  1 | SIMPLE      | a     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |    33.33 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
2 rows in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`a`.`a1` AS `a1`,`op`.`a`.`a2` AS `a2` from `op`.`a` join `op`.`b` where (`op`.`a`.`a1` > 2)
1 row in set (0.00 sec)

```


----
Good luck!  
冬日暖阳
