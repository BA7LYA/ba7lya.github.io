---
title: "More C++ Idioms - Address Of"
date: 2023-12-05T11:12:16+08:00
description: "More C++ Idioms - Address Of"
featured: false
toc: false
usePageBundles: false
featureImage: ""
featureImageAlt: ''
featureImageCap: ''
thumbnail: "/images/iso_cpp_logo.png"
shareImage: "/images/iso_cpp_logo.png"
codeLineNumbers: true
figurePositionShow: true
categories:
  - C++
tags:
  - C++ Idioms
comment: true
---

## Intent

Find the address of an object of a class that has an overloaded unary ampersand (`&`) operator.

>查找具有重载一元`&`操作符的类的对象的地址。

## Also Known As

This idiom has no known alias.

## Motivation

C++ allows overloading of the unary ampersand (`&`) operator for class types. The return type of such an operator need not be the actual address of the object. Intentions of such a class are highly debatable, but the language allows it nevertheless. The address-of idiom is a way to find the real address of an object irrespective of the overloaded unary ampersand operator and its access protection.

>C++允许重载类类型的一元`&`操作符。这种操作符的返回类型不必是对象的实际地址。
>
>这样一个类的意图是非常有争议的，但是语言允许这样做。
>
>Address-Of 惯用法是一种找到对象的真实地址的方法，而不考虑重载的一元`&`操作符及其访问保护。

In the example below, the `main` function fails to compile because the operator `&` of the class `nonaddressable` is `private`. Even if it were accessible, a conversion from its return type `double` to a pointer would not have been possible or meaningful.

>在下面的例子中，`main`函数无法编译，因为类`nonaddressable`的操作符`&`是私有的。
>
>即使它是可访问的，从它的返回类型`double`到指针的转换也是不可能的，也没有意义。

```c++
class nonaddressable 
{
public:
    typedef double useless_type;
private:
    useless_type operator&() const;
};

int main()
{
  nonaddressable na;
  nonaddressable * naptr = &na;  // Error: operator & of type nonadressable is private.
}
```

## Solution and Sample Code

The Address-of idiom retrieves the address of an object using a series of casts.

```c++
template <class T>
T * addressof(T & v)
{
  return reinterpret_cast<T *>(& const_cast<char&>(reinterpret_cast<const volatile char &>(v)));
}

int main()
{
  nonaddressable na;
  nonaddressable * naptr = addressof(na);  // No more compiler error.
}
```

### C++11

In C++11 the template `std::addressof`, in the `<memory>` header, was added to solve this problem.

Since C++17 the template is also marked `constexpr`.

## Known Uses

Boost's `addressof` utility.

## Related Idioms



## References

[1] [The template `std::addressof`](http://en.cppreference.com/w/cpp/memory/addressof)

[2] [Boost `addressof` utility](https://www.boost.org/doc/libs/1_83_0/libs/core/doc/html/core/addressof.html)
