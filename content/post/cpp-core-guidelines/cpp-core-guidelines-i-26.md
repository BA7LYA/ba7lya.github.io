---
title: "C++ Core Guidelines I.26 注解"
date: 2023-11-19T03:22:28+08:00
description: "C++ Core Guidelines I.26 注解"
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

# I.26: If you want a cross-compiler ABI, use a C-style subset

>如果需要跨编译器的ABI，请使用C风格的子集。

> :(

## Reason

Different compilers implement different binary layouts for classes, exception handling, function names, and other implementation details.

>不同的编译器为类、异常处理、函数名和其他实现细节实现不同的二进制布局。

## Exception

Common ABIs are emerging on some platforms freeing you from the more draconian restrictions.

>一些平台上出现了常见的ABI，将使用者从更严格的限制中解放出来。

## Note

If you use a single compiler, you can use full C++ in interfaces. That might require recompilation after an upgrade to a new compiler version.

>如果您使用单个编译器，则可以在接口中使用完整的C++。这可能需要在升级到新的编译器版本后重新编译。

## Enforcement

(Not enforceable) It is difficult to reliably identify where an interface forms part of an ABI.

>很难可靠地确定接口在哪里构成ABI的一部分。

## Reference

[1] [C++ Core Guidelines I.26: If you want a cross-compiler ABI, use a C-style subset](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i26-if-you-want-a-cross-compiler-abi-use-a-c-style-subset)
