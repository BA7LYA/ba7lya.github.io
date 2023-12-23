---
title: "C++ Core Guidelines F.56 注解"
date: 2023-12-22T21:56:19+08:00
description: "C++ Core Guidelines F.55 注解"
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

# F.56: Avoid unnecessary condition nesting

## Reason

Shallow nesting of conditions makes the code easier to follow. It also makes the intent clearer. Strive to place the essential code at outermost scope, unless this obscures intent.

条件的浅嵌套使代码更容易理解。它也使意图更清晰。尽量将基本代码放在最外层作用域，除非这样会模糊意图。

## Example

Use a guard-clause to take care of exceptional cases and return early.

```c++
// Bad: Deep nesting
void foo() {
    ...
    if (x) {
        computeImportantThings(x);
    }
}

// Bad: Still a redundant else.
void foo() {
    ...
    if (!x) {
        return;
    }
    else {
        computeImportantThings(x);
    }
}

// Good: Early return, no redundant else
void foo() {
    ...
    if (!x)
        return;

    computeImportantThings(x);
}
```

## Example

```c++
// Bad: Unnecessary nesting of conditions
void foo() {
    ...
    if (x) {
        if (y) {
            computeImportantThings(x);
        }
    }
}

// Good: Merge conditions + return early
void foo() {
    ...
    if (!(x && y))
        return;

    computeImportantThings(x);
}
```

## Enforcement

Flag a redundant `else`. Flag a functions whose body is simply a conditional statement enclosing a block.
