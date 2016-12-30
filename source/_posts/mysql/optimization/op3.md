---
title: MySQL优化 -- 3 子查询的优化（一）
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL优化
date: 2016-12-29 13:47:03
updated: 2016-12-29 13:47:03
tags:
---
# 子查询的优化（一）
Logical Query Optimization Subquery (1)

## What is the SubQuery

当一个查询是另一个查询的子部分时，称之为**子查询** （查询语句中嵌套有查询语句）

- 查询的子部分，包括那些情况
  1. 目标列位置  
子查询如果位于目标列，则只能是标量子查询，否则数据库可能返回类似"错误:  子查询必须只能返回一个字段"的提示。   
```sql
-- 示例  
CREATE TABLE t1 (k1 INT PRIMARY KEY, c1 INT);  

CREATE TABLE t2 (k2 INT PRIMARY KEY, c2 INT);  
INSERT INTO t2 VALUES (1,10), (2,2), (3,30);

SELECT t1.c1, (SELECT t2.c2 FROM t2) FROM t1, t2;  
Empty set (0.00 sec)

insert into t1 values (1,1), (2,2), (3,3);  
Query OK, 3 rows affected (0.01 sec)  
Records: 3  Duplicates: 0  Warnings: 0

SELECT t1.c1, (SELECT t2.c2 FROM t2) FROM t1, t2;  
ERROR 1242 (21000): Subquery returns more than 1 row

DELETE FROM T2;  
Query OK, 3 rows affected (0.01 sec)

SELECT t1.c1, (SELECT t2.c2 FROM t2) FROM t1, t2;  
Empty set (0.00 sec)  

insert into t2 values (1,10), (2,2), (3,30);  
Query OK, 3 rows affected (0.01 sec)  
Records: 3  Duplicates: 0  Warnings: 0  

SELECT t1.c1, (SELECT t2.c2 FROM t2 WHERE K2=1) FROM t1, t2;  
+------+-----------------------------------+  
| c1   | (SELECT t2.c2 FROM t2 WHERE K2=1) |  
+------+-----------------------------------+  
|    1 |                                10 |  
|    2 |                                10 |  
|    3 |                                10 |
|    1 |                                10 |  
|    2 |                                10 |  
|    3 |                                10 |  
|    1 |                                10 |  
|    2 |                                10 |  
|    3 |                                10 |  
+------+-----------------------------------+  
9 rows in set (0.00 sec)


SELECT t1.c1, (SELECT t2.c2 FROM t2 WHERE c2=1) FROM t1, t2;  
+------+-----------------------------------+
| c1   | (SELECT t2.c2 FROM t2 WHERE c2=1) |
+------+-----------------------------------+
|    1 |                              NULL |
|    2 |                              NULL |
|    3 |                              NULL |
|    1 |                              NULL |
|    2 |                              NULL |
|    3 |                              NULL |
|    1 |                              NULL |
|    2 |                              NULL |
|    3 |                              NULL |
+------+-----------------------------------+
9 rows in set (0.00 sec)


SELECT t1.c1, (SELECT t2.c2 FROM t2 WHERE c2>1) FROM t1, t2;  
ERROR 1242 (21000): Subquery returns more than 1 row

SELECT t1.c1, (SELECT t2.c2 FROM t2 WHERE c2=10) FROM t1, t2;
+------+------------------------------------+
| c1   | (SELECT t2.c2 FROM t2 WHERE c2=10) |
+------+------------------------------------+
|    1 |                                 10 |
|    2 |                                 10 |
|    3 |                                 10 |
|    1 |                                 10 |
|    2 |                                 10 |
|    3 |                                 10 |
|    1 |                                 10 |
|    2 |                                 10 |
|    3 |                                 10 |
+------+------------------------------------+
9 rows in set (0.00 sec)

INSERT INTO t2 VALUES (4,10);
Query OK, 1 row affected (0.04 sec)

SELECT t1.c1, (SELECT t2.c2 FROM t2 WHERE c2=10) FROM t1, t2;
ERROR 1242 (21000): Subquery returns more than 1 row
```

  2. FROM子句位置  
