---
title: MySQL优化 -- 4 子查询的优化（二）
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL优化
date: 2017-12-19 18:07:17
updated: 2017-12-19 18:07:17
tags:
---
# 子查询的优化（二）
Logical Query Optimization Subquery (2)

## MySQL可以优化什么格式的子查询？
MySQL支持对简单SELECT查询中的子查询优化，包括：
- 简单SELECT查询中的子查询。
- 带有DISTINCT、ORDER BY、LIMIT操作的简单SELECT查询中的子查询。

例1：
```sql
CREATE TABLE t1 (a1 INT, b1 INT);
CREATE TABLE t2 (a2 INT, b2 INT);
CREATE TABLE t3 (a3 INT, b3 INT);
插入10000行数据。

查询执行计划如下
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1<100 AND a1 IN (SELECT a2 FROM t2 WHERE t2.a2 >10);
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
| id | select_type  | table       | type   | possible_keys | key        | key_len | ref        | rows | filtered | Extra       |
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
|  1 | SIMPLE       | t1          | ALL    | NULL          | NULL       | NULL    | NULL       | 9716 |   100.00 | Using where |
|  1 | SIMPLE       | <subquery2> | eq_ref | <auto_key>    | <auto_key> | 4       | test.t1.a1 |    1 |   100.00 | NULL        |
|  2 | MATERIALIZED | t2          | ALL    | NULL          | NULL       | NULL    | NULL       | 9716 |   100.00 | Using where |
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
```

MySQL不支持对如下情况的子查询进行优化：  
- 带有UNION操作。
- 带有GROUP BY、HAVING、聚集函数。
- 使用ORDER BY中带有LIMIT。
- 内表、外表的个数超过MySQL支持的最大表的连接数（63个表）。

例2：
```sql
CREATE TABLE t1 (a1 INT, b1 INT, PRIMARY KEY (a1));
CREATE TABLE t2 (a2 INT, b2 INT, PRIMARY KEY (a2));
CREATE TABLE t3 (a3 INT, b3 INT, PRIMARY KEY (a3));
插入10000行数据。
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1>(SELECT MIN(t2.a2) FROM t2);
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+------------------------------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+------------------------------+
|  1 | PRIMARY     | t1    | range | PRIMARY       | PRIMARY | 4       | NULL | 5000 |   100.00 | Using where                  |
|  2 | SUBQUERY    | NULL  | NULL  | NULL          | NULL    | NULL    | NULL | NULL |     NULL | Select tables optimized away |
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+------------------------------+
```

## MySQL支持哪些子查询的优化技术 ？

1. 子查询合并技术,不支持  
例3：
```sql
-- 表结构同例2
mysql> explain EXTENDED SELECT * FROM t1 WHERE a1<4 AND (EXISTS (SELECT a2 FROM t2 WHERE t2.a2<5 AND t2.b2=1) OR EXISTS (SELECT a2 FROM t2 WHERE t2.a2<5 AND t2.b2=2) );
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | t1    | range | PRIMARY       | PRIMARY | 4       | NULL |    4 |   100.00 | Using where |
|  3 | SUBQUERY    | t2    | range | PRIMARY       | PRIMARY | 4       | NULL |    5 |   100.00 | Using where |
|  2 | SUBQUERY    | t2    | range | PRIMARY       | PRIMARY | 4       | NULL |    5 |   100.00 | Using where |
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+-------------+
-- t2表上执行了2次子查询,如果支持子查询合并技术,则t2表上只执行一次子查询
--
--语义等价的 SQL 语句
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE a1<10 AND EXISTS (SELECT a2 FROM t2 WHERE t2.a2<5 AND (t2.b2=1 OR t2.b2=2));
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | t1    | range | PRIMARY       | PRIMARY | 4       | NULL |   10 |   100.00 | Using where |
|  2 | SUBQUERY    | t2    | range | PRIMARY       | PRIMARY | 4       | NULL |    5 |   100.00 | Using where |
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+-------------+
-- 人为的合并查询条件为“(t2.b2=1 OR t2.b2=2)”
-- t2表上的子查询，只执行一次
```

