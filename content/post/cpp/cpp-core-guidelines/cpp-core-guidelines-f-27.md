---
title: "C++ Core Guidelines F.27 注解"
date: 2023-12-22T21:55:16+08:00
description: "C++ Core Guidelines F.26 注解"
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

# F.27: Use a `std::shared_ptr<T>` to share ownership

## Reason

Using `std::shared_ptr` is the standard way to represent shared ownership.

That is, the last owner deletes the object.

最后一个所有者删除对象。

## Example

```c++
std::shared_ptr<const Image> im { read_image(somewhere) };

std::thread t0 {shade, args0,     top_left, im};
std::thread t1 {shade, args1,    top_right, im};
std::thread t2 {shade, args2,  bottom_left, im};
std::thread t3 {shade, args3, bottom_right, im};

// detach threads
// last thread to finish deletes the image
```

## Note

Prefer a `std::unique_ptr` over a `std::shared_ptr` if there is never more than one owner at a time. `std::shared_ptr` is for shared ownership.

如果一次不超过一个所有者，首选`std::unique_ptr`而不是`std::shared_ptr`。`std::shared_ptr`表示共享所有权。

Note that pervasive use of `std::shared_ptr` has a cost (atomic operations on the `std::shared_ptr`’s reference count have a measurable aggregate cost).

注意，普遍使用`std::shared_ptr`是有代价的（对`std::shared_ptr`的引用计数的原子操作有一个可测量的总代价）。

## Alternative

Have a single object own the shared object (e.g. a scoped object) and destroy that (preferably implicitly) when all users have completed.

让一个对象拥有共享对象（例如，有作用域的对象），并在所有用户都完成时销毁它（最好是隐式地）。

## Enforcement

(Not enforceable) This is a too complex pattern to reliably detect.
