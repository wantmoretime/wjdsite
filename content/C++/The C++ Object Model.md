# The C++ Object Model
[TOC]

## 零. 基础知识

Object-Oriented Features

- Encapsulation
- Inheritance
- Polymorphism

Data members

* nonstatic data member
* static data member

Member Functions

* nonstatic member function
* static member function
* virtual function

## 一. 构造、析构、拷贝、赋值

### Constructor

有4种情况会造成编译器必须为未声明Constructor的classes合成一个default constructor。

* 含有若干个带有Default Constructor的类对象成员（扩展已有的constructor）
* 基类带有Default Constructor的派生类（扩张已有的constructor）
* 带有Virtual Function的类（合成vtbl 和 vptr）
* 带有一个Virtua Base Class的类（合成允许虚基类在执行期存取操作的代码）

至于没有存在以上四种情况，有没有自定义constructor的class，拥有的是implicit trivial default constructor （无用的constructor），实际并没有被合成出来。

在合成的default constructor中，只有base class subobjects和member class object会被初始化，所有其他的nonstatic data member（如整型、整型指针、整型数组等）都不会被初始化。因为这些对于编译器是非必要的。

### Copy Constructor

有三种情况会以一个对象作为另一个对象的初值。

```
X::X(const X&); //第一个参数必须是引用类型
```

* 对一个对象作显式的初始化操作
* 对象被单子参数传给某个函数
* 函数传回一个对象

```c++
class X{ ... };
//初始化
X a=b;//X a(b);
//对象作为传参
extern void foo(X x);
void bar()
{
    X b;
    foo(b); 
}
//传回对象
X foo(X x)
{
    X a;
    return a;
}
```

#### Default Memberwise Initialization

当class object以“相同class的另一个object”作为初值，其内部是以所谓的default memberwise initialization手法完成的。也就是把每个内建的或派生的data member的值，从一个object拷贝到另一个object上。不过它不会拷贝其中的member class object，而是以一种递归的方式实施memberwise initialization。

决定一个copy constructor是否会被合成出来标准在于class是否展现出所谓的“bitwise copy semantics”。

#### Bitwise Copy(位逐次拷贝)

存在的问题：当对象存在指针成员，拷贝后，两者指向同一块内存，当有对象被释放，两个对象都要调用delete删除内存，从而极可能引发错误。

解决方法：自定义copy constructor，重新分配内存。

#### 不要bitwise copy semantics！

1.*当class内含一个member object而后者的class声明一个copy constructor*。（合成copy contructor，以便调用member object的copy constructor）

2.*当class继承一个base class而后者存在一个copy constructor*。（合成copy contructor，以便调用base class的copy constructor）

3.*当class声明一个或多个virtual function。*（需要为class object合成一个copy constructor来将vptr初始化）

```c++
class base
{
public:
    base();
    virtual ~base();
    virtual func_base();
private:
    //some data
};
class Derive:public base
{
public:
    Derive();
    virtual ~Derive();
    virtual func_base();   //overwrite
    virtual func_derive();
private:
	//some data
};
derive a;
derive b=a;// bitwise ok
base c=a;  // 如果是bitwise将是错误的, 当调用func_base将调用基类的函数
```

每一个新生成的object需要正确设置初值，否则当基类对象以派生类对象初始化，将导致严重错误。

4.*当class派生自一个继承串链，其中有一个或多个virtual base classes*。

问题发生在于一个class object以其derived classes的某个object作为初值的情况下。

### copy assignment operator

是否是bitwise的情况和Copy Constructor类似

* 如何显式地拒绝将一个class object指定给另一个class object？

只要将copy assignment operator声明为private，并且不提供定义即可。声明为private，我们将不再允许于任何地方（除了在member function以及该class的friends之中）做赋值操作；不提供定义，则一旦某个member function或friend试图拷贝，程序链接将失败。

### destructor

应该因为“需要”而非“感觉”来提供destructor，更不要因为你不确定是否需要一个destructor，于是就提供它。

* 虚析构函数

基类中的destructor定义为virtual，保证派生类的destructor能被调用，当基类指针指向派生类的情况。



## 二. 内存模型

### linux下进程内存分布

