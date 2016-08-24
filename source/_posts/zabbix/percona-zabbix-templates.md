---
title: 安装  percona-zabbix-templates
date: 2016-08-18 18:01:53
list_number: false  
tags:
categories:
- zabbix
description: 
  zabbix 安装  percona 模板 for mysql
---


## 环境
Centos 6.5  
zabbix 2.4.6  
mysql5.7.9


## 准备工作
### 安装php php-mysql
   #### 1. 安装 php5.6 的源
   
   ```
   rpm -Uvh http://ftp.iij.ad.jp/pub/linux/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm   
   rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
   ```
   #### 2. 安装 php5.6
   ```
   yum install --enablerepo=remi --enablerepo=remi-php56 php php-opcache php-devel php-mbstring php-mcrypt php-mysqlnd php-phpunit-PHPUnit php-pecl-xdebug php-pecl-xhprof
   ```
   #### 3. 测试php安装成功
   ```
   php -v
   ```

## 安装

### 1. 安装 percona 的 repository  
```
yum install http://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm
```

### 2. Configure Zabbix Agent
   1. 安装模板
   ```
   yum install percona-zabbix-templates
   ```
   模板安装在下列位置
   ```
   Scripts are installed to  /var/lib/zabbix/percona/scripts  
   Templates are installed to /var/lib/zabbix/percona/templates
   ```
   2. 复制 模板配置文件 `/var/lib/zabbix/percona/templates/userparameter_percona_mysql.conf` 到 zabbix agent 配置目录 `/usr/local/zabbix_agent/etc/zabbix_agentd.conf.d`
   3. 编辑`zabbix_agentd.conf`,包含模板配置
   ```
   Include=/usr/local/zabbix_agent/etc/zabbix_agentd.conf.d/
   ```
   4. 重启zabbix-agent
   ```
   service zabbix-agentd restart
   ```
### 3. 配置MySQL connectivity on Agent
   1. 创建php 连接mysql 的配置文件
   ```
    vim /var/lib/zabbix/percona/scripts/ss_get_mysql_stats.php.cnf
    
    # 添加如下内容,修改为自己mysql的用户名，密码，端口
    <?php   
    $mysql_user = 'root';
    $mysql_pass = 's3cret';
    $mysql_port = 3308;
   ```
   这一步也可以不创建配置文件，直接修改`/var/lib/zabbix/percona/scripts/ss_get_mysql_stats.php`对应的项也可以。
   2. 因为percona的脚本有bug需要修改
   如果mysql不是yum 安装，socket文件不在默认位置，可以修改
`get_mysql_stats_wrapper.sh` 中的 `HOST=127.0.0.1`   
    另外需要修改两处`CACHEFILE`,如果不是默认端口，需要添加上端口号。
    ```
    CACHEFILE="/tmp/$HOST-mysql_cacti_stats.txt:3308"
    TIMEFLM=`stat -c %Y /tmp/$HOST-mysql_cacti_stats.txt:3308`
    ```
    测试percona脚本
    ```
    /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh gg
    ```
    返回数字表示成功。
    此时`/tmp/127.0.0.1-mysql_cacti_stats.txt\:3308`文件的属主是root，修改给`zabbix` 用户。
    ```
    cd /tmp
    chown zabbix:zabbix 127.0.0.1-mysql_cacti_stats.txt\:3308
    ```
   3. 配置zabbix 用户连接mysql
   在 zabbix用户home目录创建.my.cnf文件，并输入
   ```
   [client]
   user=root
   password=s3cret
   ```
   修改`/var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh`
   ```
   #RES=`HOME=~zabbix mysql -e 'SHOW SLAVE STATUS\G' | egrep '(Slave_IO_Running|Slave_SQL_Running):' | awk -F: '{print $2}' | tr '\n' ','`
   ```
   为自己的mysql命令路径
   ```
   RES=`/opt/freeware/mysql/bin/mysql -e 'SHOW SLAVE STATUS\G' | egrep '(Slave_IO_Running|Slave_SQL_Running):' | awk -F: '{print $2}' | tr '\n' ','`
   ```
   4. 测试zabbix 连接 mysql
   ```
   sudo -u zabbix -H /var/lib/zabbix/percona/scripts/get_mysql_stats_wrapper.sh running-slave
   ```
   返回0表示主库，返回1表示从库。

   
### 4. Configure Zabbix Server

1. 把 `/var/lib/zabbix/percona/templates/zabbix_agent_template_percona_mysql_server_ht_2.0.9-sver1.1.6.xml` 下载到本地。

2. Import the XML template using Zabbix UI (Configuration -> Templates -> Import) by additionally choosing “Screens”.
此处有坑，xml文件中有两个"MySQL Graphs"，请修改一个，否则报重复。

3. Create/edit hosts by assigning them “Percona Templates” group and linking the template “Percona MySQL Server Template” (Templates tab).   




---    
Good luck！  
冬日暖阳
 

