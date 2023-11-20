---
title: "C++ Core Guidelines I.25 注解"
date: 2023-11-19T03:12:11+08:00
description: "C++ Core Guidelines I.25 注解"
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

# I.25: Prefer empty abstract classes as interfaces to class hierarchies

>首选空抽象类作为类层次结构的接口。

## Reason

Abstract classes that are empty (have no non-static member data) are more likely to be stable than base classes with state.

>空的抽象类（没有非静态成员数据）比有状态的基类更稳定。

## Example

### Bad

You just knew that `Shape` would turn up somewhere :-)

```c++
// bad: interface class loaded with data
class Shape
{
public:
    Point center() const { return c; }
    virtual void draw() const;
    virtual void rotate(int);
    // ...
private:
    Point c;
    std::vector<Point> outline;
    Color col;
};
```

This will force every derived class to compute a center – even if that’s non-trivial and the center is never used. Similarly, not every `Shape` has a `Color`, and many `Shape`s are best represented without an outline defined as a sequence of `Point`s.

### Better than bad

Using an abstract class is better:

```c++
// better: Shape is a pure interface
class Shape
{
public:
    virtual Point center() const = 0;	// pure virtual functions
    virtual void draw() const = 0;		// pure virtual functions
    virtual void rotate(int) = 0;		// pure virtual functions
    virtual ~Shape() = default;

    // ...
    // ... no data members ...
    // ...
};
```

## Enforcement

(Simple) Warn if a pointer/reference to a class `C` is assigned to a pointer/reference to a base of `C` and the base class contains data members.

>如果将指向类`C`的指针/引用赋值给指向`C`基类的指针/引用并且基类包含数据成员，则发出警告。

## Reference

[1] [C++ Core Guidelines I.25: Prefer empty abstract classes as interfaces to class hierarchies](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i25-prefer-empty-abstract-classes-as-interfaces-to-class-hierarchies)
