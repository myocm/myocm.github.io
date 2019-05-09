---
title: MySQL优化 -- 7 外连接消除、嵌套连接消除与连接消除
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL优化
date: 2018-01-03 15:18:04
updated: 2018-01-03 15:18:04
tags:
---
# 外连接消除、连接消除与嵌套连接消除
Outer Join Elimination、 Join Elimination、 Nest Join Elimination

## Outer Join Elimination

### What is Outer Join ?
![外连接]( /img/markdown-img-paste-20180104153553920.png)

内连接为：A 表和 B 表都满足连接条件的部分。  
外连接为：A 表和 B 表都满足连接条件的部分 加上 A 满足连接条件而 B 不满足连接条件的部分。（A 为主连接表的情况）。  

### The type of Outer Join
1. LEFT JOIN / LEFT OUTER JOIN：左外连接    
左向外连接的结果集包括：LEFT OUTER子句中指定的左表的所有行，而不仅仅是连接列所匹配的行。如果左表的某行在右表中没有匹配行，则在相关联的结果集行中右表的所有选择列表列均为空值。       
2. RIGHT JOIN / RIGHT  OUTER  JOIN：右外连接      
右向外连接是左向外联接的反向连接。将返回右表的所有行。如果右表的某行在左表中没有匹配行，则将为左表返回空值。       
3. FULL JOIN / FULL OUTER JOIN：全外连接  
全外连接返回左表和右表中的所有行。当某行在另一个表中没有匹配行时，则另一个表的选择列表列包含空值。如果表之间有匹配行，则整个结果集行包含基表的数据值。

