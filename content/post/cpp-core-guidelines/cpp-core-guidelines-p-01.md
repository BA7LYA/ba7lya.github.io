---
title: "C++ Core Guidelines P.01 注解"
date: 2023-11-17T02:24:12+08:00
description: "C++ Core Guidelines P.01 注解"
featured: true
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
  - C++ Best Practice
  - C++ Core Guidelines
comment: true
---

# P.01: Express ideas directly in code

> 直接用代码表达想法（，而不是用注释或者让接口使用者去猜）。

## Reason

Compilers don’t read comments (or design documents) and neither do many programmers (consistently).

> 编译器不读注释（或设计文档），许多程序员也不读注释（一贯如此）。

What is expressed in code has defined semantics and can (in principle) be checked by compilers and other tools.

> 代码中表达的内容具有定义的语义，并且（原则上）可以由编译器和其他工具进行检查。

## Example

```c++
class Date {
public:
    Month month() const;  // do
    int month();          // don't
    // ...
};
```

The first declaration of `month` is explicit about returning a `Month` and about not modifying the state of the `Date` object.

The second version leaves the reader guessing and opens more possibilities for uncaught bugs.

> 注：
>
> 具体如何实现呢？可以用`using`或者`typedef`，推荐优先使用`using`。例如：
>
> ```c++
> class Date {
> public:
>     using Month = uint8_t;
> public:
>     Month month() const;
> };
> ```

## Example

### Bad

This loop is a restricted form of `std::find`:

```c++
void f(std::vector<std::string>& v) // 注：在一个字符串vector中寻找特定的字符串
{
    std::string val;
    std::cin >> val;
    // ...
    int index = -1;                    // bad, plus should use gsl::index
    for (int i = 0; i < v.size(); ++i) {
        if (v[i] == val) {
            index = i;
            break;
        }
    }
    // ...
}
```

### Good

A much clearer expression of intent would be:

```c++
void f(std::vector<std::string>& v)
{
    std::string val;
    std::cin >> val;
    // ...
    auto p = std::find(std::begin(v), std::end(v), val);  // better
    // ...
}
```

> 注：在新项目中尽可能的用C++的新标准中的语法，更简洁易懂易维护。
>

A well-designed library expresses intent (what is to be done, rather than just how something is being done) far better than direct use of language features.

> 设计良好的库比直接使用语言特性更好地表达意图（要做什么，而不仅仅是如何做某事）。

A C++ programmer should know the basics of the standard library, and use it where appropriate. Any programmer should know the basics of the foundation libraries of the project being worked on, and use them appropriately. Any programmer using these guidelines should know the guidelines support library, and use it appropriately.

> C++程序员应该了解标准库的基础知识，并在适当的地方使用它。
>
> 任何程序员都应该了解正在处理的项目的基础库的基本知识，并适当地使用它们。
>
> 任何使用这些指南的程序员都应该知道Guidelines Support Library（GSL），并适当地使用它。

## Example

### Bad

```c++
change_speed(double s);	// bad: what does s signify?
// ...
change_speed(2.3);		// error: no unit
```

### Good

A better approach is to be explicit about the meaning of the double (new speed or delta on old speed?) and the unit used:

```c++
change_speed(Speed s);    // better: the meaning of s is specified
// ...
change_speed(23_m / 10s);  // meters per second
```

We could have accepted a plain (unit-less) `double` as a delta, but that would have been error-prone. If we wanted both absolute speed and deltas, we would have defined a `Delta` type.

> 注：
>
> `10s`为`std::chrono`定义的时间字面量，`23_m`涉及到用户自定义字面量<sup>[2]</sup>。此处可以如此定义：
>
> ```c++
> constexpr unsigned long long operator''_m(unsigned long long distance_in_m) {
>     return distance_in_m;
> }
> ```

## Enforcement

Very hard in general.

- use `const` consistently
  - check if member functions modify their object;
  - check if functions modify arguments passed by pointer or reference
- flag uses of casts (casts neuter the type system) // ???
- detect code that mimics the standard library (hard)

> 注：尽量使用标准库，而不是自己手撸。即不要重复造轮子，优先使用成熟的框架或者库。

## Reference

[1] [C++ Core Guidelines P.01](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p1-express-ideas-directly-in-code)

[2] [User-defined literals](https://en.cppreference.com/w/cpp/language/user_literal)
