---
title: "C++ Core Guidelines I.03 注解"
date: 2023-11-18T03:52:15+08:00
description: "C++ Core Guidelines I.03 注解"
featured: false
toc: true
usePageBundles: false
featureImage: "/images/cpp-core-guidelines.png"
featureImageAlt: 'C++ Core Guidelines'
featureImageCap: 'source: https://github.com/isocpp/CppCoreGuidelines'
thumbnail: "/images/iso_cpp_logo.png"
shareImage: "/images/iso_cpp_logo.png"
codeLineNumbers: true
figurePositionShow: true
categories:
  - C++
tags:
  - C++ Best Practices
  - C++ Core Guidelines
comment: true
---

# I.03: Avoid singletons

>避免单例。

> 注：
>
> 为什么单例模式这么不受待见？单例模式还是有它的适用场景的。
>
> 如果不用单例的话，用什么作为替代？

## Reason

Singletons are basically complicated global objects in disguise.

>单例对象基本上是伪装的复杂全局对象。

## Example

```c++
class Singleton {
    // ... lots of stuff to ensure that only one Singleton object is created,
    // that it is initialized properly, etc.
};
```

There are many variants of the singleton idea. That’s part of the problem.

>单例思想有许多变体。这是问题的一部分。

## Note

If you don’t want a global object to change, declare it `const` or `constexpr`.

>如果你不想让全局对象改变，就声明它为`const`或`constexpr`。

## Exception

You can use the simplest “singleton” (so simple that it is often not considered a singleton) to get initialization on first use, if any:

```c++
X& myX() {
    static X my_x {3};
    return my_x;
}
```

This is one of the most effective solutions to problems related to initialization order. In a multi-threaded environment, the initialization of the static object does not introduce a race condition (unless you carelessly access a shared object from within its constructor).

>这是与初始化顺序相关的问题的**最有效**解决方案之一。
>
>在多线程环境中，静态对象的初始化不会引入竞争条件（除非您不小心从其构造函数中访问共享对象）。

Note that the initialization of a local `static` does not imply a race condition. However, if the destruction of `X` involves an operation that needs to be synchronized we must use a less simple solution.

> 注意，初始化一个局部的`static`并不意味着存在竞争条件。
>
> 但是，如果销毁`X`涉及需要同步的操作，则必须使用不那么简单的解决方案。

For example:

```c++
X& myX() {
    static auto p = new X {3};
    return *p;  // potential leak
}
```

Now someone must `delete` that object in some suitably thread-safe way.

>现在必须有人以某种适当的线程安全的方式`delete`该对象。

That’s **error-prone**, so we don’t use that technique unless

- `myX` is in multi-threaded code,
- that `X` object needs to be destroyed (e.g., because it releases a resource), and
- `X`’s destructor’s code needs to be synchronized.

If you, as many do, define a singleton as a class for which only one object is created, functions like `myX` are not singletons, and this useful technique is not an exception to the no-singleton rule.

>如果你和许多人一样，将单例定义为只为其创建一个对象的类，那么像`myX`这样的函数就不是单例，而且这种有用的技术也不是非单例规则的例外。

## Enforcement

Very hard in general.

- Look for classes with names that include `singleton`.

  - > 检查名称中包含“singleton”的类。

- Look for classes for which only a single object is created (by counting objects or by examining constructors).

  - > 检查只为其创建一个对象的类（通过计数对象或构造函数）。

- If a class `X` has a public static function that contains a function-local `static` of the class’ type `X` and returns a pointer or reference to it, ban that.

  - > 如果类`X`有一个公共静态函数，该函数包含类类型`X`的函数局部`static`，并返回指向它的指针或引用，则禁止这样做。

## Reference

[1] [C++ Core Guidelines I.03: Avoid singletons](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i3-avoid-singletons)
