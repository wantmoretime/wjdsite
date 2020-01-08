---
title: "C++函数指针"
date: 2020-01-06T20:38:00+08:00
draft: false
description: "C++函数指针及使用map管理函数指针的应用"
tags: 
 - C++
 - leetcode
categories: 
 - C/C++
---





C++函数指针以及map管理函数指针的应用

 <!--more-->



## 函数指针

函数指针指向的是一个函数而非对象。和其它指针一样，函数指针指向某种特定类型。函数指针的类型由它的返回类型和形参类型共同决定，与函数名无关。                                                                                                 --C++Primer

示例：

```C++
//函数
bool lengthCompare(const string&, const string&);
//函数类型
bool (const string&, const string&);
//函数指针
bool (*pf)(const string&, const string&);

//直接将函数名作为一个值使用，该函数自动转换为指针
pf = lengthCompare;   //pf指向名为lengthCompare的函数
pf = &lengthCompare;  //等价语句
//调用该函数,下面三种方式等价
bool b1=pf("hello","goodbye");
bool b1=(*pf)("hello","goodbye");
bool b1=lengthCompare("hello","goodbye");
```

另外，函数指针可以作为指针参数传给另一个函数去调用，或者作为另一个函数的指针参数返回。

```c++
//函数指针形参，它会自动地转换成指向函数的指针
void useBigger(const string& s1, const string&s2, bool pf(const string&, const string&));
//一个等价声明：显式的将形参定义成指向函数的指针
void useBigger(const string& s1, const string&s2, bool (*pf)(const string&, const string&));

//使用时可以直接把函数作为实参使用，它会自动转成指向该函数的指针：
useBigger(s1,s2,lengthCompare);
```

想要声明一个返回函数的指针，最简单的办法是使用类型别名：

```c++
using F = int(int*,int);    //F是函数类型。不是指针
using PF = int(*)(in*t,int); //PF是指针类型

//和函数类型的形参不同，返回函数不会自动的转换成指针，必须显式的将返回类型指定为指针：
PF f1(int); //f1本身有一个int型参数，返回一个PF类型指针
F *f1(int); //显式指定返回类型是指向函数的指针
//也可以使用下面形式直接声明f1
int (*f1(int))(int*, int);
//C++11标准中，还可以使用尾置返回类型的方式
auto f1(int) -> int(*)(int*,int);
```



## map管理函数指针

vector使用类似，这边以map为例分析。

```c++
class test
{
public:
    int func1(std::string s1);
    int func2(std::string s1);
    int func3(std::string s1)
        
    //定义函数指针及map
	typedef int(test::*pFunc)(std::string);
	std::map<std::string, pFunc> manager;
    void set_func();
    void use_func();
}

//函数
int test::func1(std::string s1)
{
    cout<<s1<<endl;
}
int test::func2(std::string s2)
{
    cout<<s2<<endl;
}
int test::func3(std::string s3)
{
    cout<<s3<<endl;
}
//...
//函数指针存入map
void test::set_func()
{
	manager["func1"] = &test::func1;
	manager["func2"] = &test::func2;
	manager["func3"] = &test::func2;
}
//调用
void test::use_func()
{
    ...
    std::string s="hello,world";
    std::string key="func1";
    int ret = (this->*manager[key])(s);
    ...
}
```

在c++11新标准中可以直接使用std::function和std::bind来管理函数指针。

```c++
//map的声明
std::map<std::string, std::function<int(std::string)>> manager;
//函数指针存入map,其中std::placeholders::_1为占位符
void test::set_func()
{
	manager["func1"] = std::bind(&test::func1,this,std::placeholders::_1);
	manager["func2"] = std::bind(&test::func2,this,std::placeholders::_1);
	manager["func3"] = std::bind(&test::func3,this,std::placeholders::_1);
}
//调用
void test::use_func()
{
    ...
    std::string s="hello,world";
    std::string key="func1";
    int ret = manager[key](s);
    ...
}
```

在使用上C++11的std::function和std::bind看着更为规范一些，在函数类型较为复杂的话，也更为方便。

