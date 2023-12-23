---
title: "C++ Core Guidelines F.42 注解"
date: 2023-12-22T21:55:31+08:00
description: "C++ Core Guidelines F.42 注解"
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

# F.42: Return a `T*` to indicate a position (only)

## Reason

That’s what pointers are good for.

这就是指针的用处。

Returning a `T*` to transfer ownership is a misuse.

返回`T*`来转移所有权是一种误用。

## Example

```c++
Node* find(Node* t, const std::string& s) { // find s in a binary tree of Nodes
    if (!t || t->name == s) return t;
    if ((auto p = find(t->left, s))) return p;
    if ((auto p = find(t->right, s))) return p;
    return nullptr;
}
```

If it isn’t the `nullptr`, the pointer returned by `find` indicates a `Node` holding `s`. Importantly, that does not imply a transfer of ownership of the pointed-to object to the caller.

如果它不是`nullptr`，`find`返回的指针表示`Node`持有`s`。

重要的是，这并不意味着将指向对象的所有权转移给调用方。

## Note

Positions can also be transferred by iterators, indices, and references.

位置也可以通过迭代器、索引和引用来转移。

A reference is often a superior alternative to a pointer [if there is no need to use `nullptr`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-ptr-ref) or [if the object referred to should not change](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines???).

引用通常是指针的更好选择，如果不需要使用`nullptr`或如果被引用的对象不应该改变。

## Note

Do not return a pointer to something that is not in the caller’s scope; see [F.43](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-dangle).

不要返回一个指向不在调用者作用域内的对象的指针。

## See also

[discussion of dangling pointer prevention](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#???)

## Enforcement

- Flag `delete`, `std::free()`, etc. applied to a plain `T*`. Only owners should be deleted.
- Flag `new`, `malloc()`, etc. assigned to a plain `T*`. Only owners should be responsible for deletion.
