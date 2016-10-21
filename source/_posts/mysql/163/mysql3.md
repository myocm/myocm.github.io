---
title: 连接MySQL
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL
date: 2016-10-13 13:43:43
updated: 2016-10-13 13:43:43
tags:
---
# 连接MySQL

参考文档： http://dev.mysql.com/doc/refman/8.0/en/connecting.html

## mysql客户端程序的连接方式
安装并启动MySQL服务端后，就可以使用MySQL自带的命令行客户端程序mysql，连接到MySQL服务器。
如下命令：
```bash
shell> mysql -hlocalhost -uroot -ppassw0rd -S /tmp/mysql-3306.sock
或
shell> mysql --host=localhsot --user=root --password=passw0rd --socket=/tmp/mysql-3306.sock
```
mysql程序支持很多参数，常用的有下面几个
- -h,--host=*hostname* 指定要连接的服务器的地址，可以是localhsot，127.0.0.1，主机名或ip地址
- -u,--user=*username*  数据库用户名
- -p,--password=*password* 数据库用户密码
- -P,--port=# 数据库端口
- -S,--socket=*socketname* Unix系统，套接字文件
- -W,--pipe=*namepipe*   Windows,命令管道文件
- --protocol=*name* (tcp, socket, pipe,memory) 连接协议，连接协议支持的操作系统和类型如下

protocol Value|Types of Connections|Supported Operating Systems
--|--|--
TCP/IP           | local,remote  | All          
Unix socket file | local only    | Unix only
Name pipe        | local,remote  | Windows only
Shared memory    | local only    | Windows only

## 程序连接MySQL
MySQL通过不同的驱动程序支持大部分的编程语言，官网给出的驱动如下：
- [Connector/ODBC](http://dev.mysql.com/downloads/connector/odbc/)
Standardized database driver for Windows, Linux, Mac OS X, and Unix platforms.

- [Connector/Net](http://dev.mysql.com/downloads/connector/net/)
Standardized database driver for .NET platforms and development.

- [Connector/J](http://dev.mysql.com/downloads/connector/j/)
Standardized database driver for Java platforms and development.

- New! [Connector/Node.js](http://dev.mysql.com/downloads/connector/nodejs/)
Standardized database driver for Node.js platforms and development.

- [Connector/Python](http://dev.mysql.com/downloads/connector/python/)
Standardized database driver for Python platforms and development.

- [Connector/C++](http://dev.mysql.com/downloads/connector/cpp/)
Standardized database driver for C++ development.

- [Connector/C (libmysqlclient)](http://dev.mysql.com/downloads/connector/c/)
A client library for C development.

- [MySQL native driver for PHP - mysqlnd](http://dev.mysql.com/downloads/connector/php-mysqlnd/)
The MySQL native driver for PHP is an additional, alternative way to connect from PHP 5.3 or newer to the MySQL Server 4.1 or newer.

### Python3.5 连接MySQL
因为MySQL官方驱动暂时还没有支持Python3.5，只支持到Python3.4，所以需要使用第三方驱动，比如[PyMySQL](https://pypi.python.org/pypi/PyMySQL#downloads)
测试代码如下：
```python
#!/usr/bin/env python

import pymysql

conn = pymysql.connect(host='localhsot', port=3306, user='root', passwd='ppassw0rd', db='mysql')

cur = conn.cursor()
cur.execute("SELECT user,host,authentication_string FROM user")

print(cur.description)
print()

for row in cur:
    print(row)

cur.close()
conn.close()
```
输出结果如下表示连接成功：
```
(('user', 254, None, 32, 32, 0, False), ('host', 254, None, 60, 60, 0, False), ('authentication_string', 252, None, 65535, 65535, 0, True))

('mysql.sys', 'localhost', '*THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE')
('root', 'localhost', '*81F5E21E35407D884A6CD4A731AEBFB6AF209E1B')
```

## 新的连接工具[MySQL Shell](http://dev.mysql.com/downloads/shell/)
现在官网提供了一个新的连接工具MySQL Shell，用于 Javascript, Python等脚本语言来连接MySQL。  
具体用法参考[官网文档](http://dev.mysql.com/doc/refman/8.0/en/mysql-shell.html)。

## 图形化工具

- [MySQL Workbench](http://dev.mysql.com/downloads/workbench/) MySQL官方的免费GUI工具
- [SQL Developer](http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html) Oracle官方的免费GUI工具，支持Oracle database和MySQL等所有支持JDBC连接的数据库。
- [SQLyog ](https://www.webyog.com/) 功能强大的第三方收费工具。
- [navicat-for-mysql](https://www.navicat.com/products/navicat-for-mysql) 功能强大的第三方收费工具。
- [Toad for MySQL](https://software.dell.com/products/toad-for-mysql/) Dell出品的免费GUI工具。
- [phpMyAdmin](https://www.phpmyadmin.net/) 大名鼎鼎的基于php的网页版免费开源工具。



----
Good luck!  
冬日暖阳
