---
title: "C++ Core Guidelines F.07 注解"
date: 2023-11-19T04:26:19+08:00
description: "C++ Core Guidelines F.07 注解"
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

# F.07: For general use, take `T*` or `T&` arguments rather than smart pointers

>对于一般用途，使用`T*`或`T&`参数而不是智能指针。

## Reason

Passing a smart pointer transfers or shares ownership and should only be used when ownership semantics are intended. A function that does not manipulate lifetime should take raw pointers or references instead.

>传递智能指针转移或共享所有权，仅在需要所有权语义时使用。
>
>不操作生命周期的函数应该接受原始指针或引用。

Passing by smart pointer restricts the use of a function to callers that use smart pointers. A function that needs a `widget` should be able to accept any `widget` object, not just ones whose lifetimes are managed by a particular kind of smart pointer.

>传递智能指针将函数的使用限制为使用智能指针的调用者。
>
>需要`widget`的函数应该能够接受任何`widget`对象，而不仅仅是那些生命周期由特定类型的智能指针管理的对象。

Passing a shared smart pointer (e.g., `std::shared_ptr`) implies a run-time cost.

>传递共享智能指针（例如，`std::shared_ptr`）意味着运行时成本。

## Example

```c++
// accepts any int*
void f(int*);

// can only accept ints for which you want to transfer ownership
void g(std::unique_ptr<int>);

// can only accept ints for which you are willing to share ownership
void g(std::shared_ptr<int>);

// doesn't change ownership, but requires a particular ownership of the caller
void h(const std::unique_ptr<int>&);

// accepts any int
void h(int&);
```

## Example

### Bad

```c++
// callee
void f(std::shared_ptr<widget>& w)
{
    // ...
    use(*w); // only use of w -- the lifetime is not used at all
    // ...
};

// caller
std::shared_ptr<widget> my_widget = /* ... */;
f(my_widget);

widget stack_widget;
f(stack_widget); // error
```

### Good

```c++
// callee
void f(widget& w)
{
    // ...
    use(w);
    // ...
};

// caller
std::shared_ptr<widget> my_widget = /* ... */;
f(*my_widget);

widget stack_widget;
f(stack_widget); // ok -- now this works
```

## Note

We can catch many common cases of dangling pointers statically (see lifetime safety profile<sup>[4]</sup>). Function arguments naturally live for the lifetime of the function call, and so have fewer lifetime problems.

>我们可以静态地捕获许多常见的悬空指针。函数参数在函数调用的生命周期内自然存在，因此生命周期问题较少。

## Enforcement

- (Simple) Warn if a function takes a parameter of a smart pointer type (that overloads `operator->` or `operator*`) that is copyable but the function only calls any of: `operator*`, `operator->` or `get()`. Suggest using a `T*` or `T&` instead.

  - >如果函数接受可复制的智能指针类型的参数（重载`operator->`或`operator*`），但函数只调用`operator*`，`operator->`或`get()`中的任何一个，则发出警告。建议用`T*`或`T&`代替。

- Flag a parameter of a smart pointer type (a type that overloads `operator->` or `operator*`) that is copyable/movable but never copied/moved from in the function body, and that is never modified, and that is not passed along to another function that could do so. That means the ownership semantics are not used. Suggest using a `T*` or `T&` instead.

  - > 标记智能指针类型（重载`operator->`或`operator*`的类型）的参数，该参数可复制/可移动，但从未在函数体中复制/移动，并且永远不会修改，也不会传递给可以这样做的另一个函数。这意味着没有使用所有权语义。建议用`T*`或`T&`代替。

## Reference

[1] [C++ Core Guidelines F.07: For general use, take `T*` or `T&` arguments rather than smart pointers](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f7-for-general-use-take-t-or-t-arguments-rather-than-smart-pointers)

[2] [C++ Core Guidelines F.60: Prefer `T*` over `T&` when “no argument” is a valid option](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-ptr-ref)

[3] [C++ Core Guidelines R: Smart pointer rule summary](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-summary-smartptrs)

[4] [C++ Core Guidelines Pro.lifetime: Lifetime safety profile](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-lifetime)
