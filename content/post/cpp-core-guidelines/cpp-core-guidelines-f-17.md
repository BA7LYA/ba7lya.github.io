---
title: "C++ Core Guidelines F.17 注解"
date: 2023-11-20T20:40:14+08:00
description: "C++ Core Guidelines F.17 注解"
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

# F.17: For “in-out” parameters, pass by reference to non-`const`

> 对于"in-out"形参，通过非`const`的引用的方式（`X&`）传递。

## Reason

This makes it clear to callers that the object is assumed to be modified.

>这使调用者清楚地知道对象被假定为已被修改。

## Example

```c++
void update(Record& r);  // assume that update writes to r
```

## Note

Some user-defined and standard library types, such as `span<T>` or the iterators are cheap to copy and may be passed by value, while doing so has mutable (in-out) reference semantics:

>一些用户定义的和标准库类型，如`span<T>`或迭代器，复制起来成本很低，可以按值传递，而这样做具有易变（in-out）引用语义。

```c++
void increment_all(span<int> a) {
  for (auto&& e : a) {
    ++e;
  }
}
```

> 注：完整的demo？

## Note

A `T&` argument can pass information into a function as well as out of it. Thus `T&` could be an in-out-parameter. That can in itself be a problem and a source of errors:

> `T&`参数可以将信息传递给函数，反之亦然。因此，`T&`可以是一个“in-out”参数。这本身就是一个问题，也是错误的来源：

```c++
void f(string& s) {
    s = "New York";  // non-obvious error
}

void g() {
    string buffer = ".................................";
    f(buffer);	// oops
    // ...
}
```

Here, the writer of `g()` is supplying a buffer for `f()` to fill, but `f()` simply replaces it (at a somewhat higher cost than a simple copy of the characters). A bad logic error can happen if the writer of `g()` incorrectly assumes the size of the `buffer`.

>在这里，`g()`的编写者为`f()`提供了一个缓冲区来填充，但`f()`只是替换了它（比简单地复制字符的成本略高）。
>
>如果`g()`的编写者错误地假设`buffer`的大小，则可能发生严重的逻辑错误。

## Enforcement

- (Moderate) ((Foundation)) Warn about functions regarding reference to non-`const` parameters that do *not* write to them.

  - > 对函数引用非`const`形参时不写的函数发出警告。

- (Simple) ((Foundation)) Warn when a non-`const` parameter being passed by reference is `std::move`d.

  - >当引用传递的非`const`形参被`std::move`时发出警告。

## Reference

[1] [C++ Core Guidelines F.17: For “in-out” parameters, pass by reference to non-`const`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f17-for-in-out-parameters-pass-by-reference-to-non-const)
