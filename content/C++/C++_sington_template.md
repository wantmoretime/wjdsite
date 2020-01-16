---
title: "C++单例模式"
date: 2020-01-09T11:40:27+08:00
draft: false
description: "C++单例模式"
tags: 
 - C++
 - parttern
categories: 
 - C/C++
---







单例模式用于保证系统里面一个类最多只能存在一个实例，例如缓冲池、数据库连接池、线程池、应用服务实例等。

 

### 饿汉式

```c++
class Singleton
{
public:
  static Singleton &get_instance()
  {
    static Singleton instance; 
    return instance;
  }
private:
    Singleton() = default;
};
```

### 懒汉式

```c++
//延迟实例化单例对象
class Singleton
{
public:
  static Singleton &get_instance()
  {
    if(NULL==instance)
      instance = new Singleton();
    return instance;
  }
private:
  static Singleton *instance;
  Singleton() = default;
};
```



### 单例类模板实例

```c++
//c++11
template <class T>
class Singleton
{
public:
  Singleton(const Singleton &rhs) = delete;
  Singleton &operator=(const Singleton &rhs) = delete;
  static T &get_instance()
  {
    static T instance;
    return instance;
  }
protected:
    Singleton() = default;
    ~Singleton() = default;
};
//使用
#define DERIVE Derive::get_instance()  //定义获取单例
class Derive : public Singleton<Derive>
{
    ...
}
```

在工作中遇到只能使用c++11以前的版本，不能使用c++使用的一些特性，=delete/=default不能使用，单例模板类的派生类中还需要声明单例模板类的实例为派生类的友元类。

```c++
//c++11
template <class T>
class Singleton
{
public:
  static T &get_instance()
  {
    static T instance;
    return instance;
  }
protected:
    Singleton(){}
    
private://声明为私有的，但不定义，可以保证不被调用
  Singleton(const Singleton &rhs);
  Singleton &operator=(const Singleton &rhs);
};
//使用
#define DERIVE Derive::get_instance()  //定义获取单例
class Derive : public Singleton<Derive>
{
	friend class Singleton<Derive>;
    ...
}
```