2. 子查询展开（子查询反嵌套）技术,支持得不够好  
例4：  
```sql
-- 表结构同例2
mysql> EXPLAIN EXTENDED SELECT * FROM t1, (SELECT * FROM t2 WHERE t2.a2 >10) v_t2 WHERE t1.a1<10 AND v_t2.a2<20;
+----+-------------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table      | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------------------+
|  1 | PRIMARY     | t1         | range | PRIMARY       | PRIMARY | 4       | NULL |   10 |   100.00 | Using where                                        |
|  1 | PRIMARY     | <derived2> | ALL   | NULL          | NULL    | NULL    | NULL | 5000 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
|  2 | DERIVED     | t2         | range | PRIMARY       | PRIMARY | 4       | NULL | 5000 |   100.00 | Using where                                        |
+----+-------------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------------------------+
-- 在表t2上的子查询被单独执行，
-- 没和表t1进行了嵌套循环连接，子查询没有被消除，
-- 所以MySQL支持子查询反嵌套技术有限
--
--
--表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1<100 AND a1 IN (SELECT a2 FROM t2 WHERE t2.a2 >10);
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
| id | select_type  | table       | type   | possible_keys | key        | key_len | ref        | rows | filtered | Extra       |
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
|  1 | SIMPLE       | t1          | ALL    | NULL          | NULL       | NULL    | NULL       | 9716 |   100.00 | Using where |
|  1 | SIMPLE       | <subquery2> | eq_ref | <auto_key>    | <auto_key> | 4       | test.t1.a1 |    1 |   100.00 | NULL        |
|  2 | MATERIALIZED | t2          | ALL    | NULL          | NULL       | NULL    | NULL       | 9716 |   100.00 | Using where |
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
-- 表 t2 上的子查询被物化
--
--
-- 表结构同例2
mysql >  EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1<100 AND a1 IN (SELECT a2 FROM t2 WHERE t2.a2 >10);
+----+-------------+-------+--------+---------------+---------+---------+----------+------+----------+-------------+
| id | select_type | table | type   | possible_keys | key     | key_len | ref      | rows | filtered | Extra       |
+----+-------------+-------+--------+---------------+---------+---------+----------+------+----------+-------------+
|  1 | SIMPLE      | t1    | range  | PRIMARY       | PRIMARY | 4       | NULL     |   88 |   100.00 | Using where |
|  1 | SIMPLE      | t2    | eq_ref | PRIMARY       | PRIMARY | 4       | op.t1.a1 |    1 |   100.00 | Using index |
+----+-------------+-------+--------+---------------+---------+---------+----------+------+----------+-------------+
-- 子查询不存在，SQL语句被转换为内连接操作，
-- 这表明MySQL只有在针对主键列进行类似的子查询时，
-- 才把子查询上拉为内连接。
-- 所以，MySQL还是支持子查询展开技术的。
```
3. 聚集子查询消除技术,不支持  
例5：
```sql
-- 表结构同例2
mysql > EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1>(SELECT MIN(t2.a2) FROM t2);
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+------------------------------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+------------------------------+
|  1 | PRIMARY     | t1    | range | PRIMARY       | PRIMARY | 4       | NULL | 5000 |   100.00 | Using where                  |
|  2 | SUBQUERY    | NULL  | NULL  | NULL          | NULL    | NULL    | NULL | NULL |     NULL | Select tables optimized away |
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------+------------------------------+
```

## MySQL支持对哪些类型的子查询进行优化 ?
- MySQL 不支持对 EXISTS 类型的子查询做进一步的优化  
例6：
```sql
表结构同例1
mysql>  EXPLAIN EXTENDED SELECT * FROM t1 WHERE EXISTS (SELECT 1 FROM t2 WHERE t1.a1 = t2.a2 and t2.a2 > 10 ) ;
+----+--------------------+-------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type        | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+--------------------+-------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY            | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
|  2 | DEPENDENT SUBQUERY | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
+----+--------------------+-------+------+---------------+------+---------+------+------+----------+-------------+
```

- MySQL 不支持对 NOT EXISTS 类型的子查询做进一步的优化  
例7：
```sql
表结构同例1
mysql>  EXPLAIN EXTENDED SELECT * FROM t1 WHERE NOT EXISTS (SELECT 1 FROM t2 WHERE t1.a1 = t2.a2 and t2.a2 > 10 ) ;
+----+--------------------+-------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type        | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+--------------------+-------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY            | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
|  2 | DEPENDENT SUBQUERY | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
+----+--------------------+-------+------+---------------+------+---------+------+------+----------+-------------+
```

