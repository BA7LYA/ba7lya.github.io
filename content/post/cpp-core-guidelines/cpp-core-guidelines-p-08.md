---
title: "C++ Core Guidelines P.08 注解"
date: 2023-11-18T00:20:32+08:00
description: "C++ Core Guidelines P.08 注解"
featured: true
draft: false
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

# P.08: Don’t leak any resources

> 不要泄露任何资源。

> 注：大实话、大废话。

## Reason

Even a slow growth in resources will, over time, exhaust the availability of those resources. This is particularly important for long-running programs, but is an essential piece of responsible programming behavior.

>随着时间的推移，即使是资源的缓慢增长也会耗尽这些资源的可用性。
>
>这对于长时间运行的程序尤其重要，但也是负责任的编程行为的基本部分。

## Example

### Bad

```c++
void f(char* name) {
    FILE* input = fopen(name, "r");
    // ...
    if (something) return;   // bad: if something == true, a file handle is leaked
    // ...
    fclose(input);
}
```

### Good

Prefer RAII (Resource Acquisition Is Initialization)<sup>[2]</sup>:

```c++
void f(char* name) {
    ifstream input {name};
    // ...
    if (something) return;   // OK: no leak
    // ...
}
```

## Note

A leak is colloquially “anything that isn’t cleaned up.” The more important classification is “anything that can no longer be cleaned up.” For example, allocating an object on the heap and then losing the last pointer that points to that allocation. This rule should not be taken as requiring that allocations within long-lived objects must be returned during program shutdown. For example, relying on system guaranteed cleanup such as file closing and memory deallocation upon process shutdown can simplify code. However, relying on abstractions that implicitly clean up can be as simple, and often safer.

> 通俗地说，泄漏就是“任何没有清理干净的东西”。更重要的分类是“任何不能再清理的东西”。例如，在堆上分配对象，然后丢失指向该分配的最后一个指针。
>
> 此规则不应被视为要求在程序关闭期间必须返回长寿命对象中的分配。例如，依赖于系统保证的清理（如在进程关闭时关闭文件和释放内存）可以简化代码。然而，依赖于隐式清理的抽象也同样简单，而且通常更安全。

## Note

Enforcing the lifetime safety profile<sup>[4]</sup> eliminates leaks. When combined with resource safety provided by RAII<sup>[2]</sup>, it eliminates the need for “garbage collection” (by generating no garbage). Combine this with enforcement of the type and bounds profiles<sup>[5]</sup> and you get complete type- and resource-safety, guaranteed by tools.

> 强制执行生命周期安全配置文件<sup>[4]</sup>可以消除泄漏。
>
> 当与RAII<sup>[2]</sup>提供的资源安全性结合使用时，它消除了“垃圾收集”的需要（通过不生成垃圾）。
>
> 将RAII<sup>[2]</sup>与类型和边界配置文件<sup>[5]</sup>相结合，您将获得由工具保证的完整的类型和资源安全。

## Enforcement

- Look at pointers: Classify them into non-owners (the default) and owners. Where feasible, replace owners with standard-library resource handles (as in the example above). Alternatively, mark an owner as such using `owner` from the GSL<sup>[6]</sup>.

  - > 检查指针：
    >
    > 将它们分为非所有者（默认值）和所有者。
    >
    > 在可行的情况下，用标准库资源句柄替换所有者（如上面的示例）。或者，使用GSL中的`owner`标记所有者。

- Look for naked `new` and `delete`

  - 检查裸`new`和`delete`

- Look for known resource allocating functions returning raw pointers (such as `fopen`, `malloc`, and `strdup`)

  - 查找返回裸指针的已知资源分配函数（如`fopen`，`malloc`和`strdup`）。

## Reference

[1] [C++ Core Guidelines P.08: Don’t leak any resources](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p8-dont-leak-any-resources)

[2] [C++ Core Guidelines R.01: RAII](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-raii)

[3] [C++ Core Guidelines R: The resource management section](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource)

[4] [C++ Core Guidelines Pro.lifetime: Lifetime safety profile](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#prolifetime-lifetime-safety-profile)

[5] [C++ Core Guidelines In.force: Enforcement](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#inforce-enforcement)

[6] [GSL: Guidelines support library](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#gsl-guidelines-support-library)
