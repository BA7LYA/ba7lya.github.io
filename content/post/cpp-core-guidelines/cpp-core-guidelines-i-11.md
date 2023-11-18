---
title: "C++ Core Guidelines I.11 注解"
date: 2023-11-19T00:54:14+08:00
description: "C++ Core Guidelines I.11 注解"
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

# I.11: Never transfer ownership by a raw pointer (T*) or reference (T&)

>永远不要通过原始指针（`T*`）或引用（`T&`）转移所有权。

## Reason

If there is any doubt whether the caller or the callee owns an object, leaks or premature destruction will occur.

>如果对调用方或被调用方是否拥有对象有任何疑问，就会发生泄漏或过早销毁。

## Example

### Bad

Consider:

```c++
X* compute(args)    // don't
{
    X* res = new X{};
    // ...
    return res;
}
```

Who deletes the returned `X`? The problem would be harder to spot if `compute` returned a reference.

### Good

Consider returning the result by value (use move semantics if the result is large):

```c++
vector<double> compute(args)  // good
{
    vector<double> res(10000);
    // ...
    return res; // <= RVO std::move()
}
```

## Alternative

Pass ownership using a “smart pointer”, such as unique_ptr (for exclusive ownership) and shared_ptr (for shared ownership). However, that is less elegant and often less efficient than returning the object itself, so use smart pointers only if reference semantics are needed.

>使用“智能指针”传递所有权，例如`unique_ptr`（独占所有权）和`shared_ptr`（共享所有权）。
>
>然而，这样做不那么优雅，而且通常比返回对象本身效率低，所以**只有在需要引用语义时才使用智能指针**。

## Alternative

Sometimes older code can’t be modified because of ABI compatibility requirements or lack of resources.

>有时由于考虑到ABI兼容性或资源受限，而无法修改旧代码。

In that case, mark owning pointers using owner from the guidelines support library:

```c++
gsl::owner<X*> compute(args)    // It is now clear that ownership is transferred
{
    gsl::owner<X*> res = new X{};
    // ...
    return res;
}
```

This tells analysis tools that `res` is an owner. That is, its value must be `delete`d or transferred to another owner, as is done here by the `return`.

>这告诉分析工具`res`是一个所有者。也就是说，它的值必须被`delete`或转移给另一个所有者，就像这里的`return`所做的那样。

`owner` is used similarly in the implementation of resource handles.

>`owner`在资源句柄的实现中的使用与其类似。

## Note

Every object passed as a raw pointer (or iterator) is assumed to be owned by the caller, so that its lifetime is handled by the caller. Viewed another way: ownership transferring APIs are relatively rare compared to pointer-passing APIs, so the default is “no ownership transfer.”

>由原始指针（或迭代器）传递的每个对象都假定为调用者所有，因此其生命周期由调用者处理。
>
>从另一个角度来看：与指针传递API相比，所有权转移API相对较少，因此默认值是“无所有权转移”。

## Enforcement

- (Simple) Warn on `delete` of a raw pointer that is not an `owner<T>`. Suggest use of standard-library resource handle or use of `owner<T>`.

  - >对非`owner<T>`的原始指针的“删除”发出警告。
    >
    >建议使用标准库资源句柄或使用`owner<T>`。

- (Simple) Warn on failure to either `reset` or explicitly `delete` an `owner` pointer on every code path.

  - > 在每个代码路径上“重置”或显式`delete`一个`owner`指针失败时发出警告。

- (Simple) Warn if the return value of `new` or a function call with an `owner` return value is assigned to a raw pointer or non-`owner` reference.

  - >如果`new`或带有`owner`返回值的函数调用的返回值被分配给原始指针或非`owner`引用，则发出警告。

## Reference

[1] [C++ Core Guidelines I.11: Never transfer ownership by a raw pointer (`T*`) or reference (`T&`)](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i11-never-transfer-ownership-by-a-raw-pointer-t-or-reference-t)

[2] [C++Core Guidelines F.07: For general use, take `T*` or `T&` arguments rather than smart pointers](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f7-for-general-use-take-t-or-t-arguments-rather-than-smart-pointers)
