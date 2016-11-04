---
title: MySQL恢复
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL恢复
date: 2016-11-04 14:52:47
updated: 2016-11-04 14:52:47
tags:
---
# MySQL恢复
- 使用MySQL恢复命令、工具
- 不同场景MySQL恢复方法


##  什么时候需要恢复数据
- 硬件故障（如磁盘损坏）
- 人为删除（如误删数据、被黑）
- 业务回滚（如游戏bug需要回档）
- 正常需求（如部署镜像库，查看历史某时刻数据）

## 数据恢复的必要条件
- 有效备份
- 完整的数据库操作日志（binlog）

## 数据恢复思路
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161104153340569.png)
1. 备份4 + binlog4 恢复到故障时间点（适用于各种数据丢失场景）
2. 挖掘binlog4 获取相关SQL语句，构造反转SQL语句并应用到数据库（只适用于记录丢失，且binlog必须是row格式）

```sql
t1(id primary key , a int)

##反转SQL语句：
insert into t1(id,a) values(1,1) --> delete t1 where id=1 and a=1
update t1 set a=5 where id=1     --> update t1 set a=1 where id=1
delete from t1 where id=1        --> insert into t1(id,a) values(1,1)
```

## 数据库恢复工具与命令
- mysqldump备份  --> source恢复
- xtrabackup备份 --> xtrabackup恢复
- binlog备份     --> mysqlbinlog恢复

### mysqldump 的还原方法
因mysqldump 是导出的sql语句，直接应用即可还原数据

`mysql -uroot -proot test < t.sql`
或
`mysql> source t.sql`

### xtrabackup 的还原方法
因为xtrabackup 是物理备份，因此，如果数据库磁盘故障，都是数据文件，可以使用此方法还原。

copy-back 时需要 mysql datadir 目录为空，并且mysql 数据库停止。


- 完整备份的还原
```
1. innobackupex --apply-log  BACK-BASE-DIR
2. innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root --copy-back ./2016-01-26_13-51-26/
```

- 增量备份还原
```

1. innobackupex --apply-log --redo-only  BACK-BASE-DIR
2. innobackupex --apply-log --redo-only  BACK-BASE-DIR  --incremental-dir=INCREMENTAL-DIR-1
3. innobackupex --apply-log --redo-only  BACK-BASE-DIR  --incremental-dir=INCREMENTAL-DIR-2  
    ......
n. innobackupex --apply-log  BACK-BASE-DIR  --incremental-dir=INCREMENTAL-DIR-n
n+1. innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root --copy-back BACK-BASE-DIR
```

- 压缩备份还原
```
解压
1. innobackupex --decompress /home/sdy/backup/2016-01-26_15-04-37     并且   find ./ -name "*.qp" | xargs rm -f \{}
应用日志
2. innobackupex --apply-log /home/sdy/backup/2016-01-26_15-04-37
还原
3. innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root --copy-back BACK-BASE-DIR
```

- 流备份还原
xbstream 格式
```
解压
1. mkdir stream  ;  xbstream -C stream  -x < stream.bak
应用日志
2. innobackupex --parallel=4 --apply-log --use-memory=200MB /home/sdy/backup/stream
还原
3. innobackupex --defaults-file=/home/sdy/mysql/bin/my.cnf --user=root --password=root --copy-back /home/sdy/backup/stream
```
  tar 格式
  ```
解压
1. mkdir tar ; cd tar ; tar xf ../tar.bak
2.应用日志
3.还原
```
### mysqlbinlog 工具恢复数据
```
mysqlbinlog --start-position=120 --stop-position=2150  mysql-bin.000008 | mysql -uroot -proot
```

## 恢复示例
### 恢复误删数据
case：误操作，删除数据忘记带完整条件，执行delete from user where age>30 [and sex=male]
需求：将删除数据还原
恢复前提：完整的数据库操作日志（binlog）

### 恢复误删表、库
case：业务被黑，表被删除了（drop table user）
需求：将表恢复
前提：备份+备份以来完整binlog

### 将数据库恢复到指定时间点
case：游戏bug，导致很多玩家利用bug刷金币
需求：游戏回滚，数据库也需要回滚
前途：有效备份+完整的数据库操作日志（binlog）

> 恢复是一件非常苦逼的差事，尽量避免做。我们要做数据卫士而不是救火队员。
（限速应该严格把控权限，数据变更操作应事先测试，操作时做好备份）
有效备份（+binlog）是重中之重，对数据库定期备份是必须的
备份是一切数据恢复的基础



----
Good luck!  
冬日暖阳