```sql
select * from book ;
+--------+------------------+-----------+
| bookid | bookname         | studentid |
+--------+------------------+-----------+
|      1 | <<培训>>         |         3 |
|      2 | <<成功秘诀>>     |         5 |
|      3 | <<红楼梦>>       |         3 |
|      4 | <<西厢记>>       |         2 |
|      5 | <<水浒传>>       |         6 |
|      6 | <<三国演义>>     |        10 |
+--------+------------------+-----------+
select * from student ;
+-----------+--------------+
| studentid | stundentname |
+-----------+--------------+
|         1 | 张三         |
|         2 | 李四         |
|         3 | 关羽         |
|         4 | 张飞         |
|         5 | 黄聪         |
|         6 | 李逵         |
|         7 | 赵娜         |
|         8 | 王萌         |
+-----------+--------------+
-- 内连接
root@127.0.0.1 [op]> SELECT *
    -> FROM book AS b, student AS s
    -> WHERE b.studentid=s.studentid
    ->  ;
+--------+------------------+-----------+-----------+--------------+
| bookid | bookname         | studentid | studentid | stundentname |
+--------+------------------+-----------+-----------+--------------+
|      4 | <<西厢记>>       |         2 |         2 | 李四         |
|      1 | <<培训>>         |         3 |         3 | 关羽         |
|      3 | <<红楼梦>>       |         3 |         3 | 关羽         |
|      2 | <<成功秘诀>>     |         5 |         5 | 黄聪         |
|      5 | <<水浒传>>       |         6 |         6 | 李逵         |
+--------+------------------+-----------+-----------+--------------+
5 rows in set (0.00 sec)

-- 左外连接
root@127.0.0.1 [op]> SELECT *
    -> FROM book AS b LEFT JOIN student AS s
    -> ON b.studentid=s.studentid
    -> ;
+--------+------------------+-----------+-----------+--------------+
| bookid | bookname         | studentid | studentid | stundentname |
+--------+------------------+-----------+-----------+--------------+
|      4 | <<西厢记>>       |         2 |         2 | 李四         |
|      1 | <<培训>>         |         3 |         3 | 关羽         |
|      3 | <<红楼梦>>       |         3 |         3 | 关羽         |
|      2 | <<成功秘诀>>     |         5 |         5 | 黄聪         |
|      5 | <<水浒传>>       |         6 |         6 | 李逵         |
|      6 | <<三国演义>>     |        10 |      NULL | NULL         |
+--------+------------------+-----------+-----------+--------------+
6 rows in set (0.00 sec)
-- 右外连接
root@127.0.0.1 [op]> SELECT *
    -> FROM book AS b RIGHT JOIN
    -> student AS s
    -> ON b.studentid=s.studentid
    -> ;
+--------+------------------+-----------+-----------+--------------+
| bookid | bookname         | studentid | studentid | stundentname |
+--------+------------------+-----------+-----------+--------------+
|      1 | <<培训>>         |         3 |         3 | 关羽         |
|      2 | <<成功秘诀>>     |         5 |         5 | 黄聪         |
|      3 | <<红楼梦>>       |         3 |         3 | 关羽         |
|      4 | <<西厢记>>       |         2 |         2 | 李四         |
|      5 | <<水浒传>>       |         6 |         6 | 李逵         |
|   NULL | NULL             |      NULL |         4 | 张飞         |
|   NULL | NULL             |      NULL |         7 | 赵娜         |
|   NULL | NULL             |      NULL |         8 | 王萌         |
|   NULL | NULL             |      NULL |         1 | 张三         |
+--------+------------------+-----------+-----------+--------------+
-- 全外连接(MySQL 还不支持 FULL OUTER JOIN)
-- 可以通过合并左外连接和右外连接等价于全外连接
SELECT * FROM book AS b
    -> LEFT OUTER JOIN student AS s
    -> ON b.studentid=s.studentid
    -> UNION
    -> SELECT * FROM book AS b
    -> RIGHT OUTER JOIN student AS s
    -> ON b.studentid=s.studentid ;
+--------+------------------+-----------+-----------+--------------+
| bookid | bookname         | studentid | studentid | stundentname |
+--------+------------------+-----------+-----------+--------------+
|      4 | <<西厢记>>       |         2 |         2 | 李四         |
|      1 | <<培训>>         |         3 |         3 | 关羽         |
|      3 | <<红楼梦>>       |         3 |         3 | 关羽         |
|      2 | <<成功秘诀>>     |         5 |         5 | 黄聪         |
|      5 | <<水浒传>>       |         6 |         6 | 李逵         |
|      6 | <<三国演义>>     |        10 |      NULL | NULL         |
|   NULL | NULL             |      NULL |         4 | 张飞         |
|   NULL | NULL             |      NULL |         7 | 赵娜         |
|   NULL | NULL             |      NULL |         8 | 王萌         |
|   NULL | NULL             |      NULL |         1 | 张三         |
+--------+------------------+-----------+-----------+--------------+
```

>自然连接
```sql
root@127.0.0.1 [op]> SELECT * FROM book
    -> NATURAL LEFT OUTER JOIN student ;
+-----------+--------+------------------+--------------+
| studentid | bookid | bookname         | stundentname |
+-----------+--------+------------------+--------------+
|         2 |      4 | <<西厢记>>       | 李四         |
|         3 |      1 | <<培训>>         | 关羽         |
|         3 |      3 | <<红楼梦>>       | 关羽         |
|         5 |      2 | <<成功秘诀>>     | 黄聪         |
|         6 |      5 | <<水浒传>>       | 李逵         |
|        10 |      6 | <<三国演义>>     | NULL         |
+-----------+--------+------------------+--------------+
6 rows in set (0.00 sec)
--
root@127.0.0.1 [op]> SELECT * FROM book
    -> NATURAL RIGHT OUTER JOIN student  ;
+-----------+--------------+--------+------------------+
| studentid | stundentname | bookid | bookname         |
+-----------+--------------+--------+------------------+
|         3 | 关羽         |      1 | <<培训>>         |
|         5 | 黄聪         |      2 | <<成功秘诀>>     |
|         3 | 关羽         |      3 | <<红楼梦>>       |
|         2 | 李四         |      4 | <<西厢记>>       |
|         6 | 李逵         |      5 | <<水浒传>>       |
|         4 | 张飞         |   NULL | NULL             |
|         7 | 赵娜         |   NULL | NULL             |
|         8 | 王萌         |   NULL | NULL             |
|         1 | 张三         |   NULL | NULL             |
+-----------+--------------+--------+------------------+
9 rows in set (0.00 sec)
```

