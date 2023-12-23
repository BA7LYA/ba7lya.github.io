---
title: "C++ Core Guidelines F.51 注解"
date: 2023-12-22T21:56:08+08:00
description: "C++ Core Guidelines F.51 注解"
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

# F.51: Where there is a choice, prefer default arguments over overloading

## Reason

Default arguments simply provide alternative interfaces to a single implementation. There is no guarantee that a set of overloaded functions all implement the same semantics. The use of default arguments can avoid code replication.

默认参数只是为单个实现提供可选接口。不能保证一组重载函数都实现相同的语义。使用默认参数可以避免代码复制。

## Note

There is a choice between using default argument and overloading when the alternatives are from a set of arguments of the same types.

当备选参数来自一组相同类型的参数时，可以选择使用默认参数还是重载。

For example:

```c++
void print(const std::string& s, format f = {});
```

as opposed to

```c++
void print(const std::string& s);  // use default format
void print(const std::string& s, format f);
```

There is not a choice when a set of functions are used to do a semantically equivalent operation to a set of types.

当使用一组函数来执行与一组类型在语义上等价的操作时，没有提供选择的机会。

For example:

```c++
void print(const char&);
void print(int);
void print(zstring);
```

> 见过不少库有这样的用法。这样做可以让库的使用者不用过于担心传入的参数形式。

## See also

[Default arguments for virtual functions](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-virtual-default-arg)

## Enforcement

Warn on an overload set where the overloads have a common prefix of parameters (e.g., `f(int)`, `f(int, const string&)`, `f(int, const string&, double)`). (Note: Review this enforcement if it’s too noisy in practice.)

对重载集中重载具有共同参数前缀的重载集发出警告。

例如，

```c++
f(int)
f(int, const std::string&)
f(int, const std::string&, double)
```

