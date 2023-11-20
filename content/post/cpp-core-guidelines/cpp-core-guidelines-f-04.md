---
title: "C++ Core Guidelines F.04 注解"
date: 2023-11-19T04:26:03+08:00
description: "C++ Core Guidelines F.04 注解"
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
  - C++ Best Practices
  - C++ Core Guidelines
comment: true
---

# F.04: If a function might have to be evaluated at compile time, declare it `constexpr`

> 如果一个函数可能需要在编译时求值，声明它为`constexpr`。

## Reason

`constexpr` is needed to tell the compiler to allow compile-time evaluation.

> `constexpr`告诉编译器允许编译时求值。

## Example

The (in)famous factorial:

```c++
constexpr int fac(int n)
{
    constexpr int max_exp = 17;      // constexpr enables max_exp to be used in Expects
    Expects(0 <= n && n < max_exp);  // prevent silliness and overflow
    int x = 1;
    for (int i = 2; i <= n; ++i) x *= i;
    return x;
}
```

This is C++14. For C++11, use a recursive formulation of `fac()`.

## Note

`constexpr` does not guarantee compile-time evaluation; it just guarantees that the function can be evaluated at compile time for constant expression arguments if the programmer requires it or the compiler decides to do so to optimize.

>`constexpr`不保证编译时求值，它只是保证，如果程序员需要或者编译器为了优化而决定这样做，可以在编译时对常量表达式参数求值函数。

```c++
constexpr int min(int x, int y) { return x < y ? x : y; }

void test(int v)
{
    int m1 = min(-1, 2);            // probably compile-time evaluation
    constexpr int m2 = min(-1, 2);  // compile-time evaluation
    int m3 = min(-1, v);            // run-time evaluation
    constexpr int m4 = min(-1, v);  // error: cannot evaluate at compile time
}
```

## Note

Don’t try to make all functions `constexpr`. Most computation is best done at run time.

## Note

Any API that might eventually depend on high-level run-time configuration or business logic should not be made `constexpr`. Such customization can not be evaluated by the compiler, and any `constexpr` functions that depended upon that API would have to be refactored or drop `constexpr`.

>任何可能最终依赖于高级运行时配置或业务逻辑的API都不应该被设置为`constexpr`。这种自定义不能被编译器评估，任何依赖于该API的`constexpr`函数都必须被重构或删除`constexpr`。

## Enforcement

Impossible and unnecessary. The compiler gives an error if a non-`constexpr` function is called where a constant is required.

>不可能也没有必要。如果在需要常量的地方调用非`constexpr`函数，编译器会给出一个错误。

## Reference

[1] [C++ Core Guidelines F.04: If a function might have to be evaluated at compile time, declare it `constexpr`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f4-if-a-function-might-have-to-be-evaluated-at-compile-time-declare-it-constexpr)
