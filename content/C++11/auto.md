---
title: "auto关键字"
date: 2020-02-27T12:45:00+08:00
draft: false
tags: 
 - C++ 11
categories: 
 - C/C++
---











 <!--more-->

使用的形式如下：

```c++
auto x = expression;
```

x的类型就是表达式计算出来结果的类型。

使用auto来推导被初始化的变量的类型，在类型未知或不便书写的情况下是非常有用的。如下：

```c++
template<class T> void printall(const vector<T>& v)
{
	for (auto p = v.begin(); p!=v.end(); ++p)
		cout << *p << "\n";
}
```

同样的代码含义，在c++98里，我们必须这么写：

```
template<class T> void printall(const vector<T>& v)
{
	for (typename vector<T>::const_iterator p = v.begin(); p!=v.end(); ++p)
		cout << *p << "\n";
}
```

当变量的数据类型依赖于模板参数时，如果不使用auto关键字，将很难确定变量的数据类型。如：

```
template<class T, class U> void multiply(const vector<T>& vt, const vector<U>& vu)
{
	// ...
	auto tmp = vt[i]*vu[i];
	// ...
}
```

在这里，tmp的数据类型应该与模板参数T和U相乘之后的结果的数据类型相同。对于程序员来说，要通过模板参数确定tmp的数据类型是一件很困难的事情。但是，对于编译器来说，一旦确定了T和U的数据类型，推断tmp的数据类型将是轻而易举的一件事情。

auto主要用于简化代码，因此并不会影响标准库规范。