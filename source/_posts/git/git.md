---
title: git
list_number: false
categories:
  - git
comments: true
date: 2016-08-22 13:57:36
updated: 2016-08-22 13:57:36
tags:
description:
  使用 git 提交本博客源码
---

为了将本项目源码保存到GitHub，需要简单了解和使用git工具。
git的入门，参考[官方文档](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%85%B3%E4%BA%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6)即可。
### 初始化本地仓库
在hexo 博客项目的根目录，执行下列命令
```
git init
```

### 查看 git 状态
```
git status
```

### 添加远程仓库关联
```
git remote add origin https://github.com/myocm/myocm.github.io.git
```

### 显示远程仓库信息
```
git remote -v
# 或
git remote show origin
```
### 取远程仓库信息
```
git fetch origin
```

### 切换到本地分支
```
git checkout site
```

### 添加文件到本地仓库
```
git add *
```
### 提交到本地仓库
```
git commit -m "site init"
```

### 检查本地分支与远程分支映射是否正确
```
git remote show origin
```

### 提交本地分支到远程分支
```
git push
```

### 从远程分支拉取到本地
```
git pull
```
---
Good luck!
冬日暖阳
![](/img/a7d496e4c27b220dab21305d7b10f229.png)
