---
title: MySQL分布式架构
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL分布式架构
date: 2016-11-11 11:37:42
updated: 2016-11-11 11:37:42
tags:
---
# MySQL分布式架构

## 分布式数据库
- 区别单机，突破单台服务器的性能瓶颈上限
- 一般由管理节点机器，计算节点，数据节点组成
- 数据节点统一由分布式数据库管理节点控制

## 分布式数据库的分类
- shared Nothing VS Shared Anything
![](/img/markdown-img-paste-20161111114807700.png)

### Shared Nothing
![](/img/markdown-img-paste-20161111114921125.png)

### Shared Anything
![](/img/markdown-img-paste-20161111115004872.png)

## MySQL分布式架构
- 分布式事务处理与XA协议
- MySQL分布式事务处理
- MySQL分布式事务的局限与改进

### 分布式事务
- Distributed（XA）Transactions
- 2pc（Two Phase Commit）Transactions
![](/img/markdown-img-paste-20161111115341556.png)

### MySQL分布式事务
- 什么是分布式事务处理  
  分布式事务一般包含3个部分
  - APP
    一般指事务的发起者
  - Resource Manager，RM
    资源管理器，主要是提供外部程序进行资源共享访问
  - Transaction Manager/Coordinator(TM or TC)
    TM或者TC是事务的协调者，对于分布式，在提交时，可能有部分节点会提交失败，这个时候，需要TC来决定哪些事务需要重新提交，哪些事务需要回滚

### XA In MySQL
- Only a Resource Manager
- InnoDB Only：innodb_support_xa

### MySQL XA事务的局限
- 客户端退出，prepare状态的XA事务被回滚
  根据分布式事务的原理，所有prepare成功的事务都应该被提交，比如下面的客户端操作
  ```
xa start '111' ;
insert into t values(1);
xa end '111' ;
xa prepare '111' ;
  ```
- Server Crash 后重启，提交prepare的事务，造成主从数据不一致
MySQL官方手册中有一段话：
"If an XA Transaction has reached the PREPARE state and the MySQL server is killed(for example,with kill -9 on Unix) or shutdown abnormally, the Transaction can be continued after the server restarts. Howerver, if the client reconnects and commits the Transaction,the Transaction will be absent from the binary log even though it has been committed. This means the data and the binary log have gone out of synchrony. An implication is that XA cannot be used safely together with replication. "

- 为什么一直没有人修复
看似简单，其实解决代价很大，另外，MySQL的XA事务用的真不多，官方觉得没必要

- MySQL官方最终修复
http://dev.mysql.com/worklog/task/?id=6860  
![](/img/markdown-img-paste-20161111153252783.png)


## 开源分布式软件MyCat

### Mycat的由来
- java语言编写
- JDK版本要求>1.7版本
- 基于阿里的cobar实现，完全兼容MySQL协议
- 有专业的维护团队，更新频繁，文档写的比较好
- 功能较为完善
- 分布式事务支持的不够好
- 支持各种数据库：MySQL、Oracle、MongoDB
- 目前不支持在线水平扩容
- 要做MySQL界的Tomcat

### Mycat架构
![](/img/markdown-img-paste-20161111153832112.png)

### Mycat的功能特点
- 数据拆分，丰富的拆分策略，数据库水平扩容
- 读写分离，支持用户友好的读写分离策略
- 兼容MySQL协议，用户可以以最小的代价接入Mycat

### 数据拆分
- 枚举算法
```
<tableRule name="sharding-by-intfile">
<rule>
<columns>user_id</columns>
<algorithm>hash-int</algorithm>
</rule>
</tableRule>
<function name="hash-int" class="org.opencloudb.route.function.PartitionByFileMap">
<property name="mapFile">partition-hash-int.txt</property>
<property name="type">0</property>
<property name="defaultNode">0</property>
</function>
```

- Range算法
```
<tableRule name="auto-sharding-long">
<rule>
<columns>user_id</columns?
<algorithm>rang-long</algorithm>
</rule>
</tableRule>
<function name="rang-long" class="org.opencloudb.route.function.AutoPartitionByLong">
<property name="mapFile">autopartiton-long.txt</property>
<property name="defaultNode">0</property>
</function>
```

- Hash取模算法
```
<tableRule name="mod-long">
<rule>
<columns>user_id</columns?
<algorithm>mod-long</algorithm>
</rule>
</tableRule>
<function name="mod-long" class="org.opencloudb.route.function.PartitionByMod">
<!-- how many data nodes -->
<property name="count">3</property>
</function>
```

### Mycat里的重要概念
- 全局ID
  - MySQL： auto_increment
  - Mycat： global sequnce number
```
<system><property name="sequnceHandlerType">0</property></system>
conf/server.xml
```
- 基于文件
```
cat sequence_conf.properties
#default global sequence
GLOBAL.HISIDS=
GLOBAL.MIMID=10001
GLOBAL.MAXID=20000
GLOBAL.CURID=10000

insert into table1(id,name) values(next value for MYCATSEQ_GLOBAL,'test');
```
实现简单、安全性较差


- 基于数据库
比直接存文件要安全的多  
但是DB要支持高可用  
![](/img/markdown-img-paste-20161111155841450.png)
- 基于本地时间戳
不依赖任何服务  
ID分配一般都较大  
![](/img/markdown-img-paste-20161111160000531.png)

###Mycat的部署与安装
- 重要概念
  - DataHost
  - DataBase Node（DBN）
  - 分区规则
  - 分区键

- 部署安装
![](/img/markdown-img-paste-20161111160226501.png)

- 启动Mycat
![](/img/markdown-img-paste-2016111116030921.png)  
![](/img/markdown-img-paste-20161111160348465.png)  
![](/img/markdown-img-paste-20161111160418120.png)  
![](/img/markdown-img-paste-20161111160442868.png)  

- 使用Mycat
![](/img/markdown-img-paste-20161111160519290.png)  







----
Good luck!  
冬日暖阳
