---
title: "C++ Core Guidelines F.20 注解"
date: 2023-11-20T20:40:25+08:00
description: "C++ Core Guidelines F.20 注解"
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

# F.20: For “out” output values, prefer return values to output parameters

>对于“out”类型的参数，选择返回值而不是输出参数。

## Reason

A return value is self-documenting, whereas a `&` could be either in-out or out-only and is liable to be misused.

This includes large objects like standard containers that use implicit move operations for performance and to avoid explicit memory management.

>返回值是自记录的，而`&`可能是"in-out"或仅"out"，容易被误用。
>
>这包括像标准容器这样的大型对象，它们使用隐式移动操作来提高性能并避免显式内存管理。

If you have multiple values to return, use a `std::tuple` or similar multi-member type.

>如果要返回多个值，请使用`std::tuple`或类似的多成员类型。

## Example

### Bad

```c++
// Bad: place pointers to elements with value x in-out
void find_all(
    const std::vector<int>&,
    std::vector<const int*>& out,
    int x
);
```

### Good

```c++
// OK: return pointers to elements with the value x
std::vector<const int*> find_all(
    const std::vector<int>&,
    int x
);
```

## Note

A `struct` of many (individually cheap-to-move) elements might be in aggregate expensive to move.

>由许多元素组成的`struct`（单独移动起来很便宜），可能在整体上移动起来很昂贵。

## Exceptions

- For non-concrete types, such as types in an inheritance hierarchy, return the object by `unique_ptr` or `shared_ptr`.

  - >对于非具体类型，例如继承层次结构中的类型，通过`unique_ptr`或`shared_ptr`返回对象。

- If a type is expensive to move (e.g., `array<BigTrivial>`), consider allocating it on the free store and return a handle (e.g., `unique_ptr`), or passing it in a reference to non-`const` target object to fill (to be used as an out-parameter).

  - >如果类型的移动代价很高（例如，`array<BigTrivial>`），请考虑在自由存储区分配它并返回一个句柄（例如，`unique_ptr`），或者将其传递给非`const`目标对象的引用来填充（用作输出参数时）。

- To reuse an object that carries capacity (e.g., `std::string`, `std::vector`) across multiple calls to the function in an inner loop: [treat it as an in/out parameter and pass by reference](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-out-multi).

  - >在内部循环中多次调用某函数时如果要重用一个承载容量的对象（例如，`std::string`，`std::vector`）：将其视为一个"in-out"形参，并通过引用的方式传递。

## Example

Assuming that `Matrix` has move operations (possibly by keeping its elements in a `std::vector`):

```c++
Matrix operator+(const Matrix& a, const Matrix& b)
{
    Matrix res;
    // ... fill res with the sum ...
    return res;
}

Matrix x = m1 + m2;  // move constructor
y = m3 + m3;         // move assignment
```

## Note

The return value optimization doesn’t handle the assignment case, but the move assignment does.

>返回值优化（RVO）不处理赋值情况，但移动赋值会处理。

> 注：？？？

## Example

```c++
struct Package {      // exceptional case: expensive-to-move object
    char header[16];
    char load[2024 - 16];
};

Package fill();       // Bad: large return value
void fill(Package&);  // OK

int val();            // OK
void val(int&);       // Bad: Is val reading its argument
```

## Enforcement

- Flag reference to non-`const` parameters that are not read before being written to and are a type that could be cheaply returned; they should be “out” return values.

  - >标志对非`const`形参的引用，这些形参在写入之前不需要读取，并且是一种可以廉价返回的类型，它们应该是“out”返回值。

## Reference

[1] [C++ Core Guidelines F.20: For “out” output values, prefer return values to output parameters](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f20-for-out-output-values-prefer-return-values-to-output-parameters)

[2] [C++ Core Guidelines F.21: To return multiple “out” values, prefer returning a struct or tuple](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f21-to-return-multiple-out-values-prefer-returning-a-struct-or-tuple)
