---
title: "C++ Core Guidelines I.01 注解"
date: 2023-11-18T03:11:46+08:00
description: "C++ Core Guidelines I.01 注解"
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

# I.01: Make interfaces explicit

>使接口显式。

## Reason

Correctness. Assumptions not stated in an interface are easily overlooked and hard to test.

>正确性。没有在接口中说明的假设很容易被忽略，也很难测试。

## Example, bad

Controlling the behavior of a function through a global (namespace scope) variable (a call mode) is implicit and potentially confusing.

> 通过全局（名称空间作用域）变量（调用模式）控制函数的行为是隐式的，并且可能令人困惑。

For example:

```c++
int round(double d) {
    return (round_up) ? std::ceil(d) : d;	// don't: "invisible" dependency
}
```

It will not be obvious to a caller that the meaning of two calls of `round(7.2)` might give different results.

>对于调用者来说，两次调用`round(7.2)`的含义可能会产生不同的结果，这一点并不明显。

## Exception

Sometimes we control the details of a set of operations by an environment variable, e.g., normal vs. verbose output or debug vs. optimized. The use of a non-local control is potentially confusing, but controls only implementation details of otherwise fixed semantics.

>有时我们通过一个环境变量来控制一组操作的细节，例如，正常输出与详细输出，调试输出与优化输出。使用非本地控制可能会造成混淆，但它只控制其他固定语义的实现细节。

## Example

### Bad

Reporting through non-local variables (e.g., `errno`) is easily ignored. For example:

```c++
// don't: no test of fprintf's return value
fprintf(connection, "logging: %d %d %d\n", x, y, s);
```

What if the connection goes down so that no logging output is produced? See I.???.

### Alternative

Throw an exception. An exception cannot be ignored.

## Alternative formulation

Avoid passing information across an interface through non-local or implicit state. Note that non-`const` member functions pass information to other member functions through their object’s state.

>避免通过非本地或隐式状态跨接口传递信息。
>
>注意，非`const`成员函数通过其对象的状态将信息传递给其他成员函数。

## Alternative formulation

An interface should be a function or a set of functions. Functions can be function templates and sets of functions can be classes or class templates.

> 接口应该是一个函数或一组函数。函数可以是函数模板，函数集可以是类或类模板。

## Enforcement

- (Simple) A function should not make control-flow decisions based on the values of variables declared at namespace scope.

  - > 函数不应该根据在命名空间范围内声明的变量的值做出控制流决策。

- (Simple) A function should not write to variables declared at namespace scope.

  - > 函数不应该写入在命名空间范围内声明的变量。

## Reference

[1] [C++ Core Guidelines I.01: Make interfaces explicit](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i1-make-interfaces-explicit)