相关子查询出现在FROM子句中，数据库可能返回类似“在FROM子句中的子查询无法参考相同查询级别中的关系”的提示，所以相关子查询不能出现在FROM子句中；  
非相关子查询出现在FROM子句中，可上拉子查询到父层，在多表连接时统一考虑连接代价然后择优。
```sql
SELECT * FROM t1, (SELECT * FROM t2 WHERE t1.k1=t2.k2);
ERROR 1248 (42000): Every derived table must have its own alias

SELECT * FROM t1, (SELECT * FROM t2 WHERE t1.k1=t2.k2) AS A_t12;
ERROR 1054 (42S22): Unknown column 't1.k1' in 'where clause'

SELECT * FROM t1, (SELECT * FROM t2);
ERROR 1248 (42000): Every derived table must have its own alias

SELECT * FROM t1, (SELECT * FROM t2) as A_t2;
+----+------+----+------+
| k1 | c1   | k2 | c2   |
+----+------+----+------+
|  1 |    1 |  1 |   10 |
|  2 |    2 |  1 |   10 |
|  3 |    3 |  1 |   10 |
|  1 |    1 |  2 |    2 |
|  2 |    2 |  2 |    2 |
|  3 |    3 |  2 |    2 |
|  1 |    1 |  3 |   30 |
|  2 |    2 |  3 |   30 |
|  3 |    3 |  3 |   30 |
|  1 |    1 |  4 |   10 |
|  2 |    2 |  4 |   10 |
|  3 |    3 |  4 |   10 |
+----+------+----+------+
12 rows in set (0.00 sec)
```

  3. WHERE子句位置  
出现在WHERE子句中的子查询，是一个条件表达式的一部分，而表达式可以分解为操作符和操作数；根据参与运算的不同的数据类型，操作符也不尽相同，如INT型有“>、<、=、<>”等操作，这对子查询均有一定的要求（如INT型的等值操作，要求子查询必须是标量子查询）。另外，子查询出现在WHERE子句中的格式，也有用谓词指定的一些操作，如IN、BETWEEN、EXISTS等。
```sql
SELECT * FROM t1 WHERE k1 IN (SELECT k2 FROM t2);
+----+------+
| k1 | c1   |
+----+------+
|  1 |    1 |
|  2 |    2 |
|  3 |    3 |
+----+------+
3 rows in set (0.00 sec)


SELECT * FROM t1 WHERE k1 >=ANY (SELECT k2 FROM t2);
+----+------+
| k1 | c1   |
+----+------+
|  1 |    1 |
|  2 |    2 |
|  3 |    3 |
+----+------+
3 rows in set (0.04 sec)

SELECT * FROM t1 WHERE k1 <=SOME (SELECT k2 FROM t2);
+----+------+
| k1 | c1   |
+----+------+
|  1 |    1 |
|  2 |    2 |
|  3 |    3 |
+----+------+
3 rows in set (0.00 sec)

SELECT * FROM t1 WHERE k1 <=ANY (SELECT k2 FROM t2);
+----+------+
| k1 | c1   |
+----+------+
|  1 |    1 |
|  2 |    2 |
|  3 |    3 |
+----+------+
3 rows in set (0.00 sec)

SELECT * FROM t1 WHERE NOT EXISTS (SELECT k2 FROM t2 WHERE k2=100);
+----+------+
| k1 | c1   |
+----+------+
|  1 |    1 |
|  2 |    2 |
|  3 |    3 |
+----+------+
3 rows in set (0.00 sec)
```
  4. JOIN/ON子句位置  
