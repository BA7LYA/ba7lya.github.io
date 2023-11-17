---
title: "C++ Core Guidelines P.02 注解"
date: 2023-11-17T21:02:03+08:00
description: "C++ Core Guidelines P.02 注解"
featured: true
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

# P.2: Write in ISO Standard C++

## Reason

This is a set of guidelines for writing ISO Standard C++.

## Note

There are environments where extensions are necessary, e.g., to access system resources. In such cases, localize the use of necessary extensions and control their use with non-core Coding Guidelines. If possible, build interfaces that encapsulate the extensions so they can be turned off or compiled away on systems that do not support those extensions.

>有些环境需要扩展，例如访问系统资源。在这种情况下，本地化使用必要的扩展，并通过非核心编码指南控制它们的使用。
>
>如果可能的话，构建封装扩展的接口，以便在不支持这些扩展的系统上关闭或编译这些扩展。

Extensions often do not have rigorously defined semantics. Even extensions that are common and implemented by multiple compilers might have slightly different behaviors and edge case behavior as a direct result of not having a rigorous standard definition. With sufficient use of any such extension, expected portability will be impacted.

>扩展通常没有严格定义的语义。
>
>即使是由多个编译器实现的通用扩展，也可能由于没有严格的标准定义而具有稍微不同的行为和边缘情况行为。
>
>如果充分使用任何这样的扩展，预期的可移植性将受到影响。

## Note

Using valid ISO C++ does not guarantee portability (let alone correctness). Avoid dependence on undefined behavior (e.g., undefined order of evaluation) and be aware of constructs with implementation defined meaning (e.g., `sizeof(int)`).

> 使用有效的ISO C++并不能保证可移植性（更不用说正确性了）。
>
> 避免依赖于未定义的行为（例如，未定义的求值顺序），并注意具有实现定义含义的结构（例如，`sizeof(int)`）。

## Note

There are environments where restrictions on use of standard C++ language or library features are necessary, e.g., to avoid dynamic memory allocation as required by aircraft control software standards. In such cases, control their (dis)use with an extension of these Coding Guidelines customized to the specific environment.

> 在某些环境中，有必要限制使用标准C++语言或库功能，例如，为了避免飞机控制软件标准所要求的动态内存分配。
>
> 在这种情况下，使用针对特定环境定制的这些编码指南的扩展来控制它们的（不）使用。

## Enforcement

Use an up-to-date C++ compiler (currently C++20 or C++17) with a set of options that do not accept extensions.

> 使用带有一组不接受扩展的选项的最新C++编译器（当前为C++ 20或C++ 17）。

## Reference

[1] [C++ Core Guidelines P.02](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p2-write-in-iso-standard-c)
