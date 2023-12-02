---
title: "C++ Core Guidelines F.18 注解"
date: 2023-11-20T20:40:17+08:00
description: "C++ Core Guidelines F.18 注解"
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

# F.18: For “will-move-from” parameters, pass by `X&&` and `std::move` the parameter

>对于"will-move-from"类型的形参，通过`X&&`的形式传递参数和使用`std::move`移动参数。

## Reason

It’s efficient and eliminates bugs at the call site: `X&&` binds to rvalues, which requires an explicit `std::move` at the call site if passing an lvalue.

>这是有效的，并且消除了调用现场的bug：`X&&`绑定到右值，如果传入的是个左值，则需要在调用现场显式地使用`std::move`。

## Example

```c++
void sink(std::vector<int>&& v)  // sink takes ownership of whatever the argument owned
{
    // usually there might be const accesses of v here
    store_somewhere(std::move(v));
    // usually no more use of v here; it is moved-from
}
```

Note that the `std::move(v)` makes it possible for `store_somewhere()` to leave `v` in a moved-from state. That could be dangerous.

>注意，`std::move(v)`使得`store_somewhere()`使`v`可能处于已被移动的状态。这可能很危险。

## Exception

Unique owner types that are move-only and cheap-to-move, such as `unique_ptr`, can also be passed by value which is simpler to write and achieves the same effect. Passing by value does generate one extra (cheap) move operation, but prefer simplicity and clarity first.

>唯一的所有者类型是只能移动的，移动起来很便宜，比如`unique_ptr `，也可以通过值来传递，这样写起来更简单，也能达到同样的效果。按值传递确实会产生一个额外的（便宜的）移动操作，但更倾向于简单和清晰。

For example:

```c++
template<class T>
void sink(std::unique_ptr<T> p)
{
    // use p ... possibly std::move(p) onward somewhere else
}   // p gets destroyed
```

## Exception

If the “will-move-from” parameter is a `shared_ptr` follow R.34<sup>[2]</sup> and pass the `shared_ptr` by value.

>如果"will-move-from"参数是`shared_ptr`，请遵循R.34<sup>[2]</sup>并按值传递`shared_ptr`。

## Enforcement

- Flag all `X&&` parameters (where X is not a template type parameter name) where the function body uses them without `std::move`.

  - >标记函数体在没有`std::move`的情况下使用的所有`X&&`形参（其中`X`不是模板类型形参名）。

- Flag access to moved-from objects.

  - >标记对移出对象的访问。

- Don’t conditionally move from objects.

  - >不要有条件地移动对象。

## Reference

[1] [C++ Core Guidelines F.18: For “will-move-from” parameters, pass by `X&&` and `std::move` the parameter](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f18-for-will-move-from-parameters-pass-by-x-and-stdmove-the-parameter)

[2] [C++ Core Guidelines R.34: Take a `shared_ptr<widget>` parameter to express shared ownership](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-sharedptrparam-owner)
