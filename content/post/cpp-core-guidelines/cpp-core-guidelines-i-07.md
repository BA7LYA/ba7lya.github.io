---
title: "C++ Core Guidelines I.07 注解"
date: 2023-11-18T23:53:58+08:00
description: "C++ Core Guidelines I.07 注解"
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
  - C++ Best Practices
  - C++ Core Guidelines
comment: true
---

# I.07: State postconditions

> 说明后置条件。

## Reason

To detect misunderstandings about the result and possibly catch erroneous implementations.

>检测对结果的误解，并可能捕获错误的实现。

## Example

### Bad

Consider:

```c++
int area(int height, int width) {
    return height * width;	// bad
}
```

Here, we (incautiously) left out the precondition specification, so it is not explicit that height and width must be positive. We also left out the postcondition specification, so it is not obvious that the algorithm (`height * width`) is wrong for areas larger than the largest integer. Overflow can happen.

> 在这里，我们（不小心）遗漏了前提条件规范，因此高度和宽度必须为正的要求并不明确。
>
> 我们还省略了后置条件规范，因此对于大于最大整数的区域，算法（高度*宽度）是错误的并不明显。溢出是可能发生的。

### Good

Consider using:

```c++
int area(int height, int width) {
    auto res = height * width;
    Ensures(res > 0);
    return res;
}
```

## Example

### Bad

Consider a famous security bug:

```c++
void f()    // problematic
{
    char buffer[MAX];
    // ...
    memset(buffer, 0, sizeof(buffer));
}
```

There was no postcondition stating that the buffer should be cleared and the optimizer eliminated the apparently redundant `memset()` call:

```c++
void f()    // better
{
    char buffer[MAX];
    // ...
    memset(buffer, 0, sizeof(buffer));
    Ensures(buffer[0] == 0);
}
```

## Note

Postconditions are often informally stated in a comment that states the purpose of a function; `Ensures()` can be used to make this more systematic, visible, and checkable.

> 后置条件通常在说明函数目的的注释中非正式地声明，`Ensure()`可用于使其更**系统化**、**可见性**和**可检查性**。

## Note

Postconditions are especially important when they relate to something that is not directly reflected in a returned result, such as a state of a data structure used.

> 当后置条件与没有直接反映在返回结果中的内容相关时，例如所使用的数据结构的状态，后置条件尤为重要。

## Example

Consider a function that manipulates a `Record`, using a `mutex` to avoid race conditions:

### Bad

```c++
mutex m;

void manipulate(Record& r)	// don't
{
    m.lock();
    // ... no m.unlock() ...
}
```

Here, we “forgot” to state that the `mutex` should be released, so we don’t know if the failure to ensure release of the `mutex` was a bug or a feature.

### Better than bad

Stating the postcondition would have made it clear:

```c++
void manipulate(Record& r)    // postcondition: m is unlocked upon exit
{
    m.lock();
    // ... no m.unlock() ...
}
```

The bug is now obvious (but only to a human reading comments).

### Good

Better still, use RAII to ensure that the postcondition (“the lock must be released”) is enforced in code:

```c++
void manipulate(Record& r)    // best
{
    lock_guard<mutex> _ {m};
    // ...
}
```

## Note

Ideally, postconditions are stated in the interface/declaration so that users can easily see them. Only postconditions related to the users can be stated in the interface. Postconditions related only to internal state belongs in the definition/implementation.

> 理想情况下，后置条件是在接口/声明中声明的，这样用户可以很容易地看到它们。
>
> 只有与用户相关的后置条件可以在接口中声明。
>
> 只与内部状态相关的后置条件属于定义/实现。

## Enforcement

(Not enforceable) This is a philosophical guideline that is infeasible to check directly in the general case. Domain specific checkers (like lock-holding checkers) exist for many toolchains.

>这是一个在一般情况下无法直接检验的哲学准则。
>
>领域特定的检查器（如锁持有检查器）存在于许多工具链中。

## Reference

[1] [C++ Core Guidelines I.07: State postconditions](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i7-state-postconditions)
