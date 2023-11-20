---
title: "C++ Core Guidelines F.19 注解"
date: 2023-11-20T20:40:21+08:00
description: "C++ Core Guidelines F.19 注解"
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

# F.19: For “forward” parameters, pass by `TP&&` and only `std::forward` the parameter

>对于需要转发的形参，通过`TP&&`的形式进行传递，只通过`std::forward`转发参数，不进行其他操作。

## Reason

If the object is to be passed onward to other code and not directly used by this function, we want to make this function agnostic to the argument `const`-ness and rvalue-ness.

>如果对象要传递给其他代码，而不是由这个函数直接使用，我们希望对这个函数屏蔽掉这个参数的`const`性和右值性。

In that case, and only that case, make the parameter `TP&&` where `TP` is a template type parameter – it both *ignores* and *preserves* `const`-ness and rvalue-ness. Therefore any code that uses a `TP&&` is implicitly declaring that it itself doesn’t care about the variable’s `const`-ness and rvalue-ness (because it is ignored), but that intends to pass the value onward to other code that does care about `const`-ness and rvalue-ness (because it is preserved). When used as a parameter `TP&&` is safe because any temporary objects passed from the caller will live for the duration of the function call. A parameter of type `TP&&` should essentially always be passed onward via `std::forward` in the body of the function.

>在这种情况下，也仅在这种情况下，将形参设为`TP&&`，其中`TP`是模板类型参数——它既“忽略”又“保留`const`性和右值性。
>
>因此，任何使用`TP&&`的代码都是在隐式地声明，它本身并不关心变量的`const`性和右值性（因为它被忽略了这些），而是打算将该值传递给其他关心`const`性和右值性的代码（因为它被保留了）。当用作参数时，`TP&&`是安全的，因为从调用者传递的任何临时对象将在函数调用期间存续。
>
>`TP&&`类型的参数本质上应该始终通过函数体中的`std::forward`进行传递。

## Example

Usually you forward the entire parameter (or parameter pack, using `...`) exactly once on every static control flow path:

```c++
template<class F, class... Args>
inline auto invoke(F f, Args&&... args)
{
    return f(std::forward<Args>(args)...);
}
```

## Example

Sometimes you may forward a composite parameter piecewise, each sub-object once on every static control flow path:

```c++
template<class PairLike>
inline auto test(PairLike&& pairlike)
{
    // ...
    f1(some, args, and, std::forward<PairLike>(pairlike).first);           // forward .first
    f2(and, std::forward<PairLike>(pairlike).second, in, another, call);   // forward .second
}
```

## Enforcement

- Flag a function that takes a `TP&&` parameter (where `TP` is a template type parameter name) and does anything with it other than `std::forward`ing it exactly once on every static path, or `std::forward`ing it more than once but qualified with a different data member exactly once on every static path.

  - >标记一个函数，如果这个函数接受`TP&&`形参（其中`TP`是模板类型形参名），并对其进行任何操作，而不是在每个静态路径上`std::forward`它一次，或者`std::forward`它多次，但在每个静态路径上使用不同的数据成员限定一次。

## Reference

[1] [C++ Core Guidelines F.19: For “forward” parameters, pass by `TP&&` and only `std::forward` the parameter](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f19-for-forward-parameters-pass-by-tp-and-only-stdforward-the-parameter)
