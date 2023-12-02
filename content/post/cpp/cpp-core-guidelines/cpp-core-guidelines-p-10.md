---
title: "C++ Core Guidelines P.10 注解"
date: 2023-11-18T01:05:34+08:00
description: "C++ Core Guidelines P.10 注解"
featured: false
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

# P.10: Prefer immutable data to mutable data

> 与可变数据相比，不可变数据更好。

> 注：尽可能多的使用`constexpr`。

## Reason

It is easier to reason about constants than about variables. Something immutable cannot change unexpectedly. Sometimes immutability enables better optimization. You can’t have a data race on a constant.

> 对常数的推理要比对变量的推理容易。
>
> 不可变的东西不会意外地改变。有时，不变性可以实现更好的优化。
>
> 常量上不会出现数据竞争。

## Reference

[1] [C++ Core Guidelines P.10: Prefer immutable data to mutable data](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p10-prefer-immutable-data-to-mutable-data)

[2] [C++Core Guidelines Con: Constants and immutability](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-const)
