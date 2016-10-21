---
title: MySQL常用函数及运算符
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL常用函数及运算符
date: 2016-10-20 16:04:13
updated: 2016-10-20 16:04:13
tags:
---

# MySQL常用函数

## 集合函数
聚合函数面向一组数据，对数据进行聚合运算后返回单一的值

MySQL 聚合函数基本语法：  
select function(列) from 表

常用聚合函数

函数 | 描述
---|---
AVG() | 返回列的平均值
COUNT(DISTINCT) | 返回列去重后的行数
COUNT() | 返回列的行数
MAX() | 返回列的最大值
MIN() | 返回列的最小值
SUM() | 返回列的总和
GROUP_CONCAT() | 返回一组值的连接字符串（MySQL独有）

例子：
```sql
create table song_list (
id bigint(20) not null comment '主键',
song_name varchar(255) not null comment '歌曲名',
artist varchar(255) not null comment '艺术家',
createtime bigint(20) default '0' comment '歌曲创建时间',
updatetime bigint(20) default '0' comment '歌曲更新时间',
album varchar(255) not null comment '专辑',
palycount int(11) default '0' comment '点播次数',
status int(11) default '0' comment '状态，是否删除',
primary key(id),
key idx_artist(artist),
key idx_album(album)
) engine=Innodb default charset=utf8 comment='歌曲列表';

insert into song_list values (1,'Good Lovin\' Gone Bad','Bad Company',0,0,'Straight Shooter',453,0);
insert into song_list values (2,'Weep No More','Bad Company',0,0,'Straight Shooter',280,0);
insert into song_list values (3,'Shooting Star','Bad Company',0,0,'Straight Shooter',530,0);
insert into song_list values (4,'大象 ','李志',0,0,'1701',560,0);
insert into song_list values (5,'定西','李志',0,0,'1701',1023,0);
insert into song_list values (6,'红雪莲','洪启',0,0,'红雪莲',220,0);
insert into song_list values (7,'风柜来的人','李宗盛',0,0,'作品李宗盛',566,0);
```

- 查询每个专辑的最大播放次数和平均播放次数。
```sql
select album,max(playcount),avg(playcount) from song_list group by album;
+------------------+----------------+----------------+
| album            | max(playcount) | avg(playcount) |
+------------------+----------------+----------------+
| 1701             |           1023 |       791.5000 |
| Straight Shooter |            530 |       421.0000 |
| 作品李宗盛       |            566 |       566.0000 |
| 红雪莲           |            220 |       220.0000 |
+------------------+----------------+----------------+
4 rows in set (0.00 sec)
```

- 查询全部歌曲中，最多的播放次数最多与最少的播放次数。
```sql
select song_name,playcount from song_list where playcount in (
select max(playcount) from song_list
union
select min(playcount) from song_list );
+-----------+-----------+
| song_name | playcount |
+-----------+-----------+
| 定西      |      1023 |
| 红雪莲    |       220 |
+-----------+-----------+
2 rows in set (0.00 sec)

select max(playcount),min(playcount) from song_list;
+----------------+----------------+
| max(playcount) | min(playcount) |
+----------------+----------------+
|           1023 |            220 |
+----------------+----------------+
1 row in set (0.00 sec)
```

- 查询播放次数最多的歌曲
```sql
-- 错误查询，和sql_mode 有关，如果在严格模式下报下面的错误，如果在非严格模式下，song_name 为随机
select song_name,max(playcount) from song_list;
+----------------------+----------------+
| song_name            | max(playcount) |
+----------------------+----------------+
| Good Lovin`'` Gone Bad |           1023 |
+----------------------+----------------+
1 row in set (0.00 sec)

-- 错误查询（如果有并列第一的话，就错了）
select song_name,playcount from song_list order by playcount desc limit 1;
+-----------+-----------+
| song_name | playcount |
+-----------+-----------+
| 定西      |      1023 |
+-----------+-----------+
1 row in set (0.00 sec)

-- 正确查询
select song_name,playcount from song_list where playcount =(select max(playcount) from song_list);
+-----------+-----------+
| song_name | playcount |
+-----------+-----------+
| 定西      |      1023 |
+-----------+-----------+
1 row in set (0.00 sec)

insert into song_list values(8,'烟花三月','童丽',0,0,'烟花三月',1023,0);
Query OK, 1 row affected (0.00 sec)

select song_name,playcount from song_list where playcount =(select max(playcount) from song_list);
+--------------+-----------+
| song_name    | playcount |
+--------------+-----------+
| 定西         |      1023 |
| 烟花三月     |      1023 |
+--------------+-----------+
2 rows in set (0.01 sec)
```

- count()函数
```sql
select count(*) from song_list;
+----------+
| count(*) |
+----------+
|        8 |
+----------+
1 row in set (0.01 sec)

