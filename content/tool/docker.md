---
title: "docker简单使用"
date: 2020-01-14T17:25:08+08:00
draft: false
description: "docker简单使用"
tags: 
 - linux
 - docker
categories: 
 - tool
---





docker的一次简单使用

 <!--more-->

### 简介

Docker 包括三个基本概念:

-  **镜像（Image）**：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
-  **容器（Container）**：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
-  **仓库（Repository）**：仓库可看着一个代码控制中心，用来保存镜像。

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。

Docker 容器通过 Docker 镜像来创建。

容器与镜像的关系类似于面向对象编程中的对象与类。

docker命令：

![image](https://raw.githubusercontent.com/wantmoretime/wjdsite/master/images/images/docker_1.jpg)



### 安装启动

centos7系统下dockers安装，麻瓜式安装：

```shell
#安装
yum install docker
#启动
service start docker
```

### 简单使用

```shell
#加载镜像
docker load < 镜像
#ps：用docker import加载的centos镜像运行报错

#查看全部的镜像
docker images 

#查看正在运行的容器
docker ps

#查看全部的容器
docker ps -a

#启动镜像容器
docker run -idt <image_ID> /bin/bash

#启动时挂载本地目录，--privileged=true解决docker内权限问题：
docker run -v /home/workspace/:/mnt/ --privileged=true -idt <image_ID>  /bin/bash

#进入本地容器运行
docker exec -it <container_ID> /bin/bash

#删除容器container：
docker rm <container_ID>

#删除镜像image：
docker rmi <image_ID>
```

