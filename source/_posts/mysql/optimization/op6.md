---
title: MySQL优化 -- 6 条件简化
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL优化
date: 2018-01-02 15:37:57
updated: 2018-01-02 15:37:57
tags:
---
# 条件简化
## What is condition of a query statement ?
### 什么是条件?

SQL查询语句中，对元组进行过滤和连接的表达式  
形式上是出现在WHERE/JOIN-ON/HAVING的子句中的表达式。  
```sql
SELECT ...
[WHERE where_condition]
[HAVING where_condition]

join_condition:
    ON conditional_expr
```
>MySQL：Generally, you should use the **ON** clause for **conditions that specify how to join tables**,  
and the **WHERE** clause to **restrict which rows you want in the result set**.

### 条件优化技术
- 条件下推  
把与单个表相关的条件，放到对单表进行扫描的过程中执行。  
```sql
SELECT *
FROM A, B
WHERE A.a=1 and A.b=B.b;
```
执行顺序：  
  - 扫描A表，并带有条件A.a=1，把A表作为嵌套循环的外表  
  - 扫描B表，执行连接操作，并带有过滤条件A.b=B.b

  >说明：   
数据库系统都支持条件下推，且无论条件对应的列对象有无索引
系统自动进行优化，不用人工介入
- 条件化简

## What is Condition Simplification ?
条件化简
 1. WHERE、HAVING和JOIN-ON条件由许多表达式组成，而这些表达式在某些时候彼此之间存在一定的联系。
 2. 利用等式和不等式的性质，可以将WHERE、HAVING和ON条件化简
 3. 但不同数据库的实现可能不完全相同。

 常见的条件化简技术
### 1. 把HAVING条件并入WHERE条件  
在SQL语句中不存在GROUPBY条件或聚集函数的情况下，可以将HAVING条件与WHERE条件的进行合并。
优点：  
便于统一、集中化解条件子句，节约多次化解时间。
```sql
select * from t3 where a3>1 having b3=3;
+----+------+
| a3 | b3   |
+----+------+
|  3 |    3 |
+----+------+
-- 可以简化为
select * from t3 where a3>1 and b3=3;
```

### 2. 去除表达式中冗余的括号  
优点：  
可以减少语法分析时产生的AND和OR树的层次。---减少CPU的消耗  
示例：  
```SQL
((a AND b) AND (c AND d))
-- 化简为
a AND b AND c AND d
```
### 3. 常量传递  
优点：  
对不同关系可以使得条件分离后有效实施“选择下推”，从而可以极大减小中间关系的规模。  
示例：
```SQL
col_1 = col_2 AND col_2 = 3
-- 化简为
col_1=3 AND col_2=3
```
>注意：  
操作符“=、<、>、<=、>=、<>、<=>、LIKE”中的任何一个，在“col_1 <操作符> col_2”条件中都可能会发生常量传递

### 4. 消除死码  
化简条件，将不必要的条件去除  
示例：
```SQL
WHERE (0 > 1 AND s1 = 5)
```
“0 > 1”使得AND恒为假，则WHERE条件恒为假。    
此时就不必要再对该SQL语句进行优化和执行了，加快了查询执行的速度。
### 5. 表达式计算  
对可以求解的表达式，进行计算，得出结果。
```SQL
WHERE col_1 = 1 + 2
-- 化简为
WHERE col_1 = 3
```
### 6. 等式变换  
 化简条件（如反转关系操作符的操作数的顺序），从而改变某些表的访问路径
 ```SQL
 -a = 3    
 -- 化简为
 a = -3
 -- 好处是如果a上有索引，则可以利用索引扫描来加快访问。
 ```
### 7. 不等式变换  
化简条件，将不必要的重复条件去除。
```SQL
a > 10 AND b = 6 AND a > 2
-- 化简为
b = 6 AND a > 10
```
### 8. 布尔表达式变换
####  谓词传递闭包  
   一些比较操作符，如“<”、“>”等，具有传递性，可以起到化简表达式的作用
   ```SQL
   a>b AND b>2
   -- 可以推导出a>b AND b>2 AND a>2，“a>2”是一个隐含条件，
   -- 这样把“a>2”和“b>2”分别下推到对应的关系上，
   -- 就可以减少参与比较操作“a>b”的元组了
   ```
#### 布尔表达式被转换为一个等价的合取范式（CNF）  
   任何一个布尔表达式都能被转换为一个等价的合取范式（CNF）  
   合取范式格式为：C1 AND C2 AND… AND Cn；其中，Ck（1<=k<=n）称为合取项，每个合取项是不包含AND的布尔表达式  
   说明：  
   1 合取项只要有一个为假，整个表达式就为假，故代码中可以在发现一个合取项为假时，即停止其他合取项的判断，加快判断速度；   
   ```SQL
   WHERE(0 > 1 AND s1 = 5)  
   ```
   2 另外因为AND操作符是可交换的，所以优化器可以按照先易后难的顺序计算表达式，一旦发现一个合取项为假时，即停止其他合取项的判断，加快判断速度。
   ```SQL
   WHERE(A.a+B.b > 100 AND A.b = 5 AND 0 > 1 )
   -- 先求解：0 > 1 ，值为假，其他不再求解
   ```
#### 索引利用
   如果一个合取项上存在索引，则先判断索引是否可用，如能利用索引快速得出合取项的值，则能加快判断速度。  
   同理，OR表达式中的子项也可以利用索引
   ```SQL
   WHERE (A.a> 100 AND A.b = 5 AND... )
   -- 情况1：A表的a列上存在索引，b列无索引，则利用a上的索引找出元组，“A.b = 5” 作为过滤条件使用
   -- 情况2：A表的a列上不存在索引，b列有索引，则利用b上的索引找出元组，“A.a> 100” 作为过滤条件使用
   ```

