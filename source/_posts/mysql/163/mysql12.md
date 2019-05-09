---
title: MySQL字符集
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL字符集和字符集校对规则
date: 2016-10-24 10:13:53
updated: 2016-10-24 10:13:53
tags:
---

# MySQL字符集


## 字符集基础

### 字符集定义
字符集 ： 数据库中字符集包含两层含义

1. 各种文字和符号的集合，包括各国家文字、标点符号、图形符号、数字等。
2. 字符的编码方式，即二进制数据与字符的映射规则。

### 字符集分类
- ASCII： 美国信息互换标准编码；英语及其他西欧语言；单字节编码，7位（bits）表示一个字符，共128个字符。
- GBK： 汉字内码扩展规范；中日韩汉字、英文、数字；双字节编码；共收录了21003个汉字，GB2312的扩展。
- UTF-8： Unicode 标准的可变长度字符编码； Unicode标准（统一码），业界统一标准，包含世界上数十种文字的系统；UTF-8使用一至四个字节为每个字符编码。
- 其他常见字符集： UTF-32，UTF-16，Big5 ， latin1

### MySQL中查看字符集
```sql
show character set;
+----------+---------------------------------+---------------------+--------+
| Charset  | Description                     | Default collation   | Maxlen |
+----------+---------------------------------+---------------------+--------+
| armscii8 | ARMSCII-8 Armenian              | armscii8_general_ci |      1 |
| ascii    | US ASCII                        | ascii_general_ci    |      1 |
| big5     | Big5 Traditional Chinese        | big5_chinese_ci     |      2 |
| binary   | Binary pseudo charset           | binary              |      1 |
| cp1250   | Windows Central European        | cp1250_general_ci   |      1 |
| cp1251   | Windows Cyrillic                | cp1251_general_ci   |      1 |
| cp1256   | Windows Arabic                  | cp1256_general_ci   |      1 |
| cp1257   | Windows Baltic                  | cp1257_general_ci   |      1 |
| cp850    | DOS West European               | cp850_general_ci    |      1 |
| cp852    | DOS Central European            | cp852_general_ci    |      1 |
| cp866    | DOS Russian                     | cp866_general_ci    |      1 |
| cp932    | SJIS for Windows Japanese       | cp932_japanese_ci   |      2 |
| dec8     | DEC West European               | dec8_swedish_ci     |      1 |
| eucjpms  | UJIS for Windows Japanese       | eucjpms_japanese_ci |      3 |
| euckr    | EUC-KR Korean                   | euckr_korean_ci     |      2 |
| gb18030  | China National Standard GB18030 | gb18030_chinese_ci  |      4 |
| gb2312   | GB2312 Simplified Chinese       | gb2312_chinese_ci   |      2 |
| gbk      | GBK Simplified Chinese          | gbk_chinese_ci      |      2 |
| geostd8  | GEOSTD8 Georgian                | geostd8_general_ci  |      1 |
| greek    | ISO 8859-7 Greek                | greek_general_ci    |      1 |
| hebrew   | ISO 8859-8 Hebrew               | hebrew_general_ci   |      1 |
| hp8      | HP West European                | hp8_english_ci      |      1 |
| keybcs2  | DOS Kamenicky Czech-Slovak      | keybcs2_general_ci  |      1 |
| koi8r    | KOI8-R Relcom Russian           | koi8r_general_ci    |      1 |
| koi8u    | KOI8-U Ukrainian                | koi8u_general_ci    |      1 |
| latin1   | cp1252 West European            | latin1_swedish_ci   |      1 |
| latin2   | ISO 8859-2 Central European     | latin2_general_ci   |      1 |
| latin5   | ISO 8859-9 Turkish              | latin5_turkish_ci   |      1 |
| latin7   | ISO 8859-13 Baltic              | latin7_general_ci   |      1 |
| macce    | Mac Central European            | macce_general_ci    |      1 |
| macroman | Mac West European               | macroman_general_ci |      1 |
| sjis     | Shift-JIS Japanese              | sjis_japanese_ci    |      2 |
| swe7     | 7bit Swedish                    | swe7_swedish_ci     |      1 |
| tis620   | TIS620 Thai                     | tis620_thai_ci      |      1 |
| ucs2     | UCS-2 Unicode                   | ucs2_general_ci     |      2 |
| ujis     | EUC-JP Japanese                 | ujis_japanese_ci    |      3 |
| utf16    | UTF-16 Unicode                  | utf16_general_ci    |      4 |
| utf16le  | UTF-16LE Unicode                | utf16le_general_ci  |      4 |
| utf32    | UTF-32 Unicode                  | utf32_general_ci    |      4 |
| utf8     | UTF-8 Unicode                   | utf8_general_ci     |      3 |
| utf8mb4  | UTF-8 Unicode                   | utf8mb4_general_ci  |      4 |
+----------+---------------------------------+---------------------+--------+
41 rows in set (0.01 sec)
```
### 新增字符集
编译时加入： --with-charset=
```sh
shell> ./configure --prefix=/opt/freeware/mysql --with-plugins=innobase --with-charset=gbk
```

