---
title: "C++ Core Guidelines I.13 注解"
date: 2023-11-19T01:34:59+08:00
description: "C++ Core Guidelines I.13 注解"
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

# I.13: Do not pass an array as a single pointer

>不要通过单个指针的方式传递数组。

> 注：数组退化，丢失size信息。

## Reason

(pointer, size)-style interfaces are error-prone. Also, a plain pointer (to array) must rely on some convention to allow the callee to determine the size.

>（pointer, size）风格的接口容易出错。此外，普通指针（指向数组）必须依赖某种约定，以让被调用方确定其大小。

## Example

Consider:

```c++
void copy_n(const T* p, T* q, int n); // copy from [p:p+n) to [q:q+n)
```

What if there are fewer than `n` elements in the array pointed to by `q`? Then, we overwrite some probably unrelated memory.

What if there are fewer than `n` elements in the array pointed to by `p`? Then, we read some probably unrelated memory.

Either is undefined behavior and a potentially very nasty bug.

>如果数组中`q`所指向的元素少于`n`个会怎样？我们会误写一些可能不相关的内存。
>
>如果数组中`p`所指向的元素少于`n`个会怎样？我们会误读一些可能不相关的内存。
>
>要么是未定义的行为，要么是潜在的非常讨厌的Bug。

## Alternative

Consider using explicit spans:

```c++
void copy(span<const T> r, span<T> r2); // copy r to r2
```

## Example

### Bad

Consider:

```c++
void draw(Shape* p, int n);  // poor interface; poor code
Circle arr[10];
// ...
draw(arr, 10);
```

Passing `10` as the `n` argument might be a mistake: the most common convention is to assume `[0:n)` but that is nowhere stated. Worse is that the call of `draw()` compiled at all: there was an implicit conversion from array to pointer (array decay) and then another implicit conversion from `Circle` to `Shape`. There is no way that `draw()` can safely iterate through that array: it has no way of knowing the size of the elements.

> 将`10`作为`n`参数传递可能是错误的：最常见的惯例是假设`[0:n]`，但没有说明。
>
> 更糟糕的是，`draw()`调用根本不会通过编译：有一个从数组到指针的隐式转换（数组退化），然后另一个从`Circle`到`Shape`的隐式转换。`draw()`无法安全地遍历该数组，它无法知道元素的大小。

> 注：// TODO：做个测试

### Good

Use a support class that ensures that the number of elements is correct and prevents dangerous implicit conversions.

>使用支持类来确保元素的数量是正确的，并防止危险的隐式转换。

For example:

```c++
void draw2(span<Circle>);
Circle arr[10];
// ...
draw2(span<Circle>(arr));	// deduce the number of elements
draw2(arr);					// deduce the element type and array size

void draw3(span<Shape>);
draw3(arr);    // error: cannot convert Circle[10] to span<Shape>
```

This `draw2()` passes the same amount of information to `draw()`, but makes the fact that it is supposed to be a range of `Circle`s explicit. See ???.

### Better

```c++
// todo: ...
template<typename T>
requires std::is_base_of_v<Base, T>
void draw(std::span<T>)
{
    // ...
}
```

## Exception

Use `zstring` and `czstring` to represent C-style, zero-terminated strings. But when doing so, use `std::string_view` or `span<char>` from the GSL to prevent range errors.

## Enforcement

- (Simple) ((Bounds)) Warn for any expression that would rely on implicit conversion of an array type to a pointer type. Allow exception for `zstring`/`czstring` pointer types.

  - >对任何依赖于数组类型到指针类型的隐式转换的表达式发出警告。允许`zstring`/`czstring`指针类型的异常。

- (Simple) ((Bounds)) Warn for any arithmetic operation on an expression of pointer type that results in a value of pointer type. Allow exception for `zstring`/`czstring` pointer types.

  - > 对指针类型表达式的任何算术运算产生指针类型的值发出警告。允许`zstring`/`czstring`指针类型的异常。

## Reference

[1] [C++ Core Guidelines I.13: Do not pass an array as a single pointer](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i13-do-not-pass-an-array-as-a-single-pointer)
