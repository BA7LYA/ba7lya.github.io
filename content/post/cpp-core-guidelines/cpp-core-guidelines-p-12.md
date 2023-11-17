---
title: "C++ Core Guidelines P.12 注解"
date: 2023-11-18T02:01:04+08:00
description: "C++ Core Guidelines P.12 注解"
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

# P.12: Use supporting tools as appropriate

>适当地使用辅助工具。

## Reason

There are many things that are done better “by machine”. Computers don’t tire or get bored by repetitive tasks. We typically have better things to do than repeatedly do routine tasks.

>有很多事情用机器可以做得更好。电脑不会对重复的任务感到疲劳或厌烦。
>
>我们通常有更好的事情要做，而不是重复做常规任务。

## Example

Run a static analyzer to verify that your code follows the guidelines you want it to follow.

>运行一个静态分析器来验证您的代码是否遵循了您希望它遵循的准则。

> 注：
>
> | short name | name                         | intro                                                        |
> | ---------- | ---------------------------- | ------------------------------------------------------------ |
> | ASAN       | Address Sanitizer            | 检测堆内存的越界和悬空指针的访问<br/>检测栈和全局对象的越界访问 |
> | TSAN       | Thread Sanitizer             | 检测数据竞争错误                                             |
> | MSAN       | Memory Sanitizer             | 检查内存泄漏和其他内存错误                                   |
> | UBSAN      | Undefined Behavior Sanitizer | 检测未定义行为                                               |

## Note

There are many other kinds of tools, such as source code repositories, build tools, etc., but those are beyond the scope of these guidelines.

> 还有许多其他类型的工具，如源代码存储库、构建工具等，但这些都超出了本指南的范围。

> 注：例如Git、CMake全家桶等。

## Note

Be careful not to become dependent on over-elaborate or over-specialized tool chains. Those can make your otherwise portable code non-portable.

> 注意不要依赖于过于复杂或过于专门化的工具链。这些会使原本可移植的代码不可移植。

> 注：优先使用跨平台的工具，例如VSCode、Vim、CMake，可以避免重复的学习时间成本和非必要的心智负担。

## Reference

[1] [C++ Core Guidelines P.12: Use supporting tools as appropriate](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p12-use-supporting-tools-as-appropriate)
