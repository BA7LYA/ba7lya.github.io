---
title: "C++ Core Guidelines F.43 注解"
date: 2023-12-22T21:55:33+08:00
description: "C++ Core Guidelines F.43 注解"
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

# F.43: Never (directly or indirectly) return a pointer or a reference to a local object

## Reason

To avoid the crashes and data corruption that can result from the use of such a dangling pointer.

以避免使用悬空指针可能导致的崩溃和数据损坏。

## Example, bad

After the return from a function its local objects no longer exist:

```c++
int* f() {
    int fx = 9;
    return &fx;  // BAD
}

void g(int* p) { // looks innocent enough :)
    int gx;
    cout << "*p == " << *p << '\n';
    *p = 999;
    cout << "gx == " << gx << '\n';
}

void h() {
    int* p = f();
    int z = *p;  // read from abandoned stack frame (bad)
    g(p);        // pass pointer to abandoned stack frame to function (bad)
}
```

Here on one popular implementation I got the output:

```c++
*p == 999
gx == 999
```

I expected that because the call of `g()` reuses the stack space abandoned by the call of `f()` so `*p` refers to the space now occupied by `gx`.

- Imagine what would happen if `fx` and `gx` were of different types.
- Imagine what would happen if `fx` or `gx` was a type with an invariant.
- Imagine what would happen if more that dangling pointer was passed around among a larger set of functions.
- Imagine what a cracker could do with that dangling pointer.

Fortunately, most (all?) modern compilers catch and warn against this simple case.

## Note

This applies to references as well:

```c++
int& f() {
    int x = 7;
    // ...
    return x;  // Bad: returns reference to object that is about to be destroyed
}
```

## Note

This applies only to non-`static` local variables. All `static` variables are (as their name indicates) statically allocated, so that pointers to them cannot dangle.

## Example, bad

Not all examples of leaking a pointer to a local variable are that obvious:

```c++
int* glob;       // global variables are bad in so many ways

template<class T>
void steal(T x) {
    glob = x();  // BAD
}

void f() {
    int i = 99;
    steal([&] { return &i; });
}

int main() {
    f();
    cout << *glob << '\n';
}
```

Here I managed to read the location abandoned by the call of `f`.

The pointer stored in `glob` could be used much later and cause trouble in unpredictable ways.

## Note

The address of a local variable can be “returned”/leaked by a return statement, by a `T&` out-parameter, as a member of a returned object, as an element of a returned array, and more.

局部变量的地址可以通过`return`语句、`T&`出参、作为返回对象的成员、作为返回数组的元素等方式“返回”/泄露。

## Note

Similar examples can be constructed “leaking” a pointer from an inner scope to an outer one; such examples are handled equivalently to leaks of pointers out of a function.

类似的例子可以构造为将指针从内部作用域“泄漏”到外部作用域，此类示例的处理等同于函数指针泄漏。

A slightly different variant of the problem is placing pointers in a container that outlives the objects pointed to.

该问题的一个稍微不同的变体是，将指针放置在比所指向的对象寿命更长的容器中。

## See also

Another way of getting dangling pointers is [pointer invalidation](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???). It can be detected/prevented with similar techniques.

## Enforcement

- Compilers tend to catch return of reference to locals and could in many cases catch return of pointers to locals.
- Static analysis can catch many common patterns of the use of pointers indicating positions (thus eliminating dangling pointers)
