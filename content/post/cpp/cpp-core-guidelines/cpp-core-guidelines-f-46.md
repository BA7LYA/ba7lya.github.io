---
title: "C++ Core Guidelines F.46 注解"
date: 2023-12-22T21:55:39+08:00
description: "C++ Core Guidelines F.46 注解"
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

# F.46: `int` is the return type for `main()`

## Reason

It’s a language rule, but violated through “language extensions” so often that it is worth mentioning. Declaring `main` (the one global `main` of a program) `void` limits portability.

这是一个语言规则，但由于“语言扩展”的频繁违反，所以值得一提。声明`main`为`void`限制了可移植性。

## Example

```c++
void main() { // bad, not C++
    /* ... */
};

int main() {
    std::cout << "This is the way to do it\n";
}
```

## Note

We mention this only because of the persistence of this error in the community. Note that despite its non-void return type, the main function does not require an explicit return statement.

我们提到这一点只是因为这个错误在社区中持续存在。

注意，尽管`main`函数的返回类型是非空的，但它不需要显式的返回语句。

## Enforcement

- The compiler should do it
- If the compiler doesn’t do it, let tools flag it
