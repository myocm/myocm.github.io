---
title: MySQL慢日志分析
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL慢日志分析
date: 2016-11-03 10:54:39
updated: 2016-11-03 10:54:39
tags:
---

# MySQL慢日志分析

## 通过MySQL Processlist定位慢查询
```sql
show processlist ;
show full processlist ; -- 打印全部sql语句
```


## MySQL Slowlog 使用与介绍
- slowlog设置
```sql
set global slow_query_log=ON/OFF; --打开关闭slowlog
set global long_query_time=1/5/0.2; --设置慢查询阈值
set global log_output="FILE"/"TABLE"; -- 设置慢查询存储位置
show variables like 'slow_query_log_file'; --查询slowlog文件位置

select @@slow_query_log,@@long_query_time,@@log_output,@@slow_query_log_file;
+------------------+-------------------+--------------+--------------------------------------------+
| @@slow_query_log | @@long_query_time | @@log_output | @@slow_query_log_file                      |
+------------------+-------------------+--------------+--------------------------------------------+
|                0 |         10.000000 | FILE         | /home/mysql/mysql8/data/archlinux-slow.log |
+------------------+-------------------+--------------+--------------------------------------------+
1 row in set (0.00 sec)
```

- 慢查询表（mysql.slow_log）
```
desc slow_log;
+----------------+---------------------+------+-----+----------------------+--------------------------------+
| Field          | Type                | Null | Key | Default              | Extra                          |
+----------------+---------------------+------+-----+----------------------+--------------------------------+
| start_time     | timestamp(6)        | NO   |     | CURRENT_TIMESTAMP(6) | on update CURRENT_TIMESTAMP(6) |
| user_host      | mediumtext          | NO   |     | NULL                 |                                |
| query_time     | time(6)             | NO   |     | NULL                 |                                |
| lock_time      | time(6)             | NO   |     | NULL                 |                                |
| rows_sent      | int(11)             | NO   |     | NULL                 |                                |
| rows_examined  | int(11)             | NO   |     | NULL                 |                                |
| db             | varchar(512)        | NO   |     | NULL                 |                                |
| last_insert_id | int(11)             | NO   |     | NULL                 |                                |
| insert_id      | int(11)             | NO   |     | NULL                 |                                |
| server_id      | int(10) unsigned    | NO   |     | NULL                 |                                |
| sql_text       | mediumblob          | NO   |     | NULL                 |                                |
| thread_id      | bigint(21) unsigned | NO   |     | NULL                 |                                |
+----------------+---------------------+------+-----+----------------------+--------------------------------+
12 rows in set (0.00 sec)

select * from slow_log limit 1 \G
```

- 慢查询日志
```
# Time: 2016-11-02T13:19:54.723779Z
# User@Host: root[root] @ localhost []  Id:   106
# Query_time: 211.696652  Lock_time: 0.000000 Rows_sent: 0  Rows_examined: 0
SET timestamp=1478092794;
call proc_t1(10000,@a);
```
## 使用pt-query-digest分析慢查询
1. 使用tcpdump抓取mysql协议包
```
tcpdump -s 65535 -x -nn -q -tttt -i any -c 1000 port 3306 > mysql.tcp.txt
```
2. 使用pt-query-digest分析包
```
pt-query-digest --type tcpdump --limit 100 --watch-server 192.168.1.11:3306 mysql.tcp.txt > mysql.slow.sql
```
3. 使用pt-query-digest分析慢日志
```
pt-query-digest --type slowlog --order-by Rows_examined:sum/--order-by Query_time:cnt  /tmp/mysql-slow.log > /tmp/slow3306.sql
```

- tcpdump 参数含义
  - `-s`  截取报文的长度，0为全部报文，默认为68字节。  
  - `-x`  当分析和打印时, tcpdump 会打印每个包的头部数据, 同时会以16进制打印出每个包的数据(但不包括连接层的头部).总共打印的数据大小不会超过整个数据包的大小与snaplen 中的最小值. 必须要注意的是, 如果高层协议数据没有snaplen 这么长,并且数据链路层(比如, Ethernet层)有填充数据, 则这些填充数据也会被打印.
  - `-nn`  不转换协议和端口号
  - `-q`   快速输入，打印更少的协议信息，从而使输出行更短。
  - `-tttt` 在每一dump行，打印一个默认格式的时间戳。
  - `-i` any  监听接口
  - `-c`  接收到指定数量的数据包后退出
  - `port` 监听端口




----
Good luck!  
冬日暖阳
