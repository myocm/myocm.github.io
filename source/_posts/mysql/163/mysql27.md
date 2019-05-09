---
title: MySQL主从复制
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL主从复制
date: 2016-11-07 14:01:40
updated: 2016-11-07 14:01:40
tags:
---
# MySQL主从复制

## 主从复制的模式
![](/img/markdown-img-paste-20161107140710955.png)

## 主从复制用途
- 实时灾备，用于故障切换
![](/img/markdown-img-paste-20161107140848670.png)  
![](/img/markdown-img-paste-20161107140929722.png)
- 读写分离，提供查询服务
![](/img/markdown-img-paste-20161107141007526.png)
- 备份，避免影响业务
![](/img/markdown-img-paste-20161107141041288.png)

## 主从复制部署
### 主从部署必要条件
- 主库开启binlog日志（设置log-bin参数）
- 主从server-id不同
- 从库服务器能连通主库

### 主从部署步骤
- 备份还原（mysqldump或xtrabackup）
- 授权（grant replication slave on *.* ）
- 配置复制，并启动（change master to）
- 查看主从复制信息（show slave status\G）

## 主从复制原理
![](/img/markdown-img-paste-2016110714143736.png)

## 主从复制存在的问题
存在问题
- 主库宕机后，数据可能丢失
- 从库只有一个sql thread，主库写压力大，复制很可能延迟
解决方法
- 半同步复制
- 并行复制

## MySQL semi-sync（半同步复制）
- 5.5集成到MySQL，以插件的形式存在，需要单独安装
- 确保事务提交后binlog至少传输到一个从库
- 不保证从库应用完这个事务的binlog
- 性能有一定的降低，响应时间会更长
- 网络异常或从库宕机，会卡住主库，直到超时或从库恢复
![](/img/markdown-img-paste-20161107143028630.png)  
![](/img/markdown-img-paste-20161107142225811.png)

### 配置半同步复制
- 安装semisync，只需要安装一次
- 主库
```
install plugin rpl_semi_sync_master SONAME 'semisync_master.so';
```
- 从库
```
install plugin rpl_semi_sync_slave SONAME 'semisync_slave.so';
```

动态设置
- 主库
```
set global rpl_semi_sync_master_enabled = 1;
set global rpl_semi_sync_master_timeout = N; -- master延迟切换为异步
```
- 从库
```
set global rpl_semi_sync_slave_enabled = 1;
```

### 半同步复制状态
![](/img/markdown-img-paste-20161107142834533.png)

## MySQL并行复制
- 官方社区版5.6中新增
- 并行是指从库多线程apply binlog
- 库级别并行应用binlog，同一个库数据更改还是串行（5.7版本并行复制基于事务组）

### 并行复制设置
```
set global slave_parallel_workers=10; --设置sql线程数为10
```

### 部分数据复制
- 主库添加参数
```
binlog_do_db=db1
binlog_ignore_db=db2
binlog_ignore_db=db3
```

- 或者在从库添加参数
```
replicate_do_db=db1
replicate_ignore_db=db1
replicate_do_table=db1.t1
replicate_ignore_table=db1.t1
replicate_wild_do_table=db%.%
replicate_wild_ignore_table=db1.%
```

## 级联复制
```
A --> B --> C
```
B中添加参数
```
log_slave_updates
```
B将把A的binlog记录到自己的binlog日志中


## 复制监控
查询从库状态
```
show slave status\G

*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.170.235.110
                  Master_User: eprep1
                  Master_Port: 3308
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000065
          Read_Master_Log_Pos: 46615397
               Relay_Log_File: mysql-relay-bin.001619
                Relay_Log_Pos: 46615610
        Relay_Master_Log_File: mysql-bin.000065
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table: information_schema.%,performance_schema.%
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 46615397
              Relay_Log_Space: 46615864
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 0b08572b-8c3e-11e5-bf6f-00163e005ac3
             Master_Info_File: /datavol/mysql/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
1 row in set (0.00 sec
```

## 复制出错处理
常见： 1062（主键冲突）  1032（记录不存在）
解决： 手动处理
或 跳过复制出错
```
set global sql_slave_skip_counter=1
```

## 总结
- MySQL主从复制是MySQL高可用性、高性能（负责均衡）的基础
- 简单、灵活，部署方式多样，可以根据不同业务场景部署不同复制结构
- MySQL主从复制目前也存在一些问题，可以根据需要部署复制增强功能来解决问题
- 复制过程中应该时刻监控复制状态，复制出差或延时可能给系统造成影响
- MySQL复制是MySQL数据库工程师必知必会的一项基本技能











----
Good luck!  
冬日暖阳
