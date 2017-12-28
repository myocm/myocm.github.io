---
title: MySQL优化 -- 5 视图重写与等价谓词重写
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL优化
date: 2017-12-27 10:24:21
updated: 2017-12-27 10:24:21
tags:
---
# 视图重写与等价谓词重写
View Rewrite and Equivalent Predicate Rewrite

## What is View and View Rewrite ?
什么是视图?
视图是数据库中基于表的一种对象, 把对表的查询固化,这种固化就是视图。

注意区分:
    视图(只有视图定义部分，不含数据)<------>物化视图（把视图查询的结果生成一张表固定下来）<------>物化(技术)(把查询结果临时保存到内存的一种优化方法)
```sql
CREATE
    [OR REPLACE]
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    [DEFINER = { user | CURRENT_USER }]
        [SQL SECURITY { DEFINER | INVOKER }]
        VIEW view_name [(column_list)]
        AS select_statement
        [WITH [CASCADED | LOCAL] CHECK OPTION]
```

### 视图的类型：
1. 用 SPJ(select-project-join) 格式构造的视图，称为简单视图。
```sql
CREATE VIEW v1 AS SELECT x, y, z FROM t;
```
2. 用非SPJ格式构造的视图（带有GROUP BY等操作），称为复杂视图。
```sql
CREATE VIEW v2 AS SELECT x, y, z FROM t ORDER BY x;
```
### 什么是视图重写?
1. 查询语句中出现视图对象
2. 查询优化后,视图对象消失
3. 消失的视图对象的查询语句, 融合到初始查询语句中

视图重写示例
```sql
CREATE TABLE t_a(a INT, b INT);
CREATE VIEW v_a AS SELECT * FROM t_a;
```

基于视图的查询命令如下：
```sql
SELECT col_a FROM v_a WHERE col_b>100;
```
经过视图重写后可变换为如下形式：
```sql
SELECT col_a FROM
(
    SELECT col_a, col_b FROM t_a
)
WHERE col_b>100;
```
未来经过优化，可以变换为如下等价形式：
```sql
SELECT col_a FROM t_a WHERE col_b>100;
```

### MySQL视图重写准则：

1. MySQL支持对视图进行优化。

2. 优化方法是把视图转为对基表的查询，然后进行类似子查询的优化。

3. MySQL通常只能重写简单视图，复杂视图不能重写。

![视图重写](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20171227110559653.png)


## How to optimize view with View Rewrite for MySQL ?  
MySQL视图重写示例---数据准备：  
```sql
CREATE TABLE t1 (a1 int UNIQUE, b1 int);
CREATE TABLE t2 (a2 int UNIQUE, b2 int);
CREATE TABLE t3 (a3 int UNIQUE, b3 int);
```
创建简单视图：
```sql
CREATE VIEW v_t_1_2 AS
    SELECT * FROM t1, t2;
```
创建复杂视图：
```sql
CREATE VIEW v_t_gd_1_2 AS
    SELECT DISTINCT t1.b1, t2.b2
    FROM t1, t2
    GROUP BY t1.b1, t2.b2;
```

