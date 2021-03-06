---
title: "线程池"
date: 2020-01-07T14:42:34+08:00
draft: true
description: "线程池原理及C++实现"
tags: 
 - C++11
 - thread pool
categories: 
 - C/C++
---







<!--more-->

## 简介

**线程池**

一种线程的使用模式。线程过多会带来调度开销，进而影响缓存局部性和整体性能。而线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。这避免了在处理短时间任务时创建与销毁线程的代价。线程池不仅能够保证内核的充分利用，还能防止过分调度。可用线程数量应该取决于可用的并发处理器、处理器内核、内存、网络sockets等的数量。 

**线程池作用**

>1. 降低资源消耗。通过重复利用已创建的线程，降低线程创建和销毁造成的消耗。
>2. 提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
>3. 提高线程的可管理性。

**线程池的使用场景**

1.单个任务处理的时间很短而请求的数目确实巨大的。

即线程创建时间和销毁时间大于线程中执行任务的时间。如果我们为每一个请求都单独创建一个线程，那么物理机的所有资源基本上都被操作系统创建线程、切换线程状态、销毁线程这些操作所占用，用于业务请求处理的资源反而减少了。

另外，一些操作系统是有最大线程数量限制的。当运行的线程数量逼近这个值的时候，操作系统会变得不稳定，这也是我们要限制线程的原因。



## 线程池原理

### 线程池组成



### 线程池工作原理





## 代码















