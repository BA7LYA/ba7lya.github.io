---
title: "C++ Core Guidelines F.03 注解"
date: 2023-11-19T04:26:01+08:00
description: "C++ Core Guidelines F.03 注解"
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

# F.03: Keep functions short and simple

>保持函数简洁。

## Reason

Large functions are hard to read, more likely to contain complex code, and more likely to have variables in larger than minimal scopes. Functions with complex control structures are more likely to be long and more likely to hide logical errors.

>大型函数很难阅读，更有可能包含复杂的代码，并且更有可能具有大于最小作用域的变量。
>
>具有复杂控制结构的函数更有可能很长，更有可能隐藏逻辑错误。

## Example

### Bad

Consider:

```c++
// takes a value and calculates the expected ASIC output, given the two mode flags.
double simple_func(double val, int flag1, int flag2)
{
    double intermediate;
    if (flag1 > 0) {
        intermediate = func1(val);
        if (flag2 % 2)
             intermediate = sqrt(intermediate);
    }
    else if (flag1 == -1) {
        intermediate = func1(-val);
        if (flag2 % 2)
             intermediate = sqrt(-intermediate);
        flag1 = -flag1;
    }
    if (abs(flag2) > 10) {
        intermediate = func2(intermediate);
    }
    switch (flag2 / 10) {
    case 1: if (flag1 == -1) return finalize(intermediate, 1.171);
            break;
    case 2: return finalize(intermediate, 13.1);
    default: break;
    }
    return finalize(intermediate, 0.);
}
```

This is too complex. How would you know if all possible alternatives have been correctly handled? Yes, it breaks other rules also.

### Good

We can refactor:

```c++
double func1_muon(double val, int flag)
{
    // ???
}

double func1_tau(double val, int flag1, int flag2)
{
    // ???
}

double simple_func(double val, int flag1, int flag2)
    // simple_func: takes a value and calculates the expected ASIC output,
    // given the two mode flags.
{
    if (flag1 > 0)
        return func1_muon(val, flag2);
    if (flag1 == -1)
        // handled by func1_tau: flag1 = -flag1;
        return func1_tau(-val, flag1, flag2);
    return 0.;
}
```

## Note

“It doesn’t fit on a screen” is often a good practical definition of “far too large.” One-to-five-line functions should be considered normal.

>“一个屏幕放不下”通常是对函数“太大”的一个很实用的定义。1~5行的函数应该被认为是正常的。

## Note

Break large functions up into smaller cohesive and named functions. Small simple functions are easily `inline`d where the cost of a function call is significant.

>将大型函数分解为较小的内聚和命名函数。简单的小函数很容易内联，因为函数调用的成本很高。

## Enforcement

- Flag functions that do not “fit on a screen.” How big is a screen? Try 60 lines by 140 characters; that’s roughly the maximum that’s comfortable for a book page.

  - > 注意不“适合屏幕”的函数。
    >
    > 屏幕有多大？试着用60行140个字符，这大概是一页书所能承受的最大值。

- Flag functions that are too complex. How complex is too complex? You could use cyclomatic complexity<sup>[2]</sup>. Try “more than 10 logical paths through.” Count a simple switch as one path.

  - >注意过于复杂的函数。
    >
    >多复杂才算太复杂？你可以使用回路复杂度。尝试“通过10条以上的逻辑路径”。将一个简单的开关算作一条路径。

## Reference

[1] [C++ Core Guidelines F.03: Keep functions short and simple](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f3-keep-functions-short-and-simple)

[2] [Cyclomatic Complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity)
