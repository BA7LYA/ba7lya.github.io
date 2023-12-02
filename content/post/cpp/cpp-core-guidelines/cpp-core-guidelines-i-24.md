---
title: "C++ Core Guidelines I.24 注解"
date: 2023-11-19T02:57:57+08:00
description: "C++ Core Guidelines I.24 注解"
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

# I.24: Avoid adjacent parameters that can be invoked by the same arguments in either order with different meaning

> 避免相邻的参数可以被相同的参数以不同的顺序调用。

## Reason

Adjacent arguments of the same type are easily swapped by mistake.

>相同类型的相邻参数很容易被错误地交换。

## Example

### Bad

Consider:

```c++
void copy_n(T* p, T* q, int n);  // copy from [p:p + n) to [q:q + n)
```

This is a nasty variant of a K&R C-style interface. It is easy to reverse the “to” and “from” arguments.

### Good

Use `const` for the “from” argument:

```c++
void copy_n(const T* p, T* q, int n);  // copy from [p:p + n) to [q:q + n)
```

## Exception

If the order of the parameters is not important, there is no problem:

```c++
int max(int a, int b);
```

## Alternative

Don’t pass arrays as pointers, pass an object representing a range (e.g., a `span`):

```c++
void copy_n(std::span<const T> p, std::span<T> q);  // copy from p to q
```

## Alternative

Define a `struct` as the parameter type and name the fields for those parameters accordingly:

```c++
struct SystemParams {
    string config_file;
    string output_path;
    seconds timeout;
};
void initialize(SystemParams p);
```

This tends to make invocations of this clear to future readers, as the parameters are often filled in by name at the call site.

## Note

Only the interface’s designer can adequately address the source of violations of this guideline.

> 只有接口的设计者才能充分解决违反这一准则的根源。

## Enforcement strategy

(Simple) Warn if two consecutive parameters share the same type.

> 如果两个连续的参数共享相同的类型，则发出警告。我们仍在寻找一种不那么简单的执行方式。

We are still looking for a less-simple enforcement.

> ???

## Reference

[1] [C++ Core Guidelines I.24: Avoid adjacent parameters that can be invoked by the same arguments in either order with different meaning](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i24-avoid-adjacent-parameters-that-can-be-invoked-by-the-same-arguments-in-either-order-with-different-meaning)
