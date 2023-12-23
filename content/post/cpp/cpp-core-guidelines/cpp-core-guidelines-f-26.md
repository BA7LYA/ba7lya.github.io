---
title: "C++ Core Guidelines F.26 注解"
date: 2023-12-22T21:55:14+08:00
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

# F.26: Use a `std::unique_ptr<T>` to transfer ownership where a pointer is needed

## Reason

Using `std::unique_ptr` is the cheapest way to pass a pointer safely.

## See also

[C.50](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-factory) regarding when to return a `shared_ptr` from a factory.

## Example

```c++
std::unique_ptr<Shape> get_shape(istream& is) { // assemble shape from input stream
    auto kind = read_header(is); // read header and identify the next shape on input
    switch (kind) {
    case kCircle:
        return std::make_unique<Circle>(is);
    case kTriangle:
        return std::make_unique<Triangle>(is);
    // ...
    }
}
```

## Note

You need to pass a pointer rather than an object if what you are transferring is an object from a class hierarchy that is to be used through an interface (base class).

您需要传递一个指针而不是一个对象，如果您正在传输的东西是一个由类层次结构的对象，它将通过一个接口（基类）使用。

## Enforcement

(Simple) Warn if a function returns a locally allocated raw pointer. Suggest using either `std::unique_ptr` or `std::shared_ptr` instead.
