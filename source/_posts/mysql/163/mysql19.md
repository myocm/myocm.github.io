---
title: 索引查询优化
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL索引查询优化
date: 2016-10-27 17:26:23
updated: 2016-10-27 17:26:23
tags:
---

# 索引查询优化

## 什么是索引（Index）
- 数据库索引查找
  - 全表扫描 VS 索引查找
  ![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161027173246214.png)  
  ![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161027174513246.png)

- 如何根据首字母找到所在行
  - 二分查找
  - B+tree
  ![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161027174805131.png)

- InnoDB表聚簇索引
  ![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161027175402230.png)

## 如何使用索引
- 创建索引
  - 单列索引
  ```sql
  create index idx_test1 on tb_student(name);
  ```
  - 联合索引
  ```sql
    create index idx_test2 on tb_student(name,age);
  ```
  ※ 索引中先根据name排序，name相同的情况下，根据age排序

- 索引维护
  - 索引维护由数据库自动完成
  - 插入/修改/删除每一个索引行都变成要给内部封装的事务
  - 索引越多，事务越长，代价越高
  - 索引越多对表的插入和索引字段修改就越慢
  ※ 控制表上索引的数量，切忌胡乱添加无用索引。

- 如何使用索引
  - 排序ORDER BY，GROUP BY，DISTINCT字段添加索引
  ```sql
  select * from tb_a order by a;
  select a,count(*) from tb_a group by a;
  idx_a(a) √
  select * from tb_a order by a,b;
  idx_a_b(a,b) √
  select * from tb_a order where c=? by a;
  idx_c_a(c,a) √
  ```
- 索引与字段选择性
  - 某个字段其值的重复程度
  ![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161027181823716.png)
  - 选择性很差的字段通常不适合创建单列索引
    - 男女比例相仿的表中性别不适合创建单列索引
    - 如果男女比例极不平衡，要查询的又是少数方（理工院校查女生）可以考虑使用索引
  - 联合索引中选择性好的字段应该排在前面
  ```sql
  select * from tab_a where gender=? and name=?;
  idx_a1(name,gender) √
  ```


- 联合索引能为前缀单列，复列查询提供帮助
  ```sql
  idx_smp(a,b,c)
  where a=? ; √
  where a=? and b=? ; √
  where a=? and c=? ; √ (部分OK)
  ```
- 合理创建联合索引，避免冗余
  ```sql
  (a),(a,b),(a,b,c) ×
  (a,b,c) √
  ```
- 长字段上的索引
  - 在非常长的字段上建立索引影响性能
  - InnoDB索引单字段(utf8) 只能取前767 bytes
  - 对长字段处理的方法
    - Email类，建立前缀索引
    ```sql
    Mail_addr varchar(2048)
    idx_mailadd(Mail_addr(30))
    ```
    - 住址类，分拆字段
    ```sql
    Home_address varchar(2048)
    idx_mailadd(Mail_addr(30)) ? -- 很可能前半段都是相同的省市区街道名称
    Province varchar(1024), City varchar(1024), District varchar(1024),Local_address varchar(1024) ... 建立联合索引或单列索引 √
    ```
- 无法使用索引的情况
  - 索引列进行数学运算或函数运算
  ```sql
  where id+1=10 ×
  where id=(10-1) √
  year(col)<2007 ×
  col<'2007-01-01' √
  ```
  - 未含复合索引的前缀字段
  ```sql
  Idx_abc(a,b,c):
  where b=? and c=?  ×
  Idx_bc(b,c) √
  ```
  - 前缀通配，`'_'` 和`'%'` 通配符
  ```sql
  LIKE '%XXX%'
  LIKE 'XXX%'
  ```
  - Where 条件使用NOT，<>,!=
  - 字段类型匹配
    - 并不绝对，但是无法预测地会造成问题，不要使用
    ```sql
    a int(11) , idx_a(a)
    where a='123' ×
    where a=123 √
    ```

- 利用索引排序
  ```sql
  idx_a_b(a,b)
  ```
  - 能够使用索引帮助排序的查询
  ```sql
  order by a
  a=3 order by b
  order by a,b
  order by a desc,b desc
  a>5 order by a
  ```
  - 不能够使用索引帮助排序的查询
  ```sql
  order by b
  a>5 order by b
  a in(1,3) order by b
  order by a asc,b desc
  ```
## 查看查询是否使用了索引
![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161027205646396.png)
- explain 是确定一个查询如何走索引最简便有效的方法
```sql
explain select * from tb_test ;
```
- 关注的项目
  - type: 查询access的方式
  - key： 本次查询最终选择使用哪个索引，NULL为未使用索引
  - key_len：选择的索引使用的前缀长度或者整个长度
  - rows： 可以理解为查询逻辑读，需要扫描过的记录行数
  - extra：额外信息，主要指fetch data的具体方式


----
Good luck!  
冬日暖阳
