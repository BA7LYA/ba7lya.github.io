---
title: "C++ Core Guidelines F.22 注解"
date: 2023-12-22T21:54:13+08:00
description: "C++ Core Guidelines F.22 注解"
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

# F.22: Use `T*` or `gsl::owner<T*>` to designate a single object

## Reason

Readability: it makes the meaning of a plain pointer clear. Enables significant tool support.

可读性：它使普通指针的含义清晰。启用重要的工具支持。

## Note

In traditional C and C++ code, plain `T*` is used for many weakly-related purposes, such as:

- Identify a (single) object (not to be deleted by this function)
- Point to an object allocated on the free store (and delete it later)
- Hold the `nullptr`
- Identify a C-style string (zero-terminated array of characters)
- Identify an array with a length specified separately
- Identify a location in an array

This makes it hard to understand what the code does and is supposed to do. It complicates checking and tool support.

这使得很难理解代码做了什么和应该做什么。它使检查和工具支持变得复杂。

## Example

```c++
void use(int* p, int n, char* s, int* q) {
    // Bad:
    // we don't know if p points to n elements;
    // assume it does not or use std::span<int>
    p[n - 1] = 666;
    
    // Bad:
    // we don't know if that s points to a zero-terminated array of char;
    // assume it does not or use zstring
    std::cout << s;
    
    // Bad:
    // we don't know if *q is allocated on the free store;
    // assume it does not or use gsl::owner
    delete q;
}
```

better

```c++
void use2(std::span<int> p, zstring s, gsl::owner<int*> q) {
    p[p.size() - 1] = 666; // OK, a range error can be caught
    std::cout << s; // OK
    delete q;  // OK
}
```

## Note

`gsl::owner<T*>` represents ownership, `zstring` represents a C-style string.

## Also

Assume that a `T*` obtained from a smart pointer to `T` (e.g., `std::unique_ptr<T>`) points to a single element.

## See also

[Support library](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#gsl-guidelines-support-library)

[Do not pass an array as a single pointer](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ri-array)

## Enforcement

(Simple) ((Bounds)) Warn for any arithmetic operation on an expression of pointer type that results in a value of pointer type.
