---
layout: post
title: Effective c++ 学习笔记
categories: CPP
description: 
keywords: CPP
---

"Effective c++ 改善程序与设计的55个具体做法" by Scott Mayers（侯捷）

我自己总结的，Python 和 C++ 两门编程语言一个很大的区别：做一件事情，Python 可能有 a/b 两种做法，
a 是朴素直观的实现，但 b 是更 pythonic 的方案，推荐绝大多数情况下使用 b 方法。同样的事情，
 C++ 有 a/b/c/d/e/f/g 等方法，ab 是正确的方法，c 有内存溢出的隐患，defg 等是更高级别的用法。
不同的方法在不同的场景下各有优劣。如果水平有限，为了避免出错，就用最简单基础的方法 a 吧。（菜！）

期待能通过阅读 Effective C++ 了解更多 defg 的相关方法。以下是读书笔记以及结合工作之中的
一些总结。随时补充、整理、更新。
 
---
## ch1 让自己习惯 c++
### item 1 联邦语言

Cpp 的四个次语言：
- C
- Objected-Oriented C++ (C with Class)
- Template C++
- STL

目前来说，貌似可能对 124 还用的多一些，Template 目前用得极少，可能后面找机会多深入看下。

### item 2 尽量以 const enum inline 代替 define

文如其名。

### item 3 尽可能在需要的地方使用 const 

const 成员函数中，可以使用 mutable 来规避限制。

### item 4 确定对象被使用前初始化

声明时立即初始对象。

初始化和赋值语句的区别，例如：

```text
int a(20);
int a = 20;
int a; a = 20;
```

类初始化时，建议对类成员成员变量，采用成员初列值的方式，而不是赋值语句。
有时这样比较繁琐，对于内置类型，两种情况性能基本没差别，可以适当放松。（但依然建议这样做。）
对于复杂的类，成员初始值方式初始化，性能更高（调用类构造函数次数更少）。

## ch2 构造/析构/赋值运算

### item 5 了解 C++ 默默边写并调用类哪些函数

类内有 reference 作为成员变量时，不能使用默认的 copy assignment 操作。

## ch3 资源管理

## ch4 设计与声明

### item 18 让接口容易被正确使用，不易被误用

文如其名，注意接口参数正确设计，命名逻辑清晰。


### item 20 宁以 pass-by-reference-to-const 替换 pass-by-value

值️传递导致额外的性能开销，因为每次都会在内存中拷贝一个对应副本。使用 const refernce 会好很多。
例如 eigen 传递一个大矩阵，就应该使用后者。

当然，对于内置类型(例如 int 和 float)，没必要这样做。 

### item 21 必须返回对象时，别妄想返回 reference

返回类的 reference 的操作，很危险。如果在栈上生成对象，可能对象的生命周期结束了，但指针的
生命周期还未结束，从而出现指针悬空。如果在堆上生成对象(new)，那何时 del 又是一个棘手的问题。
也许可以考虑使用 shared_prt ?

直接返回一个类的实例，可能是一个更好的选择，虽然有少量的性能损耗。

当然另一个选择是，永远不返回对象，函数只返回简单的内置类型。可以外界生成对象，并讲地址传入。
不过有时会造成函数不够简练？例如

```text
Rational x;  // 定义有理数类，省略掉细节
Rational y;
Rational z;
bool multiply(const Rational& x, const Rational& y, Rational* z);  //  
Rational multiply(const Rational& x, const Rational& y);  // 看上去更符合 z = x * y 的形式
                                                          // 更简洁
```



### item 22 将成员变量声明为 private

文如起名。对类进行更好的封装。


### item 23 宁以 no-member，no-friend 替换 member 函数

使用 no-member 代替 member 函数，获得更好的扩展，减少公共接口数量。

不要将所有函数都堆在类里。

（可能和直观理解相反？）

### item 24 若所有参数均需类型转换，请为此采用 no-member 函数 

文如其名。需要构造函数使用 no explicit 模式。

## ch5 实现

### item 26 尽可能延后变量定义式的出现时间

文如其名。直接在构造时即赋值。

### item 27 尽量少做转型动作

- 尽量不要转型
- 如果一定需要，尽量用cpp风格的转型 xxx_cast
- 效率优先时，不要用 dynamic_cast
- 尽量将相关语句封装，不要暴露

### item 28 避免返回 handles 指向类内部变量

两个主要的缺点：

- 权限暴露太多，外部可以通过指针访问类内的底层数据（private）
- 容易指针悬空（对象被销毁了，指针没有）

### 29



---