### Why does Outer Join Elimination ?
外连接消除：

把外连接变为内连接
```sql
A OUTER JOIN B
变形为
A JOIN B
```

外连接消除的意义：  
1. 查询优化器在处理外连接操作时所需执行的操作和时间多于内连接
2. 外连接消除后，优化器在选择多表连接顺序时，可以有更多更灵活的选择，从而可以选择更好的表连接顺序，加快查询执行的速度
3. 表的一些连接算法（如块嵌套连接和索引循环连接等）在将规模小的或筛选条件最严格的表作为“外表”（放在连接顺序的最前面，是多层循环体的外循环层），可以减少不必要的I/O开销，能加快算法执行的速度


### How to do Outer Join Elimination for MySQL ?
外连接消除的条件：  
WHERE子句中的条件满足“空值拒绝”  
（又称为“reject-NULL”条件）。  

>WHERE条件可以保证从结果中排除外连接右侧（右表）生成的值为NULL的行（即条件确保应用在右表带有空值的列对象上时，条件不满足，条件的结果值为FLASE或UNKONOWEN，这样右表就不会有值为NULL的行生成），所以能使该查询在语义上等效于内连接。

```sql
explain
SELECT * FROM X LEFT JOIN Y ON (X.X_num=Y.Y_num)
WHERE Y.Y_num IS NOT NULL;
```

