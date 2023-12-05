---
title: "More C++ Idioms - Attorney Client"
date: 2023-12-05T13:28:41+08:00
description: "More C++ Idioms - Attorney Client"
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

Control the granularity of access to the implementation details of a class.

控制对类的实现细节的访问粒度。

## Motivation

A `friend` declaration in C++ gives complete access to the internals of a class. Friend declarations are, therefore, frowned upon because they break carefully crafted encapsulation. The friendship feature of C++ does not provide any way to selectively grant access to a subset of private members of a class. Friendship in C++ is an all-or-nothing proposition.

C++中的友元声明提供了对类内部的完全访问。因此，友元声明是不受欢迎的，因为它们破坏了精心设计的封装。

C++的`friend`特性没有提供任何方法来选择性地授予对类的私有成员子集的访问权。C++中的`friend`是一个要么全有要么全无的命题。

For instance, the following class Foo declares class Bar its friend. Class Bar therefore has access to all the private members of class Foo. This may not be desirable because it increases coupling. Class Bar cannot be distributed without class Foo.

例如，下面的类`Foo`声明类`Bar`是它的友类。因此类`Bar`可以访问类`Foo`的所有私有成员。

这可能是不可取的，因为它增加了耦合。类`Bar`不能在没有类`Foo`的情况下分发。

```c++
class Foo 
{
private:
    void A(int a);
    void B(float b);
    void C(double c);
    friend class Bar;
};

class Bar {
    // This class needs access to Foo::A and Foo::B only.
    // C++ friendship rules, however, give access to all the private members of Foo.
};
```

Providing selective access to a subset of members is desirable because the remaining private members can change interface if needed without breaking client code. It helps reduce coupling between classes. The Attorney-Client idiom allows a class to precisely control the amount of access they give to their friends.

提供对成员子集的选择性访问是可取的，因为剩余的私有成员可以在不破坏客户端代码的情况下更改接口。它有助于减少类之间的耦合。

“律师-客户”惯用法允许类精确地控制它们给予友元的访问量。

## Solution and Sample Code

The Attorney-client idiom works by adding a level of indirection. A *client* class that wants to control access to its internal details, appoints an *attorney* and makes it a `friend`. The `Attorney` class is carefully crafted to serve as a proxy to the `Client`. Unlike a typical proxy class, `Attorney` replicates only a subset of `Client`’s private interface. For instance, consider class `Client` wants to control access to its implementation details. `Client` wants its `Attorney` to provide access to `Client::A` and `Client::B` only.

""律师-客户"惯用法"的工作原理是增加一层间接性。

一个“客户”类，它想要控制对内部细节的访问，指定一个“律师”，并把它变成一个“友元”。

“律师”类被精心设计为“客户”的代理。与典型的代理类不同，“代理”只复制“客户”私有接口的一个子集。

例如，考虑类“Client”想要控制对其实现细节的访问。“客户”希望其“律师”只允许访问`Client::A`和`Client::B`。

```c++
class Client 
{
private:
  void A(int a);
  void B(float b);
  void C(double c);
  friend class Attorney;
};

class Attorney {
private:
    static void callA(Client & c, int a) {
    	c.A(a);
    } 
    static void callB(Client & c, float b) {
    	c.B(b);
    }
    friend class Bar;
};

class Bar {
	// Bar now has access to only Client::A and Client::B through the Attorney.
};
```

The `Attorney` class restricts access to a cohesive set of functions. All methods of class `Attorney` are inline static, each taking a reference to an instance of the `Client` and forwarding the function calls to it. Some things about `Attorney` are idiomatic. Its implementation is entirely private, which prevents other unexpected classes gaining access to the internals of `Client`. `Attorney` determines which other classes, member functions, or free functions get access to it. It declares them as `friend` to allow access to its implementation and through it `Client`. Without `Attorney`, `Client` would have declared the same set of friends giving them unrestrained access to the internals of `Client`.

“律师”类限制了对一组内聚函数的访问。“律师”类的所有方法都是内联静态的，每个方法都引用一个“客户”类的实例，并将函数调用转发给它。关于“律师”的一些事情是惯用语。它的实现是完全私有的，这可以防止其他意想不到的类访问“客户”的内部。“代理”决定哪些其他类、成员函数或自由函数可以访问它。它将它们声明为“朋友”以允许访问它的实现，并通过它声明为“客户端”。如果没有“律师”，“客户”就会声明相同的一组朋友，让他们无限制地访问“客户”的内部。

It is possible to have multiple attorney classes providing access to different sets of implementation details of the client. For instance, class `AttorneyC` may provide access to the `Client::C` method only. An interesting case emerges where an attorney class serves as a mediator for several different classes and provides cohesive access to their implementation details. Such a design is conceivable in case of inheritance hierarchies because friendship in C++ is not inheritable, but private virtual function overrides in derived classes can be called if base's private virtual functions are accessible. In the following example, the Attorney-Client idiom is applied to class `Base` and the `main` function. `Derived::Func` gets called via polymorphism. To access the implementation details of `Derived`, however, the same idiom may be applied.

可以有多个代理类，提供对客户端不同实现细节集的访问。例如，类`AttorneyC`可能只提供对`Client::C`方法的访问。出现了一个有趣的案例，其中律师类充当几个不同类的中介，并提供对其实现细节的内聚访问。这种设计在继承层次结构的情况下是可以想象的，因为C++中的友元是不可继承的，但是如果基类的私有虚函数是可访问的，派生类中的私有虚函数重写可以被调用。

在下面的例子中，代理-客户端习语应用于类`Base`和`main`函数。`Derived::Func`通过多态性被调用。然而，要访问`Derived`的实现细节，可以使用相同的惯用法。

```c++
#include <iostream>

class Base {
    private:
        virtual void Func(int x) = 0;
        friend class Attorney;
    public:
    	virtual ~Base() {}
};

class Derived : public Base {
private:
    virtual void Func(int x)  {
        // This is called even though main is not a friend of Derived.
        cout << "Derived::Func" << endl;
    }

public:
	~Derived() {}
};

class Attorney {
private:
    static void callFunc(Base & b, int x) {
    	return b.Func(x);
    }
    friend int main();
};

int main() {
    Derived d;
    Attorney::callFunc(d, 10);
}
```

## Known Uses

Boost.Iterators library

Boost.Serialization: `class boost::serialization::access`

## Related Idioms



## References

[1] [Bolton, Alan R. (01 January 2006). "Friendship and the Attorney-Client Idiom - Dr. Dobb's"](http://drdobbs.com/184402053)