## Condition Simplification of MySQL
### 1. 把HAVING条件并入WHERE条件：不支持
### 2. 去除表达式中冗余的括号：支持  
```sql  
explain select * from t3,t1 where (a1>1) AND ((a3>1) and b3=3) ;
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------------------------------------------------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | Extra                                              |
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------------------------------------------------+
|  1 | SIMPLE      | t3    | range | PRIMARY       | PRIMARY | 4       | NULL | 5000 | Using where                                        |
|  1 | SIMPLE      | t1    | range | PRIMARY       | PRIMARY | 4       | NULL | 5000 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+-------+---------------+---------+---------+------+------+----------------------------------------------------+
-- 查询优化器处理后的语句变形为：
/* select#1 */ select `test`.`t3`.`a3` AS `a3`,`test`.`t3`.`b3` AS `b3`,
`test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`
.`t3` join `test`.`t1`
where ((`test`.`t3`.`b3` = 3) and (`test`.`t1`.`a1` > 1) and (`test`.`t3`.`a3` > 1))
```
### 3. 常量传递：支持
```sql
explain select * from t3,t1 where (a3=b3 and b3=3) ;
+----+-------------+-------+-------+---------------+---------+---------+-------+-------+-------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows  | Extra |
+----+-------------+-------+-------+---------------+---------+---------+-------+-------+-------+
|  1 | SIMPLE      | t3    | const | PRIMARY       | PRIMARY | 4       | const |     1 | NULL  |
|  1 | SIMPLE      | t1    | ALL   | NULL          | NULL    | NULL    | NULL  | 10001 | NULL  |
+----+-------------+-------+-------+---------------+---------+---------+-------+-------+-------+
-- 查询优化器处理后的语句变形为：
/* select#1 */ select `test`.`t3`.`a3` AS `a3`,`test`.`t3`.`b3` AS `b3`,
`test`.`t1`.`a1` AS `a1`,`test`.`t1`.`b1` AS `b1` from `test`
.`t3` join `test`.`t1`
where ((`test`.`t3`.`a3` = 3) and (`test`.`t3`.`b3` = 3))
```
### 4. 消除死码：支持
```sql
explain select * from t3 where 0>1 and a3>1;
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra            |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
|  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Impossible WHERE |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
```

### 5. 表达式计算：支持
```SQL
explain select * from t3 where a3=1+2;
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | t3    | const | PRIMARY       | PRIMARY | 4       | const |    1 | NULL  |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
-- 查询优化器处理后的语句变形为：
/* select#1 */ select `test`.`t3`.`a3` AS `a3`,`test`.`t3`.`b3` AS `b3`
from `test`.`t3`
 where (`test`.`t3`.`a3` = (1 + 2))
--
 EXPLAIN EXTENDED SELECT * FROM t1, t2 WHERE 0>1+2 AND a1=10;
 +----+-------------+-------+------+---------------+------+---------+------+------+----------+------------------+
 | id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra            |
 +----+-------------+-------+------+---------------+------+---------+------+------+----------+------------------+
 |  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Impossible WHERE |
 +----+-------------+-------+------+---------------+------+---------+------+------+----------+------------------+
```

### 6. 等式变换：不支持
```sql
explain select * from t3 where -a3=1;
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows  | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
|  1 | SIMPLE      | t3    | ALL  | NULL          | NULL | NULL    | NULL | 10001 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
--
explain select * from t3 where a3=-1 ;
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | t3    | const | PRIMARY       | PRIMARY | 4       | const |    1 | NULL  |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
```

### 7. 不等式变换：不支持
```sql
explain select * from t3 where a3>1 and a3>2;
+----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
|  1 | SIMPLE      | t3    | range | PRIMARY       | PRIMARY | 4       | NULL | 5001 | Using where |
+----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
-- 查询优化器处理后的语句变形为：
/* select#1 */ select `test`.`t3`.`a3` AS `a3`,`test`.`t3`.`b3` AS `b3`
from `test`.`t3`
where ((`test`.`t3`.`a3` > 1) and (`test`.`t3`.`a3` > 2))
```

###  8. 布尔表达式变换
####  谓词传递闭包：不支持
```sql
explain select * from t3 where a3>b3 and b3>2;
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows  | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
|  1 | SIMPLE      | t3    | ALL  | NULL          | NULL | NULL    | NULL | 10002 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
--  查询优化器处理后的语句变形为：
/* select#1 */ select `test`.`t3`.`a3` AS `a3`,`test`.`t3`.`b3` AS `b3`
from `test`.`t3`
where ((`test`.`t3`.`a3` > `test`.`t3`.`b3`) and (`test`.`t3`.`b3` > 2))
```

#### 布尔表达式被转换为等价的合取范式：支持  
 参见死码消除中的示例

#### 索引的利用---AND操作符是可交换的：支持
 > MySQL支持对条件按表连接的次序进行排序，优先判断连接的表涉及的条件

### 9. IS NULL表达式优化：支持  
利用索引，支持“IS NULL”表达式的优化。
```SQL
CREATE TABLE `t2` (
  `a` int(11) NOT NULL,
  `b` int(11) NOT NULL,
  `c` int(11) NOT NULL,
  `d` int(11) NOT NULL,
  `e` int(11) NOT NULL,
  PRIMARY KEY (`a`,`b`),
  KEY `c_DESC_d_DESC` (`c`,`d`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
--
explain select * from t2 where c IS NULL;
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra            |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
|  1 | SIMPLE      | NULL  | NULL | NULL          | NULL | NULL    | NULL | NULL | Impossible WHERE |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------+
```





----
Good luck!  
冬日暖阳
