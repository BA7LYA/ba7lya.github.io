---
title: "C++ Core Guidelines : Philosophy"
date: 2023-11-18T02:39:39+08:00
description: "C++ Core Guidelines : Philosophy"
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

# C++ Core Guidelines : Philosophy

The rules in this section are very general.

>本节的规则非常笼统。

Philosophy rules summary:

- P.01: Express ideas directly in code {{< ref "cpp-core-guidelines-p-01.md" >}}
- P.02: Write in ISO Standard C++
- P.03: Express intent
- P.04: Ideally, a program should be statically type safe
- P.05: Prefer compile-time checking to run-time checking
- P.06: What cannot be checked at compile time should be checkable at run time
- P.07: Catch run-time errors early
- P.08: Don’t leak any resources
- P.09: Don’t waste time or space
- P.10: Prefer immutable data to mutable data
- P.11: Encapsulate messy constructs, rather than spreading through the code
- P.12: Use supporting tools as appropriate
- P.13: Use support libraries as appropriate

Philosophical rules are generally not mechanically checkable. However, individual rules reflecting these philosophical themes are. Without a philosophical basis, the more concrete/specific/checkable rules lack rationale.

>哲学规则通常不能机械地检查。然而，反映这些哲学主题的个别规则可以。
>
>没有哲学基础，更具体的/具体的/可检查的规则会缺乏基本原理。
