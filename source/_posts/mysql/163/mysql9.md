---
title: SQL进阶
list_number: false
categories:
  - MySQL
comments: true
description:
  - SQL进阶
date: 2016-10-20 13:24:44
updated: 2016-10-20 13:24:44
tags:
---
# SQL进阶

示例：
```sql
CREATE TABLE play_list (
id bigint(20) not null comment '主键',
play_name varchar(255) default null comment '歌单名字',
userid bigint(20) not null comment '歌单作者账号id',
createtime bigint(20) default '0' comment '歌单创建时间',
updatetime bigint(20) default '0' comment '歌单更新时间',
bookedcount bigint(20) default '0' comment '歌曲的订阅人数',
trackcount int(11) default '0' comment '歌曲的数量',
status int(11) default '0' comment '状态，是否删除',
primary key (id),
key idx_createtime(createtime),
key idx_uid_ctime(userid,createtime)
) engine=innodb default charset=utf8 comment='歌单';

insert into play_list values(1,'老男孩',1,143023383,143023383,5,6,0);
insert into play_list values(2,'情歌王子',2,143023384,143023384,7,3,0);
insert into play_list values(3,'每日歌曲推荐',3,143023385,143023385,2,3,0);
insert into play_list values(4,'山河水',4,143023386,143023386,5,null,0);
insert into play_list values(5,'李荣浩',1,143023387,143023387,1,10,0);
insert into play_list values(6,'情深深',5,143023388,143023388,0,0,1);
```
## order by
- order by 语句用于根据指定的列对结果集进行排序
- order by 语句默认按照升序对记录进行排序，等价于 order by xxx asc； 可以使用desc 降序。

```sql
select * from play_list order by createtime;
+----+--------------------+--------+------------+------------+-------------+------------+--------+
| id | play_name          | userid | createtime | updatetime | bookedcount | trackcount | status |
+----+--------------------+--------+------------+------------+-------------+------------+--------+
|  1 | 老男孩             |      1 |  143023383 |  143023383 |           5 |          6 |      0 |
|  2 | 情歌王子           |      2 |  143023384 |  143023384 |           7 |          3 |      0 |
|  3 | 每日歌曲推荐       |      3 |  143023385 |  143023385 |           2 |          3 |      0 |
|  4 | 山河水             |      4 |  143023386 |  143023386 |           5 |       NULL |      0 |
|  5 | 李荣浩             |      1 |  143023387 |  143023387 |           1 |         10 |      0 |
|  6 | 情深深             |      5 |  143023388 |  143023388 |           0 |          0 |      1 |
+----+--------------------+--------+------------+------------+-------------+------------+--------+
6 rows in set (0.00 sec)


select * from play_list order by createtime desc;
+----+--------------------+--------+------------+------------+-------------+------------+--------+
| id | play_name          | userid | createtime | updatetime | bookedcount | trackcount | status |
+----+--------------------+--------+------------+------------+-------------+------------+--------+
|  6 | 情深深             |      5 |  143023388 |  143023388 |           0 |          0 |      1 |
|  5 | 李荣浩             |      1 |  143023387 |  143023387 |           1 |         10 |      0 |
|  4 | 山河水             |      4 |  143023386 |  143023386 |           5 |       NULL |      0 |
|  3 | 每日歌曲推荐       |      3 |  143023385 |  143023385 |           2 |          3 |      0 |
|  2 | 情歌王子           |      2 |  143023384 |  143023384 |           7 |          3 |      0 |
|  1 | 老男孩             |      1 |  143023383 |  143023383 |           5 |          6 |      0 |
+----+--------------------+--------+------------+------------+-------------+------------+--------+
6 rows in set (0.00 sec)
```

## distinct
- distinct 用于返回唯一不同的值
- 可以返回多列的唯一组合

```sql
select distinct userid from play_list;
+--------+
| userid |
+--------+
|      1 |
|      2 |
|      3 |
|      4 |
|      5 |
+--------+
5 rows in set (0.00 sec)


select distinct userid,play_name from play_list;
+--------+--------------------+
| userid | play_name          |
+--------+--------------------+
|      1 | 老男孩             |
|      2 | 情歌王子           |
|      3 | 每日歌曲推荐       |
|      4 | 山河水             |
|      1 | 李荣浩             |
|      5 | 情深深             |
+--------+--------------------+
6 rows in set (0.00 sec)
```

## group by
- group by 根据单列或多列对数据进行分组，通常结合聚合函数使用，如`count(*)`
- having 对分组进行过滤

