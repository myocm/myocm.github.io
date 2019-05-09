---
title: MySQL高可用架构
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL高可用架构
date: 2016-11-09 10:03:36
updated: 2016-11-09 10:03:36
tags:
---

# MySQL高可用架构

## 概述

### 高可用（High Availability）
- 应用提供持续不间断（可用）的服务的能力
- 系统高可用性的评价通常用可用率表示
![](/img/markdown-img-paste-20161109101110110.png)

### 造成不可用的原因
- 硬件故障（各种）
- 预期中的系统软硬件维护
- 软件缺陷（应用代码，服务程序都可能存在bug...）
- 攻击，泄露，人为失误...等安全事件
- 对于系统来说，不可用时间是各关键组件不可用时间的总和

### 提高可用性的主要手段
- 冗余，Redundancy
- 关键软硬件通过备用冗余避免故障时长时间的不可用
- 数据软件，硬件，存储的数据，都要通过冗余确保故障时可替换
![](/img/markdown-img-paste-20161109101529804.png)

### 数据库冗余与可用性
- 数据库服务在冗余实现上有其特殊性
  - 数据：服务“有状态” 与数据冗余
- 实现方式多种多样，同一种数据也会有多种实现方案
- 可用性目标循序渐进
  - 任何故障都不会造成数据丢失 --> 可以较快速恢复服务（高可用）

## MySQL高可用常见方案

### 基于共享存储的单活方案（极不常用）
![](/img/markdown-img-paste-20161109102821801.png)

### 基于存储复制的数据冗余单活（不常用）
![](/img/markdown-img-paste-20161109102927615.png)

### 基于MySQL主从复制（常用，普适）
![](/img/markdown-img-paste-20161109103031751.png)

### 基于集群提交通信协议的多主复制（一定场景适用）
Percona XtraDB Cluster（Galera cluster）
![](/img/markdown-img-paste-20161109103318671.png)

## MySQL主从复制方案
### 主从复制高可用方案需要改进的问题
- 主从服务器各自有IP地址，发生主从切换后应用需要修改重启
- 人工判断主库是否故障再发起切换需要花较多时间
- 主从复制存在客观延迟，切换后可能造成事务数据丢失

###  主从复制高可用方案改进
- 为了避免应用人工修改切换IP，引入VIP漂移方案  
![](/img/markdown-img-paste-20161109103809769.png)  
![](/img/markdown-img-paste-20161109103842969.png)
- 为了减少人工介入处理的时间开销引入自动探活处理机制
![](/img/markdown-img-paste-20161109104014343.png)

### 高可用中间层与RDS
- VIP解决应用切换问题
- 监控和管理服务器解决自动判断故障切换和VIP漂移
- VIP管理+探活+主从关系切换 = 高可用中间层
- 云环境+高可用中间层+底层数据库 = 一种PaaS = 基本RDS

#### 高可用中间层的两种方案
- MHA
  - 自动选择复制延迟最小的从节点并试图补全日志（但大部分主机故障情况下行不通）
  - 通常要求两从以上，会进行主从关系切换
  - 不提供VIP管理方案
- MMM
  - 提供了基本的VIP管理功能
  - 适合双主配置的一对主机，不会主动切换主从关系
  - 不支持主从数据延迟判断和补全

### 主从复制延迟
- 日志传输为什么会延迟  
![](/img/markdown-img-paste-20161109104833925.png)    
![](/img/markdown-img-paste-20161109104925267.png)  
![](/img/markdown-img-paste-20161109180036373.png)  
![](/img/markdown-img-paste-20161109180133558.png)

### 比较完善的MySQL高可用方案
- 半同步复制 + 高可用中间层 + VIP管理方案
例如：
  - 半同步复制 + MHA + Keepalive
  - 半同步复制 + RDS

### MySQL高可用框架 - MHA
- 用一个管理节点监控后端数据库主库可用性
- 提供VIP漂移接口，不提供具体方法
- 提供补全从库日志的脚本
![](/img/markdown-img-paste-20161109180638353.png)

### MHA的安装步骤
1. 规划
2. 配置服务器间域名和ssh互信访问
3. 在manager节点安装MHA node 和 manage组件及其依赖包
4. 在数据库服务器安装MHA node组件及其依赖包
5. 配置VIP管理脚本master_ip_failover和master_ip_online_change
6. 配置MHA配置文件mha_manager.cnf
7. 数据库配置主从，添加mha连接用户...etc
8. 启动MHA开始监控数据库主从集群
![](/img/markdown-img-paste-20161109181110980.png)









----
Good luck!  
冬日暖阳
