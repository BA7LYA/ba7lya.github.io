---
title: "C++ Core Guidelines I.06 注解"
date: 2023-11-18T23:40:01+08:00
description: "C++ Core Guidelines I.06 注解"
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

# I.06: Prefer `Expects()` for expressing preconditions

>首选`Expects()`来表示先决条件。

## Reason

To make it clear that the condition is a precondition and to enable tool use.

>明确条件是先决条件，并使工具能够使用。

> 注：后半句咋理解？？？

## Example

### Bad

```c++
int area(int height, int width) {
    if (height <= 0 || width <= 0) {	// obscure
        my_error();
    }
    // ...
}
```

### Good

```c++
int area(int height, int width) {
    Expects(height > 0 && width > 0);	// good
    // ...
}
```

## Note

Preconditions can be stated in many ways, including comments, `if`-statements, and `assert()`. This can make them hard to distinguish from ordinary code, hard to update, hard to manipulate by tools, and might have the wrong semantics (do you always want to abort in debug mode and check nothing in productions runs?).

> 前提条件可以以多种方式声明，包括注释、`if`语句和`assert()`。
>
> 这可能使前提条件难以与普通代码区分，难以更新，难以被工具操作，并且可能具有错误的语义（你是否总是希望在调试模式下中止，而在生产运行中不检查任何内容？）

## Note

Preconditions should be part of the interface rather than part of the implementation, but we don’t yet have the language facilities to do that. Once language support becomes available (e.g., see the contract proposal) we will adopt the standard version of preconditions, postconditions, and assertions.

>**前提条件应该是接口的一部分，而不是实现的一部分**，但是我们还没有语言工具来做到这一点。
>
>一旦语言支持可用（例如，请参阅契约提案），我们将采用标准版本的前置条件、后置条件和断言。

## Note

`Expects()` can also be used to check a condition in the middle of an algorithm.

> `Expects()`也可用于在算法中间检查条件。

## Note

No, using `unsigned` is not a good way to sidestep the problem of ensuring that a value is non-negative.

> 不，使用`unsigned`并不是避免确保值是非负的问题的好方法。

## Enforcement

(Not enforceable) Finding the variety of ways preconditions can be asserted is not feasible. Warning about those that can be easily identified (assert()) has questionable value in the absence of a language facility.

>找到断言先决条件的各种方法是不可行的。在没有语言工具的情况下，对容易识别的对象（`assert()`）发出警告的价值值得怀疑。

## Reference

[1] [C++ Core Guidelines I.06: Prefer `Expects()` for expressing preconditions](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i6-prefer-expects-for-expressing-preconditions)
