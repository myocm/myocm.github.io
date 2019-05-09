---
title: InnoDB事务锁
list_number: false
categories:
  - MySQL
comments: true
description:
  - InnoDB事务锁
date: 2016-10-26 13:53:41
updated: 2016-10-26 13:53:41
tags:
---
# InnoDB事务锁

## 什么是数据库锁
- 计算机程序锁
  - 控制对共享资源进行并发访问
  - 保护数据的完整性和一致性
- 数据库中的锁
  - 分为两大类
  ![](/img/markdown-img-paste-20161026135913863.png)
  - 主要关心的是事务lock
- 数据库事务并发
  - 如果事务允许并发修改同一条数据...
  ![](/img/markdown-img-paste-20161026140540523.png)
  - 对同一行记录的修改必须串行化
- 不同数据库系统事务锁粒度
  - 行锁
    - InnoDB，Oracle
  - 页锁
    - SQL Server
  - 表锁
    - MyISAM，Memory
  - 锁升级

## InnoDB 锁机制
- 四种基本锁模式
  - 共享锁(S)-读锁-行锁
  - 排它锁(X)-写锁-行锁
  - 意向共享锁(IS)-表锁
  - 意向排它锁(IX)-表锁  

- 意向锁
  - 意向锁总是自动先加，并且意向锁自动加自动释放。
  - 意向锁提示数据库这个session将要在接下来将要施加何种锁
  - 意向锁和X/S锁级别不同，除了阻塞全表级别的X/S锁外其他任何锁

- InnoDB锁模式互斥
![](/img/markdown-img-paste-20161026161805186.png)
![](/img/markdown-img-paste-20161026162258493.png)

- 数据库加锁操作
  - 一般的select语句不加任何锁，也不会被任何事务锁阻塞
    - 读的隔离性由MVCC确保
  - S 锁
    - 手动：select * from tb_test lock in share mode;
    - 自动：insert前
  - X 锁
    - 手动：select * from tb_test lock for update;
    - 自动：update，delete前

## InnoDB 锁实现与注意事项
- InnoDB行锁的实现
  - 通过索引项加锁实现
    - 只有条件走索引才能实现行级锁
    - 索引上有重复值，可能锁住多个记录
    - 查询有多个索引可以走，可以对不同索引加锁
    - 是否对索引加锁实际上取决于MySQL执行计划
  - 自增主键做条件更新，性能最好
- InnoDB的gap lock
  - 回顾什么是幻读，幻读是对`insert`而言的，`update`和`delete`是提交读
  - gap lock 消灭幻读
    - InnoDB消灭幻读仅仅为了确保statement模式replicate的主从一致性
  - 小心gap lock，因为会阻塞`insert`
  - 所以要推荐应用开发人员使用自增主键做条件更新，这样性能最好

## 死锁
- 什么是死锁
  ![](/img/markdown-img-paste-20161026163610705.png)
- 死锁数据库自动解决
  - 数据库挑选冲突事务中回滚代价较小的事务回滚
- 死锁预防
  - 单表死锁可以根据批量更新里的更新条件排序
  - 可能冲突的跨表事务尽量避免并发
  - 尽量缩短事务长度

## 业务加锁
- 业务流程中的悲观锁
  - 假如amount不能为负值...
  ![](/img/markdown-img-paste-20161026163950398.png)
- 如何缩短锁的时间？
  ![](/img/markdown-img-paste-20161026164057949.png)


----
Good luck!  
冬日暖阳
