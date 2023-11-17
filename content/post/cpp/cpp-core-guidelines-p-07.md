---
title: "C++ Core Guidelines P.07 注解"
date: 2023-11-17T23:52:26+08:00
description: "C++ Core Guidelines P.07 注解"
featured: true
draft: false
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
  - Best Practice
comment: true
---

# P.7: Catch run-time errors early

## Reason

Avoid “mysterious” crashes. Avoid errors leading to (possibly unrecognized) wrong results.

> 避免“神秘的”崩溃。避免错误导致（可能无法识别的）错误结果。

## Example

```c++
void increment1(int* p, int n) {	// bad: error-prone
    for (int i = 0; i < n; ++i) {
        ++p[i];
    }
}

void use1(int m) {
    const int n = 10;
    int a[n] = {};
    // ...
    increment1(a, m);	// maybe typo, maybe m <= n is supposed
                        // but assume that m == 20
    // ...
}
```

Here we made a small error in `use1` that will lead to corrupted data or a crash. The (pointer, count)-style interface leaves `increment1()` with no realistic way of defending itself against out-of-range errors. If we could check subscripts for out of range access, then the error would not be discovered until `p[10]` was accessed.

We could check earlier and improve the code:

```c++
void increment2(std::span<int> p) {
    for (int& x : p) {
        ++x;
    }
}

void use2(int m) {
    const int n = 10;
    int a[n] = {};
    // ...
    increment2({a, m});    // maybe typo, maybe m <= n is supposed
    // ...
}
```

Now, `m <= n` can be checked at the point of call (early) rather than later. If all we had was a typo so that we meant to use `n` as the bound, the code could be further simplified (eliminating the possibility of an error):

```c++
void use3(int m) {
    const int n = 10;
    int a[n] = {};
    // ...
    increment2(a);   // the number of elements of a need not be repeated
    // ...
}
```

## Example, bad

Don’t repeatedly check the same value. Don’t pass structured data as strings.

> 不要重复检查相同的值。
>
> 不要将结构化数据作为字符串传递。

```c++
Date read_date(std::istream& is);	// read date from istream
									// bad: error-prone

Date extract_date(const std::string& s);	// extract date from string

void user1(const std::string& date) {	// manipulate date
    auto d = extract_date(date);	// bad: error-prone
    // ...
}

void user2() {
    Date d = read_date(cin);
    // ...
    user1(d.to_string());
    // ...
}
```

The date is validated twice (by the `Date` constructor) and passed as a character string (unstructured data).

## Example

Excess checking can be costly. There are cases where checking early is inefficient because you might never need the value, or might only need part of the value that is more easily checked than the whole. Similarly, don’t add validity checks that change the asymptotic behavior of your interface (e.g., don’t add a `O(n)` check to an interface with an average complexity of `O(1)`).

>过多的检查可能代价高昂。
>
>有些情况下，早期检查是低效的，因为您可能永远不需要该值，或者可能只需要比整个值更容易检查的部分值。
>
>同样，不要添加改变接口渐近行为的有效性检查（例如，不要在平均复杂度为`O(1)`的接口上添加`O(n)`检查）。

```c++
class Jet {    // Physics says: e * e < x * x + y * y + z * z
    float x;
    float y;
    float z;
    float e;
public:
    Jet(float x, float y, float z, float e)
        :x(x), y(y), z(z), e(e)
    {
        // Should I check here that the values are physically meaningful?
    }

    float m() const
    {
        // Should I handle the degenerate case here?
        return std::sqrt(x * x + y * y + z * z - e * e);
    }

    // ???
};
```

The physical law for a jet ($e^2 < x^2 + y^2 + z^2$) is not an invariant because of the possibility for measurement errors.

> 由于测量误差的可能性，Jet的物理定律（$e^2 < x^2 + y^2 + z^2$）不是不变的。

## Enforcement

- Look at pointers and arrays: Do range-checking early and not repeatedly

  - >注意指针和数组：尽早进行范围检查，不要重复。

- Look at conversions: Eliminate or mark narrowing conversions

  - >注意转换：消除或标记收缩转换。

- Look for unchecked values coming from input

  - >注意来自输入的未检查值。

- Look for structured data (objects of classes with invariants) being converted into strings

  - > 注意被转换为字符串的结构化数据（具有不变量的类的对象）。

## Reference

[1] [C++ Core Guidelines P.7: Catch run-time errors early](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p7-catch-run-time-errors-early)
