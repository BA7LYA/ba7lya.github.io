---
title: "More C++ Idioms"
date: 2023-12-02T21:23:17+08:00
description: "More C++ Idioms"
featured: true
toc: false
usePageBundles: false
featureImage: ""
featureImageAlt: ''
featureImageCap: ''
thumbnail: "/images/iso_cpp_logo.png"
shareImage: "/images/iso_cpp_logo.png"
codeLineNumbers: true
figurePositionShow: true
categories:
  - C++
tags:
  - C++ Idioms
comment: true
---

## Preface

> C++ has indeed become too "expert friendly"
>
>  -- Bjarne Stroustrup, The Problem with Programming, Technology Review, Nov 2006.

Stroustrup's saying is true because experts are intimately familiar with the idioms in the language. With the increase in the idioms a programmer understands, the language becomes friendlier to them. The objective of this open content book is to present modern C++ idioms to programmers who have moderate level of familiarity with C++, and help elevate their knowledge so that C++ feels much friendlier to them. It is designed to be an exhaustive catalog of reusable idioms that expert C++ programmers often use while programming or designing using C++. This is an effort to capture their techniques and vocabulary into a single work. This book describes the idioms in a regular format: Name-Intent-Motivation-Solution-References, which is succinct and helps speed learning. By their nature, idioms tend to have appeared in the C++ community and in published work many times. An effort has been made to refer to the original source(s) where possible; if you find a reference incomplete or incorrect, please feel free to suggest or make improvements.

Stroustrup的说法是正确的，因为专家们非常熟悉语言中的习语。随着程序员理解的习语的增加，语言对他们来说变得更加友好。这本开放内容书的目的是向对C++有一定熟悉程度的程序员介绍现代C++惯用法，并帮助提高他们的知识水平，使C++对他们更友好。它被设计为C++编程专家在使用C++编程或设计时经常使用的可重用成语的详尽目录。这是将他们的技术和词汇融入到一个作品中的努力。这本书以常规的格式描述了习语：名称-意图-动机-解决方案-参考，这是简洁的，有助于加快学习。就其本质而言，惯用法往往在C++社区和已发表的作品中多次出现。在可能的情况下，已尽力参考原始来源，如果您发现参考资料不完整或不正确，请随时提出建议或进行改进。

The world is invited to catalog reusable pieces of C++ knowledge (similar to the book on design patterns by GoF). The goal here is to first build an exhaustive catalog of modern C++ idioms and later evolve it into an idiom language, just like a pattern language. Finally, the contents of this book can be redistributed under the terms of the GNU Free Documentation License.

我们邀请全世界对可重用的C++知识片段进行编目（类似于GoF关于设计模式的书）。这里的目标是首先构建现代C++惯用法的详尽目录，然后将其发展为习惯用法语言，就像模式语言一样。最后，本书的内容可以在GNU自由文档许可证的条款下重新发布。

Aimed toward: Anyone with an intermediate level of knowledge in C++ and supported language paradigms.

目标人群：任何具有中级C++知识和支持的语言范例的人。

## Table of Contents

[More C++ Idioms / Address Of]({{< ref "more-cpp-idioms-address-of.md" >}})

More C++ Idioms / Algebraic Hierarchy

More C++ Idioms / Attach by Initialization

More C++ Idioms / Barton-Nackman trick

More C++ Idioms / Base-from-Member

More C++ Idioms / Boost mutant

More C++ Idioms / Calling Virtuals During Initialization

More C++ Idioms / Capability Query

More C++ Idioms / Checked delete

More C++ Idioms / Clear-and-minimize

More C++ Idioms / Coercion by Member Template

More C++ Idioms / Compile Time Control Structures

More C++ Idioms / Computational Constructor

More C++ Idioms / Computational Constructor for Return Value Optimization

More C++ Idioms / Concrete Data Type

More C++ Idioms / Const auto ptr

More C++ Idioms / Construct On First Use

More C++ Idioms / Construction Tracker

More C++ Idioms / Contents

More C++ Idioms / Copy-and-swap

More C++ Idioms / Copy-on-write

More C++ Idioms / Counted Body

More C++ Idioms / Covariant Return Types

More C++ Idioms / Curiously Recurring Template Pattern

More C++ Idioms / Empty Base Optimization

More C++ Idioms / Enable if

More C++ Idioms / enable-if

More C++ Idioms / Erase-Remove

More C++ Idioms / Execute-Around Pointer

More C++ Idioms / Exploding Return Type

More C++ Idioms / Expression-template

More C++ Idioms / Fake Vtable

More C++ Idioms / Fast Pimpl

More C++ Idioms / Final Class

More C++ Idioms / Free Function Allocators

More C++ Idioms / Friendship and the Attorney-Client

More C++ Idioms / Function Poisoning

More C++ Idioms / Generic Container Idioms

More C++ Idioms / GNUFDL

More C++ Idioms / Guidelines

More C++ Idioms / Handle Body Hierarchy

More C++ Idioms / Hierarchy Generation

More C++ Idioms / Implicit conversions

More C++ Idioms / Include Guard Macro

More C++ Idioms / Inline Guard Macro

More C++ Idioms / Inner Class

More C++ Idioms / Int-To-Type

More C++ Idioms / Interface Class

More C++ Idioms / Iterator Pair

More C++ Idioms / Making New Friends

More C++ Idioms / Member Detector

More C++ Idioms / Metafunction

More C++ Idioms / Move Constructor

More C++ Idioms / Multi-statement Macro

More C++ Idioms / Named Constructor

More C++ Idioms / Named Loop

More C++ Idioms / Named Parameter

More C++ Idioms / Nifty Counter

More C++ Idioms / Non-copyable Mixin

More C++ Idioms / Non-throwing swap

More C++ Idioms / Non-Virtual Interface

More C++ Idioms / nullptr

More C++ Idioms / Object Generator

More C++ Idioms / Object Template

More C++ Idioms / Overload Set Creation

More C++ Idioms / Parameterized Base Class

More C++ Idioms / Policy Clone

More C++ Idioms / Polymorphic Exception

More C++ Idioms / Polymorphic Value Types

More C++ Idioms / Praise

More C++ Idioms / Preface

More C++ Idioms / Print Version

More C++ Idioms / Promotion Ladder

More C++ Idioms / Recursive Type Composition

More C++ Idioms / Requiring or Prohibiting Heap-based Objects

More C++ Idioms / Resource Acquisition Is Initialization

More C++ Idioms / Resource Return

More C++ Idioms / Return Type Resolver

More C++ Idioms / Runtime Static Initialization Order

More C++ Idioms / Runtime Static Initialization Order Idioms

More C++ Idioms / Safe bool

More C++ Idioms / Scope Guard

More C++ Idioms / SFINAE

More C++ Idioms / Shrink-to-fit

More C++ Idioms / Small Object Optimization

More C++ Idioms / Smart Pointer

More C++ Idioms / Storage Class Tracker

More C++ Idioms / Tag Dispatching

More C++ Idioms / Temporary Base Class

More C++ Idioms / Temporary Proxy

More C++ Idioms / Thin Template

More C++ Idioms / Thread-safe Copy-on-write

More C++ Idioms / Traits

More C++ Idioms / Type Erasure

More C++ Idioms / Type Generator

More C++ Idioms / Type Safe Enum

More C++ Idioms / Type Selection

More C++ Idioms / Virtual Constructor

More C++ Idioms / Virtual Friend Function
