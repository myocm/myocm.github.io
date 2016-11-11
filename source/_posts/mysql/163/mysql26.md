---
title: MySQL线上部署
list_number: false
categories:
  - MySQL
comments: true
description:
  - MySQL线上部署
date: 2016-11-07 13:35:59
updated: 2016-11-07 13:35:59
tags:
---

# MySQL线上部署

## 线上部署考虑因素
- 版本选择，5.1、5.5、5.6、5.7
- 分支选择，官方社区版、percona server、Mariadb
- 安装方式，包安装，二进制包安装，源码安装
- 路径配置，参数配置（尽量模板化，标准化）
- 一个实例多个库 or 多个实例单个库


## 编译安装及二进制安装
### 二进制安装
1. 下载软件安装包
2. 解压放到指定目录（比如/usr/local）
3. 将MySQL目录放到PATH中
4. 初始化实例，编辑配置文件，并启动
5. 账户安全设置

### 编译安装MySQL
1. 下载MySQL源码安装包
2. 安装必要的包（make cmake bison-devel ncurses-devel build-essential）
3. cmake配置mysql编译选项，可以定制需要安装的功能，[编译选项参考](http://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html)
4. make && make install
5. 初始化实例，编辑配置文件并启动
6. 账户安全设置


## 升级MySQL
例如从5.5升级到5.6
1. 下载MySQL5.6安装包并配置MySQL5.6安装包路径
2. 关闭MySQL5.5的实例，修改部分参数，使用MySQL5.6软件启动
3. 执行MySQL5.6路径下mysql_upgrade脚本
4. 验证是否成功升级

## 多实例部署MySQL数据库
1. 部署好MySQL软件
2. 编辑多个配置文件，初始化多个实例
3. 启动MySQL实例
※ 不推荐使用mysqld_multi脚本启动多个实例，多个实例使用一个配置文件不容易维护。

### 多实例部署的好处
1. 充分利用系统资源
2. 资源隔离
3. 业务、模块 隔离

## 合理部署MySQL线上库
1. 根据需求选择合适的版本及分支，建议使用或升级到较高版本5.6或5.7
2. 如果需要定制MySQL功能的话，可以考虑编译安装，否则的话，建议使用二进制包安装，比较省事
3. 根据机器配置选择部署多个MySQL实例还是单个实例，机器配置非常好的话，建议部署多实例


----
Good luck!  
冬日暖阳
