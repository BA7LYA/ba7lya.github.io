---
title: "C++ Core Guidelines P.03 注解"
date: 2023-11-17T21:33:23+08:00
description: "C++ Core Guidelines P.03 注解"
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
  - C++ Best Practices
  - C++ Core Guidelines
comment: true
---

# P.03: Express intent

>表达意图。

## Reason

Unless the intent of some code is stated (e.g., in names or comments), it is impossible to tell whether the code does what it is supposed to do.

> 除非说明了某些代码的意图（例如，在名称或注释中），否则不可能判断代码是否执行了它应该执行的操作。

## Example

### Bad

```c++
gsl::index i = 0;
while (i < v.size()) {
    // ... do something with v[i] ...
}
```

The intent of “just” looping over the elements of `v` is not expressed here. The implementation detail of an `index` is exposed (so that it might be misused), and `i` outlives the scope of the loop, which might or might not be intended. The reader cannot know from just this section of code.

> 遍历`v`的元素的意图在这里没有表示。
>
> `gsl::index`的实现细节是公开的（因此可能会被误用），并且它超出了循环的作用域，这可能是也可能不是预期的。
>
> 读者不能仅仅从这段代码中知道具体的意图。

### Better

```c++
for (const auto& x : v) {
    /* do something with the value of x */
}
```

Now, there is no explicit mention of the iteration mechanism, and the loop operates on a reference to `const` elements so that accidental modification cannot happen.

> 现在，没有明确提到迭代机制，并且循环操作对`const`元素的引用，因此不会发生意外修改。

If modification is desired, say so:

```c++
for (auto& x : v) {
    /* modify x */
}
```

For more details about for-statements, see ES.71<sup>[2]</sup>.

### Best

Sometimes better still, use a named algorithm.

> 有时使用命名算法会更好。

This example uses the `for_each` from the Ranges TS because it directly expresses the intent:

```c++
for_each(v, 	 [](int x) { /* do something with the value of x */ });
for_each(par, v, [](int x) { /* do something with the value of x */ }); // 并行
```

The last variant makes it clear that we are not interested in the order in which the elements of `v` are handled.

A programmer should be familiar with

- The guidelines support library<sup>[3]</sup>
- The ISO C++ Standard Library<sup>[4]</sup>
- Whatever foundation libraries are used for the current project(s)

## Note

Alternative formulation: Say what should be done, rather than just how it should be done.

> 另一种表述方式：说出应该做什么，而不仅仅是应该怎么做。

## Note

Some language constructs express intent better than others.

>有些语言结构比其他语言结构更能表达意图。

## Example

If two `int`s are meant to be the coordinates of a 2D point, say so:

### Bad

```c++
draw_line(int, int, int, int);  // obscure: (x1,y1,x2,y2)? (x,y,h,w)? ...?
                                // need to look up documentation to know
```

### Good

```c++
draw_line(Point, Point);        // clearer
```

## Enforcement

Look for common patterns for which there are better alternatives

- simple `for` loops vs. range-`for` loops
- `f(T*, int)` interfaces vs. `f(span<T>)` interfaces
- loop variables in too large a scope
- naked `new` and `delete`
- functions with many parameters of built-in types

There is a huge scope for cleverness and semi-automated program transformation.

> 智能和半自动化的程序转换有很大的空间。

## Reference

[1] [C++ Core Guidelines P.03 - Express intent](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p2-write-in-iso-standard-c)

[2] [C++ Core Guidelines ES.71: Prefer a range-`for`-statement to a `for`-statement when there is a choice](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-for-range)

[3] [Guidelines Support Library](https://github.com/Microsoft/GSL)

[4] [The ISO C++ Standard Library](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#sl-the-standard-library)
