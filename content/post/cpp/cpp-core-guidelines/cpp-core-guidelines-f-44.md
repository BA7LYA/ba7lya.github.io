---
title: "C++ Core Guidelines F.44 注解"
date: 2023-12-22T21:55:35+08:00
description: "C++ Core Guidelines F.44 注解"
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

# F.44: Return a `T&` when copy is undesirable and “returning no object” isn’t needed

## Reason 

The language guarantees that a `T&` refers to an object, so that testing for `nullptr` isn’t necessary.

## See also

The return of a reference must not imply transfer of ownership: [discussion of dangling pointer prevention](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???) and [discussion of ownership](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???).

## Example

```c++
class Car
{
    std::array<wheel, 4> w;
    // ...
public:
    wheel& get_wheel(int i) {
        Expects(i < w.size());
        return w[i];
    }
    // ...
};

void use() {
    Car c;
    wheel& w0 = c.get_wheel(0); // w0 has the same lifetime as c
}
```

## Enforcement

Flag functions where no `return` expression could yield `nullptr`.