## MySQL字符集与字符序
charset 和 collation  
collation： 字符序，字符的排序和比较规则，每个字符集都有对应的多套字符序。  
不同的字符序决定了字符串在比较排序中的精度和性能不同。  

### 查看MySQL字符序
```sql
show collation;
+----------------------------+----------+-----+---------+----------+---------+
| Collation                  | Charset  | Id  | Default | Compiled | Sortlen |
+----------------------------+----------+-----+---------+----------+---------+
| armscii8_bin               | armscii8 |  64 |         | Yes      |       1 |
| armscii8_general_ci        | armscii8 |  32 | Yes     | Yes      |       1 |
| ascii_bin                  | ascii    |  65 |         | Yes      |       1 |
| ascii_general_ci           | ascii    |  11 | Yes     | Yes      |       1 |
......
......
| utf8_turkish_ci            | utf8     | 201 |         | Yes      |       8 |
| utf8_unicode_520_ci        | utf8     | 214 |         | Yes      |       8 |
| utf8_unicode_ci            | utf8     | 192 |         | Yes      |       8 |
| utf8_vietnamese_ci         | utf8     | 215 |         | Yes      |       8 |
+----------------------------+----------+-----+---------+----------+---------+
244 rows in set (0.01 sec)

--其实这些信息也可以查询数据字典
select * from information_schema.COLLATIONS  order by id;
+----------------------------+--------------------+-----+------------+-------------+---------+
| COLLATION_NAME             | CHARACTER_SET_NAME | ID  | IS_DEFAULT | IS_COMPILED | SORTLEN |
+----------------------------+--------------------+-----+------------+-------------+---------+
| big5_chinese_ci            | big5               |   1 | Yes        | Yes         |       1 |
| latin2_czech_cs            | latin2             |   2 |            | Yes         |       4 |
| dec8_swedish_ci            | dec8               |   3 | Yes        | Yes         |       1 |
| cp850_general_ci           | cp850              |   4 | Yes        | Yes         |       1 |
......
......
| utf8mb4_eo_0900_ai_ci      | utf8mb4            | 273 |            | Yes         |       8 |
| utf8mb4_hu_0900_ai_ci      | utf8mb4            | 274 |            | Yes         |       8 |
| utf8mb4_hr_0900_ai_ci      | utf8mb4            | 275 |            | Yes         |       8 |
| utf8mb4_vi_0900_ai_ci      | utf8mb4            | 277 |            | Yes         |       8 |
+----------------------------+--------------------+-----+------------+-------------+---------+
244 rows in set (0.02 sec)
```

### MySQL的字符序遵从命名惯例：
- 以_ci（表示大小写不敏感 case-insenstive ）
- 以_cs（表示大小写敏感 case-senstive ）
- 以_bin（表示用编码值进行比较 binary ）

## 字符集设置级别
charset 和 collation 的设置级别：  
服务器级 >> 数据库级 >> 表级 >> 列级

