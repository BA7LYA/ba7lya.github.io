---
title: "C++ Core Guidelines I.27 注解"
date: 2023-11-19T03:22:43+08:00
description: "C++ Core Guidelines I.27 注解"
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
  - C++ Best Practices
  - C++ Core Guidelines
comment: true
---

# I.27: For stable library ABI, consider the Pimpl idiom

>如果需要稳定的库ABI，请考虑Pimpl习语。

## Reason

Because private data members participate in class layout and private member functions participate in overload resolution, changes to those implementation details require recompilation of all users of a class that uses them. A non-polymorphic interface class holding a pointer to implementation (Pimpl) can isolate the users of a class from changes in its implementation at the cost of an indirection.

>因为私有数据成员参与类布局，私有成员函数参与重载解析，所以对这些实现细节的更改需要重新编译使用它们的类的所有用户。
>
>持有实现指针（Pimpl）的非多态接口类可以以间接的代价将类的用户与实现中的更改隔离开来。

## Example

Interface

```c++
// widget.hxx
class widget
{
    class impl;
    std::unique_ptr<impl> pimpl;

public:
    widget(int); // defined in the implementation file
    ~widget();   // defined in the implementation file, where impl is a complete type

    // non-copyable
    widget(const widget&) = delete;
    widget& operator=(const widget&) = delete;

    // moveable
    widget(widget&&) noexcept = default;			// defined in the implementation file
    widget& operator=(widget&&) noexcept = default; // defined in the implementation file

    void draw(); // public API that will be forwarded to the implementation
};
```

Implementation

```c++
class widget::impl
{
    int n; // private data
public:
    void draw(const widget& w) { /* ... */ }
    impl(int n) : n(n) {}
};

widget::widget(int n) : pimpl{std::make_unique<impl>(n)} {}
widget::~widget() = default;

void widget::draw() { pimpl->draw(*this); }
```

## Enforcement

(Not enforceable) It is difficult to reliably identify where an interface forms part of an ABI.

>很难可靠地确定接口在哪里构成ABI的一部分。

## Notes

[1] [C++ Core Guidelines I.27: For stable library ABI, consider the Pimpl idiom](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i27-for-stable-library-abi-consider-the-pimpl-idiom)

[2] [GotW #100: Compilation Firewalls](https://herbsutter.com/gotw/_100/)

[3] [cppreference pimpl](http://en.cppreference.com/w/cpp/language/pimpl)
