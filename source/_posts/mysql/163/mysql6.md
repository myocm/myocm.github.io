---
title: MySQL常用数据类型
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL常用数据类型
date: 2016-10-17 14:17:28
updated: 2016-10-17 14:17:28
tags:
---
# 常用数据类型

## Number

### 整型

- tinyint
- smallint
- mediumint
- int
- bigint  
![整型范围](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161017163215545.png)
  - int(11) VS int(21)  
存储和存储范围，计算完全一样，只是在使用zerofill（默认自动加上unsigned）时，显示的宽度不一样
```sql
 create table t( a int(11) zerofill,b int(21) zerofill);

 desc t;
+-------+---------------------------+------+-----+---------+-------+
| Field | Type                      | Null | Key | Default | Extra |
+-------+---------------------------+------+-----+---------+-------+
| a     | int(11) unsigned zerofill | YES  |     | NULL    |       |
| b     | int(21) unsigned zerofill | YES  |     | NULL    |       |
+-------+---------------------------+------+-----+---------+-------+
2 rows in set (0.02 sec)

 insert into t values(1,1);
Query OK, 1 row affected (0.03 sec)

 select * from t;
+-------------+-----------------------+
| a           | b                     |
+-------------+-----------------------+
| 00000000001 | 000000000000000000001 |
+-------------+-----------------------+
1 row in set (0.00 sec)


 create table t1 (a int(11),b int (21));
Query OK, 0 rows affected (0.05 sec)

 desc t1;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| a     | int(11) | YES  |     | NULL    |       |
| b     | int(21) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
2 rows in set (0.00 sec)

 insert into t1 values(1,1);
Query OK, 1 row affected (0.04 sec)

 select * from t1;
+------+------+
| a    | b    |
+------+------+
|    1 |    1 |
+------+------+
1 row in set (0.00 sec)
```
### 浮点型
  - FLOAT (M,D)
  - DOUBLE (M,D)
  ![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161017163337402.png)
  ```sql
 create table t ( a int, b float(7,3)) engine=Innodb;
Query OK, 0 rows affected (0.05 sec)

 desc t;
+-------+------------+------+-----+---------+-------+
| Field | Type       | Null | Key | Default | Extra |
+-------+------------+------+-----+---------+-------+
| a     | int(11)    | YES  |     | NULL    |       |
| b     | float(7,3) | YES  |     | NULL    |       |
+-------+------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

 insert into t values ( 1, 1234.56789);
Query OK, 1 row affected (0.06 sec)

 select * from t;
+------+----------+
| a    | b        |
+------+----------+
|    1 | 1234.568 |
+------+----------+
1 row in set (0.00 sec)

 insert into t values ( 1, 123.456789);
Query OK, 1 row affected (0.02 sec)

 select * from t;
+------+----------+
| a    | b        |
+------+----------+
|    1 | 1234.568 |
|    1 |  123.457 |
+------+----------+
2 rows in set (0.00 sec)

 insert into t values ( 1, 12345.6789);
ERROR 1264 (22003): Out of range value for column 'b' at row 1
  ```

### 定点数
  - DECIMAL
  高精度的数据类型，常用来存储交易相关的数据  
  DECIMAL（M，N） M代表总精度，N 代表小数点后的位数（标度）  
  1<M<254    0<N<60  
  存储长度变长


  > 经验之谈：  
在存储性别、省份、类型等分类信息时选择TINYINT 或者 ENUM  
BIGINT 存储空间更大，INT和BIGINT之间通常选择BIGINT  
交易等高精度数据选择使用DECIMAL

## STRING
  - CHAR
  - VARCHAR
  - TEXT

字符与字节的区别

编码\输入字符串 | 中国 | China
---|---|---
gbk(双字节)  | varchar(2)/4字节 | varchar(5)/5字节
utf8(三字节) | varchar(2)/6字节 | varchar(5)/5字节
utf8mb4(4字节) | varchar(2)/6字节 | varchar(5)/5字节

> MySQL在5.5.3之后增加了这个utf8mb4的编码，mb4就是most bytes 4的意思，专门用来兼容四字节的unicode。  
utf8mb4是utf8的超集.  
主要目的是用来存放emoji表情等特殊字符。

emoji表情

![](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-20161018121754306.png)
> JAVA 程序支持emoji表情的要求
> - MySQL版本>5.5.3
> - JDBC驱动版本>5.1.13
> - 库和表的字符集设为utf8mb4

CHAR、VARCHAR 和TEXT的区别
- CHAR存储定长，容易造成空间浪费，最大为255字符
- VARCHAR存储变长，节省存储空间，可以存储超过255个字符
- CHAR和VARCHAR存储单位为字符
- TEXT存储单位为字节，总大小为65535字节，约为64KB
- TEXT在数据库内部大多存储格式为溢出页，效率不如CHAR
![溢出页](http://ocaw8wyva.bkt.clouddn.com/markdown-img-paste-2016101817430651.png)


- BLOB  
BLOB是“binary large object”的缩写。
- BINARY
- VARBINARY
这三个类型用来存储二进制数据，比如图片信息等。但是不推荐使用。
> 经验之谈  
> CHAR与VARCHAR定义的长度是字符，而不是字节长度。
> 存储字符串类型推荐使用VARCHAR(N),N尽量小。
> 虽然数据库可以存储二进制数据，但是性能低下，不用使用数据库存储文件，音频，图片等二进制数据。

## Date and Time
时间类型
 - DATE
 - TIME
 - DATETIME
 - TIMESTAMP  
 - YEAR(4)

类型 | 存储空间 | 存储精度  | 存储范围 | 零值|例子
---|---|---|---
DATE | 三字节|年月日 |'1000-01-01' to '9999-12-31' | '0000-00-00'  |2016-10-18
TIME | 三字节|时分秒 微秒 |'-838:59:59.000000' to '838:59:59.000000' | '0000-00-00 00:00:00' |17:53:00
DATETIME  | 四字节 | 年月日 时分秒 微秒 | '1000-01-01 00:00:00.000000' to '9999-12-31 23:59:59.999999' | '0000-00-00 00:00:00'|2016-10-18  17:53:00  
TIMESTAMP | 八字节 | 年月日 时分秒 微秒 | '1970-01-01 00:00:01.000000' to '2038-01-19 03:14:07.999999' | '0000-00-00 00:00:00' |2016-10-18  17:53:00
YEAR(4) | 一字节 | 年 |　 '1901' to '2155'　| '0000'|　2016  
注：
- MySQL5.6.4版本之后，DATETIME和TIMESTAMP支持到六位微秒  
- MySQL5.7版本起，TIME支持到六位微秒
- MySQL8.0版本起不再支持YEAR(2)
>字段类型与时区的关系  
> TIMESTAMP会根据系统时区进行转换，DATETIME不会。


- 使用BIGINT存储时间类型
在JAVA和UNIX系统中，时间是以1970年1月1日0时0分0秒到现在的秒数来标识时间，本身就是数字类型，可以使用BIGINT直接存储。





----
Good luck!  
冬日暖阳