### 服务器级  
- 系统变量（可动态设置） ：  
`-character_set_server`：默认的内部操作字符集  
`-character_set_system`：系统元数据（字段名等)字符集


- 配置文件设置：
```sql
[mysqld]
character_set_server=utf8
collation_server=utf8_general_ci
```

### 数据库级
```sql
create database db_name character set latin1 collate latin1_swedish_ci;
```

`-character_set_database` : 当前选中数据库的默认字符集
主要影响`load data` 等语句的默认字符集， `CREATE DATABASE `的字符集如果不设置，
默认使用`character_set_server`的字符集。

> load data 时，使用的是 character_set_database 变量来告诉数据库，文本文件的编码格式。

### 表级
```sql
create table tbl1;
create table tbl1 (...) default charset=utf8 default collate=utf8_bin;
```

### 列级
```sql
create table tbl1;
create table tbl1(col1 varchar(5) character set latin1 collate latin1_german1_ci);
```

### 数据存储字符集使用规则
- 使用列级别的character set 设定值；
- 若列级字符集不存在，则使用对应表级的default character set 设定值；
- 若表级字符集不存在，则使用数据库级的default character set 设定值；
- 若数据库级字符集不存在，则使用服务器级别character_set_server 设定值；

### 数据库全局字符集变量设定值
```sql
show global variables like '%char%';
+--------------------------+-------------------------------------------+
| Variable_name            | Value                                     |
+--------------------------+-------------------------------------------+
| character_set_client     | utf8mb4                                   |
| character_set_connection | utf8mb4                                   |
| character_set_database   | utf8mb4                                   |
| character_set_filesystem | binary                                    |
| character_set_results    | utf8mb4                                   |
| character_set_server     | utf8mb4                                   |
| character_set_system     | utf8                                      |
| character_sets_dir       | /opt/freeware/mysql-8.0.0/share/charsets/ |
+--------------------------+-------------------------------------------+
8 rows in set (0.06 sec)
```

修改全局字符集变量设定值
```sql
set global character_set_server=utf8;
Query OK, 0 rows affected (0.00 sec)

show global variables like '%char%';
+--------------------------+-------------------------------------------+
| Variable_name            | Value                                     |
+--------------------------+-------------------------------------------+
| character_set_client     | utf8mb4                                   |
| character_set_connection | utf8mb4                                   |
| character_set_database   | utf8mb4                                   |
| character_set_filesystem | binary                                    |
| character_set_results    | utf8mb4                                   |
| character_set_server     | utf8                                      |
| character_set_system     | utf8                                      |
| character_sets_dir       | /opt/freeware/mysql-8.0.0/share/charsets/ |
+--------------------------+-------------------------------------------+
8 rows in set (0.01 sec)

show variables like '%char%';
+--------------------------+-------------------------------------------+
| Variable_name            | Value                                     |
+--------------------------+-------------------------------------------+
| character_set_client     | utf8mb4                                   |
| character_set_connection | utf8mb4                                   |
| character_set_database   | utf8mb4                                   |
| character_set_filesystem | binary                                    |
| character_set_results    | utf8mb4                                   |
| character_set_server     | utf8mb4                                   |
| character_set_system     | utf8                                      |
| character_sets_dir       | /opt/freeware/mysql-8.0.0/share/charsets/ |
+--------------------------+-------------------------------------------+
8 rows in set (0.01 sec)

```


### 会话级字符集变量设定值
```sql
show variables like '%char%';
+--------------------------+-------------------------------------------+
| Variable_name            | Value                                     |
+--------------------------+-------------------------------------------+
| character_set_client     | utf8mb4                                   |
| character_set_connection | utf8mb4                                   |
| character_set_database   | utf8mb4                                   |
| character_set_filesystem | binary                                    |
| character_set_results    | utf8mb4                                   |
| character_set_server     | utf8mb4                                   |
| character_set_system     | utf8                                      |
| character_sets_dir       | /opt/freeware/mysql-8.0.0/share/charsets/ |
+--------------------------+-------------------------------------------+
8 rows in set (0.01 sec)
```