示例:
```sql
-- 创建表，命令如下：
CREATE TABLE t_1 (t_1_id INT UNIQUE, t_1_col_1 INT, t_1_col_2 VARCHAR(10));
CREATE TABLE t_2 (t_2_id INT UNIQUE, t_2_col_1 INT, t_2_col_2 VARCHAR(10));
-- 插入数据，命令如下：
INSERT INTO t_1 VALUES (1, 11, 't_1_1');   INSERT INTO t_1 VALUES (2, 12, NULL);
INSERT INTO t_1 VALUES (3, NULL, 't_1_3'); INSERT INTO t_1 VALUES (4, 14, 't_1_4');
INSERT INTO t_1 VALUES (5, 15, NULL);      INSERT INTO t_1 VALUES (7, NULL, NULL);

INSERT INTO t_2 VALUES (1, 11, 't_2_1');   INSERT INTO t_2 VALUES (2, NULL, 't_2_2');
INSERT INTO t_2 VALUES (3, 13, NULL);      INSERT INTO t_2 VALUES (4, 14, 't_2_4');
INSERT INTO t_2 VALUES (6, 16, 't_2_6');   INSERT INTO t_2 VALUES (7, NULL, NULL);

语句一，使用TRUE作为ON的子句，WHERE子句包括连接条件：
SELECT * FROM t_1 LEFT JOIN t_2 ON true WHERE t_1_id = t_2_id;

语句二，使用ON子句包括连接条件：
SELECT * FROM t_1 LEFT JOIN t_2 ON t_1_id = t_2_id;

语句三，使用ON和WHERE子句包括连接条件：
SELECT * FROM t_1 LEFT JOIN t_2 ON t_1_id = t_2_id WHERE t_1_id = t_2_id;

从执行结果看：
语句一，三：
SELECT * FROM t_1 LEFT JOIN t_2 ON true WHERE t_1_id = t_2_id;
+--------+-----------+-----------+--------+-----------+-----------+
| t_1_id | t_1_col_1 | t_1_col_2 | t_2_id | t_2_col_1 | t_2_col_2 |
+--------+-----------+-----------+--------+-----------+-----------+
|      1 |        11 | t_1_1     |      1 |        11 | t_2_1     |
|      2 |        12 | NULL      |      2 |      NULL | t_2_2     |
|      3 |      NULL | t_1_3     |      3 |        13 | NULL      |
|      4 |        14 | t_1_4     |      4 |        14 | t_2_4     |
|      7 |      NULL | NULL      |      7 |      NULL | NULL      |
+--------+-----------+-----------+--------+-----------+-----------+
5 rows in set (0.00 sec)

语句二：
SELECT * FROM t_1 LEFT JOIN t_2 ON t_1_id = t_2_id;
+--------+-----------+-----------+--------+-----------+-----------+
| t_1_id | t_1_col_1 | t_1_col_2 | t_2_id | t_2_col_1 | t_2_col_2 |
+--------+-----------+-----------+--------+-----------+-----------+
|      1 |        11 | t_1_1     |      1 |        11 | t_2_1     |
|      2 |        12 | NULL      |      2 |      NULL | t_2_2     |
|      3 |      NULL | t_1_3     |      3 |        13 | NULL      |
|      4 |        14 | t_1_4     |      4 |        14 | t_2_4     |
|      5 |        15 | NULL      |   NULL |      NULL | NULL      |
|      7 |      NULL | NULL      |      7 |      NULL | NULL      |
+--------+-----------+-----------+--------+-----------+-----------+
6 rows in set (0.00 sec)

可以发现，语句一，三的 where 可以保证从结果集中排除外连接右侧（右表）生成的值为 NULL 的行

从执行计划看：
语句二：
explain SELECT * FROM t_1 LEFT JOIN t_2 ON t_1_id = t_2_id;
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key    | key_len | ref           | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------+
|  1 | SIMPLE      | t_1   | NULL       | ALL  | NULL          | NULL   | NULL    | NULL          |    6 |   100.00 | NULL  |
|  1 | SIMPLE      | t_2   | NULL       | ref  | t_2_id        | t_2_id | 5       | op.t_1.t_1_id |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------+
优化器处理后：
/* select#1 */ select `op`.`t_1`.`t_1_id` AS `t_1_id`,
`op`.`t_1`.`t_1_col_1` AS `t_1_col_1`,
`op`.`t_1`.`t_1_col_2` AS `t_1_col_2`,
`op`.`t_2`.`t_2_id` AS `t_2_id`,
`op`.`t_2`.`t_2_col_1` AS `t_2_col_1`,
`op`.`t_2`.`t_2_col_2` AS `t_2_col_2`
from `op`.`t_1` left join `op`.`t_2`
on((`op`.`t_2`.`t_2_id` = `op`.`t_1`.`t_1_id`))
where 1

语句一，语句三，执行计划相同：
explain SELECT * FROM t_1 LEFT JOIN t_2 ON true WHERE t_1_id = t_2_id;
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key    | key_len | ref           | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------+
|  1 | SIMPLE      | t_1   | NULL       | ALL  | t_1_id        | NULL   | NULL    | NULL          |    6 |   100.00 | NULL  |
|  1 | SIMPLE      | t_2   | NULL       | ref  | t_2_id        | t_2_id | 5       | op.t_1.t_1_id |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------+
优化器处理后：
/* select#1 */ select `op`.`t_1`.`t_1_id` AS `t_1_id`,
`op`.`t_1`.`t_1_col_1` AS `t_1_col_1`,
`op`.`t_1`.`t_1_col_2` AS `t_1_col_2`,
`op`.`t_2`.`t_2_id` AS `t_2_id`,
`op`.`t_2`.`t_2_col_1` AS `t_2_col_1`,
`op`.`t_2`.`t_2_col_2` AS `t_2_col_2`
from `op`.`t_1` join `op`.`t_2`
where (`op`.`t_2`.`t_2_id` = `op`.`t_1`.`t_1_id`)

优化器消除了外连接，变成了内连接。
```

辨析 `ON` 和 `WHERE` 的差异:  
`ON   t_1_id = t_2_id`   
t_1_id 和 t_2_id 进行连接  

`WHERE t_1_id = t_2_id`  
当t_1_id 和 t_2_id的值相等  

