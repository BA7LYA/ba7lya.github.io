---
title: "C++ Core Guidelines I.04 注解"
date: 2023-11-18T04:13:33+08:00
description: "C++ Core Guidelines I.04 注解"
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
  - C++ Best Practices
  - C++ Core Guidelines
comment: true
---

# I.04: Make interfaces precisely and strongly typed

>使用精确和强类型的接口。

## Reason

Types are the simplest and best documentation, improve legibility due to their well-defined meaning, and are checked at compile time. Also, precisely typed code is often optimized better.

>**类型是最简单和最好的文档**，由于其定义良好的含义，可以提高可读性，并且在编译时进行检查。
>
>而且，精确类型的代码通常会得到更好的优化。

## Example

### Bad

Consider:

```c++
void pass(void* data);	// weak and under qualified type void* is suspicious
```

Callers are unsure what types are allowed and if the data may be mutated as `const` is not specified. Note all pointer types implicitly convert to `void*`, so it is easy for callers to provide this value.

>调用者不确定允许的类型是什么，以及如果没有指定`const`，数据是否可能被改变。
>
>注意，所有指针类型都隐式转换为`void*`，因此调用者很容易提供此值。

The callee must `static_cast` data to an unverified type to use it. That is error-prone and verbose.

>被调用方必须将数据`static_cast`为未经验证的类型才能使用它。这很容易出错，而且冗长。

Only use `const void*` for passing in data in designs that are indescribable in C++. Consider using a `variant` or a pointer to base instead.

>只有在C++中无法描述的设计中传入数据时才使用`const void*`。
>
>考虑使用`std::variant`或指向基类的指针（多态的方式）。

### Alternative

Often, a template parameter can eliminate the `void*` turning it into a `T*` or `T&`. For generic code these `T`s can be general or concept constrained template parameters.

>通常，模板参数可以消除`void*`，将其转换为`T*`或`T&`。对于泛型代码，这些`T`可以是一般的或概念约束的模板参数。

## Example

### Bad

Consider:

```c++
draw_rect(100, 200, 100, 500);	// what do the numbers specify?
draw_rect(p.x, p.y, 10, 20);	// what units are 10 and 20 in?
```

It is clear that the caller is describing a rectangle, but it is unclear what parts they relate to. Also, an `int` can carry arbitrary forms of information, including values of many units, so we must guess about the meaning of the four `int`s. Most likely, the first two are an `x`,`y` coordinate pair, but what are the last two?

>很明显，调用者正在描述一个矩形，但不清楚它们与哪些部分相关。
>
>此外，一个`int`类型可以携带任意形式的信息，包括许多单位的值，所以我们必须猜测这四个`int`类型的含义。最有可能的是，前两个是`x`，`y`坐标对，但最后两个是什么?

> 注：最后两个参数可以是对角的坐标`x1,y1`，也可以是矩形的长和宽`hight,width`。

### Good

Comments and parameter names can help, but we could be explicit:

```c++
void draw_rectangle(Point top_left, Point bottom_right);
void draw_rectangle(Point top_left, Size height_width);

draw_rectangle(p, Point{10, 20});  // two corners
draw_rectangle(p, Size{10, 20});   // one corner and a (height, width) pair
```

Obviously, we cannot catch all errors through the static type system (e.g., the fact that a first argument is supposed to be a top-left point is left to convention (naming and comments)).

> 显然，我们不能通过静态类型系统捕获所有错误(例如，第一个参数应该是左上角的事实留给约定（命名和注释））。

## Example

### Bad

Consider:

```c++
set_settings(true, false, 42);	// what do the numbers specify?
```

The parameter types and their values do not communicate what settings are being specified or what those values mean.

> 参数类型和它们的值不能传达指定的设置或这些值的含义。

### Good

This design is more explicit, safe and legible:

```c++
// create a object
alarm_settings s{};

// edit the object
s.enabled     = true;
s.displayMode = alarm_settings::mode::spinning_light;
s.frequency   = alarm_settings::every_10_seconds;

// pass the object
set_settings(s);
```

For the case of a set of boolean values consider using a flags `enum`; a pattern that expresses a set of boolean values.

> 对于一组布尔值的情况，考虑使用标志`enum`表示一组布尔值的模式。

```c++
enable_lamp_options(lamp_option::on | lamp_option::animate_state_transitions);
```

## Example

### Bad

In the following example, it is not clear from the interface what `time_to_blink` means: Seconds? Milliseconds?

```c++
void blink_led(int time_to_blink) {	// bad -- the unit is ambiguous
    // ...
    // do something with time_to_blink
    // ...
}

void use() {
    blink_led(2);
}
```

### Good

`std::chrono::duration` types helps making the unit of time duration explicit.

```c++
void blink_led(milliseconds time_to_blink) {	// good -- the unit is explicit
    // ...
    // do something with time_to_blink
    // ...
}

void use() {
    blink_led(1500ms);
}
```

The function can also be written in such a way that it will accept any time duration unit.

```c++
template<class rep, class period>
void blink_led(duration<rep, period> time_to_blink) {	// good -- accepts any unit
    // assuming that millisecond is the smallest relevant unit
    auto milliseconds_to_blink = duration_cast<milliseconds>(time_to_blink);
    // ...
    // do something with milliseconds_to_blink
    // ...
}

void use() {
    blink_led(2s);
    blink_led(1500ms); // std::literals::chrono_literals
}
```

## Enforcement

- (Simple) Report the use of `void*` as a parameter or return type.

  - >检查使用`void*`作为参数或返回类型。

  - > 注：使用模板参数作、基类指针、`std::variant`为优化替代方案。

- (Simple) Report the use of more than one `bool` parameter.

  - >报告使用了多个`bool`参数。

  - > 注：使用`enum`作为优化替代方案。

- (Hard to do well) Look for functions that use too many primitive type arguments.

  - >检查使用过多基本类型参数的函数。

    >注：将多个参数合并为一个参数类，替代零散的多个同类型参数的传递方案。

## Reference

[1] [C++ Core Guidelines I.04: Make interfaces precisely and strongly typed](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i4-make-interfaces-precisely-and-strongly-typed)
