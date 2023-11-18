---
title: "C++ Core Guidelines I.05 注解"
date: 2023-11-18T23:20:24+08:00
description: "C++ Core Guidelines I.05 注解"
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

# I.05: State preconditions (if any)

>说明前提条件（如果有的话）。

## Reason

Arguments have meaning that might constrain their proper use in the callee.

>实参的含义可能会限制它们在被调用对象中的正确使用。

## Example

### Bad

Consider:

```c++
double sqrt(double x);
```

### Better than bad

Here `x` must be non-negative.

The type system cannot (easily and naturally) express that, so we must use other means.

For example:

```c++
double sqrt(double x); // x must be non-negative
```

### Good

Some preconditions can be expressed as assertions. For example:

```c++
double sqrt(double x) {
    Expects(x >= 0);
    /* ... */
}
```

Ideally, that `Expects(x >= 0)` should be part of the interface of `sqrt()` but that’s not easily done. For now, we place it in the definition (function body).

## Note

Prefer a formal specification of requirements, such as `Expects(p);`.

If that is infeasible, use English text in comments, such as `// the sequence [p:q) is ordered using <`.

## Note

Most member functions have as a precondition that some class invariant holds. That invariant is established by a constructor and must be reestablished upon exit by every member function called from outside the class. We don’t need to mention it for each member function.

> 大多数成员函数都有一些类不变式作为先决条件。该不变量由构造函数建立，并且必须在退出时由从类外部调用的每个成员函数重新建立。我们不需要对每个成员函数都提到它。

> 注：？？？

## Enforcement

(Not enforceable)

## See also

The rules for passing pointers. /// ???

## Reference

[1] [C++ Core Guidelines I.05: State preconditions (if any)](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i5-state-preconditions-if-any)

[2] [C++ Core Guidelines GSL.assert: Assertions](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#gslassert-assertions)
