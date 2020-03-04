---
title: "GDB调试"
date: 2020-01-08T21:16:30+08:00
draft: false
description: "总结一下日常使用的GDB调试指令"
tags: 
 - Linux
 - gdb
categories: 
 - tool
---





在Linux下开发C/C++程序，GDB调试的一项必须熟练掌握的技能，有助于DEBUG和提高开发效率。

 <!--more-->

## 调试信息

​	一般要调试某个程序，为了能清晰地看到调试的每一行代码、调用的堆栈信息、变量名和函数名等信息，需要调试程序含有**调试符号**信息。因此在使用gcc/g++编译程序时，需要加上-g选项保留调试符号信息。

​	一般在发布版本时，我们会选择去掉调试信息来减小程序大小和提高程序执行效率，除了不加-g选项，还可以通过strip命令来去除调试符号信息。

## GDB启动

一般GDB启动有以下几种方式：

* gdb 文件名 （直接调试程序）
* gdb 文件名 core文件名（调试core文件）
* gdb attach <pid>  （调试已运行的程序）

Linux系统默认不开启程序崩溃生成core文件，可以通过`ulimit 选项 设置值` 来修改设置。 

## GDB指令

### 常见的调试指令

| 命令名称    | 命令缩写 | **命令说明**                                           |
| ----------- | -------- | ------------------------------------------------------ |
| run         | r        | 运行一个程序                                           |
| continue    | c        | 让暂停的程序继续运行                                   |
| next        | n        | 运行到下一行                                           |
| step        | s        | 如果有调用函数，进入调用的函数内部，相当于 step into   |
| until       | u        | 运行到指定行停下来                                     |
| finish      | fi       | 结束当前调用函数，到上一层函数调用处                   |
| return      | return   | 结束当前调用函数并返回指定值，到上一层函数调用处       |
| jump        | j        | 将当前程序执行流跳转到指定行或地址                     |
| print       | p        | 打印变量或寄存器值                                     |
| backtrace   | bt       | 查看当前线程的调用堆栈                                 |
| frame       | f        | 切换到当前调用线程的指定堆栈，具体堆栈通过堆栈序号指定 |
| thread      | thread   | 切换到指定线程                                         |
| break       | b        | 添加断点                                               |
| tbreak      | tb       | 添加临时断点                                           |
| delete      | del      | 删除断点                                               |
| enable      | enable   | 启用某个断点                                           |
| disable     | disable  | 禁用某个断点                                           |
| watch       | watch    | 监视某一个变量或内存地址的值是否发生变化               |
| list        | l        | 显示源码                                               |
| info        | info     | 查看断点 / 线程等信息                                  |
| ptype       | ptype    | 查看变量类型                                           |
| disassemble | dis      | 查看汇编代码                                           |
| set args    |          | 设置程序启动命令行参数                                 |
| show args   |          | 查看设置的命令行参数                                   |

### 指令使用

```shell
# 设置程序启动命令行参数
(gdb) set args xxx.conf
# 查看设置的命令行参数
(gdb) show args
# 设置断点
(gdb) b main
# 显示断点
(gdb) info break
# 禁用断点
(gdb) disable  #可添加断点编号
# 启动断点
(gdb) enable   #可添加断点编号
# 删除断点
(gdb) delete   #可添加断点编号
# 运行程序
(gdb) run
# 程序中断时堆栈信息
(gdb) backtrace
# 堆栈编号切换
(gdb) frame <n>
# 查看线程信息
(gdb) info thread
# 线程切换
(gdb) thread 2
# 打印变量信息
(gdb) print <var>  #可以是变量名var、变量地址&var、或变量地址的内容*var等等
# 查看变量类型
(gdb) ptype <变量名>
# 查看断点出程序代码
(gdb) list     #+/-上下翻
# 继续运行
(gdb) continue
# 单步运行，深入程序内部逻辑
(gdb) step
# 单步运行，不进入函数
(gdb) next
# 运行到某一行
(gdb) until 100
# 运行到函数结束
(gdb) finish  #return 指令类似
# 跳转到某一行，忽略中间过程，可能
(gdb) jump 100
# 监视一个变量或内存
(gdb) watch <变量名或内存地址>
# 设置中断时自动打印变量值
(gdb) display <变量名>
# 删除display的变量
(gdb) delete display
# 设置临时断点
(gdb) tbreak main
```

一般使用时，在程序运行前设置好运行参数和需要打印或监视的变量，打好断点，运行程序，查看中断处的堆栈信息、线程状况、变量值是否符号预期，然后继续运行调试。对于一些疑难杂症需要深入挖掘GDB其他调试功能来解决。

### 其他设置

打印设置

```shell
(gdb) help set print  #显示所有print提供设置选项
(gdb) set print element 0  #取消打印长度限制。默认200
(gdb) set print object on/off(默认) #如果打开打印其真实的类型，而不是声明的类型
(gdb) set print pretty on/off(默认) #是否以更换行缩进的方式打印结构体，或者类
(gdb) set print address on/off   #打开(默认)/关闭打印地址
(gdb) set print static-member on(默认)/off #是否打印静态数据成员
(gdb) set print thread-events on(默认)/off #当程序启动或者线程退出时打印信息
(gdb) set print array-indexes on/off  #打开/关闭(默认) 显示数组index
(gdb) set print frame-arguments  all/scalars(默认)/none #在打印堆栈帧(bt)的时候显示参数的方式
...
```

条件断点

```
break [lineNo] if [condition]
```

lineNo 是程序触发断点后需要停下的位置，condition 是断点触发的条件

多进程调试切换

```shell
(gdb) show follow-fork-mode    #  选择调试父进程还是子进程
(gdb) show follow-fork-mode  parent
(gdb) show follow-fork-mode  child
```

多线程调试锁定线程

```shell
(gdb) set scheduler-locking on/off # 调试时设置，可以锁定当前调试线
```

