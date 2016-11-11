---
title: MySQL日常运维
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL日常运维
date: 2016-11-07 15:01:51
updated: 2016-11-07 15:01:51
tags:
---

# MySQL日常运维

## DBA运维工作
日常
- 导数据、数据修改、表结构变更
- 加权限、问题处理
其他
- 数据库选型部署、设计、监控、备份、优化等

### 日常运维工作

#### 导数据及注意事项
1. 数据最终形式（csv、sql文本，还是直接导入某库中）
2. 导数据的方法（mysqldump、select into outfile ）
3. 导数据注意事项
  - 导出为csv格式需要FILE权限，并且只能数据库本地导
  - 避免锁库锁表（mysqldump使用--single-transaction选项不锁表）
  - 避免对业务造成影响，尽量在镜像库做

#### 数据修改及注意事项
1. 修改前切记做好备份
2. 开事务做，修改完检查好了再提交
3. 避免一次修改大量数据，可以分批修改
4. 避免业务高峰期做

#### 表结构变更注意事项
1. 在低峰期做
2. 表结构变更是否会有锁（5.6包含online ddl功能）
3. 使用pt-online-schema-change 完成表结构变更
  - 可以避免主从延时
  - 可以避免负载过高，可以限速

#### 问题处理（数据库慢）
1. 数据库慢在哪？
2. show processlist 查看mysql连接信息
3. 查看系统状态（iostat，top，vmstat，tcprstat）

## 总结
1. 日常工作比较简单，但是任何一个操作都可能影响线上服务
2. 结合不同环境，不同要求，选择最合适的方法处理
3. 日常工作应该求稳不求快，保障线上稳定是DBA的最大责任



----
Good luck!  
冬日暖阳
