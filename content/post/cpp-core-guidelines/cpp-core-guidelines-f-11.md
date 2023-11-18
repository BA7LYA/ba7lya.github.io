---
title: "C++ Core Guidelines F.11 注解"
date: 2023-11-19T04:26:31+08:00
description: "C++ Core Guidelines F.11 注解"
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

# F.11: Use an unnamed lambda if you need a simple function object in one place only

## Reason

That makes the code concise and gives better locality than alternatives.

## Example

```c++
auto earlyUsersEnd = std::remove_if(
    users.begin(),
    users.end(),
    [](const User &a) {
        return a.id > 100;
    }
);
```

## Exception

Naming a lambda can be useful for clarity even if it is used only once.

## Enforcement

- Look for identical and near identical lambdas (to be replaced with named functions or named lambdas).

## Reference

[1] [C++ Core Guidelines F.11 Use an unnamed lambda if you need a simple function object in one place only](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f11-use-an-unnamed-lambda-if-you-need-a-simple-function-object-in-one-place-only)
