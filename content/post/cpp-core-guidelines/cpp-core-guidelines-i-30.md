---
title: "C++ Core Guidelines I.30 注解"
date: 2023-11-19T03:22:54+08:00
description: "C++ Core Guidelines I.30 注解"
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

# I.30: Encapsulate rule violations

>封装违反了规则的代码。

## Reason

To keep code simple and safe. Sometimes, ugly, unsafe, or error-prone techniques are necessary for logical or performance reasons. If so, keep them local, rather than “infecting” interfaces so that larger groups of programmers have to be aware of the subtleties. Implementation complexity should, if at all possible, not leak through interfaces into user code.

>保持代码简单和安全。
>
>有时，出于逻辑或性能原因，丑陋、不安全或容易出错的技术是必要的。
>
>如果是这样，保持它们在本地，而不是“感染”接口。如果可能的话，实现复杂性不应该通过接口泄漏到用户代码中。

## Example

Consider a program that, depending on some form of input (e.g., arguments to `main`), should consume input from a file, from the command line, or from standard input. We might write

```c++
bool owned;
gsl::owner<istream*> inp;
switch (source) {
    case std_in:        owned = false; inp = &cin;                       break;
    case command_line:  owned = true;  inp = new istringstream{argv[2]}; break;
    case file:          owned = true;  inp = new ifstream{argv[2]};      break;
}
istream& in = *inp;
```

This violated the rule against uninitialized variables, the rule against ignoring ownership, and the rule against magic constants. In particular, someone has to remember to somewhere write

```c++
if (owned) delete inp;
```

We could handle this particular example by using `unique_ptr` with a special deleter that does nothing for `cin`, but that’s complicated for novices (who can easily encounter this problem) and the example is an example of a more general problem where a property that we would like to consider static (here, ownership) needs infrequently be addressed at run time. The common, most frequent, and safest examples can be handled statically, so we don’t want to add cost and complexity to those. But we must also cope with the uncommon, less-safe, and necessarily more expensive cases. Such examples are discussed in C++ Memory Model<sup>[2]</sup>.

So, we write a class

```c++
class Istream { [[gsl::suppress(lifetime)]]
public:
    enum Opt { from_line = 1 };
    Istream() { }
    Istream(czstring p) : owned{true}, inp{new ifstream{p}} {}            // read from file
    Istream(czstring p, Opt) : owned{true}, inp{new istringstream{p}} {}  // read from command line
    ~Istream() { if (owned) delete inp; }
    operator istream&() { return *inp; }
private:
    bool owned = false;
    istream* inp = &cin;
};
```

Now, the dynamic nature of `istream` ownership has been encapsulated. Presumably, a bit of checking for potential errors would be added in real code.

> 现在，`istream`所有权的动态特性已经被封装。
>
> 据推测，在实际代码中会添加一些对潜在错误的检查。

## Enforcement

- Hard, it is hard to decide what rule-breaking code is essential
- Flag rule suppression that enable rule-violations to cross interfaces

## Reference

[1] [C++ Core Guidelines I.30: Encapsulate rule violations](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i30-encapsulate-rule-violations)

[2] [C++ Memory Model](http://www.stroustrup.com/resource-model.pdf)
