---
title: "C++ Core Guidelines P.13 注解"
date: 2023-11-18T02:24:41+08:00
description: "C++ Core Guidelines P.13 注解"
featured: true
draft: false
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

# P.13: Use support libraries as appropriate

>酌情使用支持库。

> 注：生产环境中尽量不要重复造轮子，优先使用成熟的库。

## Reason

Using a well-designed, well-documented, and well-supported library saves time and effort; its quality and documentation are likely to be greater than what you could do if the majority of your time must be spent on an implementation. The cost (time, effort, money, etc.) of a library can be shared over many users. A widely used library is more likely to be kept up-to-date and ported to new systems than an individual application. Knowledge of a widely-used library can save time on other/future projects. So, if a suitable library exists for your application domain, use it.

> 使用设计良好、文档完备、支持良好的库可以节省时间和精力，如果您必须将大部分时间花在实现上，那么它的质量和文档可能会比您所能做的要高。库的成本（时间、精力、金钱等）可以由许多用户共享。与单个应用程序相比，广泛使用的库更有可能保持最新并移植到新系统。了解广泛使用的库可以节省其他/未来项目的时间。因此，如果存在适合您的应用程序领域的库，请使用它。

> 注：开源万岁！

> 注：平时应注意了解、收集相关的库，评估库的质量，纳入自己的武器库中。

## Example

```c++
std::sort(begin(v), end(v), std::greater<>());
```

Unless you are an expert in sorting algorithms and have plenty of time, this is more likely to be correct and to run faster than anything you write for a specific application. You need a reason not to use the standard library (or whatever foundational libraries your application uses) rather than a reason to use it.

> 除非您是排序算法方面的专家，并且有足够的时间，否则这可能比您为特定应用程序编写的任何内容都更正确，运行速度更快。
>
> 你需要一个不使用标准库（或应用程序使用的任何基础库）的理由，而不是需要一个使用它的理由。

## Note

By default use

- The ISO C++ Standard Library
- The Guidelines Support Library<sup>[2]</sup>

## Note

If no well-designed, well-documented, and well-supported library exists for an important domain, maybe you should design and implement it, and then use it.

> 如果一个重要的领域没有设计良好、文档完备、支持良好的库，也许你应该设计并实现它，然后使用它。

[1] [C++ Core Guidelines P.13: Use support libraries as appropriate](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p13-use-support-libraries-as-appropriate)

[2] [GSL: Guidelines support library](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#gsl-guidelines-support-library)
