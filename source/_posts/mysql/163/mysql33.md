---
title: MySQL5.7新特性
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL5.7新特性
date: 2016-11-11 16:13:23
updated: 2016-11-11 16:13:23
tags:
---
# MySQL5.7新特性

## MySQL版本时间轴
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161111161507737.png)

## MySQL5.7 简介
- 更强大的功能：原生JSON支持；新增安全性功能；新增地理位置支持GeoJSON，GeoHash特性
- 更强大的性能：某些场景性能较MySQL5.6提升2-3倍，Innodb只读事务优化；临时表改进；并行复制优化
- 更强大的运维：InnoDB Buffer Pool动态修改；sys schema引入；在线修改表结构等

## 功能

### JSON
- 原生支持JSON。包括字段的JSON类型和一系列JSON处理函数
- BLOB存储
- 混合存储结构化与非结构化数据
- 事务特性
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161111162051691.png)

### 安全性
- 初始化数据库： root密码由空变成了随机密码；不在自动创建test数据库
- 设置密码过期策略
```
ALTER USER 'user'@'localhost' PASSWORD EXPIRE INTERVAL 30 DAY;
```
- 锁定用户、解锁用户
```
ALTER USER 'user'@'localhost' ACCOUNT LOCK;
ALTER USER 'user'@'localhost' ACCOUNT UNLOCK;
```

### 虚拟列
```
create table order_lines(
orderid bigint,
userid bigint,
price decimal(10,2)
qty int,
sum_price decimal(10,2) generated
always as (qty*price) stored
);
```
STORED | VIRTUAL
---|---
Primary & secondary index | secondary index only
B-TREE, Full text search,R-TREE | B-TREE only
Duplication of data in base table and index | less storage
Independent of SE   | Requires SE support
Online operation | Online operation

## 性能

### 只读事务
- 绝大多数业务都是读多写少，优化只读事务意义重大
- 只读事务
```
start transaction read only
```
autocommit 下的不加锁的select语句
- MySQL5.7 通过不再为只读事务分配事务ID，不分配回滚段，降低内存开销，减少锁竞争等多种方式，优化了只读事务的开销

### 临时表
- 临时表：不需要严格一致性，重启MySQL或断开连接生命周期结束；只在当前会话中可见
- 优化策略：
  - 元数据修改不持久化
  - DML操作不写redo，关闭change buffer
  - 通过减少不必要的IO操作，提高临时表各种操作的性能

### 并行复制
- 基于MariaDB并行复制策略的改进，提升提交并行度
- 基于组提交的并行复制：从库按照主库的事务提交顺序提交，一个组提交的事务都是可以并行回放的
- slave-parallel-type：
  - DATABASE（默认） 基于库的并行复制方式
  - LOGICAL_CLOCK

### 基准测试对比
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161111163801686.png)


## 运维

### InnoDB Buffer Pool 动态修改
```
set global innodb_buffer_pool_size=402653184;
```
- innodb_buffer_pool_size=N*(innodb_buffer_pool_chunk_size*innodb_buffer_pool_instances)
- 调整buffer pool的过程，用户的新请求都会堵塞
- 查看buffer pool 调整状态
```
SHOW STATUS WHERE variable_name='innodb_buffer_pool_resize_status'
```
或者 查看 error log

### sys schema
- sys schema 是MySQL5.7.7中引入的一个系统库，包含了一系列视图、函数和存储过程
- 包含：系统运行状态；用户会话信息与资源占用；数据对象信息；SQL执行情况；索引覆盖；锁信息
- 基于performance schma的汇总与统计
- [官方手册](http://dev.mysql.com/doc/refman/5.7/en/sys-schema.html)
- 全表扫描的查询语句
```
select * from statements_with_full_table_scans;
```
- 用户会话信息与资源占用
```
select * from  host_summary;
```
- InnoDB锁等待情况
```
select * from  innodb_lock_waits;
```

### 在线修改表结构
- 在线修改表结构
  - 是否拷贝表；是否允许DML；是否允许select
- MySQL5.7版本新增alter table rename index 无需copy table
- 支持较好的操作：新增索引、删除索引、重命名字段
- 支持不好的操作：新增字段、删除字段、修改字段类型

## 总结
- MySQL5.7 是未来的趋势，拥抱变化
- 可运维性、易用性提升明显
- 在合适的场合选择合理的MySQL版本






----
Good luck!  
冬日暖阳
