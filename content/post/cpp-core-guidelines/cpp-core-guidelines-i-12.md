---
title: "C++ Core Guidelines I.12 注解"
date: 2023-11-19T01:15:02+08:00
description: "C++ Core Guidelines I.12 注解"
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

# I.12: Declare a pointer that must not be null as `not_null`

> 声明一个不能为空的指针为`not_null`。

## Reason

To help avoid dereferencing `nullptr` errors. To improve performance by avoiding redundant checks for `nullptr`.

>帮助避免解引用`nullptr`错误。通过避免对`nullptr`进行冗余检查来提高性能。

## Example

```c++
int length(const char* p);            // it is not clear whether length(nullptr) is valid

length(nullptr);                      // OK?

int length(not_null<const char*> p);  // better: we can assume that p cannot be nullptr

int length(const char* p);            // we must assume that p can be nullptr
```

By stating the intent in source, implementers and tools can provide better diagnostics, such as finding some classes of errors through static analysis, and perform optimizations, such as removing branches and null tests.

>通过在源代码中声明意图，实现者和工具可以提供更好的诊断，例如通过静态分析找到一些错误类，并执行优化，例如删除分支和空测试。

## Note

`not_null` is defined in the guidelines support library.

## Note

The assumption that the pointer to `char` pointed to a C-style string (a zero-terminated string of characters) was still implicit, and a potential source of confusion and errors. Use `czstring` in preference to `const char*`.

> 指向`char`的指针可以指向c风格的字符串（以零结尾的字符串），这其中的转换仍然是隐式的，并且是混淆和错误的潜在来源。
>
> 使用`czstring`优先于`const char*`。

```c++
// we can assume that p cannot be nullptr
// we can assume that p points to a zero-terminated array of characters
int length(not_null<czstring> p);
```

Note: `length()` is, of course, `std::strlen()` in disguise.

## Enforcement

- (Simple) ((Foundation)) If a function checks a pointer parameter against `nullptr` before access, on all control-flow paths, then warn it should be declared `not_null`.

  - > 如果一个函数在访问某个它的某个参数之前在所有控制流路径上检查这个参数是否为`nullptr`，那么应该将这个参数声明为`not_null`。

- (Complex) If a function with pointer return value ensures it is not `nullptr` on all return paths, then warn the return type should be declared `not_null`.

  - > 如果一个带有指针返回值的函数确保在所有返回路径上它不是`nullptr`，那么其返回类型应该声明为`not_null`。


## Reference

[1] [C++ Core Guidelines I.12: Declare a pointer that must not be null as `not_null`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i12-declare-a-pointer-that-must-not-be-null-as-not_null)
