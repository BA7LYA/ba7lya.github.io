---
title: "C++ Core Guidelines P.05 注解"
date: 2023-11-17T22:58:01+08:00
description: "C++ Core Guidelines P.05 注解"
featured: true
draft: false
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

# P.05: Prefer compile-time checking to run-time checking

> 比起运行时检查，编译时检查更好。

> 注：编译器检查 > 运行时检查 > 运行时报错 > 未定义行为。

## Reason

Code clarity and performance. You don’t need to write error handlers for errors caught at compile time.

> 代码清晰度和性能。
>
> 您不需要为在编译时捕获的错误编写错误处理程序。

## Example

### Bad

```c++
// Int is an alias used for integers
int bits = 0;         // don't: avoidable code
for (Int i = 1; i; i <<= 1) { // what's the point?
    ++bits;
}
if (bits < 32) {
    std::cerr << "Int too small\n";
}
```

This example fails to achieve what it is trying to achieve (because overflow is undefined) and should be replaced with a simple `static_assert`.

> 这个例子没有达到它想要达到的目标（因为overflow是未定义的），应该用一个简单的`static_assert`替换。

### Good

```c++
// Int is an alias used for integers
static_assert(sizeof(Int) >= 4);	// do: compile-time check
```

Or better still just use the type system and replace `Int` with `int32_t`.

## Example

### Bad

```c++
void read(int* p, int n);   // read max n integers into *p

int a[100];
read(a, 1000);    // bad, off the end
```

### Good

```c++
void read(std::span<int> r); // read into the range of integers r

int a[100];
read(a);        // better: let the compiler figure out the number of elements
```

## Alternative formulation

Don’t postpone to run time what can be done well at compile time.

>编译时可以做的事情不要推迟到运行时。

## Enforcement

- Look for pointer arguments. // 查找指针参数。
- Look for run-time checks for range violations. // 查找违反范围的运行时检查。

## Reference

[1] [C++ Core Guidelines P.05: Prefer compile-time checking to run-time checking](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p5-prefer-compile-time-checking-to-run-time-checking)
