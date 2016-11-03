---
title: InnoDB存储引擎
list_number: false
categories:
  - MySQL
comments: true
description:
  - InnoDB存储引擎
date: 2016-10-26 11:08:08
updated: 2016-10-26 11:08:08
tags:
---
# InnoDB存储引擎

## InnoDB存储引擎架构
![InnoDB存储引擎架构](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161026112910390.png)

## InnoDB物理文件
- 磁盘文件
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161026113305308.png)

- InnoDB系统表空间文件
  - ibdata1里存放什么
    - 回滚段
    - 所有InnoDB表元数据信息
    - Double Write，Insert buffer dump等等...
  - 自动扩展机制

- InnoDB与磁盘文件有关的参数
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-2016102611403298.png)

- InnoDB数据文件存储结构
  - 索引组织表（聚簇表）
  - 根据表逻辑主键排序
  - 数据节点每页16K
  - 根据主键寻址迅速
  - 主键值递增的insert插入效率较好
  - 主键值随机的insert插入效率较差
  - 因此，InnoDB表必须指定主键，建议使用自增数字
  ![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161026114558368.png)

## InnoDB内存缓存
- InnoDB数据块缓存池
  - 数据的读写需要经过缓存
  - 数据以整页（16K）为单位读取到缓存中
  - 缓存中的数据以LRU策略换出
  - IO效率高，性能好
  ![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161026115213733.png)

   ![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-2016102611570899.png)
- InnoDB Buffer Pool 相关参数
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161026120450261.png)

## InnoDB数据持久化
- 持久化与事务日志
  - 事务日志实时持久化
  - 内存变化数据（脏数据）增量异步刷出到磁盘
  - 实例故障靠重放日志恢复
  - 性能好，可靠，恢复快
  ![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161026120930905.png)
- InnoDB日志持久化相关参数
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161026121103109.png)

## InnoDB行级锁
- 写不阻塞读
- 不同行间的写互相不阻塞
- 并发性能好

## InnoDB事务支持
- 事务ACID特性完整支持
  - （A）回滚段失败回滚
  - （C）支持主外键约束
  - （I）事务版本+回滚段=MVCC
  - （D）事务日志持久化
  
- 默认事务隔离级别为可重复读，可以修改


----
Good luck!  
冬日暖阳