select count(1) from song_list;
+----------+
| count(1) |
+----------+
|        8 |
+----------+
1 row in set (0.00 sec)

select count(song_name) from song_list;
+------------------+
| count(song_name) |
+------------------+
|                8 |
+------------------+
1 row in set (0.00 sec)
```
※ count(`*`) 和 count(1) 基本一样，没有明显的性能差别
※ count(`*`) 和 count(song_name) 的差别是，count(song_name) 会除去 song_name is null 的情况。

- 显示每张专辑的歌曲列表
```sql
select album,group_concat(song_name) from song_list group by album;
+------------------+-------------------------------------------------+
| album            | group_concat(song_name)                         |
+------------------+-------------------------------------------------+
| 1701             | 大象 ,定西                                      |
| Straight Shooter | Good Lovin`'` Gone Bad,Weep No More,Shooting Star |
| 作品李宗盛       | 风柜来的人                                      |
| 烟花三月         | 烟花三月                                        |
| 红雪莲           | 红雪莲                                          |
+------------------+-------------------------------------------------+
5 rows in set (0.00 sec)

注意：
group_concat 能够拼接的最大字符长度，有下面的参数决定。

show global variables like '%concat%';
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| group_concat_max_len | 1024  |
+----------------------+-------+
1 row in set (0.04 sec)
```

- 使用聚合函数，对数据进行行列转换
```sql
create table test1(
user varchar(50),
`key` varchar(10),
value varchar(50)
);

insert into test1 values('li','age','18');
insert into test1 values('li','dep','2');
insert into test1 values('li','sex','male');

insert into test1 values('sun','age','44');
insert into test1 values('sun','dep','3');
insert into test1 values('sun','sex','female');

insert into test1 values('wang','age','20');
insert into test1 values('wang','dep','3');
insert into test1 values('wang','sex','male');

select * from test1;
+------+------+--------+
| user | key  | value  |
+------+------+--------+
| li   | age  | 18     |
| li   | dep  | 2      |
| li   | sex  | male   |
| sun  | age  | 44     |
| sun  | dep  | 3      |
| sun  | sex  | female |
| wang | age  | 20     |
| wang | dep  | 3      |
| wang | sex  | male   |
+------+------+--------+
9 rows in set (0.00 sec)

select user,
max(case when `key`='age' then value end ) as age,
max(case when `key`='dep' then value end ) as dep,
max(case when `key`='sex' then value end ) as sex
from test1 group by user;
+------+------+------+--------+
| user | age  | dep  | sex    |
+------+------+------+--------+
| li   | 18   | 2    | male   |
| sun  | 44   | 3    | female |
| wang | 20   | 3    | male   |
+------+------+------+--------+
3 rows in set (0.01 sec)
```

##  预定义函数
预定义函数面向单值数据，返回一对一的处理结果（聚合函数可以理解成多对一）。

预定义函数基本语法

select function(列) from 表  
select * from 表 where 列= function(value)

### 字符串函数

函数 | 描述
---|---
length() | 返回列的字节数
char_length() | 返回列的字符数
trim(),rtrim(),ltrim() | 去除两边/右边/左边的空格
substring(str,pos,[len]) | 从pos位置截取字符串str，截取len长度
locate(substr,str,[pos]) | 返回substr在str字符串中的位置
replace(str,from_str,to_str) | 将str字符串中from_str替换成to_str
lower(),upper() | 字符串转换为小写/大写

```sql
create table t ( id int (11),name varchar(100));
Query OK, 0 rows affected (0.02 sec)

insert into t values(1,'abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz');
Query OK, 1 row affected (0.00 sec)

select substring(name,26) from t;
+-----------------------------+
| substring(name,26)          |
+-----------------------------+
| zabcdefghijklmnopqrstuvwxyz |
+-----------------------------+
1 row in set (0.01 sec)


select substring(name,27) from t;
+----------------------------+
| substring(name,27)         |
+----------------------------+
| abcdefghijklmnopqrstuvwxyz |
+----------------------------+
1 row in set (0.01 sec)

select substring(name,-3) from t;
+--------------------+
| substring(name,-3) |
+--------------------+
| xyz                |
+--------------------+
1 row in set (0.00 sec)

select substring(name,27,3) from t;
+----------------------+
| substring(name,27,3) |
+----------------------+
| abc                  |
+----------------------+
1 row in set (0.00 sec)

select locate('d','abcdefg') ;
+-----------------------+
| locate('d','abcdefg') |
+-----------------------+
|                     4 |
+-----------------------+
1 row in set (0.00 sec)

