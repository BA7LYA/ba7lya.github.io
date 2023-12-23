---
title: "C++ Core Guidelines F.48 注解"
date: 2023-12-22T21:55:44+08:00
description: "C++ Core Guidelines F.48 注解"
featured: false
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

# F.48: Don’t `return std::move(local)`

## Reason

With guaranteed copy elision, it is now almost always a pessimization to expressly use `std::move` in a return statement.

有了保证的拷贝消除，现在在返回语句中明确使用`std::move`几乎总是一种最差化的做法。

## Example, bad

```c++
S f()　{
  S result;
  return std::move(result);
}
```

## Example, good

```c++
S f() {
  S result;
  return result; // RVO will do `std::move` for you
}
```

## Enforcement

This should be enforced by tooling by checking the return expression.
