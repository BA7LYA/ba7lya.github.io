---
title: "C++ Core Guidelines P.11 注解"
date: 2023-11-18T01:17:31+08:00
description: "C++ Core Guidelines P.11 注解"
featured: true
draft: false
toc: false
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
  - Best Practice
comment: true
---

# P.11: Encapsulate messy constructs, rather than spreading through the code

> 封装混乱的结构，而不是在代码中展开。

## Reason

Messy code is more likely to hide bugs and harder to write. A good interface is easier and safer to use. Messy, low-level code breeds more such code.

> 混乱的代码更容易隐藏bug，也更难编写。
>
> 好的接口使用起来更容易、更安全。
>
> 杂乱、低级的代码会滋生更多杂乱、低级的代码。

## Example

### Bad

```c++
int sz = 100;
int* p = (int*) malloc(sizeof(int) * sz);
int count = 0;
// ...
for (;;) {
    // ... read an int into x, exit loop if end of file is reached ...
    // ... check that x is valid ...
    if (count == sz)
        p = (int*) realloc(p, sizeof(int) * sz * 2);
    p[count++] = x;
    // ...
}
```

This is low-level, verbose, and error-prone. For example, we “forgot” to test for memory exhaustion.

>这是低级的、冗长的、容易出错的。例如，当我们“忘记”测试内存耗尽时。

### Good

Instead, we could use `vector`:

```c++
std::vector<int> v;
v.reserve(100); // increase the capacity of the vector to at least 100
// ...
for (int x; std::cin >> x; ) {
    // ... check that x is valid ...
    v.push_back(x);
}
```

## Note

The standards library and the GSL are examples of this philosophy. For example, instead of messing with the arrays, unions, cast, tricky lifetime issues, `gsl::owner`, etc., that are needed to implement key abstractions, such as `vector`, `span`, `lock_guard`, and `future`, we use the libraries designed and implemented by people with more time and expertise than we usually have. Similarly, we can and should design and implement more specialized libraries, rather than leaving the users (often ourselves) with the challenge of repeatedly getting low-level code well. This is a variant of the subset of superset principle that underlies these guidelines.

> 标准库和GSL就是这种理念的例子。
>
> 例如，我们没有去处理数组、联合、强制转换、棘手的生命周期问题、`gsl::owner`等实现关键抽象（如`vector`、`span`、`lock_guard`和`future`）所需要的问题，而是使用那些比我们通常拥有更多时间和专业知识的人设计和实现的库。
>
> 类似地，我们可以而且应该设计和实现更专门的库，而不是让用户（通常是我们自己）反复地获得低级代码。
>
> 这是作为这些指导原则基础的超集子集原则的一个变体。

> 注：生产环境使用代码的优先级：标准库>第三方库（可能同一场景会有多个可用的库，需要评估用哪个）>自己手搓的轮子。

## Enforcement

- Look for “messy code” such as complex pointer manipulation and casting outside the implementation of abstractions.

  - > 寻找“混乱的代码”，比如复杂的指针操作和抽象实现之外的强制转换。

## Reference

[1] [C++ Core Guidelines P.11: Encapsulate messy constructs, rather than spreading through the code](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p11-encapsulate-messy-constructs-rather-than-spreading-through-the-code)