select locate('d','abcdefgdf',5) ;
+---------------------------+
| locate('d','abcdefgdf',5) |
+---------------------------+
|                         8 |
+---------------------------+
1 row in set (0.00 sec)

select locate('d','abcdefgdf') ;
+-------------------------+
| locate('d','abcdefgdf') |
+-------------------------+
|                       4 |
+-------------------------+
1 row in set (0.00 sec)
```

### 时间函数

函数 | 描述
---|---
curdate() | 当前日期
curtime() | 当前时间
now() | 显示当前日期时间（常用）
unix_timestamp() | 当前时间戳
date_format(date,format) | 按指定格式显示时间
date_add(date,interval unit) | 计算指定日期向后加一段时间
date_sub(date,interval unit)  | 计算指定日期向前减一段时间

```sql
select curdate(),curtime(),now(),unix_timestamp();
+------------+-----------+---------------------+------------------+
| curdate()  | curtime() | now()               | unix_timestamp() |
+------------+-----------+---------------------+------------------+
| 2016-10-20 | 18:02:24  | 2016-10-20 18:02:24 |       1476957744 |
+------------+-----------+---------------------+------------------+
1 row in set (0.00 sec)


select from_unixtime( unix_timestamp()), unix_timestamp(),now();
+----------------------------------+------------------+---------------------+
| from_unixtime( unix_timestamp()) | unix_timestamp() | now()               |
+----------------------------------+------------------+---------------------+
| 2016-10-20 18:02:49              |       1476957769 | 2016-10-20 18:02:49 |
+----------------------------------+------------------+---------------------+
1 row in set (0.00 sec)

select date_format('2016-01-07 02:45:26','%D %M %Y');
+-----------------------------------------------+
| date_format('2016-01-07 02:45:26','%D %M %Y') |
+-----------------------------------------------+
| 7th January 2016                              |
+-----------------------------------------------+
1 row in set (0.00 sec)

select date_format('2016-01-07 02:45:26','%H:%i:%s');
+-----------------------------------------------+
| date_format('2016-01-07 02:45:26','%H:%i:%s') |
+-----------------------------------------------+
| 02:45:26                                      |
+-----------------------------------------------+
1 row in set (0.00 sec)

select date_format(now(),'%D %M %Y');
+-------------------------------+
| date_format(now(),'%D %M %Y') |
+-------------------------------+
| 20th October 2016             |
+-------------------------------+
1 row in set (0.00 sec)
```

- 时间日期运算

date + INTERVAL expr unit  
date - INTERVAL expr unit   

DATE_ADD(date,INTERVAL expr unit)  
DATE_SUB(date,INTERVAL expr unit)  

```sql
select now() + interval 1 day;
+------------------------+
| now() + interval 1 day |
+------------------------+
| 2016-10-21 18:05:55    |
+------------------------+
1 row in set (0.00 sec)

select now() - interval 1 hour;
+-------------------------+
| now() - interval 1 hour |
+-------------------------+
| 2016-10-20 17:06:32     |
+-------------------------+
1 row in set (0.00 sec)

select now() - interval 1 week;
+-------------------------+
| now() - interval 1 week |
+-------------------------+
| 2016-10-13 18:07:57     |
+-------------------------+
1 row in set (0.00 sec)

select date_sub(now(),interval 1 week);
+---------------------------------+
| date_sub(now(),interval 1 week) |
+---------------------------------+
| 2016-10-13 18:08:18             |
+---------------------------------+
1 row in set (0.00 sec)

select str_to_date('2016-02-29','%Y-%m-%d')+interval 1 day;
+-----------------------------------------------------+
| str_to_date('2016-02-29','%Y-%m-%d')+interval 1 day |
+-----------------------------------------------------+
| 2016-03-01                                          |
+-----------------------------------------------------+
1 row in set (0.00 sec)
```

### 数字处理函数

函数 | 描述
---|---
ABS() | 返回数值的绝对值
CEIL() | 对小数向上取整 CEIL(1.2)=2
ROUND() | 四舍五入
POW(num,n) | Num的n 次幂 POW(2,3)=8
FLOOR() | 对小数向下取整 FOOR(1.2)=1
MOD(N,M) | 取模（返回n除以m的余数）=N%M
RAND() | 取0~1之间的一个随机数

```sql
select a.num,case when a.num <=0.65 then 1 else 0 end do from   (select  rand() num from dual  ) a ;
+----------------------+----+
| num                  | do |
+----------------------+----+
| 0.022154221331844314 |  1 |
+----------------------+----+
1 row in set (0.01 sec)


select 1+ceil (rand()*4) num;
+-----+
| num |
+-----+
|   2 |
+-----+
1 row in set (0.00 sec)


