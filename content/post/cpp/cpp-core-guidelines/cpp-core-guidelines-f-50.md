---
title: "C++ Core Guidelines F.50 注解"
date: 2023-12-22T21:56:06+08:00
description: "C++ Core Guidelines F.50 注解"
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

# F.50: Use a lambda when a function won’t do (to capture local variables, or to write a local function)

## Reason

Functions can’t capture local variables or be defined at local scope; if you need those things, prefer a lambda where possible, and a handwritten function object where not. On the other hand, lambdas and function objects don’t overload; if you need to overload, prefer a function (the workarounds to make lambdas overload are ornate). If either will work, prefer writing a function; use the simplest tool necessary.

函数不能捕获局部变量或被定义在局部范围内，如果需要这些，首选lambda，如果不需要则使用手写函数对象。

另一方面，lambda和函数对象不会重载，如果需要重载，最好使用函数（使lambdas重载的变通方法很华丽）。

如果哪一种都可以，最好写一个函数。

## Example

```c++
// writing a function that should only take an int or a string
// -- overloading is natural
void f(int);
void f(const std::string&);

// writing a function object that needs to capture local state and appear
// at statement or expression scope -- a lambda is natural
std::vector<work> v = lots_of_work();
for (int tasknum = 0; tasknum < max; ++tasknum) {
    pool.run([=, &v] {
        /*
        ...
        ... process 1 / max - th of v, the tasknum - th chunk
        ...
        */
    });
}
pool.join();
```

## Exception

Generic lambdas offer a concise way to write function templates and so can be useful even when a normal function template would do equally well with a little more syntax. This advantage will probably disappear in the future once all functions gain the ability to have Concept parameters.

泛型lambda提供了一种简洁的方法来编写函数模板，因此即使普通函数模板只需要多一点语法就可以做得同样好，它也很有用。一旦将来所有函数都能够拥有概念参数，这个优势可能就会消失。

## Enforcement

Warn on use of a named non-generic lambda (e.g., `auto x = [](int i) { /*...*/; };`) that captures nothing and appears at global scope. Write an ordinary function instead.

警告使用命名的非泛型lambda，该lambda不捕获任何内容并出现在全局作用域。

相反，应该写一个普通的函数。
