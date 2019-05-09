---
title: 新建一篇文章
date: 2016-08-11 11:05:02
categories:
- 其他

description:
  如何使用Hexo创建一篇新文章

---

# 使用Hexo 创建一篇新文章

## 创建文件
在命令行或`Git Shell` 中切换到Hexo项目，运行下列命令
```
hexo n <title>  # <title> 文章标题，同时也是文件名
```
执行完成后，会在项目的`source\_posts`目录下创建一个markdown文件，例如
```
INFO  Created: F:\GitHub\myocm\source\_posts\second.md
```

## 编辑文件
使用自己喜欢的markdown编辑器打开上一步hexo创建的markdown文件，使用markdown语法编辑自己的文章。


## 发布文件
文章写完后，在命令行或`Git Shell` 中切换到Hexo项目，运行下列命令
```
hexo g   # 产生可以发布的静态文件
```
会在项目的public目录下生成网站的静态文件，拷贝所有文件到GitHub的本地文件夹，打开本机GitHub，登录并发布到GitHub服务器即可。

==========  
推荐markdown编辑器
[http://chenguanzhou.github.io/MarkDownEditor/](http://chenguanzhou.github.io/MarkDownEditor/)  
支持win10，支持本地图片上传到imgur.com 或 七牛

----
Good luck!  
冬日暖阳
![](/img/fab34411cdca2bfa7558d8f8891c9860.png)