select 1+ceil (rand()*4) num ;
+-----+
| num |
+-----+
|   2 |
+-----+
1 row in set (0.00 sec)
```

- IP地址转换函数  
inet_aton()    把ip地址字符串转换为数字  
inet_ntoa()    把数字转换成字符串ip  

```sql
select inet_aton('61.135.169.121');
+-----------------------------+
| inet_aton('61.135.169.121') |
+-----------------------------+
|                  1032300921 |
+-----------------------------+
1 row in set (0.00 sec)

select inet_ntoa(1032300921);
+-----------------------+
| inet_ntoa(1032300921) |
+-----------------------+
| 61.135.169.121        |
+-----------------------+
1 row in set (0.00 sec)
```

## 运算符

### 比较运算

函数 | 描述
IS,IS NOT | 判定布尔值IS True，IS NOT False，IS NULL
`>,>=` | 大于，大于等于
<,<= |　小于，小于等于
=,`<=>`（MySQL特有）　 |　等于，可以判断NULL的等于
!=,<> |　不等于
BETWEEN M AND N |　取 M 和 N 之间的值 等价于 >=M and <= N
IN,NOT IN |　检查是否在或不在一组值中

```sql
select * from song_list where id  between 1 and 3;
+----+----------------------+-------------+------------+------------+------------------+-----------+--------+
| id | song_name            | artist      | createtime | updatetime | album            | playcount | status |
+----+----------------------+-------------+------------+------------+------------------+-----------+--------+
|  1 | Good Lovin' Gone Bad | Bad Company |          0 |          0 | Straight Shooter |       453 |      0 |
|  2 | Weep No More         | Bad Company |          0 |          0 | Straight Shooter |       280 |      0 |
|  3 | Shooting Star        | Bad Company |          0 |          0 | Straight Shooter |       530 |      0 |
+----+----------------------+-------------+------------+------------+------------------+-----------+--------+
3 rows in set (0.00 sec)

--between ... and ... 不能从大到小
select * from song_list where id  between 3 and 1;
Empty set (0.00 sec)

select * from song_list where id  not between 1 and 3;
+----+-----------------+-----------+------------+------------+-----------------+-----------+--------+
| id | song_name       | artist    | createtime | updatetime | album           | playcount | status |
+----+-----------------+-----------+------------+------------+-----------------+-----------+--------+
|  4 | 大象            | 李志      |          0 |          0 | 1701            |       560 |      0 |
|  5 | 定西            | 李志      |          0 |          0 | 1701            |      1023 |      0 |
|  6 | 红雪莲          | 洪启      |          0 |          0 | 红雪莲          |       220 |      0 |
|  7 | 风柜来的人      | 李宗盛    |          0 |          0 | 作品李宗盛      |       566 |      0 |
|  8 | 烟花三月        | 童丽      |          0 |          0 | 烟花三月        |      1023 |      0 |
+----+-----------------+-----------+------------+------------+-----------------+-----------+--------+
5 rows in set (0.00 sec)
```


### 算术，逻辑运算

`*,/,DIV,%,MOD,-,+ `  
`NOT,AND,&&,OR,XOR,|`

```sql
select 1+2*3 a;
+---+
| a |
+---+
| 7 |
+---+
1 row in set (0.00 sec)

select 6 DIV 3 ;
+---------+
| 6 DIV 3 |
+---------+
|       2 |
+---------+
1 row in set (0.00 sec)

select 1 OR 0 ;
+--------+
| 1 OR 0 |
+--------+
|      1 |
+--------+
1 row in set (0.00 sec)

select 1 XOR 0 ;
+---------+
| 1 XOR 0 |
+---------+
|       1 |
+---------+
1 row in set (0.00 sec)

select 1 XOR 1 ;
+---------+
| 1 XOR 1 |
+---------+
|       0 |
+---------+
1 row in set (0.00 sec)
```

###  操作符的优先级

优先级 | 操作符
---|---
1 |INTERVAL
2 |BINARY, COLLATE
3 |!
4 |- (unary minus), ~ (unary bit inversion)
5 |^
6 |*, /, DIV, %, MOD
7 |-, +
8 |<<, >>
9 |&
10||
11|= (comparison), <=>, >=, >, <=, <, <>, !=, IS, LIKE, REGEXP, IN
12|BETWEEN, CASE, WHEN, THEN, ELSE
13|NOT
14|AND, &&
15|XOR
16|OR, ||
17|= (assignment), :=

操作符优先级[官方参考文档](http://dev.mysql.com/doc/refman/8.0/en/operator-precedence.html)


----
Good luck!  
冬日暖阳
