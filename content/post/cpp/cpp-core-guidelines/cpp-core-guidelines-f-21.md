---
title: "C++ Core Guidelines F.21 注解"
date: 2023-11-20T20:40:28+08:00
description: "C++ Core Guidelines F.21 注解"
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

# F.21: To return multiple “out” values, prefer returning a struct or tuple

>要返回多个“out”值，最好返回结构体或元组。

## Reason

A return value is self-documenting as an “output-only” value. Note that C++ does have multiple return values, by convention of using a `tuple` (including `pair`), possibly with the extra convenience of `tie` or structured bindings (C++17) at the call site. Prefer using a named struct where there are semantics to the returned value. Otherwise, a nameless `tuple` is useful in generic code.

>返回值作为“out-only”值会自记录。请注意，C++确实有多个返回值，通过使用元组（包括`std::pair`）的惯例，可能在调用现场使用`std::tie`或结构化绑定（C++ 17）会格外方便。如果返回值有语义，建议使用命名的结构体。另外，一个无名的元组在泛型代码中也是有用的。

## Example

### Bad

```c++
// BAD: output-only parameter documented in a comment
int f(
    const std::string& input,
    std::string& output_data	/*output only*/
)
{
    // ...
    output_data = something();
    return status;
}
```

### Good

```c++
// GOOD: self-documenting
std::tuple<int, std::string> f(const std::string& input) {
    // ...
    return {status, something()};
}
```

C++98’s standard library already used this style, because a `pair` is like a two-element `tuple`.

>C++98的标准库已经使用了这种风格，因为`std::pair`就像一个包含两个元素的`std::tuple`。

For example, given a `std::set<std::string> my_set`, consider:

```c++
// C++98
result = my_set.insert("Hello");
if (result.second) {
    do_something_with(result.first);    // workaround
}
```

With C++11 we can write this, putting the results directly in existing local variables:

```c++
Sometype iter;			// default initialize if we haven't already
Someothertype success;	// used these variables for some other purpose

std::tie(iter, success) = my_set.insert("Hello");	// normal return value
if (success) {
    do_something_with(iter);
}
```

With C++17 we are able to use “structured bindings” to declare and initialize the multiple variables:

```c++
if (auto [ iter, success ] = my_set.insert("Hello"); success) {
    do_something_with(iter);
}
```

## Exception

Sometimes, we need to pass an object to a function to manipulate its state. In such cases, passing the object by reference `T&` is usually the right technique. Explicitly passing an in-out parameter back out again as a return value is often not necessary.

>有时，我们需要将对象传递给函数以操纵其状态。
>
>在这种情况下，通过`T&`传递对象通常是正确的技术。通常不需要显式地将输入输出参数作为返回值再次传递回来。

For example:

```c++
istream& operator>>(std::istream& in, std::string& s);    // much like std::operator>>()

for (std::string s; in >> s; ) {
    // do something with line
}
```

Here, both `s` and `in` are used as in-out parameters. We pass `in` by (non-`const`) reference to be able to manipulate its state. We pass `s` to avoid repeated allocations. By reusing `s` (passed by reference), we allocate new memory only when we need to expand `s`’s capacity. This technique is sometimes called the “caller-allocated out” pattern and is particularly useful for types, such as `string` and `vector`, that needs to do free store allocations.

>在这里，`s`和`in`都用作"in-out"参数。我们通过（非`const`）引用传递`in`来操纵它的状态。我们传递`s`以避免重复分配。
>
>通过重用`s`（通过引用传递），我们只在需要扩展`s`的容量时才分配新的内存。这种技术有时被称为"caller-allocated out"模式，对于需要进行自由存储分配的类型（如`std::string`和`std::vector`）特别有用。

To compare, if we passed out all values as return values, we would get something like this:

```c++
std::pair<std::istream&, std::string> get_string(std::istream& in) {	// not recommended
    std::string s;
    in >> s;
    return {in, std::move(s)};
}

for (auto p = get_string(cin); p.first; p.second = get_string(p.first).second) {
    // do something with p.second
}
```

We consider that significantly less elegant with significantly less performance.

> 我们认为这明显不够优雅，性能也明显下降。

For a truly strict reading of this rule (F.21), the exception isn’t really an exception because it relies on in-out parameters, rather than the plain out parameters mentioned in the rule. However, we prefer to be explicit, rather than subtle.

>对于该规则的真正严格的解读（F.21），例外并不是真正的例外，因为它依赖于"in-out"参数，而不是规则中提到的普通"out"参数。
>
>然而，我们更喜欢明确的，而不是微妙的。

## Note

In many cases, it can be useful to return a specific, user-defined type.

For example:

```c++
struct Distance {
    int value;
    int unit = 1;	// 1 means meters
};

// access d1.value and d1.unit
Distance d1 = measure(obj1);

// access d2.value and d2.unit
auto d2 = measure(obj2);

// access value and unit;
// somewhat redundant to people who know measure()
auto [value, unit] = measure(obj3);

// don't; it's likely to be confusing
auto [x, y] = measure(obj4);
```

The overly-generic `std::pair` and `std::tuple` should be used only when the value returned represents independent entities rather than an abstraction.

>过于泛化的`std::pair`和`std::tuple`应该仅在返回值代表独立实体而不是抽象时使用。

Another example, use a specific type along the lines of `std::variant<T, error_code>`, rather than using the generic `std::tuple`.

>另一个例子，使用`std::variant<T, error_code>`这样的特定类型，而不是使用泛化的`std::tuple`。

## Note

When the `std::tuple` to be returned is initialized from local variables that are expensive to copy, explicit `move` may be helpful to avoid copying:

```c++
std::pair<LargeObject, LargeObject> f(const std::string& input) {
    LargeObject large1 = g(input);
    LargeObject large2 = h(input);
    // ...
    return { std::move(large1), std::move(large2) }; // no copies
}
```

Alternatively,

```c++
std::pair<LargeObject, LargeObject> f(const std::string& input) {
    // ...
    return { g(input), h(input) }; // no copies, no moves
}
```

Note this is different from the `return move(...)` anti-pattern from ES.56<sup>[2]</sup>.

## Enforcement

- Output parameters should be replaced by return values. An output parameter is one that the function writes to, invokes a non-`const` member function, or passes on as a non-`const`.

## Reference

[1] [C++ Core Guidelines F.21: To return multiple “out” values, prefer returning a struct or tuple](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f21-to-return-multiple-out-values-prefer-returning-a-struct-or-tuple)

[2] [C++ Core Guidelines ES.56: Write `std::move()` only when you need to explicitly move an object to another scope](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-move)