JOIN/ON子句可以拆分为两部分，一是JOIN块类似于FROM子句，二是ON子句块类似于WHERE子句，这两部分都可以出现子查询。子查询的处理方式同FROM子句和WHERE子句。

  5.  GROUPBY子句位置  
目标列必须和GROUPBY关联。可将子查询写在GROUPBY位置处，但子查询用在GROUPBY处没有实用意义。

  6. ORDERBY子句位置
可将子查询写在ORDERBY位置处。但ORDERBY操作是作用在整条SQL语句上的，子查询用在ORDERBY处没有实用意义。

## Type of SubQuery
- 从对象间的关系看
  1. 相关子查询  
子查询的执行依赖于外层父查询的一些属性值。子查询因依赖于父查询的参数，当父查询的参数改变时，子查询需要根据新参数值重新执行（查询优化器对相关子查询进行优化有一定意义），如：
```sql
SELECT * FROM t1 WHERE col_1 = ANY
    (SELECT col_1 FROM t2 WHERE t2.col_2 = t1.col_2);
/* 子查询语句中存在父查询的t1表的col_2列 */
```

  2. 非相关子查询  
子查询的执行，不依赖于外层父查询的任何属性值。这样子查询具有独立性，可独自求解，形成一个子查询计划先于外层的查询求解，如：
```sql
SELECT * FROM t1 WHERE col_1 = ANY
    (SELECT col_1 FROM t2 WHERE t2.col_2 = 10);
//子查询语句中（t2）不存在父查询（t1）的属性
```

  3. 相关子查询与非相关子查询的优化对比

  >扩展阅读:
  相关子查询与不相关子查询的优化（一）  
  http://blog.163.com/li_hx/blog/static/1839914132014248292396/

  >相关子查询与不相关子查询的优化（二）  
  http://blog.163.com/li_hx/blog/static/1839914132014251031411/

  >相关子查询与不相关子查询的优化（三）
  http://blog.163.com/li_hx/blog/static/183991413201421075949661/

- 从特定谓词看
  1. [NOT] IN/ALL/ANY/SOME子查询  
  语义相近，表示“[取反] 存在/所有/任何/任何”，左面是操作数，右面是子查询，是最常见的子查询类型之一。

  2. [NOT] EXISTS子查询  
  半连接语义，表示“[取反] 存在”，没有左操作数，右面是子查询，也是最常见的子查询类型之一。

  3. 其他子查询  
  除了上述两种外的所有子查询。

- 从语句的构成复杂程度看
  1. SPJ子查询 (select-project-join)   
  由选择(select)、投影(project)、连接(join)操作组成的查询。

  2. GROUPBY子查询  
  SPJ子查询加上分组、聚集操作组成的查询。

  3. 其他子查询  
  GROUPBY子查询中加上其他子句如Top-N 、LIMIT/OFFSET、集合、排序等操作。  

  后两种子查询有时合称非SPJ子查询。

