---
title: "C++ Core Guidelines F.06 注解"
date: 2023-11-19T04:26:12+08:00
description: "C++ Core Guidelines F.06 注解"
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

# F.06: If your function must not throw, declare it `noexcept`

>如果你的函数不会抛出异常，就声明它为`noexcept`。

## Reason

If an exception is not supposed to be thrown, the program cannot be assumed to cope with the error and should be terminated as soon as possible. Declaring a function `noexcept` helps optimizers by reducing the number of alternative execution paths. It also speeds up the exit after failure.

>如果不打算抛出异常，则不能假定程序能够处理错误，应尽快终止该程序。声明函数`noexcept`有助于优化器减少可选执行路径的数量。它也加速了失败后的退出。

## Example

Put `noexcept` on every function written completely in C or in any other language without exceptions. The C++ Standard Library does that implicitly for all functions in the C Standard Library.

>在完全用C或任何其他没有异常的语言编写的每个函数上都放上`noexcept`。
>
>C++标准库对C标准库中的所有函数都隐式地执行此操作。

## Note

`constexpr` functions can throw when evaluated at run time, so you might need conditional `noexcept` for some of those.

>`constexpr`函数在运行时求值时可能会抛出，所以你可能需要有条件的`noexcept`。

> 注：demo???

## Example

You can use `noexcept` even on functions that can throw:

```c++
vector<string> collect(istream& is) noexcept
{
    vector<string> res;
    for (string s; is >> s;)
        res.push_back(s);
    return res;
}
```

If `collect()` runs out of memory, the program crashes. Unless the program is crafted to survive memory exhaustion, that might be just the right thing to do; `terminate()` might generate suitable error log information (but after memory runs out it is hard to do anything clever).

## Note

You must be aware of the execution environment that your code is running when deciding whether to tag a function `noexcept`, especially because of the issue of throwing and allocation. Code that is intended to be perfectly general (like the standard library and other utility code of that sort) needs to support environments where a `bad_alloc` exception could be handled meaningfully. However, most programs and execution environments cannot meaningfully handle a failure to allocate, and aborting the program is the cleanest and simplest response to an allocation failure in those cases. If you know that your application code cannot respond to an allocation failure, it could be appropriate to add `noexcept` even on functions that allocate.

>在决定是否将函数标记为`noexcept`时，你必须了解代码运行的执行环境，特别是因为抛出和分配问题。
>
>想要完全通用的代码（比如标准库和其他类似的实用程序代码）需要支持可以有意义地处理`bad_alloc`异常的环境。
>
>然而，大多数程序和执行环境都不能有效地处理分配失败，在这种情况下，终止程序是对分配失败最干净、最简单的响应。
>
>如果您知道您的应用程序代码无法响应分配失败，那么在分配函数上添加`noexcept`可能是合适的。

Put another way: In most programs, most functions can throw (e.g., because they use `new`, call functions that do, or use library functions that reports failure by throwing), so don’t just sprinkle `noexcept` all over the place without considering whether the possible exceptions can be handled.

>换句话说：在大多数程序中，大多数函数都可能会抛出（例如，因为它们使用了`new`，调用了这样做的函数，或者使用了通过抛出报告失败的库函数），所以不要在没有考虑是否可以处理可能的异常的情况下到处使用`noexcept`。

`noexcept` is most useful (and most clearly correct) for frequently used, low-level functions.

>对于经常使用的底层函数，`noexcept`是最有用的（也是最明显正确的）。

## Note

Destructors, `swap` functions, move operations, and default constructors should never throw.

>析构函数、`swap`函数、移动操作和默认构造函数永远不应该抛出。

See also: C++ Core Guidelines C.44<sup>[2]</sup>.

## Enforcement

- Flag functions that are not `noexcept`, yet cannot throw.
- Flag throwing `swap`, `move`, destructors, and default constructors.

## Reference

[1] [C++ Core Guidelines F.06: If your function must not throw, declare it `noexcept`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f6-if-your-function-must-not-throw-declare-it-noexcept)

[2] [C++ Core Guidelines C.44: Prefer default constructors to be simple and non-throwing](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c44-prefer-default-constructors-to-be-simple-and-non-throwing)
