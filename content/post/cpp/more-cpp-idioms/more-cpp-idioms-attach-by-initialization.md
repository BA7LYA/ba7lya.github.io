---
title: "More C++ Idioms - Attach by Initialization"
date: 2023-12-05T13:15:35+08:00
description: "More C++ Idioms - Attach by Initialization"
featured: false
toc: false
usePageBundles: false
featureImage: ""
featureImageAlt: ''
featureImageCap: ''
thumbnail: "/images/iso_cpp_logo.png"
shareImage: "/images/iso_cpp_logo.png"
codeLineNumbers: true
figurePositionShow: true
categories:
  - C++
tags:
  - C++ Idioms
comment: true
---

## Intent

Attach a user-defined object to a framework before program execution begins.

在程序开始执行之前，将用户定义的对象附加到框架上。

## Also Known As

Static-object-with-constructor

## Motivation

Certain application programming frameworks, such as GUI frameworks (e.g., Microsoft MFC) and object request brokers (e.g., some CORBA implementations) use their own internal message loops (a.k.a. event loops) to control the entire application. Application programmers may or may not have the freedom to write the application-level main function. Often, the main function is buried deep inside the application framework (e.g, `AfxWinMain` in case of MFC). Lack of access to main keeps programmers from writing application-specific initialization code before the main event loop begins. The attach-by-initialization idiom is a way of executing application-specific code before the execution of a framework-controlled loop begins.

某些应用程序编程框架，如GUI框架（例如，Microsoft MFC）和对象请求代理（例如，一些CORBA实现）使用它们自己的内部消息循环（又名事件循环）来控制整个应用程序。应用程序程序员可能有也可能没有编写应用程序级主函数的自由。通常，`main`函数隐藏在应用程序框架的深处（例如，在MFC的情况下，`AfxWinMain`）。缺乏对`main`的访问使程序员无法在`main`事件循环开始之前编写特定于应用程序的初始化代码。初始化附加惯用法是在框架控制的循环开始执行之前执行特定于应用程序的代码的一种方式。

## Solution and Sample Code

In C++, global objects and static objects in the global namespace are initialized before main begins. These objects are also known as objects of static storage duration. This property of objects of static storage duration can be used to attach an object to a system if programmers are not allowed to write their own main function.

在C++中，全局命名空间中的全局对象和静态对象在`main`开始之前被初始化。这些对象也称为静态存储持续时间对象。

如果不允许程序员编写自己的`main`函数，则可以使用静态存储时间对象的这个属性将对象附加到系统中。

For instance, consider the following (smallest possible) example using Microsoft Foundation Classes (MFC):

```c++
///// File = Hello.h
class HelloApp: public CWinApp
{
public:
    virtual BOOL InitInstance ();
};

///// File = Hello.cpp
#include <afxwin.h>
#include "Hello.h"
HelloApp myApp; // Global "application" object
BOOL HelloApp::InitInstance ()
{
    m_pMainWnd = new CFrameWnd();
    m_pMainWnd->Create(0,"Hello, World!!");
    m_pMainWnd->ShowWindow(SW_SHOW);
    return TRUE;
}
```

The above example creates a window titled "Hello, World!" and nothing more. The key thing to note here is the global object myApp of type HelloApp. The myApp object is default-initialized before the execution of main. As a side-effect of initializing the objects, the constructor of CWinApp is also called. The CWinApp class is a part of the framework and invokes constructors of several other classes in the framework. During the execution of these constructors, the global object is attached to the framework. The object is later on retrieved by AfxWinMain, which is the MFC equivalent of the regular main. The HelloApp::InitInstance member function is shown just for the sake of completeness and is not an integral part of the idiom. This function is called after AfxWinMain begins execution.

上面的示例创建了一个标题为“Hello, World!”的窗口，仅此而已。这里需要注意的关键是`HelloApp`类型的全局对象`myApp`。

`myApp`对象在`main`函数执行之前被默认初始化。作为初始化对象的副作用，`CWinApp`的构造函数也被调用。`CWinApp`类是框架的一部分，并调用框架中其他几个类的构造函数。在执行这些构造函数期间，全局对象被附加到框架上。该对象稍后由`AfxWinMain`检索，它是常规`main`的MFC等效。

显示`HelloApp::InitInstance`成员函数只是为了完整起见，它并不是惯用法的组成部分。这个函数在`AfxWinMain`开始执行后被调用。

Global and static objects can be initialized in several ways: default constructor, constructors with parameters, assignment from a return value of a function, dynamic initialization, etc.

全局和静态对象可以通过几种方式初始化：默认构造函数、带参数的构造函数、从函数返回值赋值、动态初始化等。

### Caveat

In C++, objects in the same compilation unit are created in order of definition. However, the order of initialization of objects of static storage duration across different compilation units is not well defined. Objects in a namespace are created before any function/variable in that namespace is accessed. This may or may not be before main. Order of destruction is the reverse of initialization order but the initialization order itself is not standardized. Because of this undefined behavior, the static initialization order problem comes up when a constructor of a static object uses another static object that has not been initialized yet. This idiom makes it easy to run into this trap because it depends on an object of static storage duration.

在Ｃ++中，同一编译单元中的对象是按定义顺序创建的。但是，跨不同编译单元的静态存储时间对象的初始化顺序没有很好地定义。

命名空间中的对象在访问该名称空间中的任何函数/变量之前创建。这可能在`main`之前，也可能不在`main`之前。销毁顺序与初始化顺序相反，但初始化顺序本身并不是标准化的。由于这种**未定义的行为**，当静态对象的构造函数使用另一个尚未初始化的静态对象时，就会出现静态初始化顺序问题。这种惯用法很容易陷入这种陷阱，因为它依赖于静态存储持续时间的对象。

## Known Uses

Microsoft Foundation Classes (MFC)

## Related Idioms



## References

[1] [Proposed C++ language extension to improve portability of the Attach by Initialization idiom](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/1995/N0717.htm)
