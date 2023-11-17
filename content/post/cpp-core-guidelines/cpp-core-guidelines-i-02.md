---
title: "C++ Core Guidelines I.02 注解"
date: 2023-11-18T03:35:48+08:00
description: "C++ Core Guidelines I.02 注解"
featured: true
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

# I.02: Avoid non-const global variables

## Reason

Non-`const` global variables hide dependencies and make the dependencies subject to unpredictable changes.

>非`const`全局变量隐藏依赖关系，并使依赖关系受到不可预测的变化。

> 注：这里应该主要是考虑到多线程数据竞争的问题。

> 注：如果使用这个标准评判的话，C语言的编程方式岂不是都会受到质疑？

## Example

```c++
struct Data {
    // ... lots of stuff ...
} data;            // non-const data

void compute() {	// don't
    // ... use data ...
}

void output() {	// don't
    // ... use data ...
}
```

Who else might modify data?

## Warning

The initialization of global objects is not totally ordered. If you use a global object initialize it with a constant. Note that it is possible to get undefined initialization order even for `const` objects.

>全局对象的初始化不是完全有序的。
>
>如果你使用一个全局对象，用一个常量初始化它。注意，即使对于`const`对象，也有可能获得未定义的初始化顺序。

## Exception

A global object is often better than a singleton.

>全局对象通常比单例对象更好。

> 注：单例模式这么不受待见！？

## Note

Global constants are useful.

>全局常量是有用的。

## Note

The rule against global variables applies to namespace scope variables as well.

>反对全局变量的规则也适用于命名空间作用域变量。

## Alternative

If you use global (more generally namespace scope) data to avoid copying, consider passing the data as an object by reference to `const`. Another solution is to define the data as the state of some object and the operations as member functions.

> 如果您使用全局（更一般的，命名空间作用域）数据来避免复制，请考虑通过引用将数据作为对象传递给`const`。
>
> 另一种解决方案是将数据定义为某个对象的状态，将操作定义为成员函数。

## Warning

Beware of data races: If one thread can access non-local data (or data passed by reference) while another thread executes the callee, we can have a data race. Every pointer or reference to mutable data is a potential data race.

>注意数据竞争：如果一个线程可以访问非本地数据（或通过引用传递的数据），而另一个线程执行被调用者，那么就会出现数据竞争。每个指向可变数据的指针或引用都是潜在的数据竞争。

> 注：一定要用的话，加把锁是不是就可以了？！

Using global pointers or references to access and change non-`const`, and otherwise non-global, data isn’t a better alternative to non-`const` global variables since that doesn’t solve the issues of hidden dependencies or potential race conditions.

>使用全局指针或引用来访问和更改非`const`或非全局数据并不是非`const`全局变量的更好选择，因为这不能解决隐藏的依赖关系或潜在的竞争条件的问题。

## Note

You cannot have a race condition on immutable data.

> 不可变数据上不会有竞争条件。

## Note

The rule is “avoid”, not “don’t use.” Of course there will be (rare) exceptions, such as `std::cin`, `std::cout`, and `std::cerr`.

>规则是“避免”，而不是“不使用”。当然也会有（罕见的）例外，比如`std::cin`，`std::cout`和`std::cerr`。

## Enforcement

(Simple) Report all non-const variables declared at namespace scope and global pointers/references to non-const data.

> 检查在命名空间范围内声明的所有非`const`变量和指向非`const`数据的全局指针/引用。

## References

[1] [C++ Core Guidelines I.02: Avoid non-`const` global variables](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i2-avoid-non-const-global-variables)

[2] [C++ Core Guidelines F.call: Parameter passing](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#fcall-parameter-passing)