MySQL视图重写示例1：  
在简单视图上执行连接操作。  
直接用视图和表做连接操作，查询执行计划如下：  
```sql
mysql> EXPLAIN EXTENDED SELECT * FROM t1, v_t_1_2 WHERE t1.a1<20;
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+---------------------------------------+
| id | select_type | table | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                 |
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+---------------------------------------+
|  1 | SIMPLE      | t1    | range | a1            | a1   | 5       | NULL |    1 |   100.00 | Using index condition                 |
|  1 | SIMPLE      | t1    | ALL   | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | t2    | ALL   | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using join buffer (Block Nested Loop) |
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+---------------------------------------+
3 rows in set, 1 warning (0.00 sec)
-- 查询优化器优化后的语句：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1`,`test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1`,`test`.`t2`.`a2` AS `a2`,`test`.`t2`.`b2` AS `b2` from `test`.`t1` join `test`.`t1` join `test`.`t2` where (`test`.`t1`.`a1` < 20)
```
在表上执行与简单视图等价的连接操作
```sql
mysql> EXPLAIN EXTENDED SELECT * FROM t1, (SELECT * FROM t1, t2) t12 WHERE t1.a1<20;
+----+-------------+------------+-------+---------------+------+---------+------+------+----------+---------------------------------------+
| id | select_type | table      | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                 |
+----+-------------+------------+-------+---------------+------+---------+------+------+----------+---------------------------------------+
|  1 | PRIMARY     | t1         | range | a1            | a1   | 5       | NULL |    1 |   100.00 | Using index condition                 |
|  1 | PRIMARY     | <derived2> | ALL   | NULL          | NULL | NULL    | NULL |    2 |   100.00 | Using join buffer (Block Nested Loop) |
|  2 | DERIVED     | t1         | ALL   | NULL          | NULL | NULL    | NULL |    1 |   100.00 | NULL                                  |
|  2 | DERIVED     | t2         | ALL   | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using join buffer (Block Nested Loop) |
+----+-------------+------------+-------+---------------+------+---------+------+------+----------+---------------------------------------+
4 rows in set, 1 warning (0.00 sec)
-- 查询优化器优化后的语句：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1`,`t12`.`a1` AS `a1`,`t12`.`b1` AS `b1`,`t12`.`a2` AS `a2`,`t12`.`b2` AS `b2` from `test`.`t1` join (/* select#2 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1`,`test`.`t2`.`a2` AS `a2`,`test`.`t2`.`b2` AS `b2` from `test`.`t1` join `test`.`t2`) `t12` where (`test`.`t1`.`a1` < 20)
-- MySQL 对视图和子查询的优化是不同的。
```

MySQL视图重写示例2：  
在简单视图上进行聚集操作。  
基于表t1和t2的视图v_t_1_2，进行聚集操作：  

```sql
mysql> EXPLAIN EXTENDED SELECT *, (SELECT max(a1) FROM v_t_1_2) FROM t1 WHERE t1.a1<20;
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+-------------------------+
| id | select_type | table | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                   |
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+-------------------------+
|  1 | PRIMARY     | t1    | range | a1            | a1   | 5       | NULL |    1 |   100.00 | Using index condition   |
|  2 | SUBQUERY    | NULL  | NULL  | NULL          | NULL | NULL    | NULL | NULL |     NULL | No matching min/max row |
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+-------------------------+
2 rows in set, 1 warning (0.00 sec)
-- 查询优化器优化后的语句：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1`,(/* select#2 */ select max(`test`.`t1`.`a1`) from `test`.`t1` join `test`.`t2`) AS `(SELECT max(a1) FROM v_t_1_2)` from `test`.`t1` where (`test`.`t1`.`a1` < 20)
-- 视图的子查询没有被消除，但是视图被重写，用基表代替了视图。
```

MySQL视图重写示例3：  
直接用视图和表做连接操作，并执行分组操作：  
```sql
mysql> EXPLAIN EXTENDED SELECT a1, a3 FROM t3, v_t_1_2 WHERE a1<20 GROUP BY a1, a3;
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+-----------------------------------------------------------------+
| id | select_type | table | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                                           |
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+-----------------------------------------------------------------+
|  1 | SIMPLE      | t3    | index | NULL          | a3   | 5       | NULL |    1 |   100.00 | Using index; Using temporary; Using filesort                    |
|  1 | SIMPLE      | t1    | index | a1            | a1   | 5       | NULL |    1 |   100.00 | Using where; Using index; Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | t2    | index | NULL          | a2   | 5       | NULL |    1 |   100.00 | Using index; Using join buffer (Block Nested Loop)              |
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+-----------------------------------------------------------------+
3 rows in set, 1 warning (0.00 sec)
-- 查询优化器优化后的语句：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a1` AS `a1`,`test`.`t3`.`a3` AS `a3` from `test`.`t3` join `test`.`t1` join `test`.`t2` where (`test`.`t1`.`a1` < 20) group by `test`.`t1`.`a1`,`test`.`t3`.`a3`
-- 简单视图被重写了，说明 GROUP BY 对简单视图的优化没有影响
```

MySQL视图重写示例4：  
直接用视图和表做连接操作，并执行分组和去重操作操作：
```sql
mysql> EXPLAIN EXTENDED SELECT DISTINCT a1, a3 FROM t3, v_t_1_2 WHERE a1<20
    -> GROUP BY a1, a3;
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+-----------------------------------------------------------------+
| id | select_type | table | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                                           |
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+-----------------------------------------------------------------+
|  1 | SIMPLE      | t3    | index | NULL          | a3   | 5       | NULL |    1 |   100.00 | Using index; Using temporary; Using filesort                    |
|  1 | SIMPLE      | t1    | index | a1            | a1   | 5       | NULL |    1 |   100.00 | Using where; Using index; Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | t2    | index | NULL          | a2   | 5       | NULL |    1 |   100.00 | Using index; Distinct; Using join buffer (Block Nested Loop)    |
+----+-------------+-------+-------+---------------+------+---------+------+------+----------+-----------------------------------------------------------------+
3 rows in set, 1 warning (0.00 sec)
-- 查询优化器优化后的语句：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select distinct `test`.`t1`.`a1` AS `a1`,`test`.`t3`.`a3` AS `a3` from `test`.`t3` join `test`.`t1` join `test`.`t2` where (`test`.`t1`.`a1` < 20) group by `test`.`t1`.`a1`,`test`.`t3`.`a3`
```

MySQL视图重写示例5：  
在简单视图上执行外连接操作。  
```sql
mysql> EXPLAIN EXTENDED SELECT * FROM t3 LEFT JOIN v_t_1_2 V ON V.a1=t3.a3
    ->  WHERE V.a1<20;
