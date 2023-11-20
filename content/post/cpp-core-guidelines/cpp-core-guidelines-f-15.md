---
title: "C++ Core Guidelines F.15 注解"
date: 2023-11-20T20:39:59+08:00
description: "C++ Core Guidelines F.15 注解"
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

# F.15: Prefer simple and conventional ways of passing information

## Reason

Using “unusual and clever” techniques causes surprises, slows understanding by other programmers, and encourages bugs. If you really feel the need for an optimization beyond the common techniques, measure to ensure that it really is an improvement, and document/comment because the improvement might not be portable.

>使用“不寻常和聪明”的技术会导致意外，减慢其他程序员的理解速度，并助长bug。
>
>如果您确实觉得有必要在常用技术之外进行优化，请确保它确实是一种改进，并记录/注释，因为改进可能不可移植。

The following tables summarize the advice in the following Guidelines, F.16-21.

Normal parameter passing:

![](/images/param-passing-normal.png)

Advanced parameter passing:

![](/images/param-passing-advanced.png)

> 注：参数传递，以这张图为参考基本就足够了。

Use the advanced techniques only after demonstrating need, and document that need in a comment.

For passing sequences of characters see String<sup>[2]</sup>.

## Exception

To express shared ownership using `shared_ptr` types, rather than following guidelines F.16-21, follow R.34<sup>[3]</sup>, R.35<sup>[4]</sup>, and R.36<sup>[5]</sup>.

## Reference

[1] [C++ Core Guidelines F.15: Prefer simple and conventional ways of passing information](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f15-prefer-simple-and-conventional-ways-of-passing-information)

[2] [C++ Core Guidelines SL.str: String](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#slstr-string)

[3] [C++ Core Guidelines R.34: Take a `shared_ptr<widget>` parameter to express shared ownership](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-sharedptrparam-owner)

[4] [C++ Core Guidelines R.35: Take a `shared_ptr<widget>&` parameter to express that a function might reseat the shared pointer](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-sharedptrparam)

[5] [C++ Core Guidelines R.36: Take a `const shared_ptr<widget>&` parameter to express that it might retain a reference count to the object ???](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-sharedptrparam-const)
