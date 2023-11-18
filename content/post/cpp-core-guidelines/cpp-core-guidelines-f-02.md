---
title: "C++ Core Guidelines F.02 注解"
date: 2023-11-19T04:25:58+08:00
description: "C++ Core Guidelines F.02 注解"
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

# F.02: A function should perform a single logical operation

>函数应该执行单个逻辑操作。

## Reason

A function that performs a single operation is simpler to understand, test, and reuse.

> 执行单一操作的函数更容易理解、测试和重用。

## Example

### Bad

Consider:

```c++
void read_and_print()    // bad
{
    int x;
    cin >> x;
    // check for errors
    cout << x << "\n";
}
```

This is a monolith that is tied to a specific input and will never find another (different) use.

### Good

Instead, break functions up into suitable logical parts and parameterize:

```c++
int read(istream& is)    // better
{
    int x;
    is >> x;
    // check for errors
    return x;
}

void print(ostream& os, int x)
{
    os << x << "\n";
}
```

These can now be combined where needed:

```c++
void read_and_print()
{
    auto x = read(cin);
    print(cout, x);
}
```

### Good

We could further templatize `read()` and `print()` on the data type, the I/O mechanism, the response to errors, etc.

Example:

```c++
auto read = [](auto& input, auto& value)    // better
{
    input >> value;
    // check for errors
};

void print(auto& output, const auto& value)
{
    output << value << "\n";
}
```

## Enforcement

- Consider functions with more than one “out” parameter suspicious. Use return values instead, including `tuple` for multiple return values.

  - > 检查具有多个“out”参数的函数。使用返回值的方式代替，包括`std::tuple`用于多个返回值。

- Consider “large” functions that don’t fit on one editor screen suspicious. Consider factoring such a function into smaller well-named suboperations.

  - >检查不能在一个编辑器屏幕上显示的“大型”函数。
    >
    >考虑将这样的函数分解为更小的命名良好的子操作。

- Consider functions with 7 or more parameters suspicious.

  - >检查具有7个或更多参数的函数。

## Reference

[1] [C++ Core Guidelines F.02: A function should perform a single logical operation](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f2-a-function-should-perform-a-single-logical-operation)