+----+-------------+-------+------+---------------+------+---------+------------+------+----------+---------------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref        | rows | filtered | Extra                                 |
+----+-------------+-------+------+---------------+------+---------+------------+------+----------+---------------------------------------+
|  1 | SIMPLE      | t3    | ALL  | a3            | NULL | NULL    | NULL       |    1 |   100.00 | NULL                                  |
|  1 | SIMPLE      | t1    | ref  | a1            | a1   | 5       | test.t3.a3 |    1 |   100.00 | Using index condition                 |
|  1 | SIMPLE      | t2    | ALL  | NULL          | NULL | NULL    | NULL       |    1 |   100.00 | Using join buffer (Block Nested Loop) |
+----+-------------+-------+------+---------------+------+---------+------------+------+----------+---------------------------------------+
3 rows in set, 1 warning (0.00 sec)
-- 查询优化器优化后的语句：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t3`.`a3` AS `a3`,`test`.`t3`.`b3` AS `b3`,`test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1`,`test`.`t2`.`a2` AS `a2`,`test`.`t2`.`b2` AS `b2` from `test`.`t3` join `test`.`t1` join `test`.`t2` where ((`test`.`t1`.`a1` = `test`.`t3`.`a3`) and (`test`.`t1`.`a1` < 20))
```

MySQL视图重写示例6：  
直接用复杂视图和表做连接操作，查询执行计划如下：  
```sql
mysql> EXPLAIN EXTENDED SELECT * FROM t3, v_t_gd_1_2 WHERE t3.a3<20;
+----+-------------+------------+-------+---------------+------+---------+------+------+----------+---------------------------------------+
| id | select_type | table      | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                 |
+----+-------------+------------+-------+---------------+------+---------+------+------+----------+---------------------------------------+
|  1 | PRIMARY     | t3         | range | a3            | a3   | 5       | NULL |    1 |   100.00 | Using index condition                 |
|  1 | PRIMARY     | <derived2> | ALL   | NULL          | NULL | NULL    | NULL |    2 |   100.00 | Using join buffer (Block Nested Loop) |
|  2 | DERIVED     | t1         | ALL   | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using temporary; Using filesort       |
|  2 | DERIVED     | t2         | ALL   | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using join buffer (Block Nested Loop) |
+----+-------------+------------+-------+---------------+------+---------+------+------+----------+---------------------------------------+
4 rows in set, 1 warning (0.00 sec)
-- 查询优化器优化后的语句：
mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t3`.`a3` AS `a3`,`test`.`t3`.`b3` AS `b3`,`v_t_gd_1_2`.`b1` AS `b1`,`v_t_gd_1_2`.`b2` AS `b2` from `test`.`t3` join `test`.`v_t_gd_1_2` where (`test`.`t3`.`a3` < 20)
-- 复杂视图不被 MySQL 优化
```

> MySQL只对简单视图进行优化，优化方法为视图重写，重写后还会进行进一步的子查询优化，不对复杂视图进行优化。


## What is Equivalent Predicate Rewrite ?
等价谓词重写：  
把逻辑表达式重写成**等价**的且效率更高的形式  

优点：  
能有效提高查询执行效率

常见的等价谓词重写示例1：  
- LIKE规则

LIKE谓词，是SQL标准支持的一种模式匹配比较操作；  
LIKE规则，是对LIKE谓词的等价重写，即改写LIKE谓词为其他等价的谓词，以更好地利用索引进行优化。
```sql
示例如：
  name LIKE 'Abc%'
重写为
  name >='Abc' AND name <'Abd'
```  
应用LIKE规则的好处：  
转换前针对LIKE谓词，只能进行全表扫描，如果name列上存在索引，则转换后可以进行索引扫描。

LIKE其他形式还可以转换：  
LIKE匹配的表达式中，没有通配符（`%`或`_`），则与“=”等价，
```sql
如：  
  name LIKE 'Abc'  
重写为：  
  name ='Abc'  
