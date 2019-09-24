---
title: MySQL优化 -- 9 非SPJ的优化
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL优化
date: 2019-05-21 17:03:02
updated: 2019-05-21 17:03:02
tags:
---


## 1. SPJ and Non SPJ
![](/img/op9-e1179425.png)

## 2. Non SPJ optimization

### 2.1 Why do Non SPJ Optimization ?
为什么要做非 SPJ 优化 ?  
历史:  
早期的关系数据库系如 System-R ，对 GROUPBY 和聚集等操作一般都放在其所在的查询的最后进行处理，即在执行完所有的连接和选择操作之后再执行 GROUPBY 和聚集。

优缺点 :  
   处理方式比较简单，编码容易，但执行效率会比较低。  

### 2.2 GROPUP BY Optimization
```
SELECT COUNT(s.classID) as StudentNum, s.classID FROM STUDENT AS s, CLASS AS c
WHERE S.ID=c.SID
GROUP BY s.classID
```
优化方式:  
分组转换技术:     
即对分组操作、聚集操作与连接操作的`位置进行交换`

常见分组转换技术 :  
1. 分组操作下移:GROUPBY操作可能较大幅度减小关系元组的个数，如果能够 对某个关系先进行分组操作，然后再进行表之间的连接，很可能提高连接效率。这种优化方式是`把分组操作提前执行`。下移的含义，是在查询树上，让分组操作尽量靠近叶子结点，使得分组操作的结点低于一些选择操作。

2. 分组操作上移:如果连接操作能够过滤掉大部分元组，则先进行连接后进行 GROUPBY 操作，可能提高分组操作的效率。这种优化方式是`把分组操作置后执行`。上移的含义，和下移正好相反。
对于带有 GROUPBY 等操作的非 SPJ 格式的 SQL 语句，在本节之前提及的技术都 适用，只是结合了 GROUPBY 操作的语义进行分组操作。因为 GROUPBY 操作下移 或上移均不能保证重写后的查询效率一定更好，所以，要在查询优化器中采用`基于代价的方式来估算某几种路径的优劣`。


### 2.3 ORDER BY Optimization
常见 ORDER BY 优化技术 :  
1. `排序消除( Order By Elimination ， OBYE )`   
优化器在生成执行计划前，将语句中没有必要的排序操作消除(如利用索
 引)，避免在执行计划中出现排序操作或由排序导致的操作(如在索引列上排
 序，可以利用索引消除排序操作)。
2. `排序下推( Sort push down )`   
`把排序操作尽量下推到基表中`，有序的基表进行连接后的结果符合排序的语
 义，这样能避免在最终的大的连接结果集上执行排序操作。


### 2.4 DISTINCT Optimization

常见 DISTINCT 优化技术 :
1. `DISTINCT 消除( Distinct Elimination )`  
如果表中存在主键、唯一约束、索引等，则可以消除查询语句中的 DISTINCT (这种优化方式，在语义优化中也涉及，本质上是语义优化研究的范畴)。
2. `DISTINCT 推入( Distinct Push Down )`  
生成含 DISTINCT 的反半连接查询执行计划时，先进行反半连接再进行 DIS TICT 操作;也许先执行 DISTICT 操作然后再执行反半连接，可能更优;这是利 用连接语义上确保唯一功能特性进行 DISTINCT 的优化。
3. `DISTINCT 迁移( Distinct Placement )`  
对连接操作的结果执行 DISTINCT ，可能把 DISTINCT 移到一个子查询中优 先进行(有的书籍把这项技术称为“ DISTINCT 配置”)。


## 3. Non SPJ optimization of MySQL

### 3.1 GROPUP BY Optimization
`MySQL 的 GROUP BY 优化技术:`  
MySQL 对于 GROUPBY 的处理，通常采用的方式是扫描整个表、创建一个临时表 用以执行分组操作。  
查询执行计划中出现“ Using temporary” 字样表示 MySQL 采用了常规的处理方式。  
`MySQL 不支持分组转换技术`。  
`对于 GROUPBY 的优化，则尽量利用索引`。

利用索引的条件是:分组子句中的列对象源自同一个 btree 索引(不支持利用 Hash 索引进行优化)的全部或前缀部分的部分有序的键(分组使用的索引列与 索引建立的顺序不匹配则不能使用索引)。  
主要的方式有 :  
1. Loose Index Scan :  
直接用索引完成分组操作中对分组列的检索，不必考虑索引的全部键满足 WHER E 子句，只要有部分匹配 WHERE 中的列对象即可( loose ，利用索引中部分列为“松散”)
2. Tight Index Scan :  
索引中的全部键与 WHERE 子句中的列对象匹配( tight ，利用索引中的全部列为“严密”)

