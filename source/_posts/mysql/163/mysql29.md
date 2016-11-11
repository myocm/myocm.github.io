---
title: MySQL参数调优
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL参数调优
date: 2016-11-08 13:27:53
updated: 2016-11-08 13:27:53
tags:
---

# MySQL参数调优

## 为什么要调整参数
- 不同服务器之间的配置、性能不一样
- 不同业务场景对数据的需求不一样
- MySQL的默认参数只是个参考值，并不适合所有的应用场景

## 优化之前我们需要知道什么
- 服务器相关配置
- 业务相关情况
- MySQL相关配置

## 服务器关注点
- 硬件情况
- 操作系统版本
- CPU、网卡节电模式
- 服务器numa设置
- RAID缓存  
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108133214912.png)  

### 磁盘调度策略 - Write Back
- 数据写入cache即返回，数据异步地从cache刷入存储介质  
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108133552224.png)
### 磁盘调度策略 - Write Through
- 数据同时写入cache和存储介质才返回写入成功  
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108133709267.png)

### Write Back VS Write Through
- Write Back 性能优于 Write Through
- Write Through 比 Write Back 安全性高

### RAID
- RAID （`R`edundant `A`rray of `I`ndependent `D`isks）
  - 生产环境里一般不太会用裸设备，通常会使用RAID卡对一块盘或多块盘做RAID
  - RAID卡会预留一块内存，来保证数据高效的存储与读取
  - 常见的RAID类型有：RAID1、RAID0、RAID10和RAID5

- RAID0 VS RAID1
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108134441183.png)

- RAID5 VS RAID10
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108134541895.png)

- RAID如何保证数据安全
  - BBU（`B`ackup `B`attery `U`nit）
  BBU保证在WB策略下，即使服务器发生掉电或宕机，也能够将缓存中的数据写入到磁盘，从而保证数据的安全

## MySQL有哪些注意事项

### MySQL的部署安装
- 推荐的MySQL版本 >= MySQL5.5
- 推荐的MySQL存储引擎： InnoDB

### MySQL的监控 - 系统的调优依据
- 实时监控MySQL的Slow Log
- 实时监控数据库服务器的负载情况
- 实时监控MySQL内部状态值
  - binlog文件大小（MB）
  - BufferPool命中率（%）
  - CPU利用率（%）
  - 磁盘读操作延时（ms/op）
  - 磁盘读取字节数（KB/s）
  - 磁盘读取次数（次/秒）
  - 占用磁盘存储空间（MB）
  - 磁盘写操作延时（ms/op）
  - 磁盘写入字节数（KB/s）
  - 磁盘写入次数（次/秒）
  - 磁盘IO利用率（%）
  - 占用内存量（MB）
  - 内存使用率（%）
  - 一般事务提交操作（次/秒）
  - 删除操作（次/秒）
  - 插入操作（次/秒）
  - 查询操作（次/秒）
  - 更新操作（次/秒）
  - 二阶段事务提交操作（次/秒）

#### 重点关注哪些MySQL Status
- Com_Select/Update/Delete/Insert
- Bytes_received/Bytes_sent
- Buffer Pool Hit Rate
- Threads_connected/Threads_created/Threads_running

### MySQL参数调优

#### 读优化
- 合理利用索引对MySQL查询性能至关重要
- 适当的调整MySQL参数也能提升查询性能

- innodb_buffer_pool_size
  - InnoDB存储引擎自己维护一块内存区域完成新老数据的替换
- innodb_thread_concurrency
  - InnoDB内部并发控制参数，设置为0代表不做控制
  - 如果并发请求较多，参数设置较小，后进来的请求将会排队

#### 写优化
- 表结构设计上使用自增字段作为表的主键
- 只对合适的字段加索引，索引太多影响写入性能
- 监控服务器磁盘IO情况，如果写延迟较大则需要扩容
- 选择正确的MySQL版本，合理设置参数

- 有助于提高写入性能的参数
  - innodb_flush_log_at_trx_commit && sync_binlog
  - innodb log file size
  - innodb_io_capacity
  - innodb insert buffer

- 主要影响MySQL写性能的两个参数
  - innodb_flush_log_at_trx_commit
  - sync_binlog

- innodb_flush_log_at_trx_commit
控制InnoDB事务的刷新方式，一共有三个值：0,1,2
N=0 - 每隔一秒，把事务日志缓存区的数据写到日志文件中，以及把日志文件的数据刷新到磁盘上（高效，但不安全）
N=1 - 每个事务提交的时候，就把事务日志从缓存区写到日志文件中，并且刷新日志文件的数据到磁盘上，优先使用此模式保障数据安全性（低效，非常安全）
N=2 - 每个事务提交的时候，把事务日志从缓存区写到日志文件中；每隔一秒，刷新一次日志文件，但不一定刷新到磁盘上，而是取决于操作系统的调度（高效，但不安全）

-  sync_binlog
控制每次写入binlog，是否都需要进行一次持久化

#### 如何保证事务安全
- innodb_flush_log_at_trx_commit 和 sync_binlog 都设置为 1
- 事务要和binlog保持一致性
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108143340403.png)

- 串行有哪些问题
  - SAS盘一般每秒只能有150-200个Fsync
  - 换算到数据库每秒只能执行50~60个事务

- 社区和官方的改进
  - MariaDB 提出改进，即使这两个参数都是1也能做到合并效果，性能得到了大幅提高
  - 官方吸收了MariaDB的思想，并在此基础上进行了改进，性能再次得到了提高
※ 官方在MySQL5.6版本之后才做了这个优化
※ Percona和MariaDB版本在MySQL5.5已经包含了这个优化

- 性能提升
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108143836193.png)

#### InnoDB Redo log
Write ahead log （WAL）
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108144005727.png)

- Redo log有哪些问题
  - 如果写入频繁导致Redo log 里对应的最老的数据脏页还没有刷新到磁盘，此时数据库将卡住，强制刷新脏页到磁盘
  - MySQL 默认配置两个文件才10M，非常容易写满，生产环境中应适当调整大小

- innodb_io_capacity
  - InnoDB每次刷多少个脏页，决定InnoDB存储引擎的吞吐能力
  - 在SSD等高性能存储介质下，应该提高该参数以提高数据库的性能

#### Insert Buffer
- 顺序读写 VS 随机读写
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108144453251.png)

- 随机请求性能远小于顺序请求
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108144549466.png)

※ 将尽可能多的 随机请求合并为顺序请求才是提高数据库性能的关键

- MySQL从5.1 版本开始支持Insert Buffer
- MySQL5.5版本之后同时支持update和delete的Merge
- Insert Buffer 只对二级索引且非唯一索引有效
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161108144820156.png)

## 总结
- 服务器配置要合理（内核版本、磁盘调度策略、RAID卡缓存）
- 完善的监控系统，提前发现问题
- 数据库版本要跟上，不要太新，也不要太老
- 数据库性能优化：
  查询优化：索引优化为主，参数优化为辅
  写入优化：业务优化为主，参数优化为辅



----
Good luck!  
冬日暖阳
