---
title: "C++ Core Guidelines F.05 注解"
date: 2023-11-19T04:26:06+08:00
description: "C++ Core Guidelines F.05 注解"
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

# F.5: If a function is very small and time-critical, declare it `inline`

>如果函数非常小且时序要求严格，则将它声明为为“内联”（inline）函数。

## Reason

Some optimizers are good at inlining without hints from the programmer, but don’t rely on it. Measure! Over the last 40 years or so, we have been promised compilers that can inline better than humans without hints from humans. We are still waiting. Specifying inline (explicitly, or implicitly when writing member functions inside a class definition) encourages the compiler to do a better job.

>有些优化器在没有程序员提示的情况下擅长内联，但不要依赖于它。测量！
>
>在过去40年左右的时间里，我们一直被承诺编译器可以在没有人类提示的情况下比人类更好地内联。我们还在等待。
>
>指定`inline`（在类定义内编写成员函数时显式地或隐式地）可以鼓励编译器做得更好。

## Example

```c++
inline std::string cat(const string& s, const string& s2) { return s + s2; }
```

## Exception

Do not put an `inline` function in what is meant to be a stable interface unless you are certain that it will not change. An `inline` function is part of the ABI.

>不要把“内联”函数放在一个稳定的接口中，除非你确定它不会改变。
>
>内联函数是ABI的一部分。

## Note

`constexpr` implies `inline`.

> `constexpr`意味着`inline`。

## Note

Member functions defined in-class are `inline` by default.

>默认情况下，在类中定义的成员函数是`inline`。

## Exception

Function templates (including member functions of class templates `A<T>::function()` and member function templates `A::function<T>()`) are normally defined in headers and therefore inline.

> 函数模板（包括类模板`A<T>::Function`的成员函数和成员函数模板`A::Function<T>()`）通常在头文件中定义，因此是内联的。

## Enforcement

Flag `inline` functions that are more than three statements and could have been declared out of line (such as class member functions).

> 标记有问题的“内联”函数：这些函数超过3条语句，并且可以在行外声明（例如类成员函数）。

## Reference

[1] [C++ Core Guidelines F.05: If a function is very small and time-critical, declare it `inline`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f5-if-a-function-is-very-small-and-time-critical-declare-it-inline)
