---
title: "C++ Core Guidelines F.24 注解"
date: 2023-12-22T21:55:10+08:00
description: "C++ Core Guidelines F.24 注解"
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

# F.24: Use a `std::span<T>` or a `std::span_p<T>` to designate a half-open sequence

## Reason

Informal/non-explicit ranges are a source of errors.

非正式/非显式范围是错误的来源。

## Example

```c++
X* find(std::span<X> r, const X& v);    // find v in r

std::vector<X> vec;
// ...
auto p = find({vec.begin(), vec.end()}, X{});  // find X{} in vec
```

## Note

Ranges are extremely common in C++ code. Typically, they are implicit and their correct use is very hard to ensure. In particular, given a pair of arguments `(p, n)` designating an array `[p:p+n)`, it is in general impossible to know if there really are `n` elements to access following `*p`. `std::span<T>` and `std::span_p<T>` are simple helper classes designating a `[p:q)` range and a range starting with `p` and ending with the first element for which a predicate is true, respectively.

范围在C++代码中非常常见。通常，它们是隐式的，很难保证它们的正确使用。

特别是，给定一对参数`(p, n)`指定一个数组`[p:p+n]`，通常不可能知道`*p`后面是否真的有`n`个元素可供访问。

`std::span<T>`和`std::span_p<T>`是简单的辅助类，它们分别指定一个`[p:q)`范围和一个从`p`开始到谓词为真的第一个元素结束的范围。

## Example

A `std::span` represents a range of elements, but how do we manipulate elements of that range?

```c++
void f(span<int> s) {
    // range traversal (guaranteed correct)
    for (int x : s) {
        std::cout << x << '\n';
    }

    // C-style traversal (potentially checked)
    for (gsl::index i = 0; i < s.size(); ++i) {
        cout << s[i] << '\n';
    }

    // random access (potentially checked)
    s[7] = 9;

    // extract pointers (potentially checked)
    std::sort(&s[0], &s[s.size() / 2]);
}
```

## Note

A `std::span<T>` object does not own its elements and is so small that it can be passed by value.

`std::span<T>`对象不拥有自己的元素，因为它很小，所以可以按值传递。

Passing a `std::span` object as an argument is exactly as efficient as passing a pair of pointer arguments or passing a pointer and an integer count.

传递`std::span`对象作为参数与传递一对指针参数或传递一个指针和一个整数计数完全相同。

## See also

[Support library](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#gsl-guidelines-support-library)

## Enforcement

(Complex) Warn where accesses to pointer parameters are bounded by other parameters that are integral types and suggest they could use `span` instead.