- MySQL 支持对 IN 类型的子查询的优化  
例8（非相关子查询）：
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1 IN ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
| id | select_type  | table       | type   | possible_keys | key        | key_len | ref        | rows | filtered | Extra       |
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
|  1 | SIMPLE       | t1          | ALL    | NULL          | NULL       | NULL    | NULL       | 9716 |   100.00 | Using where |
|  1 | SIMPLE       | <subquery2> | eq_ref | <auto_key>    | <auto_key> | 4       | test.t1.a1 |    1 |   100.00 | NULL        |
|  2 | MATERIALIZED | t2          | ALL    | NULL          | NULL       | NULL    | NULL       | 9716 |   100.00 | Using where |
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
-- 对t2 表进行了物化， 之后进行了半连接优化
-- 优化器处理后的语句为：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` semi join (`test`.`t2`) where ((`<subquery2>`.`a2` = `test`.`t1`.`a1`) and (`test`.`t2`.`a2` > 10))
-- 进行了物化后，消除了子查询，进行了半连接优化
```
  例9（相关子查询）：
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1 IN ( SELECT a2 FROM t2 WHERE t1.a1=10) ;
+----+-------------+-------+------+---------------+------+---------+------+------+----------+--------------------------------------------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                                              |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+--------------------------------------------------------------------+
|  1 | SIMPLE      | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where                                                        |
|  1 | SIMPLE      | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where; FirstMatch(t1); Using join buffer (Block Nested Loop) |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+--------------------------------------------------------------------+
-- select_type 为  SIMPLE 说明消除了子查询
-- 优化器处理后的语句为：
show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1276
Message: Field or reference 'test.t1.a1' of SELECT #2 was resolved in SELECT #1
*************************** 2. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` semi join (`test`.`t2`) where ((`test`.`t1`.`a1` = 10) and (`test`.`t2`.`a2` = 10))
-- IN 子查询被优化为半连接
```
- MySQL 支持对 NOT IN 类型的子查询的优化  
例10（NOT IN 非相关子查询）:
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1 NOT IN ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
|  2 | SUBQUERY    | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
-- 优化器处理后的语句为：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` where (not(<in_optimizer>(`test`.`t1`.`a1`,`test`.`t1`.`a1` in ( <materialize> (/* select#2 */ select `test`.`t2`.`a2` from `test`.`t2` where (`test`.`t2`.`a2` > 10) ), <primary_index_lookup>(`test`.`t1`.`a1` in <temporary table> on <auto_key> where ((`test`.`t1`.`a1` = `materialized-subquery`.`a2`)))))))
-- NOT IN 子查询被物化，但没有消除子查询
```

- MySQL 支持对 ALL 类型的子查询的优化  
例11（`>`ALL 非相关子查询）:
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1 > ALL ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
|  2 | SUBQUERY    | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
-- 优化器处理后的语句为：
mysql>  show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` where <not>((`test`.`t1`.`a1` <= (/* select#2 */ select max(`test`.`t2`.`a2`) from `test`.`t2` where (`test`.`t2`.`a2` > 10))))
1 row in set (0.00 sec)
-- ALL 子查询被转换为MAX 运算，但是子查询没有被消除
```
  例12（=ALL 非相关子查询）:
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1 = ALL ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+--------------------+-------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type        | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+--------------------+-------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY            | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
|  2 | DEPENDENT SUBQUERY | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
+----+--------------------+-------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
-- 优化器处理后的语句为：
mysql > show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` where <not>(<in_optimizer>(`test`.`t1`.`a1`,<exists>(/* select#2 */ select 1 from `test`.`t2` where ((`test`.`t2`.`a2` > 10) and (<cache>(`test`.`t1`.`a1`) <> `test`.`t2`.`a2`)))))
-- ALL 优化为了 EXISTS
```
  例13（`<`ALL 非相关子查询）:
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1  < ALL ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
|  2 | SUBQUERY    | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
-- 优化器处理后的语句为：
mysql > show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` where <not>((`test`.`t1`.`a1` >= (/* select#2 */ select min(`test`.`t2`.`a2`) from `test`.`t2` where (`test`.`t2`.`a2` > 10))))
-- <ALL 被优化为了 >MIN
```

