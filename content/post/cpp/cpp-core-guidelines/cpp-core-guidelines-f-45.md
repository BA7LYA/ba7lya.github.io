---
title: "C++ Core Guidelines F.45 注解"
date: 2023-12-22T21:55:37+08:00
description: "C++ Core Guidelines F.45 注解"
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

# F.45: Don’t return a `T&&`

## Reason

It’s asking to return a reference to a destroyed temporary object. A `&&` is a magnet for temporary objects.

它要求返回一个被销毁的临时对象的引用。`&&`是吸引临时对象的磁铁。

## Example

A returned rvalue reference goes out of scope at the end of the full expression to which it is returned:

```c++
auto&& x = max(0, 1);   // OK, so far
foo(x);                 // Undefined behavior
```

This kind of use is a frequent source of bugs, often incorrectly reported as a compiler bug. An implementer of a function should avoid setting such traps for users.

这种用法是bug的一个常见来源，经常被错误地报告为编译器bug。函数的实现者应该避免为用户设置这样的陷阱。

The [lifetime safety profile](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-lifetime) will (when completely implemented) catch such problems.

## Example

Returning an rvalue reference is fine when the reference to the temporary is being passed “downward” to a callee; then, the temporary is guaranteed to outlive the function call (see [F.18](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-consume) and [F.19](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-forward)). However, it’s not fine when passing such a reference “upward” to a larger caller scope. For passthrough functions that pass in parameters (by ordinary reference or by perfect forwarding) and want to return values, use simple `auto` return type deduction (not `auto&&`).

当对临时对象的引用“向下”传递给被调用者时，返回一个右值引用是可以的，这会保证临时函数的寿命超过函数调用。

然而，当将这样的引用“向上”传递给更大的调用者作用域时，就不太好了。

对于传递参数（通过普通引用或完美转发）并希望返回值的直通函数，使用简单的`auto`返回类型推导（而不是`auto&&`）。

Assume that `F` returns by value:

```c++
template<class F>
auto&& wrapper(F f)
{
    log_call(typeid(f)); // or whatever instrumentation
    return f();          // BAD: returns a reference to a temporary
}
```

Better:

```c++
template<class F>
auto wrapper(F f)        // auto <=(change to)= auto&&
{
    log_call(typeid(f)); // or whatever instrumentation
    return f();          // OK
}
```

## Exception

`std::move` and `std::forward` do return `&&`, but they are just casts – used by convention only in expression contexts where a reference to a temporary object is passed along within the same expression before the temporary is destroyed. We don’t know of any other good examples of returning `&&`.

`std::move`和`std::forward`确实返回`&&`，但它们只是强制类型转换——按照约定仅在表达式上下文中使用，在临时对象被销毁之前，对临时对象的引用在同一表达式中传递。

我们不知道还有其他返回`&&`的好例子。

## Enforcement

Flag any use of `&&` as a return type, except in `std::move` and `std::forward`.
