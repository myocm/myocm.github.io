---
title: MySQL权限管理
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL权限管理
date: 2016-10-19 16:04:56
updated: 2016-10-19 16:04:56
tags:
---
# MySQL权限管理


## 连接MySQL的必要条件
- 网络要通畅
- 用户名和密码要正确
- ip需要添加到数据库白名单
- 更细粒度的验证（库、表、列权限类型等）


## 权限粒度


### 数据权限
DATA： select， insert，delete，update

### 定义权限
Database ： create ，drop ，alter
Table    ： create ，drop ，alter
VIEW/FUNCTION/TRIGGER/PROCEDURE: create ，drop ，alter

### 管理权限
Shutdown Database
Replication Slave
Replication Client
File Privilege





## 数据库有哪些权限

```sql
show privileges;
+-------------------------+---------------------------------------+-------------------------------------------------------+
| Privilege               | Context                               | Comment                                               |
+-------------------------+---------------------------------------+-------------------------------------------------------+
| Alter                   | Tables                                | To alter the table                                    |
| Alter routine           | Functions,Procedures                  | To alter or drop stored functions/procedures          |
| Create                  | Databases,Tables,Indexes              | To create new databases and tables                    |
| Create routine          | Databases                             | To use CREATE FUNCTION/PROCEDURE                      |
| Create role             | Server Admin                          | To create new roles                                   |
| Create temporary tables | Databases                             | To use CREATE TEMPORARY TABLE                         |
| Create view             | Tables                                | To create new views                                   |
| Create user             | Server Admin                          | To create new users                                   |
| Delete                  | Tables                                | To delete existing rows                               |
| Drop                    | Databases,Tables                      | To drop databases, tables, and views                  |
| Drop role               | Server Admin                          | To drop roles                                         |
| Event                   | Server Admin                          | To create, alter, drop and execute events             |
| Execute                 | Functions,Procedures                  | To execute stored routines                            |
| File                    | File access on server                 | To read and write files on the server                 |
| Grant option            | Databases,Tables,Functions,Procedures | To give to other users those privileges you possess   |
| Index                   | Tables                                | To create or drop indexes                             |
| Insert                  | Tables                                | To insert data into tables                            |
| Lock tables             | Databases                             | To use LOCK TABLES (together with SELECT privilege)   |
| Process                 | Server Admin                          | To view the plain text of currently executing queries |
| Proxy                   | Server Admin                          | To make proxy user possible                           |
| References              | Databases,Tables                      | To have references on tables                          |
| Reload                  | Server Admin                          | To reload or refresh tables, logs and privileges      |
| Replication client      | Server Admin                          | To ask where the slave or master servers are          |
| Replication slave       | Server Admin                          | To read binary log events from the master             |
| Select                  | Tables                                | To retrieve rows from table                           |
| Show databases          | Server Admin                          | To see all databases with SHOW DATABASES              |
| Show view               | Tables                                | To see views with SHOW CREATE VIEW                    |
| Shutdown                | Server Admin                          | To shut down the server                               |
| Super                   | Server Admin                          | To use KILL thread, SET GLOBAL, CHANGE MASTER, etc.   |
| Trigger                 | Tables                                | To use triggers                                       |
| Create tablespace       | Server Admin                          | To create/alter/drop tablespaces                      |
| Update                  | Tables                                | To update existing rows                               |
| Usage                   | Server Admin                          | No privileges - allow connect only                    |
+-------------------------+---------------------------------------+-------------------------------------------------------+
33 rows in set (0.00 sec)
```
※ MySQL8.0 新加了`Create role`和`Drop role`两个权限。



## MySQL赋权操作
GRANT语法[官方参考文档](http://dev.mysql.com/doc/refman/8.0/en/grant.html)

### 如何新建一个用户并赋权
```sql
create user 'sdy'@'localhost' identified by 'password';
grant select on *.* to 'sdy'@'localhost' with grant option ;
```

### 非常规操作
向 mysql 库的user 表里插入记录，根据需要选择是否向db表 和 tables_priv 表插入记录  
执行 flush privileges命令，使权限生效  

### 授权的同时创建用户
5.7.9 以后版本不推荐
```sql
grant select on *.* to 'sdy'@'localhost' identified by 'password'  with grant option ;
```

### 查看用户权限
```sql
show grants for 'mysql.sys'@'localhost';
+---------------------------------------------------------------+
| Grants for mysql.sys@localhost                                |
+---------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'mysql.sys'@'localhost'                 |
| GRANT TRIGGER ON `sys`.* TO 'mysql.sys'@'localhost'           |
| GRANT SELECT ON `sys`.`sys_config` TO 'mysql.sys'@'localhost' |
+---------------------------------------------------------------+
3 rows in set (0.00 sec)


create user 'sdy.select'@'localhost' identified by 'password';
Query OK, 0 rows affected (0.08 sec)

grant select on *.* to 'sdy.select'@'localhost' with grant option;
Query OK, 0 rows affected (0.00 sec)


show grants for 'sdy.select'@'localhost';
+-------------------------------------------------------------------+
| Grants for sdy.select@localhost                                   |
+-------------------------------------------------------------------+
| GRANT SELECT ON *.* TO 'sdy.select'@'localhost' WITH GRANT OPTION |
+-------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### 更改用户权限
- 回收不需要的权限
```sql
revoke select on *.* from 'sdy.select'@'localhost';
```
- 更改用户密码
用新密码，grant语句重新授权
```sql
ALTER USER 'jeffrey'@'localhost'
  IDENTIFIED BY 'new_password' ;
```
或者
更改数据库记录，update mysql.user 表的password（5.7.9 之前版本），authentication_string（5.7.9 之后版本）字段  
需要flush privileges ，不推荐

### 删除用户
```sql
DROP USER 'jeffrey'@'localhost';
```

### 级联授权
授权时，使用`with grant option`允许被授予权限的人，把这个权限授予他人

## MySQL对应的权限信息存储结构
- MySQL 权限信息是存在数据库表中
- MySQL 账号对应的也加密存储在数据库表中
- 每一种权限类型在元数表明是否有该权限据里都是枚举类，表明是否有该权限


### 权限相关表
- user
- db
- tables_priv
- columns_priv
![](/img/markdown-img-paste-20161019171029515.png)

### MySQL权限上有哪些问题
在5.6时代  
使用binary 二进制安装管理用户没有设置密码  
MySQL 默认的test 库不受权限控制，存在安全风险  

可以使用 `mysql_secure_installation`命令，以安全方式初始化数据字典。


----
Good luck!  
冬日暖阳