修改会话字符集变量设定值
```sql
set character_set_server=latin1;
Query OK, 0 rows affected (0.00 sec)

show global variables like '%char%';
+--------------------------+-------------------------------------------+
| Variable_name            | Value                                     |
+--------------------------+-------------------------------------------+
| character_set_client     | utf8mb4                                   |
| character_set_connection | utf8mb4                                   |
| character_set_database   | utf8mb4                                   |
| character_set_filesystem | binary                                    |
| character_set_results    | utf8mb4                                   |
| character_set_server     | utf8                                      |
| character_set_system     | utf8                                      |
| character_sets_dir       | /opt/freeware/mysql-8.0.0/share/charsets/ |
+--------------------------+-------------------------------------------+
8 rows in set (0.00 sec)

show variables like '%char%';
+--------------------------+-------------------------------------------+
| Variable_name            | Value                                     |
+--------------------------+-------------------------------------------+
| character_set_client     | utf8mb4                                   |
| character_set_connection | utf8mb4                                   |
| character_set_database   | utf8mb4                                   |
| character_set_filesystem | binary                                    |
| character_set_results    | utf8mb4                                   |
| character_set_server     | latin1                                    |
| character_set_system     | utf8                                      |
| character_sets_dir       | /opt/freeware/mysql-8.0.0/share/charsets/ |
+--------------------------+-------------------------------------------+
8 rows in set (0.00 sec)
```

### 修改表的字符集
```sql
show create table t \G
*************************** 1. row ***************************
       Table: t
Create Table: CREATE TABLE `t` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(100) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.01 sec)

alter table t convert to  character set utf8;
Query OK, 1 row affected (0.33 sec)
Records: 1  Duplicates: 0  Warnings: 0

show create table t \G
*************************** 1. row ***************************
       Table: t
Create Table: CREATE TABLE `t` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(100) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

alter table t convert to  character set utf8 collate utf8_bin;
Query OK, 1 row affected (0.06 sec)
Records: 1  Duplicates: 0  Warnings: 0

show create table t \G
*************************** 1. row ***************************
       Table: t
Create Table: CREATE TABLE `t` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(100) COLLATE utf8_bin DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
1 row in set (0.00 sec)
```

## 客户端字符集
### 连接与字符集
- `-character_set_client`: 客户端来源数据使用的字符集。
- `-character_set_connection`: 连接层字符集
- `-character_set_results`: 查询结果字符集

### 设置客户端字符集

- 动态修改

```sql
set names utf8;
Query OK, 0 rows affected (0.00 sec)

status
----------
mysql  Ver 14.14 Distrib 8.0.0-dmr, for Linux (x86_64) using  EditLine wrapper

Connection id:		4
Current database:	demo
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		8.0.0-dmr Source distribution
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	latin1
Db     characterset:	utf8mb4
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/tmp/mysql-3306.sock
Uptime:			31 min 43 sec

Threads: 1  Questions: 36  Slow queries: 0  Opens: 184  Flush tables: 2  Open tables: 156  Queries per second avg: 0.018
----------
```

- 配置文件设置

```sql
[mysql]
default-character-set=utf8
```

![](/img/markdown-img-paste-2016102411195854.png)

### 常见乱码原因
1. 数据存储字符集不能正确编码（不支持）client发来的数据：
    client(utf8) → storage(latin1)
2. 程序连接使用的字符集与通知MySQL的character_set_client等不一致或不兼容。

> 使用建议：
1. 创建数据库/表时显示的指定字符集，不使用默认。
2. 连接字符集与数据存储字符集设置一致，推荐使用utf8.
3. 驱动程序连接时显式指定字符集(set names xxx)

>mysql CAPI：初始化数据库句柄后马上用mysql_options设定MYSQL_SET_CHARSET_NAME属性为utf8.  
mysql PHP API：连接到数据库以后显式用SET NAMES语句设置一次连接字符集。  
mysql JDBC：url="jdbc:mysql://localhost:3306/demo?user=xx&password=xx&useUnicode=true&characterEncoding=utf8";


----
Good luck!  
冬日暖阳
