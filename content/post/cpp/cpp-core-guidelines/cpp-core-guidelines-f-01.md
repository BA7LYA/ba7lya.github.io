---
title: "C++ Core Guidelines F.01 注解"
date: 2023-11-19T04:25:52+08:00
description: "C++ Core Guidelines F.01 注解"
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

# F.01: “Package” meaningful operations as carefully named functions

> 将有意义的操作“打包”为仔细命名的函数。

## Reason

Factoring out common code makes code more readable, more likely to be reused, and limit errors from complex code. If something is a well-specified action, separate it out from its surrounding code and give it a name.

>分解公共代码使代码更具可读性，更有可能被重用，并限制复杂代码中的错误。
>
>如果某件事是一个明确指定的动作，那么就把它从周围的代码中分离出来，并给它起一个名字。

## Example, don’t

```c++
void read_and_print(std::istream& is)    // read and print an int
{
    int x;
    if (is >> x)
        cout << "the int is " << x << '\n';
    else
        cerr << "no int on input\n";
}
```

Almost everything is wrong with `read_and_print`. It reads, it writes (to a fixed `ostream`), it writes error messages (to a fixed `ostream`), it handles only `int`s. There is nothing to reuse, logically separate operations are intermingled and local variables are in scope after the end of their logical use. For a tiny example, this looks OK, but if the input operation, the output operation, and the error handling had been more complicated the tangled mess could become hard to understand.

>`read_and_print`几乎所有内容都是错误的。
>
>它读，它写（到一个固定的`ostream`），它写（错误消息到一个固定的`ostream`），它只处理`int`。
>
>没有什么可重用的，逻辑上独立的操作被混合在一起，局部变量在其逻辑使用结束后仍在作用域中。
>
>对于一个小示例，这看起来不错，但是如果输入操作、输出操作和错误处理更加复杂，那么混乱的局面就会变得难以理解。

## Note

If you write a non-trivial lambda that potentially can be used in more than one place, give it a name by assigning it to a (usually non-local) variable.

>如果您编写了一个可能在多个地方使用的non-trivial lambda，请通过将其赋值给一个（通常是非局部的）变量来为其命名。

## Example

```c++
sort(a, b, [](T x, T y) { return x.rank() < y.rank() && x.value() < y.value(); });
```

Naming that lambda breaks up the expression into its logical parts and provides a strong hint to the meaning of the lambda.

>命名该lambda将表达式分解为其逻辑部分，并为lambda的含义提供了强烈的提示。

```c++
auto lessT = [](T x, T y) { return x.rank() < y.rank() && x.value() < y.value(); };

sort(a, b, lessT);
```

The shortest code is not always the best for performance or maintainability.

>在性能或可维护性方面，最短的代码并不总是最好的。

## Exception

Loop bodies, including lambdas used as loop bodies, rarely need to be named. However, large loop bodies (e.g., dozens of lines or dozens of pages) can be a problem. The rule Keep functions short and simple implies “Keep loop bodies short.” Similarly, lambdas used as callback arguments are sometimes non-trivial, yet unlikely to be reusable.

>循环体，包括用作循环体的lambdas，很少需要命名。
>
>然而，大的循环体（例如，几十行或几十页）可能是一个问题。保持函数简短<sup>[2]</sup>的规则意味着“保持循环体简短”。
>
>类似地，用作回调参数的lambda有时也很重要，但不太可能被重用。

## Enforcement

- See Keep functions short and simple<sup>[2]</sup>
- Flag identical and very similar lambdas used in different places.

## Reference

[1] [C++ Core Guidelines F.01: “Package” meaningful operations as carefully named functions](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f1-package-meaningful-operations-as-carefully-named-functions)

[2] [C++ Core Guidelines F.03: Keep functions short and simple](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f3-keep-functions-short-and-simple)
