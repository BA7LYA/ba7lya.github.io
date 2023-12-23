---
title: "C++ Core Guidelines F.47 注解"
date: 2023-12-22T21:55:41+08:00
description: "C++ Core Guidelines F.47 注解"
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

# F.47: Return `T&` from assignment operators

## Reason

The convention for operator overloads (especially on concrete types) is for `operator=(const T&)` to perform the assignment and then return (non-`const`) `*this`. This ensures consistency with standard-library types and follows the principle of “do as the ints do.”

操作符重载（特别是具体类型）的约定是让`operator=(const T&)`执行赋值操作，然后返回（非`const`)`*this`。

这确保了与标准库类型的一致性，并遵循了“像整型一样做”的原则。

## Note

Historically there was some guidance to make the assignment operator return `const T&`. This was primarily to avoid code of the form `(a = b) = c` – such code is not common enough to warrant violating consistency with standard types.

历史上有一些指导要求赋值操作符返回`const T&`。

这主要是为了避免像`(a = b) = c`这样的代码，这样的代码并不常见，不足以保证违反标准类型的一致性。

## Example

```c++
class Foo
{
public:
  // ...
  Foo& operator=(const Foo& rhs) {
    // Copy members.
    // ...
    return *this;
  }
};
```

## Enforcement

This should be enforced by tooling by checking the return type (and return value) of any assignment operator.
