---
title: MySQL触发器、存储过程与自定义函数UDF
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL触发器、存储过程与自定义函数UDF
date: 2016-10-21 14:26:01
updated: 2016-10-21 14:26:01
tags:
---

# MySQL触发器、存储过程与自定义函数UDF

## 触发器（triggers）
- 触发器是加在表上的一个特殊程序，当表上出现特定的事件（insert/update/delete）时触发该程序执行。
- 可以用于数据订正； 迁移表；实现特定的业务逻辑。
- 基本语法 [官方参考文档](http://dev.mysql.com/doc/refman/8.0/en/create-trigger.html)
```sql
CREATE TRIGGER
Name: 'CREATE TRIGGER'
Description:
Syntax:
CREATE
   [DEFINER = { user | CURRENT_USER }]
   TRIGGER trigger_name
   trigger_time trigger_event
   ON tbl_name FOR EACH ROW
   [trigger_order]
   trigger_body

trigger_time: { BEFORE | AFTER }

trigger_event: { INSERT | UPDATE | DELETE }

trigger_order: { FOLLOWS | PRECEDES } other_trigger_name
```

- new.列名 更新后的列值
- old.列名 更新前的列值，只读
```sql
delimiter //
create trigger trg_upd_t
before update on t
for each row
begin
    if new.name < 'd' then
        set new.name = 'ddd';
    elseif new.name > 'd' then
        set new.name = 'zzz';
    end if;
end;  //
delimiter ;

select * from t;
+------+------------------------------------------------------+
| id   | name                                                 |
+------+------------------------------------------------------+
|    1 | abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz |
|    2 | mmmmmmmmmm                                           |
+------+------------------------------------------------------+
2 rows in set (0.00 sec)

update t set name= 'nnn' where id = 2;
Query OK, 1 row affected (0.04 sec)
Rows matched: 1  Changed: 1  Warnings: 0

select * from t;
+------+------------------------------------------------------+
| id   | name                                                 |
+------+------------------------------------------------------+
|    1 | abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz |
|    2 | zzz                                                  |
+------+------------------------------------------------------+
2 rows in set (0.00 sec)

update t set name= 'abc' where id = 2;
Query OK, 1 row affected (0.04 sec)
Rows matched: 1  Changed: 1  Warnings: 0

select * from t;
+------+------------------------------------------------------+
| id   | name                                                 |
+------+------------------------------------------------------+
|    1 | abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz |
|    2 | ddd                                                  |
+------+------------------------------------------------------+
2 rows in set (0.00 sec)

show triggers \G;
*************************** 1. row ***************************
             Trigger: trg_upd_t
               Event: UPDATE
               Table: t
           Statement: begin
if new.name < 'd' then
set new.name = 'ddd';
elseif new.name > 'd' then
set new.name = 'zzz';
end if;
end
              Timing: BEFORE
             Created: 2016-01-07 15:18:12.87
            sql_mode: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
             Definer: root@localhost
character_set_client: utf8
collation_connection: utf8_bin
  Database Collation: utf8_bin
1 row in set (0.00 sec)

ERROR:
No query specified
```
触发器对性能有损耗，应慎重使用。  
同一类事件在一个表中只能创建一次(MySQL5.7及以前版本)。MySQL8.0版本开始允许在一个表上创建多个同类型触发器。
对于事务表，触发器执行失败则整个语句回滚。  
Row格式主从复制，触发器不会在从库上执行。  
使用触发器时应防止递归执行，如：
```sql
delimiter //
create trigger trg_upd_score
before update on `stu`
for each row
begin
        update stu set score = 20
        where name = old.name;
end; //
delimiter ;
```

## 存储过程（procedures ）

定义：存储过程是存储在数据库端的一组SQL语句集，用户可以通过存储过程名和传参多次调用的程序模块。  

特点：  
a） 使用灵活，可以使用流控制语句、自定义变量等完成复杂的业务逻辑。  
b） 提高数据安全性，屏蔽应用程序直接对表的操作，易于进行审计。  
c） 减少网络传输。  
d） 提高代码维护的复杂度，实际使用中要评估场景是否合适。  

创建语法 [官方参考文档](http://dev.mysql.com/doc/refman/8.0/en/create-procedure.html)
```sql
create procedure
Name: 'CREATE PROCEDURE'
Description:
Syntax:
CREATE
    [DEFINER = { user | CURRENT_USER }]
    PROCEDURE sp_name ([proc_parameter[,...]])
    [characteristic ...] routine_body

CREATE
    [DEFINER = { user | CURRENT_USER }]
    FUNCTION sp_name ([func_parameter[,...]])
    RETURNS type
    [characteristic ...] routine_body

proc_parameter:
    [ IN | OUT | INOUT ] param_name type

func_parameter:
    param_name type

type:
    Any valid MySQL data type

characteristic:
    COMMENT 'string'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC        --//声明函数返回值是否确定
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
```

```sql
delimiter //
create procedure proc_t1
(in total int,out result int)
begin
        declare i int;
        set i = 1;
        set result = 0;
        if total <= 0 then
          set total = 1;
      end if;
        while i <= total do
          set result = result + i;
          insert into  t(id,name) values (i,result);
          set i = i+1;
        end while ;
end; //
delimiter ;


call proc_t1(5,@a);
Query OK, 1 row affected (0.05 sec)

select @a;
+------+
| @a   |
+------+
|   15 |
+------+
1 row in set (0.00 sec)

select * from t;
+------+------------------------------------------------------+
| id   | name                                                 |
+------+------------------------------------------------------+
|    1 | abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz |
|    2 | ddd                                                  |
|    1 | 1                                                    |
|    2 | 3                                                    |
|    3 | 6                                                    |
|    4 | 10                                                   |
|    5 | 15                                                   |
+------+------------------------------------------------------+
7 rows in set (0.00 sec)

show create procedure proc_t1 \G;
*************************** 1. row ***************************
           Procedure: proc_t1
            sql_mode: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
    Create Procedure: CREATE DEFINER=`root`@`localhost` PROCEDURE `proc_t1`(in total int,out result int)
begin
        declare i int;
        set i = 1;
        set result = 0;
        if total <= 0 then
          set total = 1;
      end if;
        while i <= total do
          set result = result + i;
          insert into  t(id,name) values (i,result);
          set i = i+1;
        end while ;
end
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8_bin
1 row in set (0.00 sec)

ERROR:
No query specified
```

### 流程控制语言


- IF
```sql
IF seach_condition THEN statement_list
[ELSEIF seach_condition THEN statement_list]
[ELSE statement_list]
END IF
```

- CASE
```sql
CASE case_value
WHEN when_value THEN statement_list
[ELSE statement_list]
END CASE
```

- WHILE
```sql
WHILE seach_condition
DO statement_list
END WHILE
```
- REPEAT
```sql
REPEAT  statement_list
UNTIL seach_condition
END REPEAT
```

## 自定义函数（User-Defined Function ，UDF）
自定义函数与存储过程类似，但是必须带有返回值（RETURN）。  
自定义函数与sum(),max() 等MySQL 原生函数使用方法类似：  
select func(val);  
select  * from tbl where col=func(val);  
由于自定义函数可能在遍历数据中使用，要注意性能损耗。  

基本语法 [官方参考文档](http://dev.mysql.com/doc/refman/8.0/en/create-procedure.html)

```sql
CREATE
    [DEFINER = { user | CURRENT_USER }]
    FUNCTION sp_name ([func_parameter[,...]])
    RETURNS type
    [characteristic ...] routine_body


func_parameter:
    param_name type

type:
    Any valid MySQL data type

characteristic:
    COMMENT 'string'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC        --//声明函数返回值是否确定
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }

routine_body:
    Valid SQL routine statement
```

```sql
delimiter //

CREATE function func_t1(total INT)
RETURNS INT
begin
  DECLARE i INT;
  DECLARE result INT;

  SET i = 1;
  SET result = 0;

  IF total <= 0 THEN
    SET total = 1;
  end IF;

  WHILE i <= total do
    SET result = result + 1;
    INSERT INTO t(id,name) VALUES(i,result);
    SET i = i+1;
  end WHILE;
  RETURN result;
end; //

delimiter ;

select func_t1(15) from dual;
+-------------+
| func_t1(15) |
+-------------+
|          15 |
+-------------+
1 row in set (0.06 sec)

select   * from t;
+------+------------------------------------------------------+
| id   | name                                                 |
+------+------------------------------------------------------+
|    1 | abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyz |
|    2 | ddd                                                  |
|    1 | 1                                                    |
|    2 | 3                                                    |
|    3 | 6                                                    |
|    4 | 10                                                   |
|    5 | 15                                                   |
|    1 | 1                                                    |
|    2 | 2                                                    |
|    3 | 3                                                    |
|    4 | 4                                                    |
|    5 | 5                                                    |
|    6 | 6                                                    |
|    7 | 7                                                    |
|    8 | 8                                                    |
|    9 | 9                                                    |
|   10 | 10                                                   |
|   11 | 11                                                   |
|   12 | 12                                                   |
|   13 | 13                                                   |
|   14 | 14                                                   |
|   15 | 15                                                   |
+------+------------------------------------------------------+
22 rows in set (0.00 sec)
```

※ 触发器和存储过程不利于水平扩展，多用于统计和运维操作中。




----
Good luck!  
冬日暖阳