示例2：理解 WHERE 条件对外连接优化的影响
```
sql4：
SELECT * FROM t_1 LEFT JOIN t_2 ON t_1_id = t_2_id WHERE t_1_id>0;
+--------+-----------+-----------+--------+-----------+-----------+
| t_1_id | t_1_col_1 | t_1_col_2 | t_2_id | t_2_col_1 | t_2_col_2 |
+--------+-----------+-----------+--------+-----------+-----------+
|      1 |        11 | t_1_1     |      1 |        11 | t_2_1     |
|      2 |        12 | NULL      |      2 |      NULL | t_2_2     |
|      3 |      NULL | t_1_3     |      3 |        13 | NULL      |
|      4 |        14 | t_1_4     |      4 |        14 | t_2_4     |
|      5 |        15 | NULL      |   NULL |      NULL | NULL      |
|      7 |      NULL | NULL      |      7 |      NULL | NULL      |
+--------+-----------+-----------+--------+-----------+-----------+
6 rows in set (0.00 sec)

sql5：
SELECT * FROM t_1 LEFT JOIN t_2 ON t_1_id = t_2_id WHERE t_2_id>0;
+--------+-----------+-----------+--------+-----------+-----------+
| t_1_id | t_1_col_1 | t_1_col_2 | t_2_id | t_2_col_1 | t_2_col_2 |
+--------+-----------+-----------+--------+-----------+-----------+
|      1 |        11 | t_1_1     |      1 |        11 | t_2_1     |
|      2 |        12 | NULL      |      2 |      NULL | t_2_2     |
|      3 |      NULL | t_1_3     |      3 |        13 | NULL      |
|      4 |        14 | t_1_4     |      4 |        14 | t_2_4     |
|      7 |      NULL | NULL      |      7 |      NULL | NULL      |
+--------+-----------+-----------+--------+-----------+-----------+
5 rows in set (0.00 sec)

执行计划：
sql4：
explain SELECT * FROM t_1 LEFT JOIN t_2 ON t_1_id = t_2_id WHERE t_1_id>0;
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key    | key_len | ref           | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------------+
|  1 | SIMPLE      | t_1   | NULL       | ALL  | t_1_id        | NULL   | NULL    | NULL          |    6 |   100.00 | Using where |
|  1 | SIMPLE      | t_2   | NULL       | ref  | t_2_id        | t_2_id | 5       | op.t_1.t_1_id |    1 |   100.00 | NULL        |
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------------+
优化器处理后：
/* select#1 */ select `op`.`t_1`.`t_1_id` AS `t_1_id`,
`op`.`t_1`.`t_1_col_1` AS `t_1_col_1`,
`op`.`t_1`.`t_1_col_2` AS `t_1_col_2`,
`op`.`t_2`.`t_2_id` AS `t_2_id`,
`op`.`t_2`.`t_2_col_1` AS `t_2_col_1`,
`op`.`t_2`.`t_2_col_2` AS `t_2_col_2`
from `op`.`t_1` left join `op`.`t_2`
on((`op`.`t_2`.`t_2_id` = `op`.`t_1`.`t_1_id`))
where (`op`.`t_1`.`t_1_id` > 0)

sql5：
explain  SELECT * FROM t_1 LEFT JOIN t_2 ON t_1_id = t_2_id WHERE t_2_id>0;
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key    | key_len | ref           | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------------+
|  1 | SIMPLE      | t_1   | NULL       | ALL  | t_1_id        | NULL   | NULL    | NULL          |    6 |   100.00 | Using where |
|  1 | SIMPLE      | t_2   | NULL       | ref  | t_2_id        | t_2_id | 5       | op.t_1.t_1_id |    1 |   100.00 | NULL        |
+----+-------------+-------+------------+------+---------------+--------+---------+---------------+------+----------+-------------+
优化器处理后：
/* select#1 */ select `op`.`t_1`.`t_1_id` AS `t_1_id`,
`op`.`t_1`.`t_1_col_1` AS `t_1_col_1`,
`op`.`t_1`.`t_1_col_2` AS `t_1_col_2`,
`op`.`t_2`.`t_2_id` AS `t_2_id`,
`op`.`t_2`.`t_2_col_1` AS `t_2_col_1`,
`op`.`t_2`.`t_2_col_2` AS `t_2_col_2`
from `op`.`t_1` join `op`.`t_2`
where ((`op`.`t_2`.`t_2_id` = `op`.`t_1`.`t_1_id`) and (`op`.`t_1`.`t_1_id` > 0))

可以看到 sql5 消除了外连接。
```

