---
title: "C++ Core Guidelines F.23 注解"
date: 2023-12-22T21:54:52+08:00
description: "C++ Core Guidelines F.23 注解"
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

# F.23: Use a `gsl::not_null<T>` to indicate that “null” is not a valid value

## Reason

Clarity.

A function with a `gsl::not_null<T>` parameter makes it clear that the caller of the function is responsible for any `nullptr` checks that might be necessary. Similarly, a function with a return value of `gsl::not_null<T>` makes it clear that the caller of the function does not need to check for `nullptr`.

带有`gsl::not_null<T>`形参的函数清楚地表明，函数的调用者负责可能需要的任何`nullptr`检查。

类似地，返回值为`gsl::not_null<T>`的函数明确表示该函数的调用者不需要检查`nullptr`。

## Example

`gsl::not_null<T*>` makes it obvious to a reader (human or machine) that a test for `nullptr` is not necessary before dereference. Additionally, when debugging, `owner<T*>` and `not_null<T>` can be instrumented to check for correctness.

`gsl::not_null<T*>`让读者（人类或机器）很明显，在解引用之前不需要对`nullptr`进行测试。

此外，在调试时，可以使用`gsl::owner<T*>`和`gsl::not_null<T>`来检查正确性。

Consider:

```c++
int length(Record* p);
```

When I call `length(p)` should I check if `p` is `nullptr` first?

Should the implementation of `length()` check if `p` is `nullptr`?

```c++
// it is the caller's job to make sure p != nullptr
int length(gsl::not_null<Record*> p);

// the implementor of length() must assume that p == nullptr is possible
int length(Record* p);
```

## Note

A `gsl::not_null<T*>` is assumed not to be the `nullptr`; a `T*` might be the `nullptr`; both can be represented in memory as a `T*` (so no run-time overhead is implied).

## Note

`gsl::not_null` is not just for built-in pointers.

It works for `std::unique_ptr`, `std::shared_ptr`, and other pointer-like types.

## Enforcement

- (Simple) Warn if a raw pointer is dereferenced without being tested against `nullptr` (or equivalent) within a function, suggest it is declared `gsl::not_null` instead.
- (Simple) Error if a raw pointer is sometimes dereferenced after first being tested against `nullptr` (or equivalent) within the function and sometimes is not.
- (Simple) Warn if a `gsl::not_null` pointer is tested against `nullptr` within a function.
