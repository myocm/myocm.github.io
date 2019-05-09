---
title: MySQL事务
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL事务
date: 2016-10-24 13:47:31
updated: 2016-10-24 13:47:31
tags:
---

# MySQL中的事务

## 事务定义
- 一系列有序的数据库操作：
- 要么全部成功。
- 要么全部回退到操作前状态。
- 中间状态对其他连接不可见。

## 事务的基本操作
基本操作 | 说明
---|---
start transaction; | 开始事务
commit; | 提交（全部完成）
rollback; | 回滚（回到初始状态）

## MySQL数据库事务控制操作
![](/img/markdown-img-paste-20161024140520481.png)

- 自动提交  
autocommit可以在session级别设置  
每个DML操作都自动提交  
DDL永远是自动提交，无法通过rollback回滚。  

## 事务的四个基本属性（ACID）
- 原子性(`A`tomicity)
- 一致性(`C`onsistency)
- 隔离性(`I`solation)
- 持久性(`D`urability)

### 原子性
- 包含在事务中的操作要么全部被执行，要么都不执行。
- 中途数据库或应用发生异常，未提交的事务都应该被回滚。

### 一致性
- 数据的正确性，合理性，完整性
- 数据一致性应该符合应用需要规则
  - 账户余额不能为负数？
  - 交易对象必须先有账号？
  - 用户账号不能重复？
- 事务的结果需要满足数据的一致性约束

### 隔离性
- 数据库事务在提交完成前，中间的任何数据变化对其他的事务都是不可见的
![](/img/markdown-img-paste-20161024152526554.png)
#### 数据库隔离现象
![](/img/markdown-img-paste-20161024152701975.png)
#### 数据库隔离等级
![](/img/markdown-img-paste-20161024153054942.png)
#### MySQL的事务隔离级别
- InnoDB默认标记为可重复读（Repeatable read）
- InnoDB并不是标准定义上的可重复读
- InnoDB默认在可重复读的基础上避免幻读

#### MySQL的事务隔离级别设置
- 查看隔离级别
```sql
show variables like '%iso%';
+---------------+----------------+
| Variable_name | Value          |
+---------------+----------------+
| tx_isolation  | READ-COMMITTED |
+---------------+----------------+
1 row in set (0.02 sec)
```
可在global/session下分别设置事务级别。  
建议使用read committed（同Oracle）  
或者使用默认级别repeatable read  

设置语法
```sql
set global tx_isolation = 'READ-UNCOMMITTED' ;
set global tx_isolation = 'READ-COMMITTED' ;
set global tx_isolation = 'REPEATABLE-READ' ;
set global tx_isolation = 'SERIALIZABLE' ;

set [session | global] transaction isolation level {READ-UNCOMMITTED | READ-COMMITTED | REPEATABLE-READ | SERIALIZABLE}
```



### 持久性
- 提交完成的事务对数据库的影响必须是永久性的
  - 数据库异常不会丢失事务更新
  - 通常认为成功写入磁盘的数据即为持久化成功
- 持久化的实现
![](/img/markdown-img-paste-20161024152154504.png)

## 事务与并发写
- 某个正在更新的记录在提交或回滚前不能被其他事务同时更新
![](/img/markdown-img-paste-20161024155020213.png)

## 事务回滚的实现
- 回滚段（rollback segment） 与数据前像
![](/img/markdown-img-paste-20161024155232929.png)

----
Good luck!  
冬日暖阳
