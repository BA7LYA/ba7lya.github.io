---
title: "C++ Core Guidelines F.60 注解"
date: 2023-11-20T20:40:33+08:00
description: "C++ Core Guidelines F.60 注解"
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

# F.60: Prefer `T*` over `T&` when “no argument” is a valid option

>当参数的值可以为空时，优先选择`T*`而不是`T&`。

## Reason

A pointer (`T*`) can be a `nullptr` and a reference (`T&`) cannot, there is no valid “null reference”. Sometimes having `nullptr` as an alternative to indicated “no object” is useful, but if it is not, a reference is notationally simpler and might yield better code.

>指针（`T*`）可以是`nullptr`，而引用（`T&`）不能为空，没有合法的“空引用”。
>
>有时使用`nullptr`作为指示的“no object”的替代方法是有用的，但如果不是，则引用在符号上更简单，并且可能产生更好的代码。

## Example

```c++
// gsl::zstring is a char*; that is a C-style string
std::string zstring_to_string(gsl::zstring p) {
    if (!p) {
        return string{};    // p might be nullptr; remember to check
    }
    return std::string{p};
}

void print(const std::vector<int>& r) {
    // r refers to a std::vector<int>; no check needed
}
```

## Note

It is possible, but not valid C++ to construct a reference that is essentially a `nullptr` (e.g., `T* p = nullptr; T& r = *p;`). That error is very uncommon.

>在C++中，构造一个本质上是`nullptr`的引用是可能的（例如，`T* p = nullptr; T& r = *p;`），但是无效的。
>
>这种错误很少见。

## Note

If you prefer the pointer notation (`->` and/or `*` vs. `.`), `not_null<T*>` provides the same guarantee as `T&`.

>如果比起`.`，你更喜欢指针表示法（`->`和/或`*`），`not_null<T*>`提供与`T&`相同的保证。

## Enforcement

- Flag ???

## Reference

[1] [C++ Core Guidelines F.60: Prefer `T*` over `T&` when “no argument” is a valid option](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f60-prefer-t-over-t-when-no-argument-is-a-valid-option)
