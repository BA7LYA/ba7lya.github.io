---
title: "C++ Core Guidelines P.09 注解"
date: 2023-11-18T00:41:44+08:00
description: "C++ Core Guidelines P.09 注解"
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
  - C++ Best Practice
  - C++ Core Guidelines
comment: true
---

# P.9: Don’t waste time or space

> 不要浪费时间和空间。

## Reason

This is C++.

> :)

## Note

Time and space that you spend well to achieve a goal (e.g., speed of development, resource safety, or simplification of testing) is not wasted. “Another benefit of striving for efficiency is that the process forces you to understand the problem in more depth.” - Alex Stepanov.

>您为实现目标（例如，开发速度、资源安全或简化测试）所花费的时间和空间不会被浪费。
>
>“追求效率的另一个好处是，这个过程迫使你更深入地理解问题。”——Alex Stepanov。

## Example, bad

```c++
struct X {
    char ch;
    int i;
    std::string s;
    char ch2;

    X(const X&);
    X& operator=(const X& a);
};

X waste(const char* p) {
    // input check
    if (!p) throw Nullptr_error{};
    
    // allocate
    int n = strlen(p);
    auto buf = new char[n];
    if (!buf) throw Allocation_error{};
    
    // copy p[] to buf[]
    for (int i = 0; i < n; ++i) {
        buf[i] = p[i];
    }

    // manipulate buffer
    X x;
    x.ch = 'a';
    x.s = std::string(n);    // give x.s space for *p
    for (gsl::index i = 0; i < x.s.size(); ++i) {
        x.s[i] = buf[i];  // copy buf into x.s
    }
    delete[] buf;
    return x;
}

void driver() {
    X x = waste("Typical argument"); // call X& operator=(const X& a)
    // ...
}
```

Yes, this is a caricature, but we have seen every individual mistake in production code, and worse. Note that the layout of `X` guarantees that at least 6 bytes (and most likely more) are wasted. The spurious definition of copy operations disables move semantics so that the return operation is slow (please note that the Return Value Optimization, RVO, is not guaranteed here). The use of `new` and `delete` for `buf` is redundant; if we really needed a local string, we should use a local `string`. There are several more performance bugs and gratuitous complication.

>是的，这是一幅漫画，但我们已经看到了生产代码中的每一个错误，甚至更糟。
>
>请注意，`X`的布局保证至少浪费6个字节（很可能更多）。
>
>复制操作的虚假定义禁用了移动语义，因此返回操作很慢（请注意，这里不能保证返回值优化，RVO）。
>
>对`buf`使用`new`和`delete`是多余的，如果我们真的需要一个局部字符串，我们应该使用一个局部字符串。
>
>还有几个性能错误和不必要的复杂性。

## Example

### Bad

```c++
void lower(gsl::zstring s) {
    for (int i = 0; i < std::strlen(s); ++i) {
        s[i] = std::tolower(s[i]);
    }
}
```

This is actually an example from production code. We can see that in our condition we have `i < strlen(s)`. This expression will be evaluated on every iteration of the loop, which means that `strlen` must walk through string every loop to discover its length. While the string contents are changing, it’s assumed that `tolower` will not affect the length of the string, so it’s better to cache the length outside the loop and not incur that cost each iteration.

> 这实际上是产品代码中的一个示例。我们可以看到，在我们的条件中有`i < strlen(s)`。这个表达式将在循环的每次迭代中求值，这意味着`strlen`必须遍历`zstring`的每个循环来发现它的长度。当字符串内容发生变化时，假设`std::tolower`不会影响字符串的长度，因此最好在循环之外缓存长度，这样就不会在每次迭代时都产生这个开销。

### Good

```c++
void lower(gsl::zstring s) {
    auto len = std::strlen(s);
    for (auto i = 0; i < len; ++i) {
        s[i] = std::tolower(s[i]);
    }
}
```

## Note

An individual example of waste is rarely significant, and where it is significant, it is typically easily eliminated by an expert. However, waste spread liberally across a code base can easily be significant and experts are not always as available as we would like. The aim of this rule (and the more specific rules that support it) is to eliminate most waste related to the use of C++ before it happens. After that, we can look at waste related to algorithms and requirements, but that is beyond the scope of these guidelines.

>浪费的个别例子很少是重要的，如果它是重要的，它通常很容易被专家消除。
>
>然而，在代码库中随意分布的浪费很容易造成重大影响，而且专家并不总是像我们希望的那样容易找到。
>
>该规则（以及支持该规则的更具体的规则）的目的是在使用C++之前消除与使用C++相关的大多数浪费。
>
>之后，我们可以查看与算法和需求相关的浪费，但这超出了这些指导方针的范围。

## Enforcement

Many more specific rules aim at the overall goals of simplicity and elimination of gratuitous waste.

> 许多更具体的规则旨在实现简化和消除无端浪费的总体目标。

- Flag an unused return value from a user-defined non-defaulted postfix `operator++` or `operator--` function. Prefer using the prefix form instead. (Note: “User-defined non-defaulted” is intended to reduce noise. Review this enforcement if it’s still too noisy in practice.)

  - > 注意用户定义的非默认后缀`operator++`或`operator--`函数中未使用的返回值。**优先使用前缀形式**。
    >
    > （注：“用户自定义非默认”是为了减少噪音。如果在实践中仍然过于嘈杂，请审查此执行。） // ???

## Reference

[1] [C++ Core Guidelines P.09: Don’t waste time or space](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p9-dont-waste-time-or-space)
