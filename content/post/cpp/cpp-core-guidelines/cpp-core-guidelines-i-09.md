---
title: "C++ Core Guidelines I.09 注解"
date: 2023-11-19T00:32:45+08:00
description: "C++ Core Guidelines I.09 注解"
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

# I.09: If an interface is a template, document its parameters using concepts

>如果接口是模板，则使用概念记录其参数。

## Reason

Make the interface precisely specified and compile-time checkable in the (not so distant) future.

> 在不远的将来，精确指定接口并使其在编译时可检查。

## Example

Use the C++20 style of requirements specification. For example:

```c++
template<typename Iter, typename Val>
	requires input_iterator<Iter>
          && equality_comparable_with<iter_value_t<Iter>, Val>
Iter find(Iter first, Iter last, Val v)
{
	// ...
}
```

## Enforcement

Warn if any non-variadic template parameter is not constrained by a concept (in its declaration or mentioned in a requires clause).

> 如果任何非可变参数模板参数不受概念约束（在其声明中或在require子句中提到），则发出警告。

## Reference

[1] [C++ Core Guidelines I.09: If an interface is a template, document its parameters using concepts](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i9-if-an-interface-is-a-template-document-its-parameters-using-concepts)

[2] [C++ Core Guidelines T.gp: Generic programming](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#tgp-generic-programming)

[3] [C++ Core Guidelines T.concepts: Concept rules](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#tconcepts-concept-rules)
