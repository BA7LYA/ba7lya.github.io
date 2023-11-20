---
title: "C++ Core Guidelines I.22 注解"
date: 2023-11-19T02:30:02+08:00
description: "C++ Core Guidelines I.22 注解"
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
  - C++ Best Practices
  - C++ Core Guidelines
comment: true
---

# I.22: Avoid complex initialization of global objects

>避免全局对象的复杂初始化。

## Reason

Complex initialization can lead to undefined order of execution.

>复杂的初始化可能导致未定义的执行顺序。

## Example

```c++
// file1.c
extern const X x;
const Y y = f(x);   // read x; write y

// file2.c
extern const Y y;
const X x = g(y);   // read y; write x
```

Since `x` and `y` are in different translation units the order of calls to `f()` and `g()` is undefined; one will access an uninitialized `const`. This shows that the order-of-initialization problem for global (namespace scope) objects is not limited to global variables.

> 由于`x`和`y`在不同的翻译单元，调用`f()`和`g()`的顺序是未定义的，他们中的其中一个将访问未初始化的`const`。
>
> 这表明全局（名称空间范围）对象的初始化顺序问题并不局限于全局变量。

## Note

Order of initialization problems become particularly difficult to handle in concurrent code. It is usually best to avoid global (namespace scope) objects altogether.

>在并发代码中，初始化顺序问题变得特别难以处理。通常最好完全避免全局（命名空间范围）对象。

## Enforcement

- Flag initializers of globals that call non-`constexpr` functions.
  - 检查调用非`constexpr`函数的全局变量的初始化式。
- Flag initializers of globals that access `extern` objects.
  - 检查访问`extern`对象的全局变量的初始化式。

## Reference

[1] [C++ Core Guidelines I.22: Avoid complex initialization of global objects](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i22-avoid-complex-initialization-of-global-objects)