![在这里插入图片描述](https://images.gitbook.cn/d0ee63e0-227a-11ea-9ddc-5367100c17b7)

- 程序段(代码段)：程序段为程序代码在内存中的映射，一个程序可以在内存中多有个副本。

- 全局（静态）存储区：程序中的全局变量和静态变量。

  分为DATA段和BSS段。DATA段（全局初始化区）存放初始化的全局变量和静态变量；BSS段（全局未初始化区）存放未初始化的全局变量和静态变量。程序运行结束时自动释放。其中BBS段在程序执行之前会被系统自动清0，所以未初始化的全局变量和静态变量在程序执行之前已经为0。

- 栈（stack）：用于存储局部变量、临时变量、函数的输入、输出参数和返回地址等，由程序自动分配和释放。

- 堆（heap）：C/C++ 动态分配的内存保存在此，需要在程序中显式分配、释放内存。

- 内存映射区：进程用到的动态库将被映射到此区域，用户通过 mmap() 函数创建的内存映射也映射到此区域。



### 结构体和类的内存布局

在 C++ 中，结构体和类只有一个区别：结构体默认的访问权限为 public，而类的访问默认访问权限为 private，二者在编译之后是完全相同的。

1. 派生类Derived类的虚表指针vptr（第一直接基类的vptr）
2. 第一直接基类的数据成员
3. 第二直接基类的虚表指针vptr
4. 第二直接基类的数据成员
5. 如果还有后继直接基类 那么依此类推
6. 派生类Derived类的成员变量

下面讨论带虚函数的继承情况。// ubuntu16.04 64bit， g++ v5.4.0

设置打印对象和虚函数表信息

```shell
(gdb) set print object on
(gdb) set print vtbl on
(gdb) set print pretty on
```



#### 菱形继承

```c++
class A {
    virtual void fun() {
    cout << "fun in A" << endl;
    }
    int ma=0;
};

class B: public A  {   //单继承
private:
    int m1=11;
public:
    virtual void vfun() {
        cout << "vfun in B" << endl;
    };
    virtual void vfunb(){
    cout << "vfun2 in B\n" << endl;
    }
};

class C: public A {
private:
    int m=12;

    public:
    virtual void vfun3() {};
};
class D: public B , public C{  //菱形继承
private:
    int m4=13;
public:
    virtual void vfun() {
        cout << "vfun in D" << endl;
    }
    virtual void fund() {
        cout << "vfun3 in D" << endl;
    }
};

// sizeof(A): 16
// sizeof(B); 16
// sizeof(C); 16
// sizeof(D); 40
// D d;  
/*
(gdb) p d
$1 = (D) {
  <B> = {
    <A> = {   //二义性
      _vptr.A = 0x400e08 <vtable for D+16>, 
      ma = 0
    }, 
    members of B: 
    m1 = 11
  }, 
  <C> = {
    <A> = {   //二义性
      _vptr.A = 0x400e38 <vtable for D+64>, 
      ma = 0
    }, 
    members of C: 
    m = 12
  }, 
  members of D: 
  m4 = 13
}
(gdb) info vtbl d
vtable for 'D' @ 0x400e08 (subobject @ 0x7fffffffe340):
[0]: 0x400b6c <A::fun()>
[1]: 0x400bfc <D::vfun()>
[2]: 0x400bc4 <B::vfunb()>
[3]: 0x400c28 <D::fund()>
vtable for 'C' @ 0x400e38 (subobject @ 0x7fffffffe350):
[0]: 0x400b6c <A::fun()>
[1]: 0x400bf0 <C::vfun3()>

(gdb) p b
$2 = (B) {
  <A> = {
    _vptr.A = 0x400e78 <vtable for B+16>, 
    ma = 0
  }, 
  members of B: 
  m1 = 11
}
(gdb) info vtbl b
vtable for 'B' @ 0x400e78 (subobject @ 0x7fffffffe320):
[0]: 0x400b6c <A::fun()>
[1]: 0x400b98 <B::vfun()>
[2]: 0x400bc4 <B::vfunb()>

(gdb) p c
$3 = (C) {
  <A> = {
    _vptr.A = 0x400e58 <vtable for C+16>, 
    ma = 0
  }, 
  members of C: 
  m = 12
}
(gdb) info vtbl c
vtable for 'C' @ 0x400e58 (subobject @ 0x7fffffffe330):
[0]: 0x400b6c <A::fun()>
[1]: 0x400bf0 <C::vfun3()>
*/
```



#### 菱形虚继承

```c++
class A {
    virtual void fun() {
    cout << "fun in A" << endl;
    }
    int ma=0;
};

class B: virtual public A  {    //单虚继承
private:
    int m1=11;
public:
    virtual void vfun() {
        cout << "vfun in B" << endl;
    };
    virtual void vfunb(){
    cout << "vfun2 in B\n" << endl;
    }
};

class C:virtual public A {
private:
    int m=12;

    public:
    virtual void vfun3() {};
};

class D: public B , public C{   //菱形继承
private:
    int m4=13;
public:
    virtual void vfun() {
        cout << "vfun in D" << endl;
    }
    virtual void fund() {
        cout << "fund in D" << endl;
    }
};
//D d; 
//sizeof(D); 48
/*
(gdb) p d
$1 = (D) {
  <B> = {
    <A> = {
      _vptr.A = 0x400ef0 <vtable for D+104>, 
      ma = 0
    }, 
    members of B: 
    _vptr.B = 0x400ea0 <vtable for D+24>, 
    m1 = 11
  }, 
  <C> = {
    members of C: 
    _vptr.C = 0x400ed0 <vtable for D+72>, 
    m = 12
  }, 
  members of D: 
  m4 = 13
}
(gdb) info vtbl d
vtable for 'D' @ 0x400ea0 (subobject @ 0x7fffffffe330):
[0]: 0x400b9c <D::vfun()>
[1]: 0x400b64 <B::vfunb()>
[2]: 0x400bc8 <D::fund()>
vtable for 'C' @ 0x400ed0 (subobject @ 0x7fffffffe340):
[0]: 0x400b90 <C::vfun3()>
vtable for 'A' @ 0x400ef0 (subobject @ 0x7fffffffe350):
[0]: 0x400b0c <A::fun()>

(gdb) p b
$2 = (B) {
  <A> = {
    _vptr.A = 0x401058 <vtable for B+64>, 
    ma = 0
  }, 
  members of B: 
  _vptr.B = 0x401030 <vtable for B+24>, 
  m1 = 11
}
(gdb) info vtbl b
vtable for 'B' @ 0x401030 (subobject @ 0x7fffffffe2f0):
[0]: 0x400b38 <B::vfun()>
[1]: 0x400b64 <B::vfunb()>
vtable for 'A' @ 0x401058 (subobject @ 0x7fffffffe300):
[0]: 0x400b0c <A::fun()>

(gdb) p c
$3 = (C) {
  <A> = {
    _vptr.A = 0x401000 <vtable for C+56>, 
    ma = 0
  }, 
  members of C: 
  _vptr.C = 0x400fe0 <vtable for C+24>, 
  m = 12
}
(gdb) info vtbl c
vtable for 'C' @ 0x400fe0 (subobject @ 0x7fffffffe310):
[0]: 0x400b90 <C::vfun3()>
vtable for 'A' @ 0x401000 (subobject @ 0x7fffffffe320):
[0]: 0x400b0c <A::fun()>

*/
```



#### 多重虚继承

```c++
class B   {
private:
    int m1=11;
public:
    virtual void vfun() {
        cout << "vfun in B" << endl;
    };
    virtual void vfunb(){
    cout << "vfun2 in B\n" << endl;
    }
};

class C  {
private:
    int m=12;

    public:
    virtual void vfun3() {};
};

class D:virtual public B ,virtual public C{
private:
    int m4=13;
public:
    virtual void vfun() {
        cout << "vfun in D" << endl;
    }
    virtual void fund() {
        cout << "vfun3 in D" << endl;
    }

};
/*
(gdb) p d
$1 = (D) {
  <B> = {
    _vptr.B = 0x400d90 <vtable for D+80>, 
    m1 = 11
  }, 
  <C> = {
    _vptr.C = 0x400db8 <vtable for D+120>, 
    m = 12
  }, 
  members of D: 
  _vptr.D = 0x400d60 <vtable for D+32>, 
  m4 = 13
}
(gdb) info vtbl d
vtable for 'D' @ 0x400d60 (subobject @ 0x7fffffffe330):
[0]: 0x400b70 <D::vfun()>
[1]: 0x400ba4 <D::fund()>
vtable for 'B' @ 0x400d90 (subobject @ 0x7fffffffe340):
[0]: 0x400b9b <virtual thunk to D::vfun()>
[1]: 0x400b38 <B::vfunb()>
vtable for 'C' @ 0x400db8 (subobject @ 0x7fffffffe350):
[0]: 0x400b64 <C::vfun3()>
*/
```

多重虚继承将保留多个基类虚函数表，派生类将拥有一个自己的虚表指针，如果定义了虚函数，则派生类生成一个自己的虚函数表；（否则派生类虚表指针未指向虚函数表）

### RAII和智能指针

RAII（Resource Acquisition Is Initialization）即“资源获取就是初始化”，智能指针是 RAII 思想的一种体现，它通过类的构造函数和析构函数来管理资源的申请和释放。使用智能指针可以大大减少内存泄漏的情况。

* unique_ptr
* share_ptr
* weak_ptr



## 三. 补充

### 静态数据成员&静态成员函数

1.静态数据成员

存储在全局数据区，属于类所有，由所有对象共享，不占用类对象的大小

没有this指针

2.静态成员函数

属于类所有，只能访问静态数据成员和静态成员函数

没有this指针

![img](https://img2018.cnblogs.com/blog/1053346/201909/1053346-20190918224853401-2122653082.png)

### 初始化

Effective C++ 条款4：确定对象被使用前已先被初始化

- 为内置型对象进行手工初始化，因为C++不保证初始化它们。局部内置变量不初始化，全局变量会初始化
- 构造函数最好使用成员初始化列表，而不要在构造函数本体内使用赋值操作。初始化列表列出的成员变量，其排列顺序应该和它们在class中声明的次序相同。
- 为免除“跨编译单元之初始化次序”的问题，请以local static对象代替non-local static对象。

