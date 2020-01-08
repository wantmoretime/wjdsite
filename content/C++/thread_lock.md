---
title: "线程锁"
date: 2020-01-07T14:42:34+08:00
draft: false
description: "线程锁及Pthread API接口使用"
tags: 
 - C++11
 - multi-thread
categories: 
 - C/C++
---













 多线程程序中，一般可能“同时发生多个写操作”或“同时发生读写操作”的情况时，必须加锁。如果只是同时发生多个读操作没有写操作的话，不需要加锁。

<!--more-->

多线程编程中常见的几种线程锁：互斥锁、条件变量、自旋锁、读写锁、递归锁，一般而言，锁的功能越强大，性能就会越低。下面介绍几种锁及pthread库的一些API使用。

## 同步与互斥

[同步]：是指散布在不同任务之间的若干程序片断，它们的运行必须严格按照规定的某种先后次序来运行，这种先后次序依赖于要完成的特定的任务。

[互斥]：是指散布在不同任务之间的若干程序片断，当某个任务运行其中一个程序片段时，其它任务就不能运行它们之中的任一程序片段，只能等到该任务运行完这个程序片段后才可以运行。

***互斥是指某一资源同时只允许一个访问者对其进行访问，具有唯一性和排它性。但互斥无法控制对资源的访问顺序。***

***同步是指在互斥的基础上实现对资源的有序访问。***

## 互斥锁(mutex)

互斥锁具有以下特点：

原子性和唯一性：把一个互斥锁定义为一个原子操作，由操作系统保证，当一个线程获得了互斥锁，则其他线程无法在该锁未释放时获得这个锁。

非忙等待：如果一个线程获得了一个互斥锁，则后续其他线程尝试去获取这个锁都将被挂起（不占用cpu资源），直到前面的线程释放该互斥锁，后续线程才能被唤醒并继续执行。

```c++
#include <pthread.h>
pthread_mutex_t*      mutex;
mutex = new pthread_mutex_t(); //创建一个锁对象
//phtread_mutexattr_t*  mutexattr;  //互斥锁属性，默认属性取NULL

//初始化锁
pthread_mutex_init(mutex, NULL);                     //动态初始化互斥锁
//pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;  //静态初始化互斥锁

// 对互斥锁上锁，若互斥锁已经上锁，则调用者一直阻塞，
// 直到互斥锁解锁后再上锁。
pthread_mutex_lock(mutex);

// 调用该函数时，若互斥锁未加锁，则上锁，返回 0；
// 若互斥锁已加锁，则函数直接返回失败，即 EBUSY。
pthread_mutex_trylock(mutex);

// 当线程试图获取一个已加锁的互斥量时，pthread_mutex_timedlock允许绑定线程阻塞时间。
// pthread_mutex_timedlock函数与pthread_mutex_lock函数是基本等价的；
// 但是当达成超时时间，pthread_mutex_timedlock因没拿到互斥锁而返回错误码ETIMEOUT
struct timespec timeout;
clock_gettime(CLOCK_REALTIME, &timeout);
pthread_mutex_timedlock(mutex,&timeout);

// 对指定的互斥锁解锁。
pthread_mutex_unlock(mutex);

// 销毁指定的一个互斥锁。互斥锁在使用完毕后，
// 必须要对互斥锁进行销毁，以释放资源。
pthread_mutex_destroy(mutex);
delete mutex;
```



## 条件变量(condition variable)

利用线程间共享的全局变量进行同步的一种机制，主要包括两个动作：

* 一个线程阻塞等待“条件变量的条件成立”而挂起

* 另一个线程通知“条件成立”（给出条件成立信号）

为了防止竞争，条件变量的使用总是和互斥锁结合在一起。

```c++
pthread_cond_t*                 m_cond;
pthread_condattr_t*             m_condattr;

pthread_condattr_init(m_condattr);
pthread_mutexattr_setpshared(&m_condattr,PTHREAD_PROCESS_SHARED);  //设置进程共享属性
pthread_condattr_setclock(&m_condattr, CLOCK_MONOTONIC);  //设置时间属性
pthread_cond_init(m_cond, m_condattr);

//通知
pthread_cond_signal(m_cond);   
pthread_cond_broadcast(m_cond); //唤醒所有的线程

//等待:m_mutex为互斥锁，先解锁互斥锁，以阻塞方式等待条件变量，收到信号后，再进行加锁
pthread_cond_wait(m_cond, m_mutex);//
pthread_cond_timedwait(m_cond, m_mutex, &timeout);  //指定等待时间

pthread_condattr_destroy(m_condattr);
pthread_cond_destroy(m_cond);

delete m_cond;
delete m_condattr;
```



## 自旋锁(spin lock)

自旋锁：当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。

获取锁的线程一直处于活跃状态，但是并没有执行任何有效的任务，使用这种锁会造成忙等(busy-waiting)。因此自旋锁不应该被长时间持有，否则特别的浪费CPU时间，自旋锁设计的初衷应该是短时间内进行轻量级的加锁。

自旋锁与互斥锁比较：线程在等待一个自旋锁释放会占用cpu空转避免不必要的上下文切换，等待互斥锁则是将线程挂起，不会占用cpu，代价是需要线程上下文切换的开销。

```c++
int pthread_spin_destroy(pthread_spinlock_t *lock);
int pthread_spin_init(pthread_spinlock_t *lock, int pshared);//设置自旋锁多进程共享

int pthread_spin_lock(pthread_spinlock_t *lock);
int pthread_spin_trylock(pthread_spinlock_t *lock);
int pthread_spin_unlock(pthread_spinlock_t *lock);
```



## 读写锁

读锁：可以在没有写锁的时候被多个线程持有

写锁：只能有一个写锁，是独占的(排他的)

互斥原则：

- 读-读能共存;
- 读-写不能共存;
- 写-写不能共存。

所有读写锁的实现必须确保写操作对读操作的内存影响。

```c++
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);                                         //初始化读写锁
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);    //销毁读写锁
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);     //加读锁
//阻塞: 之前对这把锁加的写锁的操作
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);  //尝试加读锁
//加锁成功: 0; 加锁失败: 错误号
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);     //加写锁
//阻塞: 上一次加写锁还没有解锁; 上一次加读锁没解锁
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);  //尝试加写锁
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);     //解锁
```

