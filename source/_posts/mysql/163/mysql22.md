---
title: MySQL日志系统
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL日志系统
date: 2016-11-02 10:54:24
updated: 2016-11-02 10:54:24
tags:
---
# MySQL日志系统

- MySQL日志的分类
- 各种日志的主要用途
- 各种日志的使用方法

## 什么是日志
- 日志（log）是一种顺序记录事件流水的文件
- 记录计算机程序运行过程中发生了什么
- 多种多样的用途
  - 帮助分析程序问题
  - 分析服务请求的特征，流量等
  - 判断工作是否成功执行
  - ......

## MySQL日志的分类

### 服务器日志
- 记录进程启动运行过程中的特殊事件，帮助分析MySQL服务遇到的问题
- 根据需求抓取特定的SQL语句，追踪性能可能存在问题的业务SQL
### 事务日志
- 记录应用程序对数据的所有更改
- 可用于数据恢复
- 可用于实例间数据同步
![](/img/markdown-img-paste-20161102110911946.png)

## 服务错误日志
- 记录实例启动运行过程中重要消息
- 配置参数
```
log_error=/data/mysql_data/node_1/mysqld.log
```
- 内容并非全是错误消息
- 如果mysqld进程无法正常启动，首先需要查看错误日志

## 慢查询日志
- 记录执行时间超过一定阈值的SQL语句
- 配置参数
```
slow_query_log=1
slow_query_log_file=/data/mysql_data/node_1/mysql_slow.log
long_query_time=5
```
- 用于分析系统中可能存在性能问题的SQL

## 综合查询日志
- 如果开启将会记录系统中所有SQL语句
- 配置参数
```
general_log=1
general_log_file=/data/mysql_data/node_1/mysql_gen.log
```
- 偶尔用于帮助分析系统问题，对性能有影响

### 查询日志的输出与文件切换
- 日志输出参数
```
log_output={file|table|none}
```
- 如果日志文件过大，可以定期截断并切换新文件
```
flush logs;
```
※ `flush logs;` 对binary log 也有效，也是截断日志，并重新生成一个新的日志文件。

## 存储引擎事务日志
- 部分存储引擎拥有重做日志（redo log）
- 如InnoDB，TokuDB等WAL(`W`rite `A`head `L`og)机制存储引擎
- 日志随着事务commit优先持久化，确保异常恢复不丢数据
- 日志顺序写性能较好

### InnoDB事务日志重用机制
- InnoDB事务日志采用两组文件交替重用
![](/img/markdown-img-paste-2016110211280515.png)  
![](/img/markdown-img-paste-20161102112735609.png)  

- 相关参数
  - innodb_log_group_home_dir  
日志目录，通常和innodb_data_home_dir相同，高并发下可以设置到不同的磁盘阵列，减少IO冲突，提高性能。
  - innodb_log_files_in_group  
该变量控制日志文件数。`默认值为2。静态变量,需要重启生效`。
  - innodb_log_buffer_size  
该变量将数据存入到内存中，可以减少大量的IO资源消耗。当事务提交时，保存脏数据，后续在刷新到磁盘。当我们调整innodb_buffer_pool_size大小时，innodb_log_buffer_size和innodb_log_file_size也应该做出相应的调整。
  - innodb_log_file_size  
该变量控制日志文件大小，默认5M，太小，`静态变量,需要重启生效`。

## 二进制日志binlog
- binlog(binary log)
- 记录数据引起数据变化的SQL语句或数据逻辑变化的内容
- MySQL服务层记录，无关存储引擎
- binlog的主要作用
  - 基于备份恢复数据
  - 数据库主从同步
  - 挖掘分析SQL语句

### 开启binlog
- 主要参数
```
log_bin=/data/mysql_data/node_1/mysql-bin
sql_log_bin=1
sync_binlog=1
```
- 查看binlog文件
```
show binary logs;
```

### binlog管理
- 主要参数
```
max_binlog_size=100MB
expire_logs_days=7
```
- binlog始终生成新文件，不会重用
- 手工清理binlog
```
purge binary logs to 'mysql-bin.000009';
purge binary logs before '2016-11-01 10:00:00';
```

### 查看binlog内容
- 日志(log)
```
show binlog events in 'mysql-bin.000011';
show binlog events in 'mysql-bin.000011' from 60 limit 3;
```
- mysqlbinlog工具
```
mysqlbinlog  /data/mysql_data/node_1/mysql-bin.000005
--start-datetime | --stop-datetime
--start-position | --stop-position
```
![](/img/markdown-img-paste-20161102114219122.png)

### binlog格式
- 主要参数
```
binlog_format={ROW|STATEMENT|MIXED}
```
- 查看row模式的binlog内容
```
mysqlbinlog --base64-output=decode-rows -v /data/mysql_data/node_1/mysql-bin.000005
```



----
Good luck!  
冬日暖阳
