---
title: "C++ Core Guidelines F.25 注解"
date: 2023-12-22T21:55:12+08:00
description: "C++ Core Guidelines F.25 注解"
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

# F.25: Use a `gsl::zstring` or a `gsl::not_null<zstring>` to designate a C-style string

## Reason

C-style strings are ubiquitous. They are defined by convention: zero-terminated arrays of characters. We must distinguish C-style strings from a pointer to a single character or an old-fashioned pointer to an array of characters.

C风格的字符串无处不在。它们是按照约定定义的：以零结尾的字符数组。

我们必须将C风格的字符串与指向单个字符的指针或指向字符数组的老式指针区分开来。

If you don’t need null termination, use `std::string_view`.

## Example

Consider:

```c++
int length(const char* p);
```

When I call `length(s)` should I check if `s` is `nullptr` first? Should the implementation of `length()` check if `p` is `nullptr`?

```c++
// the implementor of length() must assume that p == nullptr is possible
int length(gsl::zstring p);

// it is the caller's job to make sure p != nullptr
int length(gsl::not_null<zstring> p);
```

## Note

`gsl::zstring` does not represent ownership.

## See also

[Support library](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#gsl-guidelines-support-library)
