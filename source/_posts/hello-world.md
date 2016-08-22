---
title: Hello World  
date: 2016-08-10 10:10:10  
list_number: false  
categories:  
- 其他

description:  
  你看到的这个页面是怎么来的  
    --- 记如何使用GitHub pages 和 Hexo 搭建免费个人博客
---


# 你看到的这个页面是怎么来的
  --- 记如何使用GitHub pages 和 Hexo 搭建免费个人博客



## 准备工作

### 1. 注册GitHub账号  
- 登录[GitHub](https://github.com/) 注册自定义账号。
- 创建一个新的仓库（repository），name必须和用户名一致,后面跟
`.github.io `,如 `mycom.github.io`,因为`.github.io `是GitHub pages 的域名。

### 2. 注册自己的域名（可选）
- 推荐域名服务商 [GoDaddy](http://www.godaddy.com/) ，支持支付宝，还可以在网上搜索优惠码。

### 3. 下载软件
- 下载[GitHub Windows](https://windows.github.com/)
- 下载[Node.JS](http://nodejs.org/)

## 搭建步骤

### 1. 初始化本机环境
安装GitHub Windows 和 Node.JS 。
### 2. 安装Hexo
打开 Git Shell ,输入如下命令安装
``` sh
npm install -g hexo-cli
npm install hexo --save
```

### 3. 初始化Hexo
- 创建Hexo文件夹，如 ` F:\GitHub `
- 进入` Git Shell `切换到该路径下` F:\GitHub `执行下列命令
```
hexo init <folder>
cd <folder>
```
- 安装Hexo 插件，在上面的` Git Shell `窗口中，输入下列命令
```bash
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked --save
npm install hexo-renderer-stylus --save
npm install hexo-generator-feed --save
npm install hexo-generator-sitemap --save
```
- 启动hexo，在本地查看效果
```
hexo server
```
在浏览器里输入 `http://localhost:4000  `
如果能看到页面，则表示Hexo初始化成功。

### 4. 发布blog到是GitHub
- 手工发布  
  1. 产生静态部署文件     
在` Git Shell ` 中，输入以下命令
```
hexo g #生成public静态文件
```
  2. 上传文件到GitHub   
打开桌面GitHub，输入用户名密码登录GitHub，把自己创建的仓库clone到本地一个文件夹，里面应该是空的。把hexo产生的public文件夹里的文件全部拷贝到GitHub本地的文件夹里。然后在GitHub 里写上备注，commit，然后点击右上角的sync，同步到GitHub 网站。
  3. 部署完成，可以访问自己的GitHub pages域名，如mycom.github.io，查看效果。

- 使用Hexo的一键部署  
  1. 修改根目录的`_config.yml`文件中
```
deploy:
  type: git
  repo: <repository url>
  branch: [branch]
  message: [message]
```

    指定部署类型为 git  
    指定库地址即可。
    ![](http://i.imgur.com/BTRsLZn.png)
  2. 执行部署命令
```
hexo generate --deploy
```


## 使用主题
### 1. 选择主题  
网站的页面布局，扩展功能等在Hexo中是通过主题（themes）来配置和实现的。
Hexo 网站有很多主题[https://hexo.io/themes/](https://hexo.io/themes/),可以挑选自己喜欢的主题。
例如本站使用的[Jacman](https://github.com/wuchong/jacman)主题。

### 2. 使用主题  
- 下载主题
选择好自己喜欢的主题后，把主题代码clone到Hexo项目的`themes`目录。
比如下载jacman主题，使用如下命令,在博客的根目录执行
```
git clone https://github.com/wuchong/jacman.git themes/jacman
```
下载完成后，可以在`themes`目录看到`jacman`目录。
- 启用主题
修改你的博客根目录下的`_config.yml`配置文件中的`theme`属性，
将其设置为你刚刚下载的主题，如`jacman`
- 配置主题
主要是修改主题目录里面的`_config.yml`配置文件，参照主题说明设置即可。
  - 本站`jacman`主题的配置参照[如何使用 Jacman 主题](http://jacman.wuchong.me/2014/11/20/how-to-use-jacman/)  
  对于新手使用`jacman`主题时，有几处[如何使用 Jacman 主题](http://jacman.wuchong.me/2014/11/20/how-to-use-jacman/)说的比较简单，列举如下
    1. 什么是`front-matter`,就是文件最上方用`---` 隔开的对文件属性进行说明的区域，参考[front-matter](https://hexo.io/zh-cn/docs/permalinks.html)
    2. 启用`多说`评论系统的`duoshuo_shortname`是指[http://duoshuo.com/create-site/](http://duoshuo.com/create-site/)里面的多说二级域名。
- 自定义404页面
使用腾讯公益404页面，在博客source目录，建立404.html页面，放入如下代码
```
<!DOCTYPE html>
<html>
<head>
</head>
<body>
<script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8" homePageUrl="http://mydba.xyz" homePageName="回到主页"></script>
</body>
</html>
```
把`homePageUrl`修改为自己的域名即可。

## 使用自己的域名

### 1. 注册域名
参考上面的准备工作

### 2. 在GitHub 设置自己的域名
登录GitHub网站，在自己的博客项目里点击`Settings`,在  `Custom domain` 里输入自己的域名，保存。

### 3. 设置域名解析
推荐使用[DNSPod](https://www.dnspod.cn/)  做域名解析，参考[http://winterttr.me/2015/10/23/from-dns-to-github-custom-domain/](http://winterttr.me/2015/10/23/from-dns-to-github-custom-domain/)

==========  
主要参考 https://wsgzao.github.io/post/hexo-guide/
Hexo http://hexo.io/
Hexo docs https://hexo.io/zh-cn/docs/index.html
jacman  https://github.com/wuchong/jacman/wiki/%E9%85%8D%E7%BD%AE%E6%8C%87%E5%8D%97


---
Good luck!  
冬日暖阳