- 从结果的角度看
  1. 标量子查询  
  子查询返回的结果集类型是一个简单值（return a scalar，a single value）。

  2. 单行单列子查询  
  子查询返回的结果集类型是零条或一条单元组（return a zero or single row，but only a column）。相似于标量子查询,但可能返回零条元组。

  3. 多行单列子查询  
  子查询返回的结果集类型是多条元组但只有一个简单列（return multiple rows，but only a column）。

  4. 表子查询  
  子查询返回的结果集类型是一个表（多行多列）（returna table，one or more rows of one or more columns）。

  [子查询辨析](http://blog.163.com/li_hx/blog/static/1839914132014220111710312/)

## Why does DB  need to optimize SubQuery
为什么要作子查询优化呢

在数据库实现早期，查询优化器对子查询一般采用嵌套执行的方式，即对父查询中的每一行，都执行一次子查询，这样子查询会执行很多次。这种执行方式效率很低。

而对子查询进行优化，可能带来几个数量级的查询效率的提高。
子查询转变成为连接操作之后，会得到如下好处：
1. 子查询不用执行很多次。
2. 优化器可以根据统计信息来选择不同的连接方法和不同的连接顺序。

子查询中的连接条件、过滤条件分别变成了父查询的连接条件、过滤条件，优化器可以对这些条件进行下推，以提高执行效率。

## How to optimize SubQuery
1. 子查询合并（Subquery Coalescing）  
在某些条件下（语义等价：两个查询块产生同样的结果集），多个子查询能够合并成一个子查询（合并后还是子查询，以后可以通过其他技术消除掉子查询）。这样可以把多次表扫描、多次连接减少为单次表扫描和单次连接，如：  
```sql
SELECT * FROM t1 WHERE a1<10 AND (
    EXISTS (SELECT a2 FROM t2 WHERE t2.a2<5 AND t2.b2=1) OR
    EXISTS (SELECT a2 FROM t2 WHERE t2.a2<5 AND t2.b2=2)
);

可优化为：

SELECT * FROM t1 WHERE a1<10 AND (
    EXISTS (SELECT a2 FROM t2 WHERE t2.a2<5 AND (t2.b2=1 OR t2.b2=2)
    /*两个ESISTS子句合并为一个，条件也进行了合并 */
);
```

2. 子查询展开（Subquery Unnesting）  
又称子查询反嵌套，又称为子查询上拉。
把一些子查询置于外层的父查询中，作为连接关系与外层父查询并列，其实质是把某些子查询重写为等价的多表连接操作（展开后，子查询不存在了，外部查询变成了多表连接）。

  带来的好处是，有关的访问路径、连接方法和连接顺序可能被有效使用，使得查询语句的层次尽可能的减少。

  常见的IN/ANY/SOME/ALL/EXISTS依据情况转换为半连接（SEMI JOIN）、普通类型的子查询消除等情况属于此类，如：
  ```sql
SELECT * FROM t1, (SELECT * FROM t2 WHERE t2.a2 >10) v_t2
WHERE t1.a1<10 AND v_t2.a2<20;

可优化为：

SELECT * FROM t1, t2 WHERE t1.a1<10 AND t2.a2<20 AND t2.a2 >10;
/* 子查询变为了t1、t2表的连接操作，相当于把t2表从子查询中上拉了一层 */
```

3. 聚集子查询消除（Aggregate Subquery Elimination）
通常，一些系统支持的是标量聚集子查询消除。如：
```sql
SELECT * FROM t1 WHERE t1.a1>(SELECT avg(t2.a2) FROM t2);
```

- 子查询展开优化的技术手段
  - 展开的条件  
    1. 如果子查询中出现了聚集、GROUPBY、DISTINCT子句，则子查询只能单独求解，不可以上拉到外层。
    2. 如果子查询只是一个简单格式的（SPJ格式）查询语句，则可以上拉子查询到外层，这样往往能提高查询效率。子查询上拉，讨论的就是这种格式，这也是子查询展开技术处理的范围。  
  - 遵循规则
  把子查询上拉到上层查询，前提是上拉（展开）后的结果不能带来多余的元组，所以子查询展开需要遵循如下规则
    1. 如果上层查询的结果没有重复（即SELECT子句中包含主码），则可以展开其子查询。并且展开后的查询的SELECT子句前应加上DISTINCT标志。
    2. 如果上层查询的SELECT语句中有DISTINCT标志，可以直接进行子查询展开。如果内层查询结果没有重复元组，则可以展开。
  - 子查询展开的具体步骤
    1. 将子查询和外层查询的FROM子句连接为同一个FROM子句，并且修改相应的运行参数。
    2. 将子查询的谓词符号进行相应修改(如：“IN”修改为“=”)。
    3. 将子查询的WHERE条件作为一个整体与外层查询的WHERE条件合并，并用AND条件连接词连接，从而保证新生成的谓词与原旧谓词的上下文意思相同，且成为一个整体。





----
Good luck!  
冬日暖阳
