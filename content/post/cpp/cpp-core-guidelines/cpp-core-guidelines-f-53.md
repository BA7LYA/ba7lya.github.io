---
title: "C++ Core Guidelines F.53 注解"
date: 2023-12-22T21:56:12+08:00
description: "C++ Core Guidelines F.53 注解"
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

# F.53: Avoid capturing by reference in lambdas that will be used non-locally, including returned, stored on the heap, or passed to another thread

## Reason

Pointers and references to locals shouldn’t outlive their scope. Lambdas that capture by reference are just another place to store a reference to a local object, and shouldn’t do so if they (or a copy) outlive the scope.

指针和对局部变量的引用不应该超出它们的作用域。

通过引用捕获的lambdas只是存储对局部对象的引用的另一个地方，如果它们（或副本）超出了作用域，就不应该这样做。

## Example

```c++
int local = 42;

// BAD
// Want a reference to local.
// Note, that after program exits this scope,
// local no longer exists, therefore
// process() call will have undefined behavior!
thread_pool.queue_work([&] { process(local); }); // pass by reference

// GOOD
// Want a copy of local.
// Since a copy of local is made, it will
// always be available for the call.
thread_pool.queue_work([=] { process(local); }); // pass by value
```

## Note

If a non-local pointer must be captured, consider using `unique_ptr`; this handles both lifetime and synchronization.

If the `this` pointer must be captured, consider using `[*this]` capture, which creates a copy of the entire object.

## Enforcement

- (Simple) Warn when capture-list contains a reference to a locally declared variable
- (Complex) Flag when capture-list contains a reference to a locally declared variable and the lambda is passed to a non-`const` and non-local context
