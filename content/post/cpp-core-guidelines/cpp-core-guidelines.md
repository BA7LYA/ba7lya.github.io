---
title: "C++ Core Guidelines"
date: 2023-11-18T02:39:39+08:00
description: "C++ Core Guidelines"
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

# C++ Core Guidelines

> 注：以下的“我”、“我们”的视角为C++ Core Guidelines的Editor，包括Bjarne Stroustrup和Herb Sutter。

## Abstract

This document is a set of guidelines for using C++ well. The aim of this document is to help people to use modern C++ effectively. By “modern C++” we mean effective use of the ISO C++ standard (currently C++20, but almost all of our recommendations also apply to C++17, C++14 and C++11). In other words, what would you like your code to look like in 5 years’ time, given that you can start now? In 10 years’ time?

>本文档是一套使用C++的指南。
>
>本文档的目的是帮助人们有效地使用现代C++。所谓“现代C++”，我们指的是有效地使用ISO C++标准（目前是C++ 20，但我们几乎所有的建议也适用于C++ 17、C++ 14和C++ 11）。换句话说，如果现在可以开始，您希望5年后的代码是什么样子？10年后呢？

The guidelines are focused on relatively high-level issues, such as interfaces, resource management, memory management, and concurrency. Such rules affect application architecture and library design. Following the rules will lead to code that is statically type safe, has no resource leaks, and catches many more programming logic errors than is common in code today. And it will run fast – you can afford to do things right.

>这些指导方针关注于相对高级的问题，例如接口、资源管理、内存管理和并发性。这些规则影响应用程序体系结构和库设计。
>
>遵循这些规则将产生静态类型安全的代码，没有资源泄漏，并且捕获比现在代码中常见的更多的编程逻辑错误。
>
>而且它会跑得很快——你可以很容易地把事情做对。

We are less concerned with low-level issues, such as naming conventions and indentation style. However, no topic that can help a programmer is out of bounds.

>我们不太关心低级别的问题，比如命名约定和缩进样式。然而，任何可以帮助程序员的话题都是可以接受的。

Our initial set of rules emphasizes safety (of various forms) and simplicity. They might very well be too strict. We expect to have to introduce more exceptions to better accommodate real-world needs. We also need more rules.

>我们最初的一套规则强调（各种形式的）安全性和简单性。他们可能太严格了。
>
>我们希望引入更多的异常来更好地适应现实世界的需求。
>
>我们还需要更多的规则。

You will find some of the rules contrary to your expectations or even contrary to your experience. If we haven’t suggested you change your coding style in any way, we have failed! Please try to verify or disprove rules! In particular, we’d really like to have some of our rules backed up with measurements or better examples.

>你会发现一些规则与你的期望相反，甚至与你的经验相反。
>
>如果我们没有建议你以任何方式改变你的编码风格，我们就失败了！
>
>请尝试验证或反驳规则！特别是，我们真的希望我们的一些规则有测量或更好的例子来支持。

You will find some of the rules obvious or even trivial. Please remember that one purpose of a guideline is to help someone who is less experienced or coming from a different background or language to get up to speed.

>你会发现一些规则是显而易见的，甚至是微不足道的。
>
>请记住，指南的一个目的是帮助那些经验不足或来自不同背景或语言的人跟上速度。

Many of the rules are designed to be supported by an analysis tool. Violations of rules will be flagged with references (or links) to the relevant rule. We do not expect you to memorize all the rules before trying to write code. One way of thinking about these guidelines is as a specification for tools that happens to be readable by humans.

>许多规则被设计为由分析工具支持。违反规则的行为将被标记为相关规则的引用（或链接）。
>
>我们不期望您在尝试编写代码之前记住所有的规则。考虑这些指导方针的一种方式是将其视为人类可读的工具规范。

The rules are meant for gradual introduction into a code base. We plan to build tools for that and hope others will too.

>这些规则旨在逐步引入代码库。我们计划为此开发工具，并希望其他人也能这样做。

Comments and suggestions for improvements are most welcome. We plan to modify and extend this document as our understanding improves and the language and the set of available libraries improve.

>欢迎对改进提出意见和建议。我们计划随着我们理解的提高以及语言和可用库集的改进而修改和扩展此文档。

## Philosophy

The rules in this section are very general.

>本节的规则非常笼统。

Philosophy rules summary:

