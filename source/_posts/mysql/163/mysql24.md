---
title: MySQL备份
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL备份
date: 2016-11-03 15:36:25
updated: 2016-11-03 15:36:25
tags:
---

# MySQL备份

## 基础知识
### 备份用途
- 数据灾备
  - 应对硬件故障数据丢失
  - 应对人为或程序bug导致数据误删除
- 制作镜像库以供服务
  - 需要将数据迁移、统计分析等用途
  - 需要为线上数据建立建立一个镜像

### 备份内容
- 数据
  - 数据文件或文本格式数据
- 操作日志(binlog)
  - 数据库变更日志
  ![](/img/markdown-img-paste-20161103154539948.png)

### 冷备份与热备份
- 冷备份
  - 关闭数据库服务，完整拷贝数据文件
- 热备份
  - 在不影响数据库读写服务的情况下备份数据库

### 物理备份与逻辑备份
- 物理备份
  - 以数据页的形式拷贝数据文件
- 逻辑备份
  - 导出为裸数据或者SQL(insert)语句

### 本地备份与远程备份
- 本地备份
  - 在数据库服务器本地进行备份
- 远程备份
  - 远程连接数据库进行备份

### 全量备份与增量备份
- 全量备份
  - 备份完整的数据库
- 增量备份
  - 只备份上一次备份以来发生修改的数据
![](/img/markdown-img-paste-20161103155256846.png)

### 备份周期
考虑因素：
- 数据库大小（决定备份时间）
- 恢复速度要求（快速or慢速）
- 备份方式（全量or增量）


## 常用工具及用法
- mysqldump - 逻辑备份，热备
- xtrabackup - 物理备份，热备
- Lvm/zfs snapshot - 物理备份
- mydumper - 逻辑备份，热备
- cp - 物理备份，冷备

### mysqldump
MySQL官方自带的命令行工具
主要示例：
1. 使用mysqldump备份表，库，实例
2. 使用mysqldump制作一致性备份
3. 使用mysqldump导出数据为csv格式
```sql
1)mysqldump -uroot -p123456 --socket=XXX --all-databases > XXX.sql
2)mysqldump -uroot -p123456 --socket=XXX --databases db2 > XXX.sql
3)mysqldump -uroot -p123456 --socket=XXX db2 t1 > XXX.sql
4)
create database db3;
surce XXX.sql;
5)mysqldump --single-transaction -uroot -p123456 --all-databases > XXX.sql
6)mysqldump -utest -ptest -hXXX -P3306 --all-databases > XXX.sql
7)mysqldump --single-transaction -uroot -p123456 db1 -T XXX
8)mysqldump --single-transaction -uroot -p123456 db1 -T XXX
```

### xtrabackup
特点：
- 开源，在线备份InnoDB表
- 支持限速备份，避免对业务造成影响
- 支持流备
- 支持增量备份
- 支持备份文件压缩与加密
- 支持并行备份与恢复，速度快
[官方网站](https://www.percona.com/downloads/XtraBackup/)  
[代码地址](https://github.com/percona/percona-xtrabackup/tree/release-2.4.4)

#### xtrabackup备份原理
- 基于InnoDB的crash-recovery功能
- 备份期间允许用户读写，写请求产生redo日志
- 从磁盘上拷贝数据文件
- 从InnoDB redo log file 实时拷贝走备份期间产生的所有redo日志
- 恢复的时候  数据文件+redo日志 = 一致性备份
![](/img/markdown-img-paste-20161103175351142.png)

#### 实用脚本innobackupex
- 开源Perl脚本，封装调用xtrabackup及一系列相关工具与OS操作，最终完成备份过程
- 支持备份InnoDB和其他引擎的表
- 备份一致性保证

- innobackupex备份基本流程
![](/img/markdown-img-paste-20161103175631788.png)

- innobackupex使用示例
1. 全量备份
```
默认在BACKUP-ROOT-DIR下面产生时间戳备份子目录
innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root /home/sdy/backup/innoback/

可以自定义备份子目录名称，使用  --no-timestamp 选项
innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root  --no-timestamp /home/sdy/backup/innoback/full_back
```
2. 增量备份
```
innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root --incremental /home/sdy/backup/innoback/   --incremental-basedir=/home/sdy/backup/innoback/2016-01-21_07-24-37
或
innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root --incremental  --incremental-basedir=/home/sdy/backup/innoback/2016-01-21_07-24-37   /home/sdy/backup/innoback/
```
3. 流方式备份
```
xbstream格式
innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root  --stream=xbstream /home/sdy/backup/innoback > /home/sdy/backup/innoback/stream_bak
或
tar格式
innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root  --stream=tar /home/sdy/backup/innoback > /home/sdy/backup/innoback/stream_bak.tar
```
4. 并行备份
```
innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root --parallel=4  /home/sdy/backup/innoback
```
5. 限流备份
```
限流10M
innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root --throttle=10  /home/sdy/backup/innoback
```
6. 压缩备份
```
innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root  --compress --compress-threads=4 /home/sdy/backup/innoback
```

7. 常用参数介绍
```
--kill-long-queries-timeout=#
                      This option specifies the number of seconds innobackupex
                      waits between starting FLUSH TABLES WITH READ LOCK and
                      killing those queries that block it. Default is 0
                      seconds, which means innobackupex will not attempt to
                      kill any queries.

--kill-long-query-type=all | update
                      This option specifies which types of queries should be
                      killed to unblock the global lock. Default is "all".

--slave-info        This option is useful when backing up a replication slave
                      server. It prints the binary log position and name of the
                      master server. It also writes this information to the
                      "xtrabackup_slave_info" file as a "CHANGE MASTER"
                      command. A new slave for this master can be set up by
                      starting a slave server on this backup and issuing a
                      "CHANGE MASTER" command with the binary log position
                      saved in the "xtrabackup_slave_info" file.
```

## 如何制定数据库备份策略
需要考虑的因素
1. 数据库是不是都是innodb引擎表    -->  备份方式，热备or冷备
2. 数据量大小                     -->  逻辑备份or物理备份，全量or增量
3. 数据库本地空间是否充足          -->  备份到本地or远程
4. 需要多快恢复                   -->  备份频率 小时or天


----
Good luck!  
冬日暖阳
