---
title: "C++ Core Guidelines F.54 注解"
date: 2023-12-22T21:56:15+08:00
description: "C++ Core Guidelines F.54 注解"
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

# F.54: When writing a lambda that captures `this` or any class data member, don’t use `[=]` default capture

## Reason

It’s confusing.

Writing `[=]` in a member function appears to capture by value, but actually captures data members by reference because it actually captures the invisible `this` pointer by value. If you meant to do that, write `this` explicitly.

在成员函数中写入`[=]`似乎是按值捕获，但实际上是通过引用捕获数据成员，因为它实际上是按值捕获不可见的`this`指针。

如果你打算这样做，明确地写`this`。

## Example

```c++
class My_class
{
    int x = 0;
    // ...

    void f()
    {
        int i = 0;
        // ...

        // BAD: "looks like" copy/value capture
        // [&] has identical semantics and copies the this pointer under the current rules
        // [=,this] and [&,this] are not much better, and confusing
        auto lambda = [=] { use(i, x); };

        x = 42;
        lambda(); // calls use(0, 42);
        x = 43;
        lambda(); // calls use(0, 43);

        // ...

        auto lambda2 = [i, this] { use(i, x); }; // ok, most explicit and least confusing

        // ...
    }
};
```

## Note

If you intend to capture a copy of all class data members, consider C++17 `[*this]`.

## Enforcement

Flag any lambda capture-list that specifies a capture-default of `[=]` and also captures `this` (whether explicitly or via the default capture and a use of `this` in the body)
