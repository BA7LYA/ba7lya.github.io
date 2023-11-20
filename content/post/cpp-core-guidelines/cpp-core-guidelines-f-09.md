---
title: "C++ Core Guidelines F.09 注解"
date: 2023-11-19T04:26:24+08:00
description: "C++ Core Guidelines F.09 注解"
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

# F.09: Unused parameters should be unnamed

## Reason

Readability. Suppression of unused parameter warnings.

## Example

```c++
widget* find(const set<widget>& s, const widget& w, Hint);   // once upon a time, a hint was used
```

## Note

Allowing parameters to be unnamed was introduced in the early 1980s to address this problem.

If parameters are conditionally unused, declare them with the `[[maybe_unused]]` attribute.

>如果参数是根据条件可能不会被不使用的，用`[[maybe_unused]]`属性声明它们。

For example:

```c++
template <typename Value>
Value* find(
    const set<Value>& s,
    const Value& v,
    [[maybe_unused]] Hint h)
{
    if constexpr (sizeof(Value) > CacheSize)
    {
        // a hint is used only if Value is of a certain size
    }
}
```

## Enforcement

Flag named unused parameters.

## Reference

[1] [C++ Core Guidelines F.09: Unused parameters should be unnamed](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f9-unused-parameters-should-be-unnamed)