外连接消除总结:

1. 注意外连接与内连接的语义差别

2. 外连接优化的条件：空值拒绝

3. 外连接优化的本质：语义上是外连接，但WHER条件使得外连接可以蜕化为内连接




## Join Elimination
连接消除：  
去掉不必要的连接对象，则减少了连接操作

连接消除的条件：  
无固定模式，需要具体分析

连接消除情况一：  
唯一键/主键作为连接条件，三表内连接，中间表的列只作为连接条件，则可以去掉中间表

CREATE TABLE A (a1 INT UNIQUE, a2 VARCHAR(9), a3 INT);  
CREATE TABLE B (b1 INT UNIQUE, b2 VARCHAR(9), c2 INT);  
CREATE TABLE C (c1 INT UNIQUE, c2 VARCHAR(9), c3 INT);  

B的列在WHERE条件子句中只作为等值连接条件存在，则查询可以去掉对B的连接操作：  
SELECT A.`*`, C.`*` FROM **A JOIN B ON (a1=b1) JOIN CON (b1=c1)**;  
相当于：  
SELECT A.`*`, C.`*` FROM **A JOIN C ON (a1= c1)**;  

连接消除情况二：  
一些特殊形式，可以消除连接操作（可消除的表除了作为连接对象外，不出现在任何子句中）。

例：
SELECT MAX(a1) FROM A, B;`/* 在这样格式中的MIN、MAX函数操作可以消除连接，去掉B表不影响结果；其他聚集函数不可以 */`   
SELECT DISTINCT a3 FROM A, B; `/* 对连接结果中的a3列执行去重操作*/`  
SELECT a1 FROM A, B GROUP BY a1; `/* 对连接结果中的a1列执行分组操作 */`  


