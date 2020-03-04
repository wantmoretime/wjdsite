---
title: "I/O多路复用"
date: 2020-02-11T12:14:08+08:00
draft: true
description: "description"
tags: 
 - 网络编程
 - epoll
 - I/O

---









I/O多路复用（multiplexing）的本质是通过一种机制（系统内核缓冲I/O数据），让单个进程可以监视多个文件描述符，一旦某个描述符就绪（一般是读就绪或写就绪），能够通知程序进行相应的读写操作。

 <!--more-->

