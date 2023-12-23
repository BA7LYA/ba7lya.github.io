---
title: "C++ Core Guidelines F.49 注解"
date: 2023-12-22T21:55:51+08:00
description: "C++ Core Guidelines F.49 注解"
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

# F.49: Don’t return `const T`

## Reason

It is not recommended to return a `const` value. Such older advice is now obsolete; it does not add value, and it interferes with move semantics.

不建议返回`const`值。这样的老建议现在已经过时了，它不会增加价值，而且会干扰移动语义。

## Example

```c++
const vector<int> fct();    // bad: that "const" is more trouble than it is worth

void g(vector<int>& vx) {
    // ...
    fct() = vx;   // prevented by the "const"
    // ...
    vx = fct(); // expensive copy: move semantics suppressed by the "const"
    // ...
}
```

The argument for adding `const` to a return value is that it prevents (very rare) accidental access to a temporary. The argument against is that it prevents (very frequent) use of move semantics.

将`const`添加到返回值的理由是，它可以防止（非常罕见的）意外访问临时对象。

反对的理由是，它阻止（非常频繁地）使用移动语义。

## See also

[F.20, the general item about “out” output values](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-out)

## Enforcement

Flag returning a `const` value. To fix: Remove `const` to return a non-`const` value instead.