连接消除情况三：  
主外键关系的表进行的连接，可消除主键表，这不会影响对外键表的查询。  
例：
```
创建表，命令如下：
CREATE TABLE B (b1 INT, b2 VARCHAR(2), PRIMARY KEY(b1));
CREATE TABLE A (a1 INT, a2 VARCHAR(2), FOREIGN KEY(a1) REFERENCES B(b1) );/* A作为外键表参照主键表B */
CREATE TABLE C (c1 INT, c2 VARCHAR(2));

插入数据，命令如下：
INSERT INTO B VALUES(1, 'B1');
INSERT INTO B VALUES(2, 'B2');
INSERT INTO B VALUES(3, 'B3');

INSERT INTO A VALUES(1, 'A1');
INSERT INTO A VALUES(null, 'A2');
INSERT INTO A VALUES(3, 'A3');

INSERT INTO C VALUES(1, 'C1');
INSERT INTO C VALUES(2, 'C2');
INSERT INTO C VALUES(NULL, 'C3');

对主外键参照的表进行内连接，可以消除主键表 --- MySQL不支持
sql1:
三个表做内连接，但目标列不包括主键表B的对象，主键表B只作为连接对象和连接条件存在  
SELECT A.*, C.* FROM A,B,C WHERE A.a1=B.b1 AND B.b1=C.c1;
//主键表B作为连接对象和连接条件存在
+------+------+------+------+
| a1   | a2   | c1   | c2   |
+------+------+------+------+
|    1 | A1   |    1 | C1   |
+------+------+------+------+

EXPLAIN  SELECT A.*, C.* FROM A,B,C WHERE A.a1=B.b1 AND B.b1=C.c1;
+----+-------------+-------+------------+--------+---------------+---------+---------+---------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref     | rows | filtered | Extra                                              |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | C     | NULL       | ALL    | NULL          | NULL    | NULL    | NULL    |    3 |   100.00 | Using where                                        |
|  1 | SIMPLE      | A     | NULL       | ALL    | a1            | NULL    | NULL    | NULL    |    3 |    33.33 | Using where; Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | B     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | op.C.c1 |    1 |   100.00 | Using index                                        |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------+------+----------+----------------------------------------------------+
优化器处理后：
 /* select#1 */ select `op`.`a`.`a1` AS `a1`,
 `op`.`a`.`a2` AS `a2`,
 `op`.`c`.`c1` AS `c1`,
 `op`.`c`.`c2` AS `c2`
 from `op`.`a`
 join `op`.`b`
 join `op`.`c`
 where ((`op`.`a`.`a1` = `op`.`c`.`c1`) and (`op`.`b`.`b1` = `op`.`c`.`c1`))

sql2：
只有表A和表C进行连接，但WHERE子句多一条判定条件“A.a1 IS NOT NULL”；

SELECT A.*, C.* FROM A,C WHERE A.a1=C.c1 AND A.a1 IS NOT NULL; //只有表A和C进行连接
+------+------+------+------+
| a1   | a2   | c1   | c2   |
+------+------+------+------+
|    1 | A1   |    1 | C1   |
+------+------+------+------+

执行计划：
explain SELECT A.*, C.* FROM A,C WHERE A.a1=C.c1 AND A.a1 IS NOT NULL;
+----+-------------+-------+------------+------+---------------+------+---------+---------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref     | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+---------+------+----------+-------------+
|  1 | SIMPLE      | C     | NULL       | ALL  | NULL          | NULL | NULL    | NULL    |    3 |   100.00 | Using where |
|  1 | SIMPLE      | A     | NULL       | ref  | a1            | a1   | 5       | op.C.c1 |    1 |   100.00 | NULL        |
+----+-------------+-------+------------+------+---------------+------+---------+---------+------+----------+-------------+
优化器处理后：
/* select#1 */ select `op`.`a`.`a1` AS `a1`,
`op`.`a`.`a2` AS `a2`,
`op`.`c`.`c1` AS `c1`,
`op`.`c`.`c2` AS `c2`
from `op`.`a`
join `op`.`c`
where ((`op`.`a`.`a1` = `op`.`c`.`c1`) and (`op`.`c`.`c1` is not null))


sql3：
只有表A和表C进行连接,但WHERE子句比第二条SQL的WHERE子句内容更为简单
SELECT A.*, C.* FROM A,C WHERE A.a1=C.c1;
+------+------+------+------+
| a1   | a2   | c1   | c2   |
+------+------+------+------+
|    1 | A1   |    1 | C1   |
+------+------+------+------+

explain  SELECT A.*, C.* FROM A,C WHERE A.a1=C.c1;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | C     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |   100.00 | NULL                                               |
|  1 | SIMPLE      | A     | NULL       | ALL  | a1            | NULL | NULL    | NULL |    3 |    33.33 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
优化器处理后：
/* select#1 */ select `op`.`a`.`a1` AS `a1`,
`op`.`a`.`a2` AS `a2`,
`op`.`c`.`c1` AS `c1`,
`op`.`c`.`c2` AS `c2`
from `op`.`a`
join `op`.`c`
where (`op`.`a`.`a1` = `op`.`c`.`c1`)


第一条SQL：
SELECT A.*, C.* FROM A,B,C WHERE A.a1=B.b1 AND B.b1=C.c1;

第二条SQL：
SELECT A.*, C.* FROM A,C WHERE A.a1=C.c1 AND A.a1 IS NOT NULL;

第三条SQL：
SELECT A.*, C.* FROM A,C WHERE A.a1=C.c1;




对主外键参照的表进行外连接，可以消除主键表  --- MySQL不支持

SELECT A.*, C.* FROM A LEFT JOIN B ON (a1=b1) JOIN C ON (a1=c1);
表A外连接表B，然后连接表C，查询目标列没有表B的列，表B没有被消除
+------+------+------+------+
| a1   | a2   | c1   | c2   |
+------+------+------+------+
|    1 | A1   |    1 | C1   |
+------+------+------+------+

explain SELECT A.*, C.* FROM A LEFT JOIN B ON (a1=b1) JOIN C ON (a1=c1);
+----+-------------+-------+------------+--------+---------------+---------+---------+---------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref     | rows | filtered | Extra                                              |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | C     | NULL       | ALL    | NULL          | NULL    | NULL    | NULL    |    3 |   100.00 | NULL                                               |
|  1 | SIMPLE      | A     | NULL       | ALL    | a1            | NULL    | NULL    | NULL    |    3 |    33.33 | Using where; Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | B     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | op.C.c1 |    1 |   100.00 | Using index                                        |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------+------+----------+----------------------------------------------------+
优化器处理后：
/* select#1 */ select `op`.`a`.`a1` AS `a1`,
`op`.`a`.`a2` AS `a2`,
`op`.`c`.`c1` AS `c1`,
`op`.`c`.`c2` AS `c2`
from `op`.`a`
left join `op`.`b` on((`op`.`b`.`b1` = `op`.`c`.`c1`))
join `op`.`c`
where (`op`.`a`.`a1` = `op`.`c`.`c1`)
```
连接消除总结：  