- [P.01: Express ideas directly in code]({{< ref "cpp-core-guidelines-p-01.md" >}})
- [P.02: Write in ISO Standard C++]({{< ref "cpp-core-guidelines-p-02.md" >}})
- [P.03: Express intent]({{< ref "cpp-core-guidelines-p-03.md" >}})
- [P.04: Ideally, a program should be statically type safe]({{< ref "cpp-core-guidelines-p-04.md" >}})
- [P.05: Prefer compile-time checking to run-time checking]({{< ref "cpp-core-guidelines-p-05.md" >}})
- [P.06: What cannot be checked at compile time should be checkable at run time]({{< ref "cpp-core-guidelines-p-06.md" >}})
- [P.07: Catch run-time errors early]({{< ref "cpp-core-guidelines-p-07.md" >}})
- [P.08: Don’t leak any resources]({{< ref "cpp-core-guidelines-p-08.md" >}})
- [P.09: Don’t waste time or space]({{< ref "cpp-core-guidelines-p-09.md" >}})
- [P.10: Prefer immutable data to mutable data]({{< ref "cpp-core-guidelines-p-10.md" >}})
- [P.11: Encapsulate messy constructs, rather than spreading through the code]({{< ref "cpp-core-guidelines-p-11.md" >}})
- [P.12: Use supporting tools as appropriate]({{< ref "cpp-core-guidelines-p-12.md" >}})
- [P.13: Use support libraries as appropriate]({{< ref "cpp-core-guidelines-p-13.md" >}})

Philosophical rules are generally not mechanically checkable. However, individual rules reflecting these philosophical themes are. Without a philosophical basis, the more concrete/specific/checkable rules lack rationale.

>哲学规则通常不能机械地检查。然而，反映这些哲学主题的个别规则可以。
>
>没有哲学基础，更具体的/具体的/可检查的规则会缺乏基本原理。

## Interfaces

An interface is a contract between two parts of a program. Precisely stating what is expected of a supplier of a service and a user of that service is essential. Having good (easy-to-understand, encouraging efficient use, not error-prone, supporting testing, etc.) interfaces is probably the most important single aspect of code organization.

>接口是程序两个部分之间的契约。准确地说明对服务提供者和服务用户的期望是至关重要的。
>
>拥有良好的接口（易于理解、鼓励有效使用、不容易出错、支持测试等）可能是代码组织中最重要的一个方面。

Interface rule summary:

- [I.01: Make interfaces explicit]({{< ref "cpp-core-guidelines-i-01.md" >}})
- [I.02: Avoid non-`const` global variables]({{< ref "cpp-core-guidelines-i-02.md" >}})
- [I.03: Avoid singletons]({{< ref "cpp-core-guidelines-i-03.md" >}})
- [I.04: Make interfaces precisely and strongly typed]({{< ref "cpp-core-guidelines-i-04.md" >}})
- [I.05: State preconditions (if any)]({{< ref "cpp-core-guidelines-i-05.md" >}})
- [I.06: Prefer `Expects()` for expressing preconditions]({{< ref "cpp-core-guidelines-i-06.md" >}})
- [I.07: State postconditions]({{< ref "cpp-core-guidelines-i-07.md" >}})
- [I.08: Prefer `Ensures()` for expressing postconditions]({{< ref "cpp-core-guidelines-i-08.md" >}})
- [I.09: If an interface is a template, document its parameters using concepts]({{< ref "cpp-core-guidelines-i-09.md" >}})
- [I.10: Use exceptions to signal a failure to perform a required task]({{< ref "cpp-core-guidelines-i-10.md" >}})
- [I.11: Never transfer ownership by a raw pointer (`T*`) or reference (`T&`)]({{< ref "cpp-core-guidelines-i-11.md" >}})
- [I.12: Declare a pointer that must not be null as `not_null`]({{< ref "cpp-core-guidelines-i-12.md" >}})
- I.13: Do not pass an array as a single pointer
- I.22: Avoid complex initialization of global objects
- I.23: Keep the number of function arguments low
- I.24: Avoid adjacent parameters that can be invoked by the same arguments in either order with different meaning
- I.25: Prefer empty abstract classes as interfaces to class hierarchies
- I.26: If you want a cross-compiler ABI, use a C-style subset
- I.27: For stable library ABI, consider the Pimpl idiom
- I.30: Encapsulate rule violations