```
CREATE TABLE t_god (a INT, b INT, c INT, d INT, e INT);
CREATE INDEX t_god_idx_1 ON t_god (a,b,c);
CREATE INDEX t_god_idx_2 ON t_god (d);

示例 1 :索引列上执行 GROUPBY ，支持 GROUPBY 优化(没有使用“ Using file sort” 类似的操作进行排序):
mysql> EXPLAIN  SELECT a FROM t_god GROUP BY a;
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key         | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t_god | NULL       | index | t_god_idx_1   | t_god_idx_1 | 15      | NULL |    1 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`t_god`.`a` AS `a` from `op`.`t_god` group by `op`.`t_god`.`a`
1 row in set (0.00 sec)

示例 2 :索引列上执行 ORDERBY ， MySQL 支持 ORDERBY 优化:
mysql> EXPLAIN SELECT a FROM t_god ORDER BY a;
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key         | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t_god | NULL       | index | NULL          | t_god_idx_1 | 15      | NULL |    1 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`t_god`.`a` AS `a` from `op`.`t_god` order by `op`.`t_god`.`a`
1 row in set (0.00 sec)

示例3 索引列上执行ORDERBY、GROUPBY，MySQL支持ORDERBY优化也支持G ROUPBY 优化
mysql> EXPLAIN SELECT a FROM t_god GROUP BY a ORDER BY a;
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key         | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t_god | NULL       | index | t_god_idx_1   | t_god_idx_1 | 15      | NULL |    1 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`t_god`.`a` AS `a` from `op`.`t_god` group by `op`.`t_god`.`a` order by `op`.`t_god`.`a`
1 row in set (0.00 sec)

示例4 带有聚集操作，索引列上执行GROUPBY，MySQL支持GROUPBY优化:
mysql> EXPLAIN SELECT a, MIN(b) FROM t_god WHERE c>2 GROUP BY a;
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+--------------------------+
| id | select_type | table | partitions | type  | possible_keys | key         | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | t_god | NULL       | index | t_god_idx_1   | t_god_idx_1 | 15      | NULL |    1 |   100.00 | Using where; Using index |
+----+-------------+-------+------------+-------+---------------+-------------+---------+------+------+----------+--------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`t_god`.`a` AS `a`,min(`op`.`t_god`.`b`) AS `MIN(b)` from `op`.`t_god` where (`op`.`t_god`.`c` > 2) group by `op`.`t_god`.`a`
1 row in set (0.00 sec)
```

### 3.2 ORDER BY Optimization
`MySQL 的 ORDER BY 优化技术:`

```
创建表，命令如下:
CREATE TABLE t_o1 (a1 INT UNIQUE, b1 INT);
CREATE TABLE t_o2 (a2 INT UNIQUE, b2 INT);

示例1 在索引列上进行排序操作，MySQL支持利用索引进行排序优化。
mysql> EXPLAIN SELECT a1 FROM t_o1 ORDER BY a1;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t_o1  | NULL       | index | NULL          | a1   | 5       | NULL |    1 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`t_o1`.`a1` AS `a1` from `op`.`t_o1` order by `op`.`t_o1`.`a1`
1 row in set (0.00 sec)

示例2 排序下推，MySQL不支持。在非索引列上执行连接，然后排序:
mysql> EXPLAIN  SELECT * FROM t_o1, t_o2 WHERE b1=b2 ORDER BY b1;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | t_o1  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using temporary; Using filesort                    |
|  1 | SIMPLE      | t_o2  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
2 rows in set, 1 warning (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `op`.`t_o1`.`a1` AS `a1`,`op`.`t_o1`.`b1` AS `b1`,`op`.`t_o2`.`a2` AS `a2`,`op`.`t_o2`.`b2` AS `b2` from `op`.`t_o1` join `op`.`t_o2` where (`op`.`t_o2`.`b2` = `op`.`t_o1`.`b1`) order by `op`.`t_o1`.`b1`
1 row in set (0.00 sec)
```
### 3.3 DISTINCT Optimization

```
示例 1 MySQL 支持对于 DISTINCT 消除的优化技术。
在有主键的 e1 列上执行 DISTINCT ，查询执行计划如下:
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

示例 2 MySQL 不支持对于 DISTINCT 推入的优化技术。
a2 列是唯一列，又处于反半连接的语义( NOT EXISTS )，完全可以把 DISTIN CT 下推到表 t_o2 中先执行，然后再执行反半连接操作:
mysql> EXPLAIN  SELECT DISTINCT b1 FROM t_o1 WHERE NOT EXISTS (SELECT 1 FROM t_o2 WHERE b1=a2);
+----+--------------------+-------+------------+------+---------------+------+---------+------------+------+----------+------------------------------+
| id | select_type        | table | partitions | type | possible_keys | key  | key_len | ref        | rows | filtered | Extra                        |
+----+--------------------+-------+------------+------+---------------+------+---------+------------+------+----------+------------------------------+
|  1 | PRIMARY            | t_o1  | NULL       | ALL  | NULL          | NULL | NULL    | NULL       |    1 |   100.00 | Using where; Using temporary |
|  2 | DEPENDENT SUBQUERY | t_o2  | NULL       | ref  | a2            | a2   | 5       | op.t_o1.b1 |    1 |   100.00 | Using index                  |
+----+--------------------+-------+------------+------+---------------+------+---------+------------+------+----------+------------------------------+
2 rows in set, 2 warnings (0.00 sec)

mysql> show warnings \G
*************************** 1. row ***************************
  Level: Note
   Code: 1276
Message:
*************************** 2. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select distinct `op`.`t_o1`.`b1` AS `b1` from `op`.`t_o1` where (not(exists(/* select#2 */ select 1 from `op`.`t_o2` where (`op`.`t_o1`.`b1` = `op`.`t_o2`.`a2`))))
2 rows in set (0.00 sec)
```
### 3.4 LIMIT Optimization