注意连接消除与外连接消除的技术差别  
连接消除去掉的是被连接的某个对象  
外连接消除去掉的是外连接的语义，变形为内连接  



## Nest Join Elimination

嵌套连接消除：  

连接存在多个层次，用括号标识连接的优先次序。   
嵌套连接消除，就是消除嵌套的连接层次，把多个层次的连接减少为较少层次的连接，尽量“扁平化”    

例：  
```
创建表，命令如下：
CREATE TABLE B (b1 INT, b2 VARCHAR(9));
CREATE TABLE A (a1 INT, a2 VARCHAR(9));
CREATE TABLE C (c1 INT, c2 VARCHAR(9));
插入数据，命令如下：
INSERT INTO B VALUES(1, 'B1'), (NULL, 'B2'), (31, 'B31'), (32, 'B32'), (NULL, 'B4'),(5, 'B5'), (6, 'B6');
INSERT INTO A VALUES(1, 'A1'), (null, 'A2'), (NULL, 'A31'), (32, 'A32'), (4, 'A4'), (5, 'A5'), (NULL, 'A6');
INSERT INTO C VALUES(1, 'C1'), (NULL, 'C2'), (31, 'C31'), (NULL, 'C32'), (4, 'C4'), (NULL, 'C5'),(6, 'A6');

SELECT * FROM A JOIN (B JOIN C ON B.b1=C.c1) ON A.a1=B.b1  WHERE A.a1 > 1;
SQL语句的语义是B和C先连接，然后再和A连接

explain SELECT * FROM A JOIN (B JOIN C ON B.b1=C.c1) ON A.a1=B.b1  WHERE A.a1 > 1;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | A     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    7 |    33.33 | Using where                                        |
|  1 | SIMPLE      | B     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    7 |    14.29 | Using where; Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | C     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    7 |    14.29 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
优化器处理后：
/* select#1 */ select `op`.`a`.`a1` AS `a1`,
`op`.`a`.`a2` AS `a2`,
`op`.`b`.`b1` AS `b1`,
`op`.`b`.`b2` AS `b2`,
`op`.`c`.`c1` AS `c1`,
`op`.`c`.`c2` AS `c2`
from `op`.`a`
join `op`.`b`
join `op`.`c`
where ((`op`.`b`.`b1` = `op`.`a`.`a1`) and (`op`.`c`.`c1` = `op`.`a`.`a1`) and (`op`.`a`.`a1` > 1))
```
嵌套连接消除总结：

1. 嵌套连接消除的连接的层次,这是一种连接的语义顺序的变化

2. 连接消除，消掉的是一些被连接的对象

3. 外连接消除，消掉的是外连接的语义，使得外连接变形为内连接

----
Good luck!  
冬日暖阳
