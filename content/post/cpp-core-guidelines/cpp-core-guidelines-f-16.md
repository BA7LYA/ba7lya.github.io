---
title: "C++ Core Guidelines F.16 注解"
date: 2023-11-20T20:40:07+08:00
description: "C++ Core Guidelines F.16 注解"
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

# F.16: For “in” parameters, pass cheaply-copied types by value and others by reference to `const`

>对于“输入”形参，通过值传递的方式传递低成本复制的类型，其他类型通过“const &”的形式。

## Reason

Both let the caller know that a function will not modify the argument, and both allow initialization by rvalues.

>两种形式都会让调用者知道函数不会修改参数，并且两种形式都允许通过右值进行初始化。

What is “cheap to copy” depends on the machine architecture, but two or three words (doubles, pointers, references) are usually best passed by value. When copying is cheap, nothing beats the simplicity and safety of copying, and for small objects (up to two or three words) it is also faster than passing by reference because it does not require an extra indirection to access from the function.

>什么是“低成本复制”取决于机器架构，但两或三个字（双精度，指针，引用）通常最好**按值传递**。当复制成本较低时，没有什么比复制的**简单性和安全性**更好，并且对于小对象（最多两或三个字），它也比通过引用传递更**快**，因为它不需要从函数访问额外的间接。

## Example

```c++
void f1(const string& s);	// OK: pass by reference to const; always cheap
void f2(	  string  s);	// bad: potentially expensive

void f3(	  int  x);		// OK: Unbeatable
void f4(const int& x);		// bad: overhead on access in f4()
```

For advanced uses (only), where you really need to optimize for rvalues passed to “input-only” parameters:

> 对于高级用途（仅限），当你确实需要优化传递给“仅输入”参数的右值时：

- If the function is going to unconditionally move from the argument, take it by `&&`. See F.18<sup>[2]</sup>.

  - > 如果函数将无条件地移动实参，用`&&`传参。

- If the function is going to keep a copy of the argument, in addition to passing by `const&` (for lvalues), add an overload that passes the parameter by `&&` (for rvalues) and in the body `std::move`s it to its destination. Essentially this overloads a “will-move-from”; see F.18<sup>[2]</sup>.

  - > 如果函数要保留实参的副本，除了通过`const&`（对于左值）传递外，还可以添加一个重载，通过`&&`（对于右值）传递参数，并在函数体中将其`std::move`移动到目的地。从本质上讲，这重载了“will-move-from”。

- In special cases, such as multiple “input + copy” parameters, consider using perfect forwarding. See F.19<sup>[3]</sup>.

  - >在特殊情况下，例如多个“输入+复制”参数，可以考虑使用完美转发。

## Example

```c++
int multiply(int, int); // just input ints, pass by value

// suffix is input-only but not as cheap as an int, pass by const&
std::string& concatenate(std::string&, const std::string& suffix);

void sink(std::unique_ptr<widget>);  // input only, and moves ownership of the widget
```

Avoid “esoteric techniques” such as passing arguments as `T&&` “for efficiency”. Most rumors about performance advantages from passing by `&&` are false or brittle (but see [F.18](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-consume) and [F.19](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-forward)).

> 避免“深奥的技术”，例如“为了效率”将参数传递为`T&&`。大多数关于通过`&&`来获得性能优势的谣言都是虚假的或脆弱的。

## Notes

A reference can be assumed to refer to a valid object (language rule). There is no (legitimate) “null reference.” If you need the notion of an optional value, use a pointer, `std::optional`, or a special value used to denote “no value.”

>可以假定引用引用的是一个有效的对象（语言规则）。不允许（合法的）空引用。
>
>如果您需要可选值的概念，请使用指针、`std::optional`或一个用于表示“空值”的特殊值。

## Enforcement

- (Simple) ((Foundation)) Warn when a parameter being passed by value has a size greater than `2 * sizeof(void*)`. Suggest using a reference to `const` instead.

  - >当值传递的参数的大小大于`2 * sizeof(void*)`时发出警告，建议使用`const &`替代。

- (Simple) ((Foundation)) Warn when a parameter passed by reference to `const` has a size less or equal than `2 * sizeof(void*)`. Suggest passing by value instead.

  - >当`const &`传递的形参的`size<=2 * sizeof(void*)`时发出警告。建议按值传递。

- (Simple) ((Foundation)) Warn when a parameter passed by reference to `const` is `move`d.

  - >当通过`const &`传递的形参被`std::move`时发出警告。

## Exception

To express shared ownership using `shared_ptr` types, follow R.34<sup>[4]</sup> or R.36<sup>[5]</sup>, depending on whether or not the function unconditionally takes a reference to the argument.

>要使用`std::shared_ptr`类型表达共享所有权，请遵循R.34<sup>[4]</sup>或R.36<sup>[5]</sup>，具体取决于函数是否无条件地引用参数。

## Reference

[1] [C++ Core Guidelines F.16: For “in” parameters, pass cheaply-copied types by value and others by reference to `const`](/post/cpp-core-guidelines-f-16.md)

[2] [C++ Core Guidelines F.18: For “will-move-from” parameters, pass by `X&&` and `std::move` the parameter](/post/cpp-core-guidelines-f-18.md)

[3] [C++ Core Guidelines F.19: For “forward” parameters, pass by `TP&&` and only `std::forward` the parameter](/post/cpp-core-guidelines-f-19.md)

[4] [C++ Core Guidelines R.34: Take a `shared_ptr<widget>` parameter to express shared ownership](/post/cpp-core-guidelines-r-34.md)

[5] [C++ Core Guidelines R.36: Take a `const shared_ptr<widget>&` parameter to express that it might retain a reference count to the object ???](/post/cpp-core-guidelines-r-36.md)