```
如果name列上存在索引，则可以利用索引提高查询效率  


- BETWEEN-AND规则
BETWEEN-AND谓词，是SQL标准支持的一种范围比较操作；  
BETWEEN-AND规则，是BETWEEN-AND谓词的等价重写，即改写BETWEEN-AND谓词为其他等价的谓词，以更好地利用索引进行优化。  
```sql
如：  
  sno BETWEEN 10 AND 20  
重写为：  
  sno>=10 AND sno <=20  
```

应用BETWEEN-AND规则的好处是：  
如果sno上建立了索引，则可以用索引扫描代替原来BETWEEN-AND谓词限定的全表扫描，从而提高了查询的效率。  

- IN转换OR规则：  
IN是只IN操作符操作，不是IN子查询。   
IN转换OR规则，就是IN谓词的OR等价重写，即改写IN谓词为等价的OR谓词，以更好地利用索引进行优化。  
将IN谓词等价重写为若干个OR谓词，**可能** 会提高执行效率。  
```sql
如：
   age IN (8，12，21)
重写为：
  age=8 OR age=12 OR age=21
```
应用IN转换OR规则后效率是否能够提高，需要看数据库对IN谓词是否只支持全表扫描。

如果数据库对IN谓词只支持全表扫描且OR谓词中表的age列上存在索引，则转换后查询效率会提高。

- IN转换ANY规则

IN转换ANY规则，就是IN谓词的ANY等价重写，即改写IN谓词为等价的ANY谓词。

IN可以转换为OR，OR可以转为ANY，所以可以直接把IN转换为ANY。
将IN谓词等价重写为ANY谓词，可能会提高执行效率。
```sql
如：
  age IN (8，12，21)
重写为：
  age ANY(8, 12, 21)
```
应用IN转换ANY规则后效率是否能够提高，依赖于数据库对于ANY操作的支持情况。  
OR转换ANY规则，就是OR谓词的ANY等价重写，即改写OR谓词为等价的ANY谓词，以更好地利用MIN/MAX操作进行优化。
```sql
如：
  sal>1000 OR
  dno=3 AND (sal>1100 OR sal>base_sal+100) OR
  sal>base_sal+200 OR
  sal>base_sal×2

重写为：
  dno=3 AND (sal>1100 OR sal>base_sal+100) OR
  sal> ANY (1000,base_sal+200,base_sal×2)
```
OR转换ANY规则，依赖于数据库对于ANY操作的支持情况。

PostgreSQL V9.2.3和MySQL V5.6.10目前都不支持本条规则。

- ALL/ANY转换集函数规则
就是ALL/ANY谓词改写为等价的聚集函数MIN/MAX谓词操作，以更好地利用MIN/MAX操作进行优化。
```sql
如：
  sno>ANY(10, 2*5+3,sqrt(9))
重写为：
  sno>sqrt(9)
```
上面这个ALL/ANY转换集函数规则的示例，有两点需要注意：
1. 示例中存在“>”和“ANY”，其意是在找出`(10, 2*5+3,sqrt(9))`中的最小值，所以可以重写为`sno>sqrt(9)`。
通常，聚集函数MAX()、MIN()等的执行效率一般都比ANY、ALL谓词的执行效率高，因此在这种情况下对其进行重写，可以起到比较好的效果。
2. 如果有索引存在，求解MAX/MIN的效率更高。

- NOT规则

NOT谓词的等价重写，如下：
```sql
  NOT (col_1 !=2)    重写为  col_1=2
  NOT (col_1 !=col_2)重写为  col_1=col_2
  NOT (col_1 =col_2) 重写为  col_1!=col_2
  NOT (col_1 <col_2) 重写为  col_1>=col_2
  NOT (col_1 >col_2) 重写为  col_1<=col_2
```

NOT规则重写的好处：
  如果col_1上建立了索引，则可以用索引扫描代替原来的全表扫描，从而提高查询的效率。

- OR重写并集规则
```sql
  OR条件重写为并集操作，形如下SQL示例：
    SELECT *
    FROM student
    WHERE(sex=’f’ AND age>15) OR age>18；
```
  假设所有条件表达式的列上都有索引（即sex列和age列上都存在索引），数据库可能对于示例中的WHERE语句强迫查询优化器使用顺序	扫描，因为这个语句要检索的是OR操作的集合。

  为了能利用索引处理上面的查询，可以将语句改成如下形式：
```sql
    SELECT *
    FROM student
    WHERE sex=’f’ and age>15
    UNION
    SELECT *
    FROM student
    WHERE age>18;
```
改写后的形式，可以分别利用列sex和age上的索引，进行索引扫描，然后再提供执行UNION操作获得最终结果。





## How to do Equivalent Predicate Rewrite for MySQL ?
?



----
Good luck!  
冬日暖阳
