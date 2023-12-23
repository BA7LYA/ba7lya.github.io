---
title: "C++ Core Guidelines F.55 注解"
date: 2023-12-22T21:56:17+08:00
description: "C++ Core Guidelines F.55 注解"
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

# F.55: Don’t use `va_arg` arguments

## Reason

Reading from a `va_arg` assumes that the correct type was actually passed. Passing to varargs assumes the correct type will be read. This is fragile because it cannot generally be enforced to be safe in the language and so relies on programmer discipline to get it right.

从`va_arg`中读取会假设实际传递了正确的类型。传递给变参会假设将读取正确的类型。

这是脆弱的，因为它通常不能在语言中强制执行为安全的，因此依赖于程序员的纪律来实现它。

>你妹，那你设计这功能还不让人用！！！

## Example

```c++
int sum(...) {
    // ...
    while (/*...*/) {
        result += va_arg(list, int); // BAD, assumes it will be passed ints
    }
    // ...
}

sum(3, 2); // ok
sum(3.14159, 2.71828); // BAD, undefined

template<class ...Args>
auto sum(Args... args) { // GOOD, and much more flexible
    return (... + args); // note: C++17 "fold expression"
}

sum(3, 2); // ok: 5
sum(3.14159, 2.71828); // ok: ~5.85987
```

## Alternatives

- overloading
- variadic templates
- `variant` arguments
- `initializer_list` (homogeneous)

## Note

Declaring a `...` parameter is sometimes useful for techniques that don’t involve actual argument passing, notably to declare “take-anything” functions so as to disable “everything else” in an overload set or express a catchall case in a template metaprogram.

## Enforcement

- Issue a diagnostic for using `va_list`, `va_start`, or `va_arg`.
- Issue a diagnostic for passing an argument to a vararg parameter of a function that does not offer an overload for a more specific type in the position of the vararg. To fix: Use a different function, or `[[suppress(types)]]`.
