---
title: "C++ Core Guidelines I.08 注解"
date: 2023-11-19T00:20:08+08:00
description: "C++ Core Guidelines I.08 注解"
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

# I.08: Prefer `Ensures()` for expressing postconditions

## Reason

To make it clear that the condition is a postcondition and to enable tool use.

> 以明确条件是后置条件，并启用工具使用。

## Example

```c++
void f() {
    char buffer[MAX];
    // ...
    memset(buffer, 0, MAX);
    Ensures(buffer[0] == 0);
}
```

## Note

Postconditions can be stated in many ways, including comments, `if`-statements, and `assert()`. This can make them hard to distinguish from ordinary code, hard to update, hard to manipulate by tools, and might have the wrong semantics.

> 后置条件可以通过多种方式声明，包括注释、`if`语句和`assert()`。
>
> 这可能使它们难以与普通代码区分，难以更新，难以被工具操作，并且可能具有错误的语义。

## Alternative

Postconditions of the form “this resource must be released” are best expressed by RAII.

>RAII最容易表达形式为“此资源必须被释放”的后置条件。

## Note

Ideally, that `Ensures` should be part of the interface, but that’s not easily done. For now, we place it in the definition (function body). Once language support becomes available (e.g., see the contract proposal) we will adopt the standard version of preconditions, postconditions, and assertions.

>理想情况下，`Ensures`应该是接口的一部分，但这并不容易做到。现在，我们把它放在定义（函数体）中。
>
>一旦语言支持可用（例如，请参阅契约提案），我们将采用标准版本的前置条件、后置条件和断言。

## Enforcement

(Not enforceable) Finding the variety of ways postconditions can be asserted is not feasible. Warning about those that can be easily identified (`assert()`) has questionable value in the absence of a language facility.

>找到断言后置条件的各种方法是不可行的。在没有语言功能的情况下，那些容易识别的（`assert()`）的警告的价值值得怀疑。

## Reference

[1] [C++ Core Guidelines I.08: Prefer `Ensures()` for expressing postconditions]()
