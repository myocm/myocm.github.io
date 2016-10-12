---
title: MySQL 安装
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL 安装
date: 2016-10-11 18:36:22
updated: 2016-10-12 18:36:22
tags:
---
# 轻松安装MySQL
官方参考文档
[http://dev.mysql.com/doc/refman/8.0/en/installing.html](http://dev.mysql.com/doc/refman/8.0/en/installing.html
)

## windows 下的安装
1. 图形化工具安装  
   1. 下载[MySQL Installer](http://dev.mysql.com/downloads/installer/)
   2. 选择合适的安装类型，根据安装向导完成安装即可。

2. 绿色安装
   1. 下载MySQL的[zip压缩包](http://dev.mysql.com/downloads/mysql/)
   2. 解压到本地磁盘
   3. 创建参数文件my.ini，可以复制my-default.ini，并改名。
   4. 编辑my.ini 文件。指定`basedir`,`datadir` 等参数。
   5. 初始化数据字典。MySQL5.7.7 之前默认自带数据字典，MySQL5.7.7之后需要自己初始化数据字典。
   6. 启动MySQL服务器。 在cmd窗口运行` D:\Program\mysql-5.7.15-winx64\bin>mysqld.exe`即可。首次用初始化生成的随机密码进入后，要求立即修改root的密码，其他语句无法执行。输入` alter user 'root'@'localhost' identified by 'passw0rd';`
   7. 关闭MySQL服务器。 在cmd窗口运行` D:\Program\mysql-5.7.15-winx64\bin>mysqladmin.exe shutdown -uroot -p` 输入密码`passw0rd`即可。
   8. 添加MySQL命令到PATH路径。就不需要使用命令的全路径了。
   9. 创建MySQL的Windows服务。


- my.ini 配置参考
```
[client]
port = 3306                          # 设置服务器的端口为3306
default-character-set = utf8         # 设置客户端默认字符集为utf8

[mysqld]

# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
innodb_buffer_pool_size = 128M

# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin

# These are commonly set, remove the # and set as required.
basedir = D:\Program\mysql-5.7.15-winx64
datadir = D:\Program\mysql-5.7.15-winx64\data
port = 3306
server_id = 1

character_set_server = utf8              # 设置服务器字符集为utf8
collation_server = utf8_general_ci       # 设置服务器字符集校对规则为utf8_general_ci
init_connect = 'SET NAMES utf8'          # 设置客户端连接时的字符集为utf8
init_connect = 'set autocommit = 0'      # 设置服务器关闭自动提交
transaction_isolation = READ-COMMITTED   # 设置服务器的事务隔离级别为 提交读
lower_case_table_names = 1               # 设置服务器表名为小写字母
open_files_limit = 10240                 # 设置服务器可以打开的文件数
max_connections = 2000                   # 设置服务器可以建立的最大连接数，默认为150


####innodb set
innodb_buffer_pool_size = 256M           # 设置innodb的内存大小为256M
innodb_log_buffer_size = 20M             # 设置innodb的日志缓冲区
innodb_file_per_table=1                  # 设置innodb单独表空间，不使用共享表空间
innodb_open_files = 1000                 # 设置innodb可以打开的文件数


# slow_query_log                         # 设置服务器是否开启慢查询
# long_query_time = 5                    # 设置服务器慢查询的阈值，单位为秒，即大于这个时间的查询会被记录到慢查询日志。
# key_buffer_size = 50M                  # 设置服务器索引缓存空间

# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
sort_buffer_size = 2M
read_rnd_buffer_size = 2M

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```
- 初始化数据字典参考
```
# 启动一个cmd命令窗口，切换到参数文件my.ini指定的datadir目录
# 输入下列命令
D:\...\data>D:\Program\mysql-5.7.15-winx64\bin\mysqld.exe --initialize
或
D:\...\data>D:\Program\mysql-5.7.15-winx64\bin\mysqld.exe --initialize-insecure

mysqld.exe --initialize 为安全初始化，会生成一个带随机密码的root账号。密码在数据目录的错误日志文件（主机名.err）中。
mysqld.exe --initialize-insecure 为非安全初始化，生成的root账号密码为空。
```
- 创建windows服务参考
```
# --install 后面可以指定服务名，默认为MySQL
C:\>D:\Program\mysql-5.7.15-winx64\bin\mysqld.exe --install mysql
Service successfully installed.

# 使用windows服务启动 MySQL
C:\> net start mysql

# 使用windows服务停止 MySQL
C:\> net stop mysql

# 移除windows服务
C:\>D:\Program\mysql-5.7.15-winx64\bin\mysqld.exe --remove mysql
Service successfully removed.
```
## Linux 下的安装
1. 从发行版仓库安装  
   各Linux发行版都提供从仓库安装MySQL。使用对应发行版从仓库安装软件的方式安装即可。  
   如果Linux发行版仓库里的版本比较老，可以去MySQL官方下载[官方仓库源](http://dev.mysql.com/downloads/repo/)。然后从MySQL官方仓库源安装即可。

2. 源码安装  
   从5.7版本起与5.6及之前版本的源码编译安装有所不同，从源码安装时，要参考对应版本的安装说明。  
   下面的步骤适用于5.7及以后版本，以MySQL8.0.0开发版为例。  
   1. 下载源码 http://dev.mysql.com/downloads/mysql/
   2. 解压并创建编译目录
```
tar zxf mysql-boost-8.0.0-dmr.tar.gz
cd mysql-boost-8.0.0-dmr
mkdir bld
```
   3. 编译和安装程序
```
cd bld
cmake .. -DWITH_BOOST=../boost \                       # 从5.7 版本起需要boost库，如果下载含有boost库的源码就可以不用下载，否则需要指定下载参数。
-DCMAKE_INSTALL_PREFIX=/opt/freeware/mysql-8.0.0 \     # 指定安装前缀，默认为 /usr/local/mysql
-DCPACK_MONOLITHIC_INSTALL=true \                      # 生成一个安装包，还是多个安装包
-DDEFAULT_CHARSET=utf8mb4 \                            # 指定默认字符集
-DDEFAULT_COLLATION=utf8mb4_general_ci \               # 指定默认字符集校验规则
-DTMPDIR=/tmp \                                        # 指定默认临时文件目录
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \                      # 开启ARCHIVE存储引擎
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \                    # 开启BLACKHOLE存储引擎
-DWITH_PERFSCHEMA_STORAGE_ENGINE=1 \                   # 开启PERFSCHEMA存储引擎
-DENABLED_LOCAL_INFILE=1 \                             # 开启支持本地操作文件权限
-DWITH_EMBEDDED_SERVER=0 \                             # 关闭嵌入式服务器
-DWITH_TEST_TRACE_PLUGIN=0 \                           # 关闭测试跟踪插件
-DWITH_KEYRING_TEST=0 \                                # 关闭KEYRING测试
-DINSTALL_MYSQLTESTDIR= \                              # 不安装测试文件
-DWITH_SYSTEMD=0                                       # 是否支持systemd 启动,在 redhat7等，systemd 系统上启用，其他系统可以不用这个参数。

# 编译  -j4 以4个job（即线程）编译
make -j4

# 安装
make install
```
   4. 初始化数据库  
      1. 编辑配置文件
```
      useradd -m mysql
      passwd mysql
      su - mysql
      mkdir mysql8
      cd mysql8
      mkdir bin data pid log mylogs
      cd bin
      cp /opt/freeware/mysql8.0.0/support-files/my-default.cnf my.cnf


      vim my.cnf
      # 参考上面my.ini配置
```
      2. 初始化数据字典
```
shell> mysqld --defaults-file=~/mysql8/bin/my.cnf --initialize --user=mysql
# root密码记录在错误日志中
```
   5. 启动数据库服务器并修改root密码
```
      启动：
      shell> mysqld --defaults-file=~/mysql8/bin/my.cnf

      另起一个终端：
      shell> mysql -uroot -p  --socket=/tmp/mysql-3306.sock

      登录后
      mysql> alter user 'root'@'localhost' identified by 'root';
      mysql> flush privileges;
      退出后，可使用新密码登录。
```
   6. 创建启动脚本
```
shell> vim start.sh
#!/bin/bash

nohup mysqld_safe --defaults-file=./my.cnf &

shell> chmod +x start.sh
```
  7. 创建停库脚本
```
#!/bin/bash
mysqladmin shutdown -uroot -p -S /tmp/mysql-3306.sock
```
  8. 如果在编译时参数`-DWITH_SYSTEMD=1`，开启支持`systemd`方式启动的话，则不会生成`mysqld_safe`命令，上面的启动脚本就不能工作，需要执行下列步骤启用`systemd`方式启动。
     1. 复制脚本到指定位置。在MySQL安装目录下，有`usr`目录，如下：
     ```
     shell> cd opt/freeware/mysql-8.0.0
     shell> ls -l
     drwxr-xr-x  2 root root  4096 2016-10-12 16:10:05 bin
     -rw-r--r--  1 root root 17987 2016-08-25 20:32:09 COPYING
     drwxr-xr-x  2 root root    55 2016-10-12 16:09:39 docs
     drwxr-xr-x  3 root root  4096 2016-10-12 16:09:41 include
     drwxr-xr-x  4 root root   172 2016-10-12 16:10:04 lib
     drwxr-xr-x  4 root root    30 2016-10-12 16:09:58 man
     -rw-r--r--  1 root root  2478 2016-08-25 20:32:09 README
     drwxr-xr-x 28 root root  4096 2016-10-12 16:10:06 share
     drwxr-xr-x  2 root root   112 2016-10-12 16:10:06 support-files
     drwxr-xr-x  3 root root    17 2016-10-12 16:10:05 usr

     # 复制usr目录里的文件到系统的/usr对应目录里
     shell> cp -rp usr /usr
     ```
     2. 编辑MySQL的参数文件
     ```
     shell> vim /etc/my.cnf
     [mysqld]

      # Remove leading # and set to the amount of RAM for the most important data
      # cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
      innodb_buffer_pool_size = 128M

      # Remove leading # to turn on a very important data integrity option: logging
      # changes to the binary log between backups.
      # log_bin

      # These are commonly set, remove the # and set as required.
      basedir = /opt/freeware/mysql-8.0.0
      datadir = /home/mysql/mysql8/data
      port = 3306
      server_id = 1
      socket = /tmp/mysql-3306.sock


      init_connect = 'SET NAMES utf8mb4'
      init_connect = 'set autocommit = 0'  
      transaction_isolation = READ-COMMITTED
      lower_case_table_names = 1
      skip-external-locking

      # Remove leading # to set options mainly useful for reporting servers.
      # The server defaults are faster for transactions and fast SELECTs.
      # Adjust sizes as needed, experiment to find the optimal values.
      # join_buffer_size = 128M
      # sort_buffer_size = 2M
      # read_rnd_buffer_size = 2M

      sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
     ```
     3. 启动MySQL服务
     ```
     # 查看mysqld服务状态
     shell> systemctl status mysqld
     # 启动
     shell> systemctl start mysqld
     # 停止
     shell> systemctl stop mysqld
     # 重启
     shell> systemctl restart mysqld
     ```
     4. 设置开机启动MySQL服务
     ```
     # 设置开机启动
     shell> systemctl enable mysqld
     # 关闭开机启动
     shell> systemctl disable mysqld
     ```






----
Good luck!  
冬日暖阳
