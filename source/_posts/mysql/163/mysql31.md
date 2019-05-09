---
title: MySQL读写分离
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL读写分离
date: 2016-11-10 15:01:53
updated: 2016-11-10 15:01:53
tags:
---

# MySQL读写分离

## MySQL读写分离概述

### 读写分离 - What
- 在高并发查询场景下，为了满足应用访问需求，通常都会部署多个从库提供查询服务以提升数据库查询的扩展性，将查询分发到从库，让从库分担主库查询负载的技术，我们通常称为读写分离
![](/img/markdown-img-paste-20161110150550728.png)

### 读写分离 - Why
![](/img/markdown-img-paste-20161110161053462.png)   
![](/img/markdown-img-paste-20161110161226636.png)      
![](/img/markdown-img-paste-20161110161319923.png)     
![](/img/markdown-img-paste-20161110161420860.png)   


### 读写分离 - How
![](/img/markdown-img-paste-20161110161450249.png)  
![](/img/markdown-img-paste-20161110161526576.png)  


### 读写分离主要问题
缺点：
1. 数据可能不一致
2. 部署、实现相对复杂  
![](/img/markdown-img-paste-20161110161728463.png)

## 读写分离常见方案
- MySQL Proxy
- Amoeba For MySQL
- MySQL Router
- 应用端实现

### MySQL Proxy
MySQL Proxy 是MySQL官方开发的一款读写分离中间件，开发多年一直未GA，在功能上存在许多缺陷，目前官方已经放弃开发
- 使用C/C++ 开发服务模块
- 使用lua脚本语言分析sql语句进行读写分离  
![](/img/markdown-img-paste-20161110175403769.png)

### MySQL Proxy （Atlas）
Atlas是360公司开发改进的一款MySQL Proxy分支，基于MySQL Proxy0.8.2，修正大量bug，并添加许多使用功能。已经大规模运行于360公司的各种业务中，经过线上大并发业务考验，也有许多其他公司在使用，开发者与使用者比较活跃
Fix：
- Lua 改为C语言，提高性能
- 并发下字符集修正
- 避免死等待
- 主库宕机仍可以读
- MySQL新版兼容等

Add：
- 从库动态上下线
- 分库分表
- 从库指定负载权重
- IP过滤等

安装
准备：
Atlas： https://github.com/Qihoo360/Atlas/releases

操作：
1. sudo dpkg -i Atlas-2.2-debian7.0-x86_64.deb
2. 编辑配置文件
3. /usr/local/mysql-proxy/bin/mysql-proxyd instancename start


### Amoeba for MySQL
Amoeba是类似MySQL Proxy的中间件，采用java语言编写，具有读写分离、数据切分和过滤等功能，是非常早的一款国产MySQL读写分离软件
- 支持读写分离
- 支持数据拆分
- 对事务支持不好
- 数据拆分比较简单
![](/img/markdown-img-paste-20161110180125249.png)


### MySQL Router
MySQL Router 是MySQL官方开发的一款一个轻量级的用来实现高可用和扩展性的中间件，支持读写分离负载均衡，支持跟MySQL Fabric一起使用，刚GA，稳定性等还有待考验

MySQL Router提供2个端口，读写1个，只读端口1个
读写端口：first available
只读端口：round robin
![](/img/markdown-img-paste-2016111018245511.png)

----
Good luck!  
冬日暖阳
