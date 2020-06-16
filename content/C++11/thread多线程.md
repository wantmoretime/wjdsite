---
title: "std::thread"
date: 2020-06-16T10:45:00+08:00
draft: false
tags: 
 - C++ 11
 - 多线程
 - 源码
categories: 
 - C/C++
---







C++11之前的多线程使用pthread库是基于C语言的标准库，C++11标准库支持了自己的多线程库<thread>,在std命名空间下。



 <!--more-->

​	线程在构造关联的线程对象后立即开始执行，从作为构造函数参数提供的执行函数开始。 执行函数的返回值将被忽略，如果它通过引发异常终止，则将调用std :: terminate。执行函数可以通过std :: promise或修改共享变量（如果需要同步，请参见std :: mutex和std :: atomic）将其返回值或异常传达给调用方。

​	std::thread的不可拷贝和不可赋值的，但是是可以移动拷贝和移动赋值的。

## 源码内部实现

### Member types

| Member type                    | Definition               |
| ------------------------------ | ------------------------ |
| `native_handle_type`(optional) | *implementation-defined* |

| Member classes                                            |                                                       |
| --------------------------------------------------------- | ----------------------------------------------------- |
| [ id](https://en.cppreference.com/w/cpp/thread/thread/id) | represents the *id* of a thread (public member class) |

| Member functions                                             |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [(constructor)](https://en.cppreference.com/w/cpp/thread/thread/thread) | constructs new thread object                                 |
| [(destructor)](https://en.cppreference.com/w/cpp/thread/thread/%7Ethread) | destructs the thread object, underlying thread must be joined or detached |
| [operator=](https://en.cppreference.com/w/cpp/thread/thread/operator%3D) | moves the thread object                                      |

| Observers                                                    |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [joinable](https://en.cppreference.com/w/cpp/thread/thread/joinable) | checks whether the thread is joinable, i.e. potentially running in parallel context |
| [get_id](https://en.cppreference.com/w/cpp/thread/thread/get_id) | returns the id of the thread                                 |
| [native_handle](https://en.cppreference.com/w/cpp/thread/thread/native_handle) | returns the underlying implementation-defined thread handle  |
| [ hardware_concurrency](https://en.cppreference.com/w/cpp/thread/thread/hardware_concurrency) [static] | returns the number of concurrent threads supported by the implementation |

| Operations                                                   |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [join](https://en.cppreference.com/w/cpp/thread/thread/join) | waits for a thread to finish its execution                   |
| [detach](https://en.cppreference.com/w/cpp/thread/thread/detach) | permits the thread to execute independently from the thread handle |
| [swap](https://en.cppreference.com/w/cpp/thread/thread/swap) | swaps two thread objects                                     |

| Non-member functions                                         |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [ std::swap(std::thread)](https://en.cppreference.com/w/cpp/thread/thread/swap2) | specializes the [std::swap](https://en.cppreference.com/w/cpp/algorithm/swap) algorithm |



## Example

例1：

```c++
#include <iostream>
#include <thread>
#include <chrono>
 
void foo()
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
void bar()
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
int main()
{
    std::thread t1(foo); //启动线程执行foo()函数,函数异常退出会调用std::terminate
    std::thread t2(bar);
 
    std::cout << "thread 1 id: " << t1.get_id() << '\n'
              << "thread 2 id: " << t2.get_id() << '\n';
 
    std::swap(t1, t2);
 
    std::cout << "after std::swap(t1, t2):" << '\n'
              << "thread 1 id: " << t1.get_id() << '\n'
              << "thread 2 id: " << t2.get_id() << '\n';
 
    t1.swap(t2);
 
    std::cout << "after t1.swap(t2):" << '\n'
              << "thread 1 id: " << t1.get_id() << '\n'
              << "thread 2 id: " << t2.get_id() << '\n';
 
    t1.join(); //等待线程执行退出
    t2.join();
}

//g++ -o main main.cpp -std=c++11 -lpthread
```

例2：

```c++
#include <iostream>
#include <thread>
#include <chrono>
 
void foo()
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
int main()
{
    std::thread t;
    std::cout << "before starting, joinable: " << std::boolalpha << t.joinable()
              << '\n';
 
    t = std::thread(foo);
    std::cout << "after starting, joinable: " << t.joinable() 
              << '\n';
 
    t.join();
    std::cout << "after joining, joinable: " << t.joinable() 
              << '\n';
}

```

例3：

```c++
#include <iostream>
#include <chrono>
#include <thread>
 
void independentThread() 
{
    std::cout << "Starting concurrent thread.\n";
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Exiting concurrent thread.\n";
}
 
void threadCaller() 
{
    std::cout << "Starting thread caller.\n";
    std::thread t(independentThread);
    t.detach();
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Exiting thread caller.\n";
}
 
int main() 
{
    threadCaller();
    std::this_thread::sleep_for(std::chrono::seconds(5));
}
```



参考：

https://en.cppreference.com/w/cpp/thread/thread