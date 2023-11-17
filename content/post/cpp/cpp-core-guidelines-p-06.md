---
title: "C++ Core Guidelines P.06 注解"
date: 2023-11-17T23:17:28+08:00
description: "C++ Core Guidelines P.06 注解"
featured: true
draft: false
toc: false
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
  - C++ Best Practice
  - C++ Core Guidelines
comment: true
---

# P.6: What cannot be checked at compile time should be checkable at run time

>在编译时不能检查的应该在运行时检查。

## Reason

Leaving hard-to-detect errors in a program is asking for crashes and bad results.

> 在程序中留下难以检测的错误会导致崩溃和糟糕的结果。

## Note

Ideally, we catch all errors (that are not errors in the programmer’s logic) at either compile time or run time. It is impossible to catch all errors at compile time and often not affordable to catch all remaining errors at run time. However, we should endeavor to write programs that in principle can be checked, given sufficient resources (analysis programs, run-time checks, machine resources, time).

> 理想情况下，我们在编译时或运行时捕获所有错误。
>
> 在编译时捕获所有错误是不可能的，在运行时捕获所有剩余的错误通常也负担不起。
>
> 然而，我们应该努力编写原则上可以被检查的程序，给予足够的资源（分析程序、运行时检查、机器资源、时间）。

## Example, bad

```c++
// separately compiled, possibly dynamically loaded
extern void f(int* p);

void g(int n)
{
    // bad: the number of elements is not passed to f()
    f(new int[n]);
}
```

Here, a crucial bit of information (the number of elements) has been so thoroughly “obscured” that static analysis is probably rendered infeasible and dynamic checking can be very difficult when `f()` is part of an ABI so that we cannot “instrument” that pointer. We could embed helpful information into the free store, but that requires global changes to a system and maybe to the compiler. What we have here is a design that makes error detection very hard.

> 在这里，关键的信息位（元素的数量）被完全“模糊”了，以至于当`f()`是ABI的一部分时，静态分析可能变得不可行，动态检查可能非常困难，因此我们无法检测该指针。我们可以将有用的信息嵌入到自由存储区（堆）中，但这需要对系统和编译器进行全局更改。这里的设计使得错误检测非常困难。

## Example, bad

We can of course pass the number of elements along with the pointer:

```c++
// separately compiled, possibly dynamically loaded
extern void f2(int* p, int n);

void g2(int n)
{
    f2(new int[n], m);  // bad: a wrong number of elements can be passed to f()
}
```

Passing the number of elements as an argument is better (and far more common) than just passing the pointer and relying on some (unstated) convention for knowing or discovering the number of elements. However (as shown), a simple typo can introduce a serious error. The connection between the two arguments of `f2()` is conventional, rather than explicit.

> 将元素数量作为参数传递比仅仅传递指针并依靠某种（未声明的）约定来知道或发现元素数量更好（也更常见）。
>
> 然而（如上所示），一个简单的打字错误可能会导致严重的错误。`f2()`的两个参数之间的连接是常规的，而不是显式的。

Also, it is implicit that `f2()` is supposed to `delete` its argument (or did the caller make a second mistake?).

> 此外，`f2()`应该`delete`它的参数是隐含的（或者调用者犯了第二个错误？）

## Example, bad

The standard library resource management pointers fail to pass the size when they point to an object:

```c++
// separately compiled, possibly dynamically loaded
// NB: this assumes the calling code is ABI-compatible, using a
// compatible C++ compiler and the same stdlib implementation
extern void f3(std::unique_ptr<int[]>, int n);

void g3(int n)
{
    f3(std::make_unique<int[]>(n), m);	// bad: pass ownership and size separately
}
```

## Example, good

We need to pass the pointer and the number of elements as an integral object:

> 我们需要将指针和元素数量作为一个整体对象进行传递。

```c++
extern void f4(vector<int>&);   // separately compiled, possibly dynamically loaded
extern void f4(span<int>);      // separately compiled, possibly dynamically loaded
                                // NB: this assumes the calling code is ABI-compatible, using a
                                // compatible C++ compiler and the same stdlib implementation

void g3(int n)
{
    vector<int> v(n);
    f4(v);                     // pass a reference, retain ownership
    f4(span<int>{v});          // pass a view, retain ownership
}
```

This design carries the number of elements along as an integral part of an object, so that errors are unlikely and dynamic (run-time) checking is always feasible, if not always affordable.

> 这种设计将元素数量作为对象的组成部分，因此不太可能出现错误，并且运行时检查总是可行的~~，如果不是总是负担得起的话~~。

## Example

How do we transfer both ownership and all information needed for validating use?

> 我们如何转移所有权和验证使用所需的所有信息？

```c++
// OK: move
std::vector<int> f5(int n) {
    std::vector<int> v(n);
    // ... initialize v ...
    return v; // <= RVO: implicit std::move()
}

// bad: loses n
std::unique_ptr<int[]> f6(int n) {
    auto p = std::make_unique<int[]>(n);
    // ... initialize *p ...
    return p; // <=  return p but without n
}

// bad: loses n and we might forget to delete
gsl::owner<int*> f7(int n) {
    gsl::owner<int*> p = new int[n];
    // ... initialize *p ...
    return p;
}
```

## Example

- ??? // TODO

- show how possible checks are avoided by interfaces that pass polymorphic base classes around, when they actually know what they need? Or strings as “free-style” options // TODO

  - > 说明传递多态基类的接口在实际上知道自己需要什么时，是如何避免可能的检查的？或者字符串作为“自由风格”选项。

## Enforcement

- Flag (pointer, count)-style interfaces (this will flag a lot of examples that can’t be fixed for compatibility reasons)

  - > 标志（指针，计数）风格的接口（这将标记许多由于兼容性原因无法修复的示例） // ???

- ??? // TODO

## Reference

[1] [C++ Core Guidelines P.6: What cannot be checked at compile time should be checkable at run time](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p6-what-cannot-be-checked-at-compile-time-should-be-checkable-at-run-time)