```sql
select userid,count(*) from play_list group by userid;
+--------+----------+
| userid | count(*) |
+--------+----------+
|      1 |        2 |
|      2 |        1 |
|      3 |        1 |
|      4 |        1 |
|      5 |        1 |
+--------+----------+
5 rows in set (0.00 sec)


-- having过滤
select userid,count(*) from play_list group by userid having count(*) >=2;
+--------+----------+
| userid | count(*) |
+--------+----------+
|      1 |        2 |
+--------+----------+
1 row in set (0.00 sec)
```

## like
模糊匹配

通配符 | 描述
---|---
`%`  | 代表一个或多个字符
`_`  | 代表一个字符
`[charlist]` | 匹配括号中的任何单一字符
`[^charlist]`或`[!charlist]` | 匹配不在括号中的任何单一字符

```sql
select * from play_list where play_name like '%男孩%';
+----+-----------+--------+------------+------------+-------------+------------+--------+
| id | play_name | userid | createtime | updatetime | bookedcount | trackcount | status |
+----+-----------+--------+------------+------------+-------------+------------+--------+
|  1 | 老男孩    |      1 |  143023383 |  143023383 |           5 |          6 |      0 |
+----+-----------+--------+------------+------------+-------------+------------+--------+
1 row in set (0.01 sec)
```

## limit,offset
取指定条数，分页

```sql
select * from play_list where createtime between 143023383 and 143023389 limit 2 offset 3;
+----+-----------+--------+------------+------------+-------------+------------+--------+
| id | play_name | userid | createtime | updatetime | bookedcount | trackcount | status |
+----+-----------+--------+------------+------------+-------------+------------+--------+
|  4 | 山河水    |      4 |  143023386 |  143023386 |           5 |       NULL |      0 |
|  5 | 李荣浩    |      1 |  143023387 |  143023387 |           1 |         10 |      0 |
+----+-----------+--------+------------+------------+-------------+------------+--------+
2 rows in set (0.01 sec)

select * from play_list where createtime between 143023383 and 143023389 limit 3,2;
+----+-----------+--------+------------+------------+-------------+------------+--------+
| id | play_name | userid | createtime | updatetime | bookedcount | trackcount | status |
+----+-----------+--------+------------+------------+-------------+------------+--------+
|  4 | 山河水    |      4 |  143023386 |  143023386 |           5 |       NULL |      0 |
|  5 | 李荣浩    |      1 |  143023387 |  143023387 |           1 |         10 |      0 |
+----+-----------+--------+------------+------------+-------------+------------+--------+
2 rows in set (0.00 sec)


select * from play_list where createtime between 143023383 and 143023389 limit 1,2;
+----+--------------------+--------+------------+------------+-------------+------------+--------+
| id | play_name          | userid | createtime | updatetime | bookedcount | trackcount | status |
+----+--------------------+--------+------------+------------+-------------+------------+--------+
|  2 | 情歌王子           |      2 |  143023384 |  143023384 |           7 |          3 |      0 |
|  3 | 每日歌曲推荐       |      3 |  143023385 |  143023385 |           2 |          3 |      0 |
+----+--------------------+--------+------------+------------+-------------+------------+--------+
2 rows in set (0.00 sec)

select * from play_list where createtime between 143023383 and 143023389 limit 1 offset 2;
+----+--------------------+--------+------------+------------+-------------+------------+--------+
| id | play_name          | userid | createtime | updatetime | bookedcount | trackcount | status |
+----+--------------------+--------+------------+------------+-------------+------------+--------+
|  3 | 每日歌曲推荐       |      3 |  143023385 |  143023385 |           2 |          3 |      0 |
+----+--------------------+--------+------------+------------+-------------+------------+--------+
1 row in set (0.00 sec)

select * from play_list where createtime between 143023383 and 143023389 limit 0,2;
+----+--------------+--------+------------+------------+-------------+------------+--------+
| id | play_name    | userid | createtime | updatetime | bookedcount | trackcount | status |
+----+--------------+--------+------------+------------+-------------+------------+--------+
|  1 | 老男孩       |      1 |  143023383 |  143023383 |           5 |          6 |      0 |
|  2 | 情歌王子     |      2 |  143023384 |  143023384 |           7 |          3 |      0 |
+----+--------------+--------+------------+------------+-------------+------------+--------+
2 rows in set (0.00 sec)
```

## case when
case when 实现类似编程语言的if else 功能， 可以对SQL 的输出结果进行选择判断。

```sql
select play_name,
(case when trackcount is null then 0 else trackcount  end ) as trackcount
from play_list;
+--------------------+------------+
| play_name          | trackcount |
+--------------------+------------+
| 老男孩             |          6 |
| 情歌王子           |          3 |
| 每日歌曲推荐       |          3 |
| 山河水             |          0 |
| 李荣浩             |         10 |
| 情深深             |          0 |
+--------------------+------------+
6 rows in set (0.00 sec)
```

----
Good luck!  
冬日暖阳
