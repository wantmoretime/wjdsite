---
title: "Github+Hugo搭建个人博客网站"
date: 2020-01-04T18:38:29+08:00
draft: false
description: "从零开始搭建属于自己的静态网站"
tags: 
 - hugo
 - github
categories: 
 - tool
---





本篇blog主要介绍使用Github+hugo,简单搭建个人博客。

 <!--more-->



## 前期准备

- Git
- Go(go版本1.11以上)

## hugo环境搭建（win10环境下）

版本下载路径：https://github.com/gohugoio/hugo/releases

新建一个文件夹hugo，并将该文件夹路径添加到windows系统环境变量PATH下。

下载需要的版本，解压到Hugo文件夹下即可，不需要安装。



## hugo操作

使用window命令行cmd或git bash命令行工具进行操作都可以，下面以git bash操作为例。

**创建新的静态网站**

```
hugo$ hugo new site new-site
```

hugo文件夹下将新建一个new-site文件夹，里面包含的文件结构及作用:
|- archetypes ：存放default.md，头文件格式 
|- content ：content目录存放博客文章（markdown文件） 
|- data ：存放自定义模版，导入的toml文件（或json，yaml） 
|- layouts ：layouts目录存放的是网站的模板文件 
|- static ：static目录存放图片，css等静态资源 
|- themes ：存放网站主题文件
|- config.toml ：config.toml是网站的配置文件

现在为网站添加一个主题，在hugo官网找一个想要的主题，然后克隆到hugo/new-site/theme文件夹下。

比如添加的hugo-primer主题，在hugo/new-site/theme路径下执行:

`hugo/new-site/theme$ git clone https://github.com/qqhann/hugo-primer.git`

将主题添加到config配置文件中，在hugo/new-site路径下执行：

`hugo/new-site$ echo 'theme = "hugo-primer"' >> config.toml`

**创建自己的一个blog文件**

在hugo路径下执行：`$ hugo new posts/first-post.md`，将在content的文件夹下生成posts文件夹及posts文件夹下first-post.md文件。

编辑first-post.md文件，并去掉draft: true（草稿）这一行，否则在生成页面将被忽略，简单示例如下。

```
---
title: "First Post"
date: 2020-01-04T00:17:54+08:00
---

## Hello world
```

**本地启动hugo服务**

```
hugo/new-site$ hugo server
```

浏览器打开`http://localhost:1313`即可看到你的网页了。

**hugo生成静态网页**

```
hugo/new-site$ hugo
```

生成public文件夹，下面即包含你的静态网页，也就是待发布的网站内容。

## Github发布静态网站

新建git仓库newsite.github.io，不要生成readme文件。

下面在本地git进行操作

```git
$ cd hugo/new-site/public  //进入待发布的文件夹下
$ git init //(初始化git仓库)
$ git add .
$ git commit -m "first commit"
$ git remote add origin https://github.com/newsite.github.io.git   //添加远程仓库
$ git push -u origin master
```

如果创建仓库时，生成其他文件，则需要先进行 `git pull origin master`下拉操作。

在你的github页面上就可以看到推上去的文件了。

在github上设置个人网站页如下：

```
github-> setting -> GitHub Pages -> source
选择master分支
```

如果想要选择master/docs分支的话，需要你将本地整个新生成new-site文件夹全部推到github上，本地config.toml配置加一句`publishDir = "docs"`，让静态网页生成到docs文件夹下面。

设置刷新后，你就可以在这个地方看到你个人的网站地址了。

如果你本地配置文件config.toml中的baseURL值和你现在发布的URL不一致，修改配置文件的值，然后hugo一下，重新生成静态网页，重新push到github即可。

现在，你就可以看到你自己的静态博客网页了！

