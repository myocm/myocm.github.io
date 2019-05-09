---
title: MySQL性能测试
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL性能测试
date: 2016-10-31 10:57:22
updated: 2016-10-31 10:57:22
tags:
---

#  MySQL性能测试

## 为什么需要做性能测试

- 对线上产品缺乏心理预估
- 重现线上异常
- 规划未来的业务增长
- 测试不同硬件软件配置

## 性能测试的分类

### 设备层的测试
- 关注指标
  - 服务器、磁盘性能
  - 磁盘坏块率
  - 服务器寿命

### 业务层的测试
- 针对业务进行测试


### 数据库层的测试
- 什么情况下要做MySQL的测试
  - 测试不同的MySQL分支版本
  - 测试不同的MySQL版本
  - 测试不同的MySQL参数搭配

## MySQL测试分类
  - CPU Bound
  - IO Bound
  ![](/img/markdown-img-paste-2016103111195853.png)

## 常见的测试工具
- 开源的MySQL性能测试工具
  - sysbench
  - tpcc-mysql
  - mysqlslap
- 针对业务编写的性能测试工具
  - blogbench

## 性能测试衡量指标
  - 服务吞吐量(TPS，QPS)
  - 服务响应时间
  - 服务并发性

## Sysbench
  - 业界较为出名的性能测试工具
  - 可以测试磁盘、CPU、数据库
  - 支持多种数据库：Oracle、DB2、MySQL
  - 需要自己下载编译安装
  - 建议版本：sysbench0.5以上，当前版本1.0
  - [主页](https://github.com/akopytov/sysbench)

### 编译Sysbench
- 下载sysbench
  - Git clone https://github.com/akopytov/sysbench
- 编译&安装
  - ./autogen.sh
  - ./configure
  - make  

```
centos6.5 运行./autogen.sh时，报错libtoolize: command not found
yum install libtool*  即可


运行./configure --prefix=/opt/freeware/sysbench  报如下错误
configure: error: mysql_config executable not found
********************************************************************************
ERROR: cannot find MySQL libraries. If you want to compile with MySQL support,
       please install the package containing MySQL client libraries and headers.
       On Debian-based systems the package name is libmysqlclient-dev.
       On RedHat-based systems, it is mysql-devel.
       If you have those libraries installed in non-standard locations,
       you must either specify file locations explicitly using
       --with-mysql-includes and --with-mysql-libs options, or make sure path to
       mysql_config is listed in your PATH environment variable. If you want to
       disable MySQL support, use --without-mysql option.
********************************************************************************

yum  install  mysql-devel

安装完成后，运行sysbench 命令，报错
./sysbench: error while loading shared libraries: libmysqlclient.so.21: cannot open shared object file: No such file or directory

是因为mysql是编译安装，对应的共享对象没有注册
注册共享对象即可：
1. cat /etc/ld.so.conf  ##查看共享库注册目录
include ld.so.conf.d/*.conf
2. cd ld.so.conf.d/
如果有mysql-x86_64.conf则修改，如果没有则创建一个
内容为mysql动态库的连接目录，如：
/usr/lib64/mysql
3. 如果/usr/lib64/mysql目录不存在，则创建目录，
在里面创建动态链接库的软连接，如：
lrwxrwxrwx 1 root root 54 2016-10-31 14:03:54 libmysqlclient.so -> /opt/freeware/mysql-8.0.0/lib/libmysqlclient.so.21.0.0
lrwxrwxrwx 1 root root 54 2016-10-31 14:03:37 libmysqlclient.so.21 -> /opt/freeware/mysql-8.0.0/lib/libmysqlclient.so.21.0.0
4. 运行ldconfig 命令重新载入动态配置，使上述配置生效。
```

### Sysbench流程
- 常见做法
![](/img/markdown-img-paste-20161031141917966.png)

#### Prepare语法
![](/img/markdown-img-paste-20161031142038603.png)  
![](/img/markdown-img-paste-20161031142424884.png)
```
./sysbench --test=/home/sdy/sysbench/sysbench/sysbench/tests/db/parallel_prepare.lua --oltp_tables_count=1 --rand-init=on --oltp-table-size=10000 --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=root --mysql-password=root --mysql-db=demo --max-requests=0 prepare
```
#### Sysbench表结构
```sql  
mysql> show create table sbtest1 \G
*************************** 1. row ***************************
       Table: sbtest1
Create Table: CREATE TABLE `sbtest1` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `k` int(10) unsigned NOT NULL DEFAULT '0',
  `c` char(120) NOT NULL DEFAULT '',
  `pad` char(60) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  KEY `k_1` (`k`)
) ENGINE=InnoDB AUTO_INCREMENT=10001 DEFAULT CHARSET=utf8mb4 MAX_ROWS=1000000
1 row in set (0.00 sec)
```

#### Run语法
![](/img/markdown-img-paste-20161031144359270.png)  
![](/img/markdown-img-paste-20161031144520508.png)
```
./sysbench --test=/home/sdy/sysbench/sysbench/sysbench/tests/db/oltp.lua --oltp-table-size=10000 --oltp_tables_count=1 --num-threads=100 --oltp-read-only=off --report-interval=10 --rand-type=uniform --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=root --mysql-password=root --mysql-db=demo --max-time=1000 --max-requests=0 run
```

#### Cleanup
- 手动drop掉表和数据库
- 使用sysbench提供的cleanup命令
![](/img/markdown-img-paste-20161031151050504.png)  
```
./sysbench --test=/home/sdy/sysbench/sysbench/sysbench/tests/db/parallel_prepare.lua --oltp_tables_count=1 --rand-init=on --oltp-table-size=10000 --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=root --mysql-password=root --mysql-db=demo --max-requests=0 cleanup
```

#### 特殊情况
- 写入测试（纯Insert操作）
![](/img/markdown-img-paste-20161031144837322.png)

#### Sysbench输出解读
![](/img/markdown-img-paste-2016103115085522.png)

## Tpcc-mysql
- TPC-C是专门针对联机交易处理系统(OLTP系统)的规范
- Tpcc-mysql 由percona根据规范实现

### TPCC流程
![](/img/markdown-img-paste-20161031151833719.png)  
![](/img/markdown-img-paste-20161031151901758.png)
- 下载tpcc-mysql
  - bzr branch lp:~percona-dev/perconatools/tpcc-mysql
- 编译安装
  - cd tpcc-mysql
  - make
    - export LD_LIBRARY_PATH=$MYSQL_HOME/lib
    - export C_INCLUDE_PATH=$MYSQL_HOME/include
    - export PATH=$MYSQL_HOME/bin:$PATH
### 使用tpcc-mysql的步骤
![](/img/markdown-img-paste-20161031152312810.png)

- 创建表结果
手工建库，使用下面的脚本建表和约束。
  - create_table.sql
  - add_fkey_idx.sql

- Tpcc-load
tpcc-load [server] [DB] [user] [pass] [warehouse]
![](/img/markdown-img-paste-2016103115271801.png)
```
./tpcc_load 127.0.0.1 tpcc root root 1
```

- Tpcc-start
tpcc-start -h server_host -P port -d database_name -u mysql_user -p mysql-password -w warehouses -c connections -r warmup_time -l running_time -i report_interval -f report_file
![](/img/markdown-img-paste-20161031153014782.png)
```
./tpcc_start -h 127.0.0.1 -P 3306 -d tpcc -u root -p root -w 1 -c 120 -r 30 -l 180 -i 10 -f report_file
```

- Tpcc-mysql输出解读
  - 运行过程的输出
  ![](/img/markdown-img-paste-2016103115313357.png)
  - 运行结束输出结果
  ![](/img/markdown-img-paste-20161031153159795.png)
```
 10, 411(2):3.794|8.120, 412(0):1.193|3.019, 41(0):0.529|1.412, 41(0):4.648|6.283, 41(0):7.640|9.171
###第一列代表时间，秒， 第二列，新增订单执行成功次数（超时次数）：90% 业务的响应时间| 最大响应时间
               第三列，支付业务
               第四列，订单查询业务
                    第五列，物流业务
               最后一列，仓储业务


<Raw Results>
 [0] sc:7202  lt:9  rt:0  fl:0     ##新增订单   sc  成功次数  lt 超出次数 rt 重试次数  fl 失败次数
 [1] sc:7209  lt:3  rt:0  fl:0     ##支付业务
 [2] sc:721  lt:0  rt:0  fl:0      ##订单查询业务
 [3] sc:721  lt:0  rt:0  fl:0      ##物流业务
 [4] sc:721  lt:0  rt:0  fl:0      ##仓储业务
in 180 sec.

<Raw Results2(sum ver.)>            ##计算两次，确保计算正确
 [0] sc:7204  lt:9  rt:0  fl:0
 [1] sc:7209  lt:3  rt:0  fl:0
 [2] sc:721  lt:0  rt:0  fl:0
 [3] sc:721  lt:0  rt:0  fl:0
 [4] sc:721  lt:0  rt:0  fl:0

<Constraint Check> (all must be [OK])   ##测试场景业务覆盖范围要求
[transaction percentage]
       Payment: 43.48% (>=43.0%) [OK]
  Order-Status: 4.35% (>= 4.0%) [OK]
      Delivery: 4.35% (>= 4.0%) [OK]
   Stock-Level: 4.35% (>= 4.0%) [OK]
[response time (at least 90% passed)]
     New-Order: 99.88%  [OK]
       Payment: 99.96%  [OK]
  Order-Status: 100.00%  [OK]
      Delivery: 100.00%  [OK]
   Stock-Level: 100.00%  [OK]

<TpmC>
   2403.667 TpmC      ##每分钟事务数          
```

## 总结
- IO Bound 测试数据量要远大于内存、CPU Bound 测试数据量要小于内存
- 测试时间建议大于60分钟，减小误差
- Sysbench更倾向于测试MySQL性能、TPCC更接近于业务
- 运行测试程序需要同时监控机器负载，MySQL各项监控指标



----
Good luck!  
冬日暖阳
