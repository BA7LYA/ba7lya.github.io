---
title: "C++ Core Guidelines F.08 注解"
date: 2023-11-19T04:26:22+08:00
description: "C++ Core Guidelines F.08 注解"
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

# F.08: Prefer pure functions

>首选纯函数。

## Reason

Pure functions are easier to reason about, sometimes easier to optimize (and even parallelize), and sometimes can be memoized.

>纯函数更容易推理，有时更容易优化（甚至并行化），有时可以记忆。

## Example

```c++
template<class T>
auto square(T t) { return t * t; }
```

## Enforcement

Not possible.

## Reference

[1] [C++ Core Guidelines F.08: Prefer pure functions](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f8-prefer-pure-functions)
