---
title: "Linux常用命令"
date: 2020-01-08T21:16:30+08:00
draft: false
description: "常用的Linux命令汇总"
tags: 
 - Linux
 - cmd
categories: 
 - tool
---







 <!--more-->

## ipcs/ipcrm

功能：查看/删除IPC的信息

使用：





## strace







查看负载：

w

uptime

top



## dmesg





## vmstat





## mpstat

功能：用于打印一次每个CPU的统计信息，**可用于查看CPU的调度是否均匀**。

使用：

mpstat -P ALL 1





## pidstat

该命令用于打印各个进程对CPU的占用情况，类似top命令中显示的内容。pidstat的优势在于，可以滚动的打印进程运行情况，而不像top那样会清屏。

pidstat 1



## sar

功能：系统活动状态报告。

sar -n DEV 1            用于报告网络统计数据

sar -n TCP,ETCP 1   可以用于粗略的判断网络的吞吐量，如发起的网络连接数量和接收的网络连接数量





## awk







## sed





## strip

从特定文件中剥掉一些符号信息和调试信息



## nm

列出符号信息

## file



## strings

