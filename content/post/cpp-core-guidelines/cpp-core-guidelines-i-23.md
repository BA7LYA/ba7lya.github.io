---
title: "C++ Core Guidelines I.23 注解"
date: 2023-11-19T02:39:11+08:00
description: "C++ Core Guidelines I.23 注解"
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

# I.23: Keep the number of function arguments low

>尽量减少函数参数的数量。

## Reason

Having many arguments opens opportunities for confusion. Passing lots of arguments is often costly compared to alternatives.

> 参数太多会导致混乱。与优化替代方案相比，传递大量参数通常代价高昂。

## Discussion

The two most common reasons why functions have too many parameters are:

### Missing an abstraction.

There is an abstraction missing, so that a compound value is being passed as individual elements instead of as a single object that enforces an invariant. This not only expands the parameter list, but it leads to errors because the component values are no longer protected by an enforced invariant.

> 缺少适当的抽象。
>
> 因此复合值作为单独的元素传递，而不是作为强制不变量的单个对象传递。
>
> 这不仅使参数列表膨胀，而且还会导致错误，因为组件值不再受到强制不变量的保护。

### Violating “one function, one responsibility.”

The function is trying to do more than one job and should probably be refactored.

> 违反了“一个功能，一个责任”（单一责任原则）。该函数试图完成多个任务，可能应该进行重构。

## Example

The standard-library `merge()` is at the limit of what we can comfortably handle:

```c++
template<
	class InputIterator1,
	class InputIterator2,
	class OutputIterator,
	class Compare>
OutputIterator merge(
	InputIterator1 first1,
	InputIterator1 last1,
	InputIterator2 first2,
	InputIterator2 last2,
	OutputIterator result,
	Compare        comp
);
```

Note that this is because of problem above – missing abstraction. Instead of passing a range (abstraction), STL passed iterator pairs (unencapsulated component values). Here, we have four template arguments and six function arguments.

To simplify the most frequent and simplest uses, the comparison argument can be defaulted to `<`:

```c++
template<
	class InputIterator1,
    class InputIterator2,
    class OutputIterator>
OutputIterator merge(
	InputIterator1 first1,
	InputIterator1 last1,
	InputIterator2 first2,
	InputIterator2 last2,
	OutputIterator result
);
```

This doesn’t reduce the total complexity, but it reduces the surface complexity presented to many users.

To really reduce the number of arguments, we need to bundle the arguments into higher-level abstractions:

```c++
template<
	class InputRange1,
    class InputRange2,
    class OutputIterator>
OutputIterator merge(
    InputRange1 r1,
    InputRange2 r2,
    OutputIterator result
);
```

Grouping arguments into “bundles” is a general technique to reduce the number of arguments and to increase the opportunities for checking.

Alternatively, we could use a standard library concept to define the notion of three types that must be usable for merging:

```c++
template<class In1, class In2, class Out>
	requires mergeable<In1, In2, Out>
Out merge(In1 r1, In2 r2, Out result);
```

## Example

The safety Profiles recommend replacing

```c++
void f(int* some_ints, int some_ints_length);  // BAD: C style, unsafe
```

with

```c++
void f(gsl::span<int> some_ints);              // GOOD: safe, bounds-checked
```

Here, using an abstraction has safety and robustness benefits, and naturally also reduces the number of parameters.

## Note

How many parameters are too many? Try to use fewer than four (4) parameters. There are functions that are best expressed with four individual parameters, but not many.

> 多少个参数算多？尽量使用少于4个参数。

## Alternative

Use better abstraction: Group arguments into meaningful objects and pass the objects (by value or by reference).

>更好的抽象：将参数分组为有意义的对象，并传递对象（通过值或引用）。

## Alternative

Use default arguments or overloads to allow the most common forms of calls to be done with fewer arguments.

>使用默认参数或重载以允许使用更少的参数完成最常见形式的调用。

## Enforcement

- Warn when a function declares two iterators (including pointers) of the same type instead of a range or a view.

  - > 当函数声明两个相同类型的迭代器（包括指针）而不是范围或视图时发出警告。

- (Not enforceable) This is a philosophical guideline that is infeasible to check directly.

  - > 这是一个哲学指导方针，不可能直接检查。

## Reference

[1] [C++ Core Guidelines I.23: Keep the number of function arguments low](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i23-keep-the-number-of-function-arguments-low)