- MySQL 支持对 SOME 类型的子查询的优化  
例14（`>`SOME）:
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1  > SOME ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
|  2 | SUBQUERY    | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
-- 优化器处理后的语句为：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` where <nop>((`test`.`t1`.`a1` > (/* select#2 */ select min(`test`.`t2`.`a2`) from `test`.`t2` where (`test`.`t2`.`a2` > 10))))
-- >SOME 子查询被转换为 >MIN
```
  例15（=SOME）:
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1  = SOME ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
| id | select_type  | table       | type   | possible_keys | key        | key_len | ref        | rows | filtered | Extra       |
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
|  1 | SIMPLE       | t1          | ALL    | NULL          | NULL       | NULL    | NULL       | 9716 |   100.00 | Using where |
|  1 | SIMPLE       | <subquery2> | eq_ref | <auto_key>    | <auto_key> | 4       | test.t1.a1 |    1 |   100.00 | NULL        |
|  2 | MATERIALIZED | t2          | ALL    | NULL          | NULL       | NULL    | NULL       | 9716 |   100.00 | Using where |
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)
-- 优化器处理后的语句为：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` semi join (`test`.`t2`) where ((`<subquery2>`.`a2` = `test`.`t1`.`a1`) and (`test`.`t2`.`a2` > 10))
-- 进行了物化和半连接，消除了子查询
```
  例16（`<`SOME）:
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1  < SOME ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
|  2 | SUBQUERY    | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
-- 优化器处理后的语句为：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` where <nop>((`test`.`t1`.`a1` < (/* select#2 */ select max(`test`.`t2`.`a2`) from `test`.`t2` where (`test`.`t2`.`a2` > 10))))
-- <SOME 被优化成了 <MAX
```

- MySQL 支持对 ANY 类型的子查询的优化  
例17（>ANY）:
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1  > ANY ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
|  2 | SUBQUERY    | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
-- 优化器处理后的语句为：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` where <nop>((`test`.`t1`.`a1` > (/* select#2 */ select min(`test`.`t2`.`a2`) from `test`.`t2` where (`test`.`t2`.`a2` > 10))))
-- >ANY  被优化为 >MIN
```
  例18（=ANY）:
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1  = ANY ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
| id | select_type  | table       | type   | possible_keys | key        | key_len | ref        | rows | filtered | Extra       |
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
|  1 | SIMPLE       | t1          | ALL    | NULL          | NULL       | NULL    | NULL       | 9716 |   100.00 | Using where |
|  1 | SIMPLE       | <subquery2> | eq_ref | <auto_key>    | <auto_key> | 4       | test.t1.a1 |    1 |   100.00 | NULL        |
|  2 | MATERIALIZED | t2          | ALL    | NULL          | NULL       | NULL    | NULL       | 9716 |   100.00 | Using where |
+----+--------------+-------------+--------+---------------+------------+---------+------------+------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)
-- 优化器处理后的语句为：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` semi join (`test`.`t2`) where ((`<subquery2>`.`a2` = `test`.`t1`.`a1`) and (`test`.`t2`.`a2` > 10))
-- =ANY 物化后通过半连接 消除了子查询
```
  例19（`<`ANY）:
```sql
表结构同例1
mysql> EXPLAIN EXTENDED SELECT * FROM t1 WHERE t1.a1  < ANY ( SELECT a2 FROM t2 WHERE t2.a2 > 10 );
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | t1    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
|  2 | SUBQUERY    | t2    | ALL  | NULL          | NULL | NULL    | NULL | 9716 |   100.00 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
-- 优化器处理后的语句为：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`.`t1` where <nop>((`test`.`t1`.`a1` < (/* select#2 */ select max(`test`.`t2`.`a2`) from `test`.`t2` where (`test`.`t2`.`a2` > 10))))
-- <ANY 被优化为 <MAX
```

## MySQL子查询总结

MySQL 支持对简单 SELECT 子查询的优化，  
不支持对 EXISTS 和 NOT EXISTS 的只查询优化，  
但是对 IN, ALL, SOME, ANY 谓词支持一定的优化。


----
Good luck!  
冬日暖阳