MySQL 的 LIMIT 优化技术 :

1. LIMIT 对单表扫描的影响:如果索引扫描可用且花费低于全表扫描，则用索引
扫描实现 LIMIT ( LIMIT 取很少量的行，否则优化器更倾向于使用全表扫描)。

```
create table t3(a1 int , a2 int , a3 int, key id3(a3)) ;
insert into t3 values(1, 2, 11) ;
insert into t3 values(2, 3, 12) ;
insert into t3 values(3, 4, 13) ;
mysql> explain select * from t3 where a3 > 10 ;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t3    | NULL       | ALL  | id3           | NULL | NULL    | NULL |    3 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> explain select * from t3 where a3 > 10 limit 1;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | t3    | NULL       | range | id3           | id3  | 5       | NULL |    3 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)
```
2. LIMIT 对排序的影响:  
如果 LIMIT 和 ORDERBY 子句协同使用，当取到 LIMIT
 设定个数的有序元组数后，后续的排序操作将不再进行。

3. LIMIT 对去重的影响:  
如果 LIMIT 和 DISTINCT 子句协同使用，当取到 LIMIT 设定个数的唯一的元组数后，后续的去重操作将不再进行。

4. LIMIT 受分组的影响:  
如果 LIMIT 和 GROUPBY 子句协同使用， GROUPBY 按索引有序计算每个组的总数的过程中， LIMIT 操作不必计数直到下一个分组开始 计算。
```
GROUP    LIMIT
 1----    +1
 1
 1
 2----    +1
 2
 3----    +1
 ...      ...
```

5. LIMIT 0 :  
直接返回空结果集。
```
mysql> explain select * from t3 where a1 > 10 order by a3 limit 0;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra      |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Zero limit |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------+
1 row in set, 1 warning (0.00 sec)
```
6. MySQL 支持对不带 HAVING 子句的 LIMIT 进行优化。

### 3.5 SET Optimization
MySQL 的集合操作优化技术:  
1. MySQL 语法:  
```
SELECT ...
UNION [ALL | DISTINCT] SELECT ...
[UNION [ALL | DISTINCT] SELECT ...]
```
2. 查询重写规则: OR重写并集规则 ---MySQL不支持
3. 需要引入代价估算的去评估重写后的代价，比较复杂

ORDER BY 子句去除
```
create table tk1 (a int, b int , c int);
create table tk2 (a int, b int , c int);

EXPLAIN
(SELECT c FROM tk1 WHERE c<21 ORDER BY c)
UNION
(SELECT c FROM tk2 WHERE c<11 ORDER BY c);

mysql> EXPLAIN
    -> (SELECT c FROM tk1 WHERE c<21 ORDER BY c)
    -> UNION
    -> (SELECT c FROM tk2 WHERE c<11 ORDER BY c);
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type  | table      | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | PRIMARY      | tk1        | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where     |
|  2 | UNION        | tk2        | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where     |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL  | NULL          | NULL | NULL    | NULL | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+

mysql> EXPLAIN
    -> (SELECT c FROM tk1 WHERE c<21 ORDER BY c limit 3)
    -> UNION
    -> (SELECT c FROM tk2 WHERE c<11 ORDER BY c limit 5);
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
| id | select_type  | table      | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                       |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
|  1 | PRIMARY      | tk1        | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where; Using filesort |
|  2 | UNION        | tk2        | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where; Using filesort |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL  | NULL          | NULL | NULL    | NULL | NULL |     NULL | Using temporary             |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
3 rows in set, 1 warning (0.00 sec)
```

### 3.6 Other Optimization
MySQL 的其他优化技术:  
1. SELECT DISTINCT a FROM t1 LIMIT 1;
2. SELECT DISTICT MAX(a) FROM t1;

## 4. Logical query optimization and Heuristics Rule
1. Logical query optimization---relational algebra, 数学规则  
理论依据,公式推导(定理)
2. Heuristics Rule--- 经验规则  
实践中检验得出的经验 ( 公理 )

示例 : 常见的一些启发式规则
1. 嵌套连接消除 : 如果都是内连接 , 则可以把表示嵌套关系的括号去掉 Ajoin(BjoinC) == AjoinBjoinC
2. 选择操作下推
3. 投影操作下推

示例 : 常见的一些经验规则
1. 在索引键上执行排序操作 , 通常利用索引的有序性按序读取数据而不 进行排序
2. 选择率低于 `10%` 时 , 利用索引的效果通常比读表数据的效果好
3. 当表的数据量较少时,全表扫描可能优于其它方式(如利用索引的
方式)

----
Good luck!  
冬日暖阳
