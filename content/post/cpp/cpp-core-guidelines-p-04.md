---
title: "C++ Core Guidelines P.04 注解"
date: 2023-11-17T22:42:06+08:00
description: "C++ Core Guidelines P.04 注解"
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
  - Best Practice
comment: true
---

# P.4: Ideally, a program should be statically type safe

## Reason

Ideally, a program would be completely statically (compile-time) type safe.

> 理想情况下，程序应该是完全静态（编译时）类型安全的。

Unfortunately, that is not possible.

> 不幸的是，这是不可能的。

Problem areas:

- unions
- casts
- array decay<sup>[2]</sup> // 数组退化
- range errors
- narrowing conversions

## Note

These areas are sources of serious problems (e.g., crashes and security violations). We try to provide alternative techniques.

> 这些区域是严重问题的来源（例如，崩溃和安全违规）。我们试图提供替代技术。

## Enforcement

We can ban, restrain, or detect the individual problem categories separately, as required and feasible for individual programs. Always suggest an alternative.

> 我们可以根据个别程序的需要和可行性，分别禁止、限制或检测个别问题类别。总是提出一个替代方案。

For example:

- unions – use `std::variant` (in C++17)
- casts – minimize their use; templates can help // demo ???
- array decay – use `span` (from the GSL)
- range errors – use `span`
- narrowing conversions – minimize their use and use `narrow` or `narrow_cast` (from the GSL) where they are necessary

## Reference

[1] [C++ Core Guidelines P.4: Ideally, a program should be statically type safe](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p4-ideally-a-program-should-be-statically-type-safe)

[2] [What is array-to-pointer conversion aka. decay?](https://stackoverflow.com/questions/1461432/what-is-array-to-pointer-conversion-aka-decay)
