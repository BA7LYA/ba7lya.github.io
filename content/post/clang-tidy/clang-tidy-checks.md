---
title: "Clang-Tidy Checks"
date: 2023-11-22T21:48:37+08:00
description: "Clang-Tidy Checks"
featured: true
toc: true
usePageBundles: false
featureImage: "/images/LLVM-logo.png"
featureImageAlt: 'C++ Core Guidelines'
featureImageCap: 'source: https://llvm.org/Logo.html'
thumbnail: "/images/LLVM-logo.png"
shareImage: "/images/LLVM-logo.png"
codeLineNumbers: true
figurePositionShow: true
categories:
  - C++
tags:
  - C++ Best Practices
  - Clang-Tidy
comment: true
---

# Clang-Tidy Checks

> 注：Clang-Tidy是C++最佳实践之集大成者，规则众多，文档清晰，实践性强，易于使用，支持定制规则，十分强大。

## abseil-*

Checks related to Abseil library.

> 注：本项主要针对Google的Abseil库。

### abseil-cleanup-ctad

Suggests switching the initialization pattern of `absl::Cleanup` instances from the factory function to class template argument deduction (CTAD), in C++17 and higher.

>建议在C++ 17和更高版本中将`absl::Cleanup`实例的初始化模式从工厂函数切换到类模板参数推导（CTAD）<sup>[2]</sup>。

Original

```c++
auto c1 = absl::MakeCleanup([] {});
const auto c2 = absl::MakeCleanup(std::function<void()>([] {}));
```

Suggestion

```c++
absl::Cleanup c1 = [] {};
const absl::Cleanup c2 = std::function<void()>([] {});
```

### abseil-duration-addition

Check for cases where addition should be performed in the `absl::Time` domain. When adding two values, and one is known to be an `absl::Time`, we can infer that the other should be interpreted as an `absl::Duration` of a similar scale, and make that inference explicit.

>检查应该在`absl::Time`域中执行加法的情况。当两个值相加时，其中一个已知为`absl::Time`，我们可以推断出另一个应该被解释为类似尺度的`absl::Duration`，让这个推论更明确。

Examples:

```c++
// Original - Addition in the integer domain
int x;
absl::Time t;
int result = absl::ToUnixSeconds(t) + x;

// Suggestion - Addition in the absl::Time domain
int result = absl::ToUnixSeconds(t + absl::Seconds(x));
```

### abseil-duration-comparison

Checks for comparisons which should be in the `absl::Duration` domain instead of the floating point or integer domains.

>检查比较是否应该在`absl::Duration`域中，而不是浮点数或整数域中。

Note: In cases where a `Duration` was being converted to an integer and then compared against a floating-point value, truncation during the `Duration` conversion might yield a different result. In practice this is very rare, and still indicates a bug which should be fixed.

> 在将`Duration`转换为整数然后与浮点值进行比较的情况下，在`Duration`转换期间进行截断可能会产生不同的结果。
>
> 在实践中，这是非常罕见的，仍然表明这一个应该修复的错误。

Examples 1:

```c++
// Original - Comparison in the floating point domain
double x;
absl::Duration d;
if (x < absl::ToDoubleSeconds(d)) ...

// Suggested - Compare in the absl::Duration domain instead
if (absl::Seconds(x) < d) ...
```

Examples 2:

```c++
// Original - Comparison in the integer domain
int x;
absl::Duration d;
if (x < absl::ToInt64Microseconds(d)) ...

// Suggested - Compare in the absl::Duration domain instead
if (absl::Microseconds(x) < d) ...
```

### abseil-duration-conversion-cast

Checks for casts of `absl::Duration` conversion functions, and recommends the right conversion function instead.

>检查`absl::Duration`转换函数的强制转换，并推荐正确的转换函数。

Examples 1:

```c++
// Original - Cast from a double to an integer
absl::Duration d;
int i = static_cast<int>(absl::ToDoubleSeconds(d));

// Suggested - Use the integer conversion function directly.
int i = absl::ToInt64Seconds(d);
```

Examples 2:

```c++
// Original - Cast from a double to an integer
absl::Duration d;
double x = static_cast<double>(absl::ToInt64Seconds(d));

// Suggested - Use the integer conversion function directly.
double x = absl::ToDoubleSeconds(d);
```

### abseil-duration-division

`absl::Duration` arithmetic works like it does with integers. That means that division of two `absl::Duration` objects returns an `int64` with any fractional component truncated toward 0.

>`absl::Duration`算术运算的工作方式与整数类似。这意味着两个`absl::Duration`对象的除法返回一个`int64`，其中任何小数部分都被截断为0。

For example:

```c++
absl::Duration d = absl::Seconds(3.5);
int64 sec1 = d / absl::Seconds(1);     // Truncates toward 0.
int64 sec2 = absl::ToInt64Seconds(d);  // Equivalent to division.
assert(sec1 == 3 && sec2 == 3);

double dsec = d / absl::Seconds(1);  // WRONG: Still truncates toward 0.
assert(dsec == 3.0);
```

If you want floating-point division, you should use either the `absl::FDivDuration()` function, or one of the unit conversion functions such as `absl::ToDoubleSeconds()`.

>如果你想要浮点除法，你应该使用`absl::FDivDuration()`函数，或者一个单位转换函数，比如`absl:: todoublesseconds()`。

For example:

```c++
absl::Duration d = absl::Seconds(3.5);
double dsec1 = absl::FDivDuration(d, absl::Seconds(1));  // GOOD: No truncation.
double dsec2 = absl::ToDoubleSeconds(d);                 // GOOD: No truncation.
assert(dsec1 == 3.5 && dsec2 == 3.5);
```

This check looks for uses of `absl::Duration` division that is done in a floating-point domain, and recommends the use of a function that returns a floating-point value.

>该检查查找在浮点域中使用的`absl::Duration`除法，并建议使用返回浮点值的函数。

### abseil-duration-factory-float

Checks for cases where the floating-point overloads of various `absl::Duration` factory functions are called when the more-efficient integer versions could be used instead.

>在调用各种`absl::Duration`工厂函数的浮点重载处，检查可以使用更有效的整数版本的情况。

This check will not suggest fixes for literals which contain fractional floating point values or non-literals. It will suggest removing superfluous casts.

>此检查不会建议修复包含小数浮点值或非字面量的文字。它将建议移除多余的转换。

Examples 1:

```c++
// Original - Providing a floating-point literal.
absl::Duration d = absl::Seconds(10.0);

// Suggested - Use an integer instead.
absl::Duration d = absl::Seconds(10);
```

Examples 2:

```c++
// Original - Explicitly casting to a floating-point type.
absl::Duration d = absl::Seconds(static_cast<double>(10));

// Suggested - Remove the explicit cast
absl::Duration d = absl::Seconds(10);
```

### abseil-duration-factory-scale

Checks for cases where arguments to `absl::Duration` factory functions are scaled internally and could be changed to a different factory function. This check also looks for arguments with a zero value and suggests using `absl::ZeroDuration()` instead.

>检查`absl::Duration`工厂函数的参数是否在内部缩放，是否可以更改为不同的工厂函数。
>
>此检查还查找具有零值的参数，并建议使用`absl::ZeroDuration()`代替。

Examples 1:

```c++
// Original - Internal multiplication.
int x;
absl::Duration d = absl::Seconds(60 * x);

// Suggested - Use absl::Minutes instead.
absl::Duration d = absl::Minutes(x);
```

Examples 2:

```c++
// Original - Internal division.
int y;
absl::Duration d = absl::Milliseconds(y / 1000.);

// Suggested - Use absl:::Seconds instead.
absl::Duration d = absl::Seconds(y);
```

Examples 3:

```c++
// Original - Zero-value argument.
absl::Duration d = absl::Hours(0);

// Suggested = Use absl::ZeroDuration instead
absl::Duration d = absl::ZeroDuration();
```

### abseil-duration-subtraction

Checks for cases where subtraction should be performed in the `absl::Duration` domain. When subtracting two values, and the first one is known to be a conversion from `absl::Duration`, we can infer that the second should also be interpreted as an `absl::Duration`, and make that inference explicit.

>检查应该在`absl::Duration`域中执行减法的情况。当减去两个值时，已知第一个值是从`absl::Duration`转换而来，我们可以推断第二个值也应该被解释为`absl::Duration`，让这个推论更明确。

Examples 1:

```c++
// Original - Subtraction in the double domain
double x;
absl::Duration d;
double result = absl::ToDoubleSeconds(d) - x;

// Suggestion - Subtraction in the absl::Duration domain instead
double result = absl::ToDoubleSeconds(d - absl::Seconds(x));
```

Examples 2:

```c++
// Original - Subtraction of two Durations in the double domain
absl::Duration d1, d2;
double result = absl::ToDoubleSeconds(d1) - absl::ToDoubleSeconds(d2);

// Suggestion - Subtraction in the absl::Duration domain instead
double result = absl::ToDoubleSeconds(d1 - d2);
```

Note: As with other `clang-tidy` checks, it is possible that multiple fixes may overlap (as in the case of nested expressions), so not all occurrences can be transformed in one run. In particular, this may occur for nested subtraction expressions. Running `clang-tidy` multiple times will find and fix these overlaps.

>与其他“clang-tidy”检查一样，多个修复可能会重叠（就像嵌套表达式的情况一样），因此一次运行可能不会检查到所有的结果。特别是，这可能发生在嵌套减法表达式中。多次运行`clang-tidy`将找到并修复这些重叠。

### abseil-duration-unnecessary-conversion

Finds and fixes cases where `absl::Duration` values are being converted to numeric types and back again.

>查找并修复`absl::Duration`值被转换为数字类型并再次转换回来的情况。

Floating-point examples 1:

```c++
// Original - Conversion to double and back again
absl::Duration d1;
absl::Duration d2 = absl::Seconds(absl::ToDoubleSeconds(d1));

// Suggestion - Remove unnecessary conversions
absl::Duration d2 = d1;
```

Floating-point examples 2:

```c++
// Original - Division to convert to double and back again
absl::Duration d2 = absl::Seconds(absl::FDivDuration(d1, absl::Seconds(1)));

// Suggestion - Remove division and conversion
absl::Duration d2 = d1;
```

Integer examples 1:

```c++
// Original - Conversion to integer and back again
absl::Duration d1;
absl::Duration d2 = absl::Hours(absl::ToInt64Hours(d1));

// Suggestion - Remove unnecessary conversions
absl::Duration d2 = d1;
```

Integer examples 2:

```c++
// Original - Integer division followed by conversion
absl::Duration d2 = absl::Seconds(d1 / absl::Seconds(1));

// Suggestion - Remove division and conversion
absl::Duration d2 = d1;
```

Unwrapping scalar operations:

```c++
// Original - Multiplication by a scalar
absl::Duration d1;
absl::Duration d2 = absl::Seconds(absl::ToInt64Seconds(d1) * 2);

// Suggestion - Remove unnecessary conversion
absl::Duration d2 = d1 * 2;
```

Note: Converting to an integer and back to an `absl::Duration` might be a truncating operation if the value is not aligned to the scale of conversion. In the rare case where this is the intended result, callers should use `absl::Trunc` to truncate explicitly.

>如果值未与转换的时间尺度（时、分、秒）对齐，则转换为整数并返回到`absl::Duration`可能是截断操作。
>
>在极少数情况下，如果这是预期的结果，调用者应该使用`absl::Trunc`显式截断。

## boost-*

Checks related to Boost library.

### boost-use-to-string

This check finds conversion from integer type like int to `std::string` or `std::wstring` using `boost::lexical_cast`, and replace it with calls to `std::to_string` and `std::to_wstring`.

>该检查查找使用`boost::lexical_cast`从整数类型（如`int`）到`std::string`或`std::wstring`的转换，并将其替换为对`std::to_string`和`std::to_wstring`的调用。

It doesn’t replace conversion from floating points despite the `to_string` overloads, because it would change the behavior.

>尽管`to_string`重载，但它不会替换从浮点数的转换，因为它会改变行为。

```c++
auto str  = boost::lexical_cast<std::string>(42);
auto wstr = boost::lexical_cast<std::wstring>(2137LL);

// Will be changed to
auto str  = std::to_string(42);
auto wstr = std::to_wstring(2137LL);
```

## bugprone-*

Checks that target bug-prone code constructs.

### bugprone-argument-comment

Checks that argument comments match parameter names.

>检查实参注释是否与形参名称匹配。

The check understands argument comments in the form `/*parameter_name=*/` that are placed right before the argument.

>该检查理解以`/*parameter_name=*/`形式放置在参数前面的参数注释。

```c++
void f(bool foo);

...

f(/*bar=*/true);
// warning: argument name 'bar' in comment does not match parameter name 'foo'
```

The check tries to detect typos and suggest automated fixes for them.

#### Options

##### `StrictMode`

When `false` (default value), the check will ignore leading and trailing underscores and case when comparing names – otherwise they are taken into account.

>当为`false`（默认值）时，检查将在比较名称时忽略前导和尾随的下划线和大小写，否则将考虑它们。

##### `IgnoreSingleArgument`

When `true`, the check will ignore the single argument.

##### `CommentBoolLiterals`

When `true`, the check will add argument comments in the format `/*ParameterName=*/` right before the boolean literal argument.

Before:

```c++
void foo(bool TurnKey, bool PressButton);
foo(true, false);
```

After:

```c++
void foo(bool TurnKey, bool PressButton);
foo(/*TurnKey=*/true, /*PressButton=*/false);
```

##### `CommentIntegerLiterals`

When true, the check will add argument comments in the format `/*ParameterName=*/` right before the integer literal argument.

Before:

```c++
void foo(int MeaningOfLife);
foo(42);
```

After:

```c++
void foo(float Pi);
foo(/*Pi=*/3.14159);
```

##### `CommentStringLiterals`

When true, the check will add argument comments in the format `/*ParameterName=*/` right before the string literal argument.

Before:

```c++
void foo(const char *String);
void foo(const wchar_t *WideString);

foo("Hello World");
foo(L"Hello World");
```

After:

```c++
void foo(const char *String);
void foo(const wchar_t *WideString);

foo(/*String=*/"Hello World");
foo(/*WideString=*/L"Hello World");
```

##### `CommentCharacterLiterals`

When true, the check will add argument comments in the format /*ParameterName=*/ right before the character literal argument.

Before:

```c++
void foo(char *Character);
foo('A');
```

After:

```c++
void foo(char *Character);
foo(/*Character=*/'A');
```

##### `CommentUserDefinedLiterals`

When true, the check will add argument comments in the format /*ParameterName=*/ right before the user defined literal argument.

Before:

```c++
void foo(double Distance);
double operator"" _km(long double);
foo(402.0_km);
```

After:

```c++
void foo(double Distance);
double operator"" _km(long double);
foo(/*Distance=*/402.0_km);
```

##### `CommentNullPtrs`

When true, the check will add argument comments in the format /*ParameterName=*/ right before the nullptr literal argument.

Before:

```c++
void foo(A* Value);
foo(nullptr);
```

After:

```c++
void foo(A* Value);
foo(/*Value=*/nullptr);
```

### bugprone-assert-side-effect

Finds `assert()` with side effect.

The condition of `assert()` is evaluated only in debug builds so a condition with side effect can cause different behavior in debug / release builds.

>`assert()`的条件仅在调试版本中评估，因此具有副作用的条件可能在调试/发布版本中导致不同的行为。

#### Options

##### `AssertMacros`

A comma-separated list of the names of assert macros to be checked.

>要检查的断言宏的名称的逗号分隔列表。

##### `CheckFunctionCalls`

Whether to treat non-`const` member and non-member functions as they produce side effects. Disabled by default because it can increase the number of false positive warnings.

>是否对产生副作用的非`const`成员和非成员函数进行处理。
>
>默认情况下禁用，因为它会增加误报警告的数量。

##### `IgnoredFunctions`

A semicolon-separated list of the names of functions or methods to be considered as not having side-effects. Regular expressions are accepted, e.g. `[Rr]ef(erence)?$` matches every type with suffix `Ref`, `ref`, Reference and reference. The default is empty. If a name in the list contains the sequence `::` it is matched against the qualified typename (i.e. `namespace::Type`, otherwise it is matched against only the type name (i.e. `Type`).

### bugprone-assignment-in-if-condition

Finds assignments within conditions of `if` statements. Such assignments are bug-prone because they may have been intended as equality tests.

>在`if`语句的条件中查找赋值。这样的赋值很容易出现bug，因为它们可能被用作相等性测试。

This check finds all assignments within if conditions, including ones that are not flagged by `-Wparentheses` due to an extra set of parentheses, and including assignments that call an overloaded operator=().

>该检查查找`if`条件中的所有赋值，包括那些由于额外的圆括号而没有被`-Wparentheses`标记的赋值，以及调用重载操作符`=()`的赋值。
>

The identified assignments violate [BARR group “Rule 8.2.c”](https://barrgroup.com/embedded-systems/books/embedded-c-coding-standard/statement-rules/if-else-statements).

>被标记的赋值违反BARR组规则8.2。

```c++
int f = 3;

// This is identified by both `Wparentheses` and this check - should it have been: `if (f == 4)` ?
if(f = 4) {
  f = f + 1;
}

// the assignment here `(f = 6)` is identified by this check, but not by `-Wparentheses`.
// Should it have been `(f == 6)` ?
if((f == 5) || (f = 6)) {
  f = f + 2;
}
```

### bugprone-bad-signal-to-kill-thread

Finds `pthread_kill` function calls when a thread is terminated by raising `SIGTERM` signal and the signal kills the entire process, not just the individual thread. Use any signal except `SIGTERM`.

>发现`pthread_kill`函数调用时，一个线程被终止引发`SIGTERM`信号，信号杀死整个进程，而不仅仅是单个线程。
>
>使用除`SIGTERM`以外的任何信号。

```c++
pthread_kill(thread, SIGTERM);
```

This check corresponds to the CERT C Coding Standard rule [POS44-C. Do not use signals to terminate threads](https://wiki.sei.cmu.edu/confluence/display/c/POS44-C.+Do+not+use+signals+to+terminate+threads).

### bugprone-bool-pointer-implicit-conversion

Checks for conditions based on implicit conversion from a `bool` pointer to `bool`.

>根据从`bool`指针到`bool`指针的隐式转换检查条件。

Example:

```c++
bool *p;
if (p) {
  // Never used in a pointer-specific way.
}
```

### bugprone-branch-clone

Checks for repeated branches in if/else if/else chains, consecutive repeated branches in switch statements and identical true and false branches in conditional operators.

>检查if/else if/else链中的重复分支、`switch`语句中的连续重复分支以及条件操作符中的相同`true`和`false`分支。

```c++
if (test_value(x)) {
  y++;
  do_something(x, y);
} else {
  y++;
  do_something(x, y);
}
```

In this simple example (which could arise e.g. as a copy-paste error) the `then` and `else` branches are identical and the code is equivalent the following shorter and cleaner code:

>在这个简单的例子中（例如，可能出现复制粘贴错误），`then`和`else`分支是相同的，代码相当于以下更短更清晰的代码。

```c++
test_value(x); // can be omitted unless it has side effects
y++;
do_something(x, y);
```

If this is the intended behavior, then there is no reason to use a conditional statement; otherwise the issue can be solved by fixing the branch that is handled incorrectly.

>如果这是预期的行为，那么就没有理由使用条件语句;否则，可以通过修复处理不正确的分支来解决问题。

The check also detects repeated branches in longer `if/else if/else` chains where it would be even harder to notice the problem.

>检查还检测到在较长的“if/else if/else”链中重复的分支，在这些链中更难注意到问题。

In `switch` statements the check only reports repeated branches when they are consecutive, because it is relatively common that the `case:` labels have some natural ordering and rearranging them would decrease the readability of the code.

>在`switch`语句中，检查只报告连续的重复分支，因为`case:`标签通常有一些自然的顺序，重新排列它们会降低代码的可读性。

For example:

```c++
switch (ch) {
case 'a':
  return 10;
case 'A':
  return 10;
case 'b':
  return 11;
case 'B':
  return 11;
default:
  return 10;
}
```

Here the check reports that the `'a'` and `'A'` branches are identical (and that the `'b'` and `'B'` branches are also identical), but does not report that the `default:` branch is also identical to the first two branches. If this is indeed the correct behavior, then it could be implemented as:

>这里，检查报告`a`和`A`分支是相同的（`b`和`B`分支也是相同的），但不报告`default:`分支也与前两个分支相同。
>
>如果这确实是正确的行为，那么它可以实现为：

```c++
switch (ch) {
case 'a':
case 'A':
  return 10;
case 'b':
case 'B':
  return 11;
default:
  return 10;
}
```

Here the check does not warn for the repeated `return 10;`, which is good if we want to preserve that `'a'` is before `'b'` and `default:` is the last branch.

>这里的检查不会对重复的`return 10;`发出警告，如果我们想保留`a`在`b`之前并且`default:`是最后一个分支，这是很好的。

Switch cases marked with the `[[fallthrough]]` attribute are ignored.

Finally, the check also examines conditional operators and reports code like:

```c++
return test_value(x) ? x : x;
```

Unlike `if` statements, the check does not detect chains of conditional operators.

>与`if`语句不同，该检查不检测条件操作符链。

Note: This check also reports situations where branches become identical only after preprocessing.

>注意：此检查还报告只有在预处理之后分支才变得相同的情况。

### bugprone-casting-through-void

Detects unsafe or redundant two-step casting operations involving `void*`.

>检测涉及`void*`的不安全或冗余的两步强制转换操作。

Two-step type conversions via `void*` are discouraged for several reasons.

>由于几个原因，不鼓励通过`void*`进行两步类型转换。

- They obscure code and impede its understandability, complicating maintenance.

  - >它们模糊了代码，阻碍了代码的可理解性，使维护变得复杂。

- These conversions bypass valuable compiler support, erasing warnings related to pointer alignment. It may violate strict aliasing rule and leading to undefined behavior.

  - >这些转换绕过了有价值的编译器支持，消除了与指针对齐相关的警告。
    >
    >它可能违反严格的混叠规则并导致未定义的行为。

- In scenarios involving multiple inheritance, ambiguity and unexpected outcomes can arise due to the loss of type information, posing runtime issues.

  - >在涉及多重继承的场景中，由于类型信息的丢失，可能会产生歧义和意外结果，从而引发运行时问题。

In summary, avoiding two-step type conversions through `void*` ensures clearer code, maintains essential compiler warnings, and prevents ambiguity and potential runtime errors, particularly in complex inheritance scenarios.

>总之，通过`void*`避免两步类型转换可以确保代码更清晰，保持必要的编译器警告，并防止歧义和潜在的运行时错误，特别是在复杂的继承场景中。

Examples:

```c++
using IntegerPointer = int *;
double *ptr;

static_cast<IntegerPointer>(static_cast<void *>(ptr)); // WRONG
reinterpret_cast<IntegerPointer>(reinterpret_cast<void *>(ptr)); // WRONG
(IntegerPointer)(void *)ptr; // WRONG
IntegerPointer(static_cast<void *>(ptr)); // WRONG
```

### bugprone-compare-pointer-to-member-virtual-function

Detects unspecified behavior about equality comparison between pointer to member virtual function and anything other than null-pointer-constant.

>检测关于成员虚函数指针与非空指针常量之间相等比较的未指定行为。

```c++
struct A {
  void f1();
  void f2();
  virtual void f3();
  virtual void f4();

  void g1(int);
};

void fn() {
  bool r1 = (&A::f1 == &A::f2);  // ok
  bool r2 = (&A::f1 == &A::f3);  // bugprone
  bool r3 = (&A::f1 != &A::f3);  // bugprone
  bool r4 = (&A::f3 == nullptr); // ok
  bool r5 = (&A::f3 == &A::f4);  // bugprone

  void (A::*v1)() = &A::f3;
  bool r6 = (v1 == &A::f1); // bugprone
  bool r6 = (v1 == nullptr); // ok

  void (A::*v2)() = &A::f2;
  bool r7 = (v2 == &A::f1); // false positive, but potential risk if assigning other value to v2.

  void (A::*v3)(int) = &A::g1;
  bool r8 = (v3 == &A::g1); // ok, no virtual function match void(A::*)(int) signature.
}
```

Provide warnings on equality comparisons involve pointers to member virtual function or variables which is potential pointer to member virtual function and any entity other than a null-pointer constant.

>对相等比较提供警告，包括指向成员虚函数的指针或可能指向成员虚函数和除空指针常量以外的任何实体的变量的指针。

In certain compilers, virtual function addresses are not conventional pointers but instead consist of offsets and indexes within a virtual function table (vtable). Consequently, these pointers may vary between base and derived classes, leading to unpredictable behavior when compared directly. This issue becomes particularly challenging when dealing with pointers to pure virtual functions, as they may not even have a valid address, further complicating comparisons.

>在某些编译器中，虚函数地址不是传统的指针，而是由虚函数表（vtable）中的偏移量和索引组成。
>
>因此，这些指针可能在基类和派生类之间变化，直接比较时会导致不可预测的行为。
>
>当处理指向纯虚函数的指针时，这个问题变得特别具有挑战性，因为它们甚至可能没有有效地址，从而使比较进一步复杂化。

Instead, it is recommended to utilize the `typeid` operator or other appropriate mechanisms for comparing objects to ensure robust and predictable behavior in your codebase. By heeding this detection and adopting a more reliable comparison method, you can mitigate potential issues related to unspecified behavior, especially when dealing with pointers to member virtual functions or pure virtual functions, thereby improving the overall stability and maintainability of your code. In scenarios involving pointers to member virtual functions, it’s only advisable to employ `nullptr` for comparisons.

>相反，建议使用`typeid`操作符或其他适当的机制来比较对象，以确保代码库中的健壮和可预测的行为。通过注意这种检测并采用更可靠的比较方法，您可以减轻与未指定行为相关的潜在问题，特别是在处理指向成员虚函数或纯虚函数的指针时，从而提高代码的整体稳定性和可维护性。在涉及指向成员虚函数的指针的场景中，只建议使用`nullptr`进行比较。

#### Limitations

Does not analyze values stored in a variable. For variable, only analyze all virtual methods in the same `class` or `struct` and diagnose when assigning a pointer to member virtual function to this variable is possible.

>不分析存储在变量中的值。
>
>对于变量，只分析同一类或结构中的所有虚方法，并诊断何时可以将指向成员虚函数的指针赋值给该变量。

### bugprone-copy-constructor-init

Finds copy constructors where the constructor doesn’t call the copy constructor of the base class.

>查找构造函数未调用基类的复制构造函数的复制构造函数。

```c++
class Copyable {
public:
  Copyable() = default;
  Copyable(const Copyable &) = default;

  int memberToBeCopied = 0;
};

class X2 : public Copyable {
  X2(const X2 &other) {} // Copyable(other) is missing
};
```

Also finds copy constructors where the constructor of the base class don’t have parameter.

>还查找基类的构造函数没有参数的复制构造函数。

```c++
class X3 : public Copyable {
  X3(const X3 &other) : Copyable() {} // other is missing
};
```

Failure to properly initialize base class sub-objects during copy construction can result in undefined behavior, crashes, data corruption, or other unexpected outcomes. The check ensures that the copy constructor of a derived class properly calls the copy constructor of the base class, helping to prevent bugs and improve code quality.

>在复制构造过程中未能正确初始化基类子对象可能导致未定义的行为、崩溃、数据损坏或其他意想不到的结果。
>
>检查确保派生类的复制构造函数正确调用基类的复制构造函数，有助于防止错误并提高代码质量。

Limitations:

- It won’t generate warnings for empty classes, as there are no class members (including base class sub-objects) to worry about.

  - >它不会为空类生成警告，因为没有需要担心的类成员（包括基类子对象）。

- It won’t generate warnings for base classes that have copy constructor private or deleted.

  - >它不会为复制构造函数为私有或已删除的基类生成警告。

- It won’t generate warnings for base classes that are initialized using other non-default constructor, as this could be intentional.

  - 它不会为使用其他非默认构造函数初始化的基类生成警告，因为这可能是故意的。

The check also suggests a fix-its in some cases.

>在某些情况下，检查还建议进行修复。

### bugprone-dangling-handle

Detect dangling references in value handles like `std::string_view`. These dangling references can be a result of constructing handles from temporary values, where the temporary is destroyed soon after the handle is created.

>检测值句柄中的悬空引用，如`std::string_view`。
>
>这些悬空引用可能是由临时值构造句柄的结果，其中临时句柄在创建后很快被销毁。

Examples:

```c++
string_view View = string();  // View will dangle.
string A;
View = A + "A";  // still dangle.

vector<string_view> V;
V.push_back(string());  // V[0] is dangling.
V.resize(3, string());  // V[1] and V[2] will also dangle.

string_view f() {
  // All these return values will dangle.
  return string();
  string S;
  return S;
  char Array[10]{};
  return Array;
}
```

#### Options

##### `HandleClasses`

A semicolon-separated list of class names that should be treated as handles. By default only `std::basic_string_view` and `std::experimental::basic_string_view` are considered.

>应被视为句柄的类名的分号分隔列表。
>
>默认情况下，只考虑`std::basic_string_view`和`std::experimental::basic_string_view`。

### bugprone-dynamic-static-initializers

Finds instances of static variables that are dynamically initialized in header files.

>查找在头文件中动态初始化的静态变量的实例。

This can pose problems in certain multithreaded contexts. For example, when disabling compiler generated synchronization instructions for static variables initialized at runtime (e.g. by `-fno-threadsafe-statics`), even if a particular project takes the necessary precautions to prevent race conditions during initialization by providing their own synchronization, header files included from other projects may not. Therefore, such a check is helpful for ensuring that disabling compiler generated synchronization for static variable initialization will not cause problems.

>这可能会在某些多线程上下文中造成问题。
>
>例如，当禁用编译器为在运行时初始化的静态变量生成的同步指令时（例如，通过`-fno-threadsafe- stats`），即使特定项目在初始化期间采取必要的预防措施，通过提供自己的同步来防止竞争条件，包括来自其他项目的头文件也可能不会。
>
>因此，这样的检查有助于确保禁用编译器为静态变量初始化生成的同步不会导致问题。

Consider the following code:

```c++
int foo() {
  static int k = bar();
  return k;
}
```

When synchronization of static initialization is disabled, if two threads both call `foo` for the first time, there is the possibility that k will be double initialized, creating a race condition.

>当静态初始化的同步被禁用时，如果两个线程都是第一次调用`foo`，那么k就有可能被双初始化，从而创建一个竞争条件。

### bugprone-easily-swappable-parameters

Finds function definitions where parameters of convertible types follow each other directly, making call sites prone to calling the function with swapped (or badly ordered) arguments.

>查找相同的或具有可转换类型的相邻的参数的函数定义，因为这使得调用者容易使用交换的（或顺序错误的）参数调用函数。

```c++
void drawPoint(int X, int Y) { /* ... */ }
FILE *open(const char *Dir, const char *Name, Flags Mode) { /* ... */ }
```

A potential call like `drawPoint(-2, 5)` or `openPath("a.txt", "tmp", Read)` is perfectly legal from the language’s perspective, but might not be what the developer of the function intended.

>从语言的角度来看，像`drawPoint(- 2,5)`或`openPath("a.t txt"， "tmp"， Read)`这样的潜在调用是完全合法的，但可能不是函数开发人员想要的。

More elaborate and type-safe constructs, such as opaque typedefs or strong types should be used instead, to prevent a mistaken order of arguments.

>应该使用更精细和类型安全的结构，例如不透明类型或强类型，以防止错误的参数顺序。

```c++
struct Coord2D { int X; int Y; };
void drawPoint(const Coord2D Pos) { /* ... */ }

FILE *open(const Path &Dir, const Filename &Name, Flags Mode) { /* ... */ }
```

Due to the potentially elaborate refactoring and API-breaking that is necessary to strengthen the type safety of a project, no automatic fix-its are offered.

>由于可能需要复杂的重构和API破坏来加强项目的类型安全，因此没有提供自动修复。

#### Options

##### Extension/Relaxation options

Relaxation (or extension) options can be used to broaden the scope of the analysis and fine-tune the enabling of more mixes between types. Some mixes may depend on coding style or preference specific to a project, however, it should be noted that enabling all of these relaxations model the way of mixing at call sites the most. These options are expected to make the check report for more functions, and report longer mixable ranges.

>松弛（或扩展）选项可用于扩大分析范围，并微调类型之间更多混合的启用。
>
>一些混合可能取决于特定于项目的编码风格或偏好，然而，应该注意的是，启用所有这些松弛模型在调用站点的混合方式最多。
>
>这些选项将使检查报告具有更多的功能，并报告更长的可混合范围。

`QualifiersMix`

Whether to consider parameters of some cvr-qualified `T` and a differently cvr-qualified `T` (i.e. `T` and `const T`, `const T` and `volatile T`, etc.) mixable between one another. If `false`, the check will consider differently qualified types unmixable. True turns the warnings on. Defaults to `false`.

The following example produces a diagnostic only if `QualifiersMix` is enabled:

```c++
void *memcpy(const void *Destination, void *Source, std::size_t N) { /* ... */ }
```

`ModelImplicitConversions`

Whether to consider parameters of type `T` and `U` mixable if there exists an implicit conversion from `T` to `U` and `U` to `T`. If `false`, the check will not consider implicitly convertible types for mixability. `True` turns warnings for implicit conversions on. Defaults to `true`.

The following examples produce a diagnostic only if ModelImplicitConversions is enabled:

```c++
void fun(int Int, double Double) { /* ... */ }
void compare(const char *CharBuf, std::string String) { /* ... */ }
```

**Note**

Changing the qualifiers of an expression’s type (e.g. from `int` to `const int`) is defined as an implicit conversion in the C++ Standard. However, the check separates this decision-making on the mixability of differently qualified types based on whether QualifiersMix was enabled.

For example, the following code snippet will only produce a diagnostic if both `QualifiersMix` and `ModelImplicitConversions` are enabled:

```c++
void fun2(int Int, const double Double) { /* ... */ }
```

##### Filtering options

Filtering options can be used to lessen the size of the diagnostics emitted by the checker, whether the aim is to ignore certain constructs or dampen the noisiness.

`MinimumLength`

The minimum length required from an adjacent parameter sequence to be diagnosed. Defaults to 2. Might be any positive integer greater or equal to 2. If 0 or 1 is given, the default value 2 will be used instead.For example, if 3 is specified, the examples above will not be matched.

`IgnoredParameterNames`

The list of parameter **names** that should never be considered part of a swappable adjacent parameter sequence. The value is a ;-separated list of names. To ignore unnamed parameters, add “” to the list verbatim (not the empty string, but the two quotes, potentially escaped!). **This option is case-sensitive!** By default, the following parameter names, and their Uppercase-initial variants are ignored: “” (unnamed parameters), iterator, begin, end, first, last, lhs, rhs.

`IgnoredParameterTypeSuffixes`

The list of parameter **type name suffixes** that should never be considered part of a swappable adjacent parameter sequence. Parameters which type, as written in the source code, end with an element of this option will be ignored. The value is a ;-separated list of names. **This option is case-sensitive!**By default, the following, and their lowercase-initial variants are ignored: bool, It, Iterator, InputIt, ForwardIt, BidirIt, RandomIt, random_iterator, ReverseIt, reverse_iterator, reverse_const_iterator, RandomIt, random_iterator, ReverseIt, reverse_iterator, reverse_const_iterator, Const_Iterator, ConstIterator, const_reverse_iterator, ConstReverseIterator. In addition, _Bool (but not _bool) is also part of the default value.

`SuppressParametersUsedTogether`

Suppresses diagnostics about parameters that are used together or in a similar fashion inside the function’s body. Defaults to true. Specifying false will turn off the heuristics.Currently, the following heuristics are implemented which will suppress the warning about the parameter pair involved:The parameters are used in the same expression, e.g. `f(a, b)` or `a < b`.The parameters are further passed to the same function to the same parameter of that function, of the same overload. E.g. `f(a, 1)` and `f(b, 2)` to some `f(T, int)`.NoteThe check does not perform path-sensitive analysis, and as such, “same function” in this context means the same function declaration. If the same member function of a type on two distinct instances are called with the parameters, it will still be regarded as “same function”.The same member field is accessed, or member method is called of the two parameters, e.g. `a.foo()` and `b.foo()`.Separate `return` statements return either of the parameters on different code paths.

`NamePrefixSuffixSilenceDissimilarityTreshold`

The number of characters two parameter names might be different on *either* the head or the tail end with the rest of the name the same so that the warning about the two parameters are silenced. Defaults to 1. Might be any positive integer. If 0, the filtering heuristic based on the parameters’ names is turned off.This option can be used to silence warnings about parameters where the naming scheme indicates that the order of those parameters do not matter.For example, the parameters `LHS` and `RHS` are 1-dissimilar suffixes of each other: `L` and `R` is the different character, while `HS` is the common suffix. Similarly, parameters `text1, text2, text3` are 1-dissimilar prefixes of each other, with the numbers at the end being the dissimilar part. If the value is at least 1, such cases will not be reported.

#### Limitations

This check is designed to check function signatures!

The check does not investigate functions that are generated by the compiler in a context that is only determined from a call site. These cases include variadic functions, functions in C code that do not have an argument list, and C++ template instantiations. Most of these cases, which are otherwise swappable from a caller’s standpoint, have no way of getting “fixed” at the definition point. In the case of C++ templates, only primary template definitions and explicit specializations are matched and analyzed.

None of the following cases produce a diagnostic:

```c++
int printf(const char *Format, ...) { /* ... */ }
int someOldCFunction() { /* ... */ }

template <typename T, typename U>
int add(T X, U Y) { return X + Y };

void theseAreNotWarnedAbout() {
    printf("%d %d\n", 1, 2);   // Two ints passed, they could be swapped.
    someOldCFunction(1, 2, 3); // Similarly, multiple ints passed.

    add(1, 2); // Instantiates 'add<int, int>', but that's not a user-defined function.
}
```

Due to the limitation above, parameters which type are further dependent upon template instantiations to *prove* that they mix with another parameter’s is not diagnosed.

```c++
template <typename T>
struct Vector {
  typedef T element_type;
};

// Diagnosed: Explicit instantiation was done by the user, we can prove it
// is the same type.
void instantiated(int A, Vector<int>::element_type B) { /* ... */ }

// Diagnosed: The two parameter types are exactly the same.
template <typename T>
void exact(typename Vector<T>::element_type A,
           typename Vector<T>::element_type B) { /* ... */ }

// Skipped: The two parameters are both 'T' but we cannot prove this
// without actually instantiating.
template <typename T>
void falseNegative(T A, typename Vector<T>::element_type B) { /* ... */ }
```

In the context of implicit conversions (when ModelImplicitConversions is enabled), the modelling performed by the check warns if the parameters are swappable and the swapped order matches implicit conversions. It does not model whether there exists an unrelated third type from which *both* parameters can be given in a function call. This means that in the following example, even while `strs()` clearly carries the possibility to be called with swapped arguments (as long as the arguments are string literals), will not be warned about.

```c++
struct String {
    String(const char *Buf);
};

struct StringView {
    StringView(const char *Buf);
    operator const char *() const;
};

// Skipped: Directly swapping expressions of the two type cannot mix.
// (Note: StringView -> const char * -> String would be **two**
// user-defined conversions, which is disallowed by the language.)
void strs(String Str, StringView SV) { /* ... */ }

// Diagnosed: StringView implicitly converts to and from a buffer.
void cStr(StringView SV, const char *Buf() { /* ... */ }
```

### bugprone-empty-catch

Detects and suggests addressing issues with empty `catch` statements.

>检测并建议使用空`catch`语句解决问题。

```c++
try {
  // Some code that can throw an exception
} catch(const std::exception&) {
}
```

Having empty catch statements in a codebase can be a serious problem that developers should be aware of. Catch statements are used to handle exceptions that are thrown during program execution. When an exception is thrown, the program jumps to the nearest catch statement that matches the type of the exception.

>在代码库中使用空`catch`语句可能是开发人员应该注意的一个严重问题。
>
>`catch`语句用于处理程序执行期间抛出的异常。当抛出异常时，程序跳转到与异常类型匹配的最近的`catch`语句。

Empty catch statements, also known as “swallowing” exceptions, catch the exception but do nothing with it. This means that the exception is not handled properly, and the program continues to run as if nothing happened.

>空`catch`语句，也称为“吞下”异常，捕获异常但不处理异常。
>
>这意味着异常没有得到正确处理，程序继续运行，就好像什么都没发生一样。

This can lead to several issues, such as:

- Hidden Bugs: If an exception is caught and ignored, it can lead to hidden bugs that are difficult to diagnose and fix. The root cause of the problem may not be apparent, and the program may continue to behave in unexpected ways.
- Security Issues: Ignoring exceptions can lead to security issues, such as buffer overflows or null pointer dereferences. Hackers can exploit these vulnerabilities to gain access to sensitive data or execute malicious code.
- Poor Code Quality: Empty catch statements can indicate poor code quality and a lack of attention to detail. This can make the codebase difficult to maintain and update, leading to longer development cycles and increased costs.
- Unreliable Code: Code that ignores exceptions is often unreliable and can lead to unpredictable behavior. This can cause frustration for users and erode trust in the software.

To avoid these issues, developers should always handle exceptions properly. This means either fixing the underlying issue that caused the exception or propagating the exception up the call stack to a higher-level handler. If an exception is not important, it should still be logged or reported in some way so that it can be tracked and addressed later.

If the exception is something that can be handled locally, then it should be handled within the catch block. This could involve logging the exception or taking other appropriate action to ensure that the exception is not ignored.

Here is an example:

```c++
try {
  // Some code that can throw an exception
} catch (const std::exception& ex) {
  // Properly handle the exception, e.g.:
  std::cerr << "Exception caught: " << ex.what() << std::endl;
}
```

If the exception cannot be handled locally and needs to be propagated up the call stack, it should be re-thrown or new exception should be thrown.

Here is an example:

```c++
try {
  // Some code that can throw an exception
} catch (const std::exception& ex) {
  // Re-throw the exception
  throw;
}
```

In some cases, catching the exception at this level may not be necessary, and it may be appropriate to let the exception propagate up the call stack. This can be done simply by not using `try/catch` block.

Here is an example:

```c++
void function() {
  // Some code that can throw an exception
}

void callerFunction() {
  try {
    function();
  } catch (const std::exception& ex) {
    // Handling exception on higher level
    std::cerr << "Exception caught: " << ex.what() << std::endl;
  }
}
```

Other potential solution to avoid empty catch statements is to modify the code to avoid throwing the exception in the first place. This can be achieved by using a different API, checking for error conditions beforehand, or handling errors in a different way that does not involve exceptions. By eliminating the need for try-catch blocks, the code becomes simpler and less error-prone.

Here is an example:

```c++
// Old code:
try {
  mapContainer["Key"].callFunction();
} catch(const std::out_of_range&) {
}

// New code
if (auto it = mapContainer.find("Key"); it != mapContainer.end()) {
  it->second.callFunction();
}
```

In conclusion, empty catch statements are a bad practice that can lead to hidden bugs, security issues, poor code quality, and unreliable code. By handling exceptions properly, developers can ensure that their code is robust, secure, and maintainable.

#### Options

##### `IgnoreCatchWithKeywords`

This option can be used to ignore specific catch statements containing certain keywords. If a `catch` statement body contains (case-insensitive) any of the keywords listed in this semicolon-separated option, then the catch will be ignored, and no warning will be raised. Default value: @TODO;@FIXME.

##### `AllowEmptyCatchForExceptions`

This option can be used to ignore empty catch statements for specific exception types. By default, the check will raise a warning if an empty catch statement is detected, regardless of the type of exception being caught. However, in certain situations, such as when a developer wants to intentionally ignore certain exceptions or handle them in a different way, it may be desirable to allow empty catch statements for specific exception types. To configure this option, a semicolon-separated list of exception type names should be provided. If an exception type name in the list is caught in an empty catch statement, no warning will be raised. Default value: empty string.

### bugprone-exception-escape

Finds functions which may throw an exception directly or indirectly, but they should not.

>查找可能直接或间接抛出异常的函数，但它们不应该抛出异常。

The functions which should not throw exceptions are the following:

- Destructors
- Move constructors
- Move assignment operators
- The `main()` functions
- `swap()` functions
- Functions marked with `throw()` or `noexcept`
- Other functions given as option

A destructor throwing an exception may result in undefined behavior, resource leaks or unexpected termination of the program. Throwing move constructor or move assignment also may result in undefined behavior or resource leak. The `swap()` operations expected to be non throwing most of the cases and they are always possible to implement in a non throwing way. Non throwing `swap()` operations are also used to create move operations. A throwing `main()` function also results in unexpected termination.

>析构函数抛出异常可能导致未定义的行为、资源泄漏或程序的意外终止。
>
>抛出移动构造函数或移动赋值运算符也可能导致未定义行为或资源泄漏。
>
>`swap()`操作在大多数情况下都是非抛出的，并且它们总是可以以非抛出的方式实现。非抛出的`swap()`操作也用于创建移动操作。
>
>抛出`main()`函数也会导致意外终止。

Functions declared explicitly with `noexcept(false)` or `throw(exception)` will be excluded from the analysis, as even though it is not recommended for functions like `swap()`, `main()`, move constructors, move assignment operators and destructors, it is a clear indication of the developer’s intention and should be respected.

>用`noexcept(false)`或`throw(exception)`显式声明的函数将被排除在分析之外，因为即使不建议像`swap()`、`main()`、移动构造函数，移动赋值操作符和析构函数这样的函数抛出异常，这些显式声明也清楚地表明了开发人员的意图，应该得到尊重。

WARNING! This check may be expensive on large source files.

>警告！对于大型源文件，此检查可能代价高昂。（检查所需时间比较久）

#### Options

- `FunctionsThatShouldNotThrow`

  - Comma separated list containing function names which should not throw. An example value for this parameter can be `WinMain` which adds function `WinMain()` in the Windows API to the list of the functions which should not throw. Default value is an empty string.

  - >包含不应抛出的函数名的逗号分隔列表。
    >
    >这个参数的一个示例值可以是`WinMain`，它将Windows API中的函数`WinMain()`添加到不应该抛出的函数列表中。
    >
    >默认值为空字符串。

- `IgnoredExceptions`

  - Comma separated list containing type names which are not counted as thrown exceptions in the check. Default value is an empty string.

  - >包含类型名的逗号分隔列表，这些类型名在检查中不被视为抛出异常。
    >
    >默认值为空字符串。

### bugprone-fold-init-type

The check flags type mismatches in [folds](https://en.wikipedia.org/wiki/Fold_(higher-order_function)) like `std::accumulate` that might result in loss of precision. `std::accumulate` folds an input range into an initial value using the type of the latter, with `operator+` by default.

This can cause loss of precision through:

Truncation: The following code uses a floating point range and an int initial value, so truncation will happen at every application of `operator+` and the result will be 0, which might not be what the user expected.

```c++
auto a = {0.5f, 0.5f, 0.5f, 0.5f};
return std::accumulate(std::begin(a), std::end(a), 0);
```

Overflow: The following code also returns `0`.

```c++
auto a = {65536LL * 65536 * 65536};
return std::accumulate(std::begin(a), std::end(a), 0);
```

### bugprone-forward-declaration-namespace

Checks if an unused forward declaration is in a wrong namespace.

The check inspects all unused forward declarations and checks if there is any declaration/definition with the same name existing, which could indicate that the forward declaration is in a potentially wrong namespace.

```c++
namespace na { struct A; }
namespace nb { struct A {}; }
nb::A a;
// warning : no definition found for 'A', but a definition with the same name
// 'A' found in another namespace 'nb::'
```

This check can only generate warnings, but it can’t suggest a fix at this point.

### bugprone-forwarding-reference-overload

The check looks for perfect forwarding constructors that can hide copy or move constructors. If a non const lvalue reference is passed to the constructor, the forwarding reference parameter will be a better match than the const reference parameter of the copy constructor, so the perfect forwarding constructor will be called, which can be confusing. For detailed description of this issue see: Scott Meyers, Effective Modern C++, Item 26.

Consider the following example:

```c++
class Person {
public:
  // C1: perfect forwarding ctor
  template<typename T>
  explicit Person(T&& n) {}

  // C2: perfect forwarding ctor with parameter default value
  template<typename T>
  explicit Person(T&& n, int x = 1) {}

  // C3: perfect forwarding ctor guarded with enable_if
  template<typename T, typename X = enable_if_t<is_special<T>, void>>
  explicit Person(T&& n) {}

  // C4: variadic perfect forwarding ctor guarded with enable_if
  template<typename... A,
    enable_if_t<is_constructible_v<tuple<string, int>, A&&...>, int> = 0>
  explicit Person(A&&... a) {}

  // C5: perfect forwarding ctor guarded with requires expression
  template<typename T>
  requires requires { is_special<T>; }
  explicit Person(T&& n) {}

  // C6: perfect forwarding ctor guarded with concept requirement
  template<Special T>
  explicit Person(T&& n) {}

  // (possibly compiler generated) copy ctor
  Person(const Person& rhs);
};
```

The check warns for constructors C1 and C2, because those can hide copy and move constructors. We suppress warnings if the copy and the move constructors are both disabled (deleted or private), because there is nothing the perfect forwarding constructor could hide in this case. We also suppress warnings for constructors like C3-C6 that are guarded with an `enable_if` or a concept, assuming the programmer was aware of the possible hiding.

#### Background

For deciding whether a constructor is guarded with enable_if, we consider the types of the constructor parameters, the default values of template type parameters and the types of non-type template parameters with a default literal value. If any part of these types is `std::enable_if` or `std::enable_if_t`, we assume the constructor is guarded.

### bugprone-implicit-widening-of-multiplication-result

The check diagnoses instances where a result of a multiplication is implicitly widened, and suggests (with fix-it) to either silence the code by making widening explicit, or to perform the multiplication in a wider type, to avoid the widening afterwards.

This is mainly useful when operating on very large buffers.

For example, consider:

```c++
void zeroinit(char* base, unsigned width, unsigned height) {
  for(unsigned row = 0; row != height; ++row) {
    for(unsigned col = 0; col != width; ++col) {
      char* ptr = base + row * width + col;
      *ptr = 0;
    }
  }
}
```

This is fine in general, but if `width * height` overflows, you end up wrapping back to the beginning of `base` instead of processing the entire requested buffer.

Indeed, this only matters for pretty large buffers (4GB+), but that can happen very easily for example in image processing, where for that to happen you “only” need a ~269MPix image.

#### Options

##### `UseCXXStaticCastsInCppSources`

When suggesting fix-its for C++ code, should C++-style `static_cast<>()`’s be suggested, or C-style casts. Defaults to `true`.

##### `UseCXXHeadersInCppSources`

When suggesting to include the appropriate header in C++ code, should `<cstddef>` header be suggested, or `<stddef.h>`. Defaults to `true`.

Examples:

```c++
long mul(int a, int b) {
  return a * b; // warning: performing an implicit widening conversion
    			// to type 'long' of a multiplication performed in type 'int'
}

char* ptr_add(char *base, int a, int b) {
  return base + a * b;	// warning: result of multiplication in type 'int' is used as a pointer
    					// offset after an implicit widening conversion to type 'ssize_t'
}

char ptr_subscript(char *base, int a, int b) {
  return base[a * b];	// warning: result of multiplication in type 'int' is used as a pointer
    					// offset after an implicit widening conversion to type 'ssize_t'
}
```

### bugprone-inaccurate-erase

Checks for inaccurate use of the `erase()` method.

Algorithms like `remove()` do not actually remove any element from the container but return an iterator to the first redundant element at the end of the container. These redundant elements must be removed using the `erase()` method. This check warns when not all of the elements will be removed due to using an inappropriate overload.

For example, the following code erases only one element:

```c++
std::vector<int> xs;
// ...
xs.erase(std::remove(xs.begin(), xs.end(), 10));
```

Call the two-argument overload of `erase()` to remove the subrange:

```c++
std::vector<int> xs;
// ...
xs.erase(std::remove(xs.begin(), xs.end(), 10), xs.end());
```

### bugprone-inc-dec-in-conditions

Detects when a variable is both incremented/decremented and referenced inside a complex condition and suggests moving them outside to avoid ambiguity in the variable’s value.

When a variable is modified and also used in a complex condition, it can lead to unexpected behavior. The side-effect of changing the variable’s value within the condition can make the code difficult to reason about. Additionally, the developer’s intended timing for the modification of the variable may not be clear, leading to misunderstandings and errors. This can be particularly problematic when the condition involves logical operators like `&&` and `||`, where the order of evaluation can further complicate the situation.

Consider the following example:

```c++
int i = 0;
// ...
if (i++ < 5 && i > 0) {
  // do something
}
```

In this example, the result of the expression may not be what the developer intended. The original intention of the developer could be to increment `i` after the entire condition is evaluated, but in reality, i will be incremented before `i > 0` is executed. This can lead to unexpected behavior and bugs in the code. To fix this issue, the developer should separate the increment operation from the condition and perform it separately.

For example, they can increment `i` in a separate statement before or after the condition is evaluated. This ensures that the value of `i` is predictable and consistent throughout the code.

```c++
int i = 0;
// ...
i++;
if (i <= 5 && i > 0) {
  // do something
}
```

Another common issue occurs when multiple increments or decrements are performed on the same variable inside a complex condition.

For example:

```c++
int i = 4;
// ...
if (i++ < 5 || --i > 2) {
  // do something
}
```

There is a potential issue with this code due to the order of evaluation in C++. The `||` operator used in the condition statement guarantees that if the first operand evaluates to `true`, the second operand will not be evaluated. This means that if `i` were initially `4`, the first operand `i < 5` would evaluate to `true` and the second operand `i > 2` would not be evaluated. As a result, the decrement operation `--i` would not be executed and `i` would hold value `5`, which may not be the intended behavior for the developer.

To avoid this potential issue, the both increment and decrement operation on `i` should be moved outside the condition statement.

### bugprone-incorrect-enable-if

Detects incorrect usages of `std::enable_if` that don’t name the nested `type` type.

In C++11 introduced `std::enable_if` as a convenient way to leverage SFINAE. One form of using `std::enable_if` is to declare an unnamed template type parameter with a default type equal to `typename std::enable_if<condition>::type`. If the author forgets to name the nested type `type`, then the code will always consider the candidate template even if the condition is not met.

Below are some examples of code using `std::enable_if` correctly and incorrect examples that this check flags.

```c++
template <
	typename T,
	typename = typename std::enable_if<T::some_trait>::type
        >
void valid_usage() { ... }

template <
	typename T,
	typename = std::enable_if_t<T::some_trait>
        >
void valid_usage_with_trait_helpers() { ... }

// The below code is not a correct application of SFINAE. Even if
// T::some_trait is not true, the function will still be considered in the
// set of function candidates. It can either incorrectly select the function
// when it should not be a candidates, and/or lead to hard compile errors
// if the body of the template does not compile if the condition is not
// satisfied.
template <
	typename T,
	typename = std::enable_if<T::some_trait>
        >
void invalid_usage() { ... }

// The tool suggests the following replacement for 'invalid_usage':
template <
	typename T,
	typename = typename std::enable_if<T::some_trait>::type
        >
void fixed_invalid_usage() { ... }
```

C++14 introduced the trait helper `std::enable_if_t` which reduces the likelihood of this error. C++20 introduces constraints, which generally supersede the use of `std::enable_if`. See [modernize-type-traits](https://clang.llvm.org/extra/clang-tidy/checks/modernize/type-traits.html) for another tool that will replace `std::enable_if` with `std::enable_if_t`, and see [modernize-use-constraints](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-constraints.html) for another tool that replaces `std::enable_if` with C++20 constraints. Consider these newer mechanisms where possible.

### bugprone-incorrect-roundings

Checks the usage of patterns known to produce incorrect rounding.

Programmers often use:

```c++
(int)(double_expression + 0.5)
```

to round the double expression to an integer.

The problem with this:

1. It is unnecessarily slow.
2. It is incorrect. The number 0.499999975 (smallest representable float number below 0.5) rounds to 1.0.
   - Even worse behavior for negative numbers where both -0.5f and -1.4f both round to 0.0.

### bugprone-infinite-loop

Finds obvious infinite loops (loops where the condition variable is not changed at all).

Finding infinite loops is well-known to be impossible (halting problem). However, it is possible to detect some obvious infinite loops, for example, if the loop condition is not changed. This check detects such loops. A loop is considered infinite if it does not have any loop exit statement (`break`, `continue`, `goto`, `return`, `throw` or a call to a function called as `[[noreturn]]`) and all of the following conditions hold for every variable in the condition:

- It is a local variable.
- It has no reference or pointer aliases.
- It is not a structure or class member.

Furthermore, the condition must not contain a function call to consider the loop infinite since functions may return different values for different calls.

For example, the following loop is considered infinite i is not changed in the body:

```c++
int i = 0, j = 0;
while (i < 10) {
  ++j;
}
```

### bugprone-integer-division

Finds cases where integer division in a floating point context is likely to cause unintended loss of precision.

No reports are made if divisions are part of the following expressions:

- operands of operators expecting integral or bool types,
- call expressions of integral or bool types, and
- explicit cast expressions to integral or bool types,

as these are interpreted as signs of deliberateness from the programmer.

Examples:

```c++
float floatFunc(float);
int intFunc(int);
double d;
int i = 42;

// Warn, floating-point values expected.
d = 32 * 8 / (2 + i);
d = 8 * floatFunc(1 + 7 / 2);
d = i / (1 << 4);

// OK, no integer division.
d = 32 * 8.0 / (2 + i);
d = 8 * floatFunc(1 + 7.0 / 2);
d = (double)i / (1 << 4);

// OK, there are signs of deliberateness.
d = 1 << (i / 2);
d = 9 + intFunc(6 * i / 32);
d = (int)(i / 32) - 8;
```

### bugprone-lambda-function-name

Checks for attempts to get the name of a function from within a lambda expression. The name of a lambda is always something like `operator()`, which is almost never what was intended.

Example:

```c++
void FancyFunction() {
  [] { printf("Called from %s\n", __func__); }();
  [] { printf("Now called from %s\n", __FUNCTION__); }();
}
```

Output:

```c++
Called from operator()
Now called from operator()
```

Likely intended output:

```c++
Called from FancyFunction
Now called from FancyFunction
```

#### Options

##### `IgnoreMacros`

The value true specifies that attempting to get the name of a function from within a macro should not be diagnosed.

The default value is `false`.

### bugprone-macro-parentheses

Finds macros that can have unexpected behavior due to missing parentheses.

Macros are expanded by the preprocessor as-is. As a result, there can be unexpected behavior; operators may be evaluated in unexpected order and unary operators may become binary operators, etc.

When the replacement list has an expression, it is recommended to surround it with parentheses. This ensures that the macro result is evaluated completely before it is used.

It is also recommended to surround macro arguments in the replacement list with parentheses. This ensures that the argument value is calculated properly.

### bugprone-macro-repeated-side-effects

Checks for repeated argument with side effects in macros.

### bugprone-misplaced-operator-in-strlen-in-alloc

Finds cases where `1` is added to the string in the argument to `strlen()`, `strnlen()`, `strnlen_s()`, `wcslen()`, `wcsnlen()`, and `wcsnlen_s()` instead of the result and the value is used as an argument to a memory allocation function (`malloc()`, `calloc()`, `realloc()`, `alloca()`) or the `new[]` operator in C++. The check detects error cases even if one of these functions (except the `new[]` operator) is called by a constant function pointer. Cases where `1` is added both to the parameter and the result of the `strlen()`-like function are ignored, as are cases where the whole addition is surrounded by extra parentheses.

C example code:

```c++
void bad_malloc(char *str) {
  char *c = (char*) malloc(strlen(str + 1));
}
```

The suggested fix is to add `1` to the return value of `strlen()` and not to its argument. In the example above the fix would be

```c++
char *c = (char*) malloc(strlen(str) + 1);
```

C++ example code:

```c++
void bad_new(char *str) {
  char *c = new char[strlen(str + 1)];
}
```

As in the C code with the `malloc()` function, the suggested fix is to add `1` to the return value of `strlen()` and not to its argument.

In the example above the fix would be

```c++
char *c = new char[strlen(str) + 1];
```

Example for silencing the diagnostic:

```c++
void bad_malloc(char *str) {
  char *c = (char*) malloc(strlen((str + 1)));
}
```

### bugprone-misplaced-pointer-arithmetic-in-alloc

Finds cases where an integer expression is added to or subtracted from the result of a memory allocation function (`malloc()`, `calloc()`, `realloc()`, `alloca()`) instead of its argument. The check detects error cases even if one of these functions is called by a constant function pointer.

Example code:

```c++
void bad_malloc(int n) {
  char *p = (char*) malloc(n) + 10;
}
```

The suggested fix is to add the integer expression to the argument of `malloc` and not to its result. In the example above the fix would be

```c++
char *p = (char*) malloc(n + 10);
```

### bugprone-misplaced-widening-cast

This check will warn when there is a cast of a calculation result to a bigger type. If the intention of the cast is to avoid loss of precision then the cast is misplaced, and there can be loss of precision. Otherwise the cast is ineffective.

Example code:

```c++
long f(int x) {
    return (long)(x * 1000);
}
```

The result `x * 1000` is first calculated using `int` precision. If the result exceeds `int` precision there is loss of precision. Then the result is casted to `long`.

If there is no loss of precision then the cast can be removed or you can explicitly cast to `int` instead.

If you want to avoid loss of precision then put the cast in a proper location, for instance:

```c++
long f(int x) {
    return (long)x * 1000;
}
```

#### Implicit casts

Forgetting to place the cast at all is at least as dangerous and at least as common as misplacing it.

If [`CheckImplicitCasts`](https://clang.llvm.org/extra/clang-tidy/checks/bugprone/misplaced-widening-cast.html#cmdoption-arg-CheckImplicitCasts) is enabled the check also detects these cases, for instance:

```c++
long f(int x) {
    return x * 1000;
}
```

#### Floating point

Currently warnings are only written for integer conversion. No warning is written for this code:

```c++
double f(float x) {
    return (double)(x * 10.0f);
}
```

#### Options

##### `CheckImplicitCasts`

If true, enables detection of implicit casts.

Default is `false`.

### bugprone-move-forwarding-reference

Warns if `std::move` is called on a forwarding reference, for example:

```c++
template <typename T>
void foo(T&& t) {
  bar(std::move(t));
}
```

[Forwarding references](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4164.pdf) should typically be passed to `std::forward` instead of `std::move`, and this is the fix that will be suggested.

(A forwarding reference is an rvalue reference of a type that is a deduced function template argument.)

In this example, the suggested fix would be

```c++
bar(std::forward<T>(t));
```

#### Background

Code like the example above is sometimes written with the expectation that `T&&` will always end up being an rvalue reference, no matter what type is deduced for `T`, and that it is therefore not possible to pass an lvalue to `foo()`.

However, this is not true. Consider this example:

```c++
std::string s = "Hello, world";
foo(s);
```

This code compiles and, after the call to `foo()`, `s` is left in an indeterminate state because it has been moved from. This may be surprising to the caller of `foo()` because no `std::move` was used when calling `foo()`.

The reason for this behavior lies in the special rule for template argument deduction on function templates like `foo()` – i.e. on function templates that take an rvalue reference argument of a type that is a deduced function template argument. (See section [temp.deduct.call]/3 in the C++11 standard.)

If `foo()` is called on an lvalue (as in the example above), then `T` is deduced to be an lvalue reference. In the example, `T` is deduced to be `std::string &`. The type of the argument `t` therefore becomes `std::string& &&`; by the reference collapsing rules, this collapses to `std::string&`.

This means that the `foo(s)` call passes `s` as an lvalue reference, and `foo()` ends up moving `s` and thereby placing it into an indeterminate state.

### bugprone-multi-level-implicit-pointer-conversion

Detects implicit conversions between pointers of different levels of indirection.

Conversions between pointer types of different levels of indirection can be dangerous and may lead to undefined behavior, particularly if the converted pointer is later cast to a type with a different level of indirection. For example, converting a pointer to a pointer to an `int` (`int**`) to a `void*` can result in the loss of information about the original level of indirection, which can cause problems when attempting to use the converted pointer. If the converted pointer is later cast to a type with a different level of indirection and dereferenced, it may lead to access violations, memory corruption, or other undefined behavior.

Consider the following example:

```c++
void foo(void* ptr);

int main() {
  int x = 42;
  int* ptr = &x;
  int** ptr_ptr = &ptr;
  foo(ptr_ptr); // warning will trigger here
  return 0;
}
```

In this example, `foo()` is called with `ptr_ptr` as its argument. However, `ptr_ptr` is a `int**` pointer, while `foo()` expects a `void*` pointer. This results in an implicit pointer level conversion, which could cause issues if `foo()` dereferences the pointer assuming it’s a `int*` pointer.

Using an explicit cast is a recommended solution to prevent issues caused by implicit pointer level conversion, as it allows the developer to explicitly state their intention and show their reasoning for the type conversion. Additionally, it is recommended that developers thoroughly check and verify the safety of the conversion before using an explicit cast. This extra level of caution can help catch potential issues early on in the development process, improving the overall reliability and maintainability of the code.

### bugprone-multiple-new-in-one-expression

Finds multiple `new` operator calls in a single expression, where the allocated memory by the first `new` may leak if the second allocation fails and throws exception.

C++ does often not specify the exact order of evaluation of the operands of an operator or arguments of a function. Therefore if a first allocation succeeds and a second fails, in an exception handler it is not possible to tell which allocation has failed and free the memory. Even if the order is fixed the result of a first `new` may be stored in a temporary location that is not reachable at the time when a second allocation fails. It is best to avoid any expression that contains more than one `operator new` call, if exception handling is used to check for allocation errors.

Different rules apply for are the short-circuit operators `||` and `&&` and the `,` operator, where evaluation of one side must be completed before the other starts. Expressions of a list-initialization (initialization or construction using `{` and `}` characters) are evaluated in fixed order. Similarly, condition of a `?` operator is evaluated before the branches are evaluated.

The check reports warning if two `new` calls appear in one expression at different sides of an operator, or if `new` calls appear in different arguments of a function call (that can be an object construction with `()` syntax). These `new` calls can be nested at any level. For any warning to be emitted the `new` calls should be in a code block where exception handling is used with catch for `std::bad_alloc` or `std::exception`. At `||`, `&&`, `,`, `?` (condition and one branch) operators no warning is emitted. No warning is emitted if both of the memory allocations are not assigned to a variable or not passed directly to a function. The reason is that in this case the memory may be intentionally not freed or the allocated objects can be self-destructing objects.

Examples:

```
struct A {
  int Var;
};
struct B {
  B();
  B(A *);
  int Var;
};
struct C {
  int *X1;
  int *X2;
};

void f(A *, B *);
int f1(A *);
int f1(B *);
bool f2(A *);

void foo() {
  A *PtrA;
  B *PtrB;
  try {
    // Allocation of 'B'/'A' may fail after memory for 'A'/'B' was allocated.
    f(new A, new B); // warning: memory allocation may leak if an other allocation is sequenced after it and throws an exception; order of these allocations is undefined

    // List (aggregate) initialization is used.
    C C1{new int, new int}; // no warning

    // Allocation of 'B'/'A' may fail after memory for 'A'/'B' was allocated but not yet passed to function 'f1'.
    int X = f1(new A) + f1(new B); // warning: memory allocation may leak if an other allocation is sequenced after it and throws an exception; order of these allocations is undefined

    // Allocation of 'B' may fail after memory for 'A' was allocated.
    // From C++17 on memory for 'B' is allocated first but still may leak if allocation of 'A' fails.
    PtrB = new B(new A); // warning: memory allocation may leak if an other allocation is sequenced after it and throws an exception

    // 'new A' and 'new B' may be performed in any order.
    // 'new B'/'new A' may fail after memory for 'A'/'B' was allocated but not assigned to 'PtrA'/'PtrB'.
    (PtrA = new A)->Var = (PtrB = new B)->Var; // warning: memory allocation may leak if an other allocation is sequenced after it and throws an exception; order of these allocations is undefined

    // Evaluation of 'f2(new A)' must be finished before 'f1(new B)' starts.
    // If 'new B' fails the allocated memory for 'A' is supposedly handled correctly because function 'f2' could take the ownership.
    bool Z = f2(new A) || f1(new B); // no warning

    X = (f2(new A) ? f1(new A) : f1(new B)); // no warning

    // No warning if the result of both allocations is not passed to a function
    // or stored in a variable.
    (new A)->Var = (new B)->Var; // no warning

    // No warning if at least one non-throwing allocation is used.
    f(new(std::nothrow) A, new B); // no warning
  } catch(std::bad_alloc) {
  }

  // No warning if the allocation is outside a try block (or no catch handler exists for std::bad_alloc).
  // (The fact if exceptions can escape from 'foo' is not taken into account.)
  f(new A, new B); // no warning
}
```

### bugprone-multiple-statement-macro

Detect multiple statement macros that are used in unbraced conditionals. Only the first statement of the macro will be inside the conditional and the other ones will be executed unconditionally.

Example:

```
#define INCREMENT_TWO(x, y) (x)++; (y)++
if (do_increment)
  INCREMENT_TWO(a, b);  // (b)++ will be executed unconditionally.
```

### bugprone-no-escape

Finds pointers with the `noescape` attribute that are captured by an asynchronously-executed block. The block arguments in `dispatch_async()` and `dispatch_after()` are guaranteed to escape, so it is an error if a pointer with the `noescape` attribute is captured by one of these blocks.

The following is an example of an invalid use of the `noescape` attribute.

```
void foo(__attribute__((noescape)) int *p) {
  dispatch_async(queue, ^{
    *p = 123;
  });
});
```

### bugprone-non-zero-enum-to-bool-conversion

Detect implicit and explicit casts of `enum` type into `bool` where `enum` type doesn’t have a zero-value enumerator. If the `enum` is used only to hold values equal to its enumerators, then conversion to `bool` will always result in `true` value. This can lead to unnecessary code that reduces readability and maintainability and can result in bugs.

May produce false positives if the `enum` is used to store other values (used as a bit-mask or zero-initialized on purpose). To deal with them, `// NOLINT` or casting first to the underlying type before casting to `bool` can be used.

It is important to note that this check will not generate warnings if the definition of the enumeration type is not available. Additionally, C++11 enumeration classes are supported by this check.

Overall, this check serves to improve code quality and readability by identifying and flagging instances where implicit or explicit casts from enumeration types to boolean could cause potential issues.

#### Example

```
enum EStatus {
  OK = 1,
  NOT_OK,
  UNKNOWN
};

void process(EStatus status) {
  if (!status) {
    // this true-branch won't be executed
    return;
  }
  // proceed with "valid data"
}
```

#### Options

##### `EnumIgnoreList`

Option is used to ignore certain enum types when checking for implicit/explicit casts to bool. It accepts a semicolon-separated list of (fully qualified) enum type names or regular expressions that match the enum type names. The default value is an empty string, which means no enums will be ignored.

### bugprone-not-null-terminated-result

Finds function calls where it is possible to cause a not null-terminated result. Usually the proper length of a string is `strlen(src) + 1` or equal length of this expression, because the null terminator needs an extra space. Without the null terminator it can result in undefined behavior when the string is read.

The following and their respective `wchar_t` based functions are checked:

```
memcpy`, `memcpy_s`, `memchr`, `memmove`, `memmove_s`, `strerror_s`, `strncmp`, `strxfrm
```

The following is a real-world example where the programmer forgot to increase the passed third argument, which is `size_t length`. That is why the length of the allocated memory is not enough to hold the null terminator.

```
static char *stringCpy(const std::string &str) {
  char *result = reinterpret_cast<char *>(malloc(str.size()));
  memcpy(result, str.data(), str.size());
  return result;
}
```

In addition to issuing warnings, fix-it rewrites all the necessary code. It also tries to adjust the capacity of the destination array:

```
static char *stringCpy(const std::string &str) {
  char *result = reinterpret_cast<char *>(malloc(str.size() + 1));
  strcpy(result, str.data());
  return result;
}
```

Note: It cannot guarantee to rewrite every of the path-sensitive memory allocations.

#### Transformation rules of ‘memcpy()’

It is possible to rewrite the `memcpy()` and `memcpy_s()` calls as the following four functions: `strcpy()`, `strncpy()`, `strcpy_s()`, `strncpy_s()`, where the latter two are the safer versions of the former two. It rewrites the `wchar_t` based memory handler functions respectively.

##### Rewrite based on the destination array

- If copy to the destination array cannot overflow [1] the new function should be the older copy function (ending with `cpy`), because it is more efficient than the safe version.
- If copy to the destination array can overflow [1] and [`WantToUseSafeFunctions`](https://clang.llvm.org/extra/clang-tidy/checks/bugprone/not-null-terminated-result.html#cmdoption-arg-WantToUseSafeFunctions) is set to true and it is possible to obtain the capacity of the destination array then the new function could be the safe version (ending with `cpy_s`).
- If the new function is could be safe version and C++ files are analyzed and the destination array is plain `char`/`wchar_t` without `un/signed` then the length of the destination array can be omitted.
- If the new function is could be safe version and the destination array is `un/signed` it needs to be casted to plain `char *`/`wchar_t *`.

- [1] It is possible to overflow:

  If the capacity of the destination array is unknown.If the given length is equal to the destination array’s capacity.

##### Rewrite based on the length of the source string

- If the given length is `strlen(source)` or equal length of this expression then the new function should be the older copy function (ending with `cpy`), as it is more efficient than the safe version (ending with `cpy_s`).
- Otherwise we assume that the programmer wanted to copy ‘N’ characters, so the new function is `ncpy`-like which copies ‘N’ characters.

#### Transformations with ‘strlen()’ or equal length of this expression

It transforms the `wchar_t` based memory and string handler functions respectively (where only `strerror_s` does not have `wchar_t` based alias).

##### Memory handler functions

`memcpy` Please visit the [Transformation rules of ‘memcpy()’](https://clang.llvm.org/extra/clang-tidy/checks/bugprone/not-null-terminated-result.html#memcpytransformation) section.

`memchr` Usually there is a C-style cast and it is needed to be removed, because the new function `strchr`’s return type is correct. The given length is going to be removed.

`memmove` If safe functions are available the new function is `memmove_s`, which has a new second argument which is the length of the destination array, it is adjusted, and the length of the source string is incremented by one. If safe functions are not available the given length is incremented by one.

`memmove_s` The given length is incremented by one.

##### String handler functions

`strerror_s` The given length is incremented by one.

`strncmp` If the third argument is the first or the second argument’s `length + 1` it has to be truncated without the `+ 1` operation.

`strxfrm` The given length is incremented by one.

#### Options

##### `WantToUseSafeFunctions`

The value true specifies that the target environment is considered to implement ‘_s’ suffixed memory and string handler functions which are safer than older versions (e.g. ‘memcpy_s()’). The default value is true.

### bugprone-optional-value-conversion

Detects potentially unintentional and redundant conversions where a value is extracted from an optional-like type and then used to create a new instance of the same optional-like type.

These conversions might be the result of developer oversight, leftovers from code refactoring, or other situations that could lead to unintended exceptions or cases where the resulting optional is always initialized, which might be unexpected behavior.

To illustrate, consider the following problematic code snippet:

```
#include <optional>

void print(std::optional<int>);

int main()
{
  std::optional<int> opt;
  // ...

  // Unintentional conversion from std::optional<int> to int and back to
  // std::optional<int>:
  print(opt.value());

  // ...
}
```

A better approach would be to directly pass `opt` to the `print` function without extracting its value:

```
#include <optional>

void print(std::optional<int>);

int main()
{
  std::optional<int> opt;
  // ...

  // Proposed code: Directly pass the std::optional<int> to the print
  // function.
  print(opt);

  // ...
}
```

By passing `opt` directly to the print function, unnecessary conversions are avoided, and potential unintended behavior or exceptions are minimized.

Value extraction using `operator *` is matched by default. The support for non-standard optional types such as `boost::optional` or `absl::optional` may be limited.

#### Options

##### `OptionalTypes`

Semicolon-separated list of (fully qualified) optional type names or regular expressions that match the optional types.

Default value is ::std::optional;::absl::optional;::boost::optional.

##### `ValueMethods`

Semicolon-separated list of (fully qualified) method names or regular expressions that match the methods.

Default value is `::value$`;`::get$`.

### bugprone-posix-return

Checks if any calls to `pthread_*` or `posix_*` functions (except `posix_openpt`) expect negative return values. These functions return either `0` on success or an `errno` on failure, which is positive only.

Example buggy usage looks like:

```
if (posix_fadvise(...) < 0) {
```

This will never happen as the return value is always non-negative. A simple fix could be:

```
if (posix_fadvise(...) > 0) {
```

### bugprone-redundant-branch-condition

Finds condition variables in nested `if` statements that were also checked in the outer `if` statement and were not changed.

Simple example:

```
bool onFire = isBurning();
if (onFire) {
  if (onFire)
    scream();
}
```

Here onFire is checked both in the outer `if` and the inner `if` statement without a possible change between the two checks. The check warns for this code and suggests removal of the second checking of variable onFire.

The checker also detects redundant condition checks if the condition variable is an operand of a logical “and” (`&&`) or a logical “or” (`||`) operator:

```
bool onFire = isBurning();
if (onFire) {
  if (onFire && peopleInTheBuilding > 0)
    scream();
}
bool onFire = isBurning();
if (onFire) {
  if (onFire || isCollapsing())
    scream();
}
```

In the first case (logical “and”) the suggested fix is to remove the redundant condition variable and keep the other side of the `&&`. In the second case (logical “or”) the whole `if` is removed similarly to the simple case on the top.

The condition of the outer `if` statement may also be a logical “and” (`&&`) expression:

```
bool onFire = isBurning();
if (onFire && fireFighters < 10) {
  if (someOtherCondition()) {
    if (onFire)
      scream();
  }
}
```

The error is also detected if both the outer statement is a logical “and” (`&&`) and the inner statement is a logical “and” (`&&`) or “or” (`||`). The inner `if` statement does not have to be a direct descendant of the outer one.

No error is detected if the condition variable may have been changed between the two checks:

```
bool onFire = isBurning();
if (onFire) {
  tryToExtinguish(onFire);
  if (onFire && peopleInTheBuilding > 0)
    scream();
}
```

Every possible change is considered, thus if the condition variable is not a local variable of the function, it is a volatile or it has an alias (pointer or reference) then no warning is issued.

#### Known limitations

The `else` branch is not checked currently for negated condition variable:

```
bool onFire = isBurning();
if (onFire) {
  scream();
} else {
  if (!onFire) {
    continueWork();
  }
}
```

The checker currently only detects redundant checking of single condition variables.

More complex expressions are not checked:

```
if (peopleInTheBuilding == 1) {
  if (peopleInTheBuilding == 1) {
    doSomething();
  }
}
```

### bugprone-reserved-identifier

cert-dcl37-c and cert-dcl51-cpp redirect here as an alias for this check.

Checks for usages of identifiers reserved for use by the implementation.

The C and C++ standards both reserve the following names for such use:

- identifiers that begin with an underscore followed by an uppercase letter;
- identifiers in the global namespace that begin with an underscore.

The C standard additionally reserves names beginning with a double underscore, while the C++ standard strengthens this to reserve names with a double underscore occurring anywhere.

Violating the naming rules above results in undefined behavior.

```
namespace NS {
  void __f(); // name is not allowed in user code
  using _Int = int; // same with this
  #define cool__macro // also this
}
int _g(); // disallowed in global namespace only
```

The check can also be inverted, i.e. it can be configured to flag any identifier that is _not_ a reserved identifier. This mode is for use by e.g. standard library implementors, to ensure they don’t infringe on the user namespace.

This check does not (yet) check for other reserved names, e.g. macro names identical to language keywords, and names specifically reserved by language standards, e.g. C++ ‘zombie names’ and C future library directions.

This check corresponds to CERT C Coding Standard rule [DCL37-C. Do not declare or define a reserved identifier](https://wiki.sei.cmu.edu/confluence/display/c/DCL37-C.+Do+not+declare+or+define+a+reserved+identifier) as well as its C++ counterpart, [DCL51-CPP. Do not declare or define a reserved identifier](https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL51-CPP.+Do+not+declare+or+define+a+reserved+identifier).

#### Options

##### `Invert`

If true, inverts the check, i.e. flags names that are not reserved.

Default is false.

##### `AllowedIdentifiers`

Semicolon-separated list of regular expressions that the check ignores.

Default is an empty list.

### bugprone-shared-ptr-array-mismatch

Finds initializations of C++ shared pointers to non-array type that are initialized with an array.

If a shared pointer `std::shared_ptr<T>` is initialized with a new-expression `new T[]` the memory is not deallocated correctly. The pointer uses plain `delete` in this case to deallocate the target memory. Instead a `delete[]` call is needed. A `std::shared_ptr<T[]>` calls the correct delete operator.

The check offers replacement of `shared_ptr<T>` to `shared_ptr<T[]>` if it is used at a single variable declaration (one variable in one statement).

Example:

```
std::shared_ptr<Foo> x(new Foo[10]); // -> std::shared_ptr<Foo[]> x(new Foo[10]);
//                     ^ warning: shared pointer to non-array is initialized with array [bugprone-shared-ptr-array-mismatch]
std::shared_ptr<Foo> x1(new Foo), x2(new Foo[10]); // no replacement
//                                   ^ warning: shared pointer to non-array is initialized with array [bugprone-shared-ptr-array-mismatch]

std::shared_ptr<Foo> x3(new Foo[10], [](const Foo *ptr) { delete[] ptr; }); // no warning

struct S {
  std::shared_ptr<Foo> x(new Foo[10]); // no replacement in this case
  //                     ^ warning: shared pointer to non-array is initialized with array [bugprone-shared-ptr-array-mismatch]
};
```

This check partially covers the CERT C++ Coding Standard rule [MEM51-CPP. Properly deallocate dynamically allocated resources](https://wiki.sei.cmu.edu/confluence/display/cplusplus/MEM51-CPP.+Properly+deallocate+dynamically+allocated+resources) However, only the `std::shared_ptr` case is detected by this check.

### bugprone-signal-handler

Finds specific constructs in signal handler functions that can cause undefined behavior. The rules for what is allowed differ between C++ language versions.

Checked signal handler rules for C:

- Calls to non-asynchronous-safe functions are not allowed.

Checked signal handler rules for up to and including C++14:

- Calls to non-asynchronous-safe functions are not allowed.
- C++-specific code constructs are not allowed in signal handlers. In other words, only the common subset of C and C++ is allowed to be used.
- Calls to functions with non-C linkage are not allowed (including the signal handler itself).

The check is disabled on C++17 and later.

Asynchronous-safety is determined by comparing the function’s name against a set of known functions. In addition, the function must come from a system header include and in a global namespace. The (possible) arguments passed to the function are not checked. Any function that cannot be determined to be asynchronous-safe is assumed to be non-asynchronous-safe by the check, including user functions for which only the declaration is visible. Calls to user-defined functions with visible definitions are checked recursively.

This check implements the CERT C Coding Standard rule [SIG30-C. Call only asynchronous-safe functions within signal handlers](https://www.securecoding.cert.org/confluence/display/c/SIG30-C.+Call+only+asynchronous-safe+functions+within+signal+handlers) and the rule [MSC54-CPP. A signal handler must be a plain old function](https://wiki.sei.cmu.edu/confluence/display/cplusplus/MSC54-CPP.+A+signal+handler+must+be+a+plain+old+function). It has the alias names `cert-sig30-c` and `cert-msc54-cpp`.

#### Options

##### `AsyncSafeFunctionSet`

Selects which set of functions is considered as asynchronous-safe (and therefore allowed in signal handlers).

It can be set to the following values:

`minimal`

- Selects a minimal set that is defined in the CERT SIG30-C rule. and includes functions `abort()`, `_Exit()`, `quick_exit()` and `signal()`.

`POSIX`

- Selects a larger set of functions that is listed in POSIX.1-2017 (see [this link](https://pubs.opengroup.org/onlinepubs/9699919799/functions/V2_chap02.html#tag_15_04_03) for more information).
- The following functions are included: `_Exit`, `_exit`, `abort`, `accept`, `access`, `aio_error`, `aio_return`, `aio_suspend`, `alarm`, `bind`, `cfgetispeed`, `cfgetospeed`, `cfsetispeed`, `cfsetospeed`, `chdir`, `chmod`, `chown`, `clock_gettime`, `close`, `connect`, `creat`, `dup`, `dup2`, `execl`, `execle`, `execv`, `execve`, `faccessat`, `fchdir`, `fchmod`, `fchmodat`, `fchown`, `fchownat`, `fcntl`, `fdatasync`, `fexecve`, `ffs`, `fork`, `fstat`, `fstatat`, `fsync`, `ftruncate`, `futimens`, `getegid`, `geteuid`, `getgid`, `getgroups`, `getpeername`, `getpgrp`, `getpid`, `getppid`, `getsockname`, `getsockopt`, `getuid`, `htonl`, `htons`, `kill`, `link`, `linkat`, `listen`, `longjmp`, `lseek`, `lstat`, `memccpy`, `memchr`, `memcmp`, `memcpy`, `memmove`, `memset`, `mkdir`, `mkdirat`, `mkfifo`, `mkfifoat`, `mknod`, `mknodat`, `ntohl`, `ntohs`, `open`, `openat`, `pause`, `pipe`, `poll`, `posix_trace_event`, `pselect`, `pthread_kill`, `pthread_self`, `pthread_sigmask`, `quick_exit`, `raise`, `read`, `readlink`, `readlinkat`, `recv`, `recvfrom`, `recvmsg`, `rename`, `renameat`, `rmdir`, `select`, `sem_post`, `send`, `sendmsg`, `sendto`, `setgid`, `setpgid`, `setsid`, `setsockopt`, `setuid`, `shutdown`, `sigaction`, `sigaddset`, `sigdelset`, `sigemptyset`, `sigfillset`, `sigismember`, `siglongjmp`, `signal`, `sigpause`, `sigpending`, `sigprocmask`, `sigqueue`, `sigset`, `sigsuspend`, `sleep`, `sockatmark`, `socket`, `socketpair`, `stat`, `stpcpy`, `stpncpy`, `strcat`, `strchr`, `strcmp`, `strcpy`, `strcspn`, `strlen`, `strncat`, `strncmp`, `strncpy`, `strnlen`, `strpbrk`, `strrchr`, `strspn`, `strstr`, `strtok_r`, `symlink`, `symlinkat`, `tcdrain`, `tcflow`, `tcflush`, `tcgetattr`, `tcgetpgrp`, `tcsendbreak`, `tcsetattr`, `tcsetpgrp`, `time`, `timer_getoverrun`, `timer_gettime`, `timer_settime`, `times`, `umask`, `uname`, `unlink`, `unlinkat`, `utime`, `utimensat`, `utimes`, `wait`, `waitpid`, `wcpcpy`, `wcpncpy`, `wcscat`, `wcschr`, `wcscmp`, `wcscpy`, `wcscspn`, `wcslen`, `wcsncat`, `wcsncmp`, `wcsncpy`, `wcsnlen`, `wcspbrk`, `wcsrchr`, `wcsspn`, `wcsstr`, `wcstok`, `wmemchr`, `wmemcmp`, `wmemcpy`, `wmemmove`, `wmemset`, `write`.
- The function `quick_exit` is not included in the POSIX list but it is included here in the set of safe functions.

The default value is `POSIX`.

### bugprone-signed-char-misuse

cert-str34-c redirects here as an alias for this check. For the CERT alias, the DiagnoseSignedUnsignedCharComparisons option is set to false.

Finds those `signed char` -> integer conversions which might indicate a programming error. The basic problem with the `signed char`, that it might store the non-ASCII characters as negative values. This behavior can cause a misunderstanding of the written code both when an explicit and when an implicit conversion happens.

When the code contains an explicit `signed char` -> integer conversion, the human programmer probably expects that the converted value matches with the character code (a value from [0..255]), however, the actual value is in [-128..127] interval. To avoid this kind of misinterpretation, the desired way of converting from a `signed char` to an integer value is converting to `unsigned char` first, which stores all the characters in the positive [0..255] interval which matches the known character codes.

In case of implicit conversion, the programmer might not actually be aware that a conversion happened and char value is used as an integer. There are some use cases when this unawareness might lead to a functionally imperfect code. For example, checking the equality of a `signed char` and an `unsigned char` variable is something we should avoid in C++ code. During this comparison, the two variables are converted to integers which have different value ranges. For `signed char`, the non-ASCII characters are stored as a value in [-128..-1] interval, while the same characters are stored in the [128..255] interval for an `unsigned char`.

It depends on the actual platform whether plain `char` is handled as `signed char` by default and so it is caught by this check or not. To change the default behavior you can use `-funsigned-char` and `-fsigned-char` compilation options.

Currently, this check warns in the following cases: - `signed char` is assigned to an integer variable - `signed char` and `unsigned char` are compared with equality/inequality operator - `signed char` is converted to an integer in the array subscript

See also: [STR34-C. Cast characters to unsigned char before converting to larger integer sizes](https://wiki.sei.cmu.edu/confluence/display/c/STR34-C.+Cast+characters+to+unsigned+char+before+converting+to+larger+integer+sizes)

A good example from the CERT description when a `char` variable is used to read from a file that might contain non-ASCII characters. The problem comes up when the code uses the `-1` integer value as EOF, while the 255 character code is also stored as `-1` in two’s complement form of char type. See a simple example of this below. This code stops not only when it reaches the end of the file, but also when it gets a character with the 255 code.

```
#define EOF (-1)

int read(void) {
  char CChar;
  int IChar = EOF;

  if (readChar(CChar)) {
    IChar = CChar;
  }
  return IChar;
}
```

A proper way to fix the code above is converting the `char` variable to an `unsigned char` value first.

```
#define EOF (-1)

int read(void) {
  char CChar;
  int IChar = EOF;

  if (readChar(CChar)) {
    IChar = static_cast<unsigned char>(CChar);
  }
  return IChar;
}
```

Another use case is checking the equality of two `char` variables with different signedness. Inside the non-ASCII value range this comparison between a `signed char` and an `unsigned char` always returns `false`.

```
bool compare(signed char SChar, unsigned char USChar) {
  if (SChar == USChar)
    return true;
  return false;
}
```

The easiest way to fix this kind of comparison is casting one of the arguments, so both arguments will have the same type.

```
bool compare(signed char SChar, unsigned char USChar) {
  if (static_cast<unsigned char>(SChar) == USChar)
    return true;
  return false;
}
```

#### Options

`CharTypdefsToIgnore`

A semicolon-separated list of typedef names. In this list, we can list typedefs for `char` or `signed char`, which will be ignored by the check. This is useful when a typedef introduces an integer alias like `sal_Int8` or `int8_t`. In this case, human misinterpretation is not an issue.

`DiagnoseSignedUnsignedCharComparisons`

When true, the check will warn on `signed char`/`unsigned char` comparisons, otherwise these comparisons are ignored.

By default, this option is set to true.

### bugprone-sizeof-container

The check finds usages of `sizeof` on expressions of STL container types. Most likely the user wanted to use `.size()` instead.

All class/struct types declared in namespace `std::` having a const `size()` method are considered containers, with the exception of `std::bitset` and `std::array`.

Examples:

```
std::string s;
int a = 47 + sizeof(s); // warning: sizeof() doesn't return the size of the container. Did you mean .size()?

int b = sizeof(std::string); // no warning, probably intended.

std::string array_of_strings[10];
int c = sizeof(array_of_strings) / sizeof(array_of_strings[0]); // no warning, definitely intended.

std::array<int, 3> std_array;
int d = sizeof(std_array); // no warning, probably intended.
```

### bugprone-sizeof-expression

The check finds usages of `sizeof` expressions which are most likely errors.

The `sizeof` operator yields the size (in bytes) of its operand, which may be an expression or the parenthesized name of a type. Misuse of this operator may be leading to errors and possible software vulnerabilities.

#### Suspicious usage of `sizeof(K)`

A common mistake is to query the `sizeof` of an integer literal. This is equivalent to query the size of its type (probably `int`). The intent of the programmer was probably to simply get the integer and not its size.

```
#define BUFLEN 42
char buf[BUFLEN];
memset(buf, 0, sizeof(BUFLEN));  // sizeof(42) ==> sizeof(int)
```

#### Suspicious usage of `sizeof(expr)`

In cases, where there is an enum or integer to represent a type, a common mistake is to query the `sizeof` on the integer or enum that represents the type that should be used by `sizeof`. This results in the size of the integer and not of the type the integer represents:

```
enum data_type {
  FLOAT_TYPE,
  DOUBLE_TYPE
};

struct data {
  data_type type;
  void* buffer;
  data_type get_type() {
    return type;
  }
};

void f(data d, int numElements) {
  // should be sizeof(float) or sizeof(double), depending on d.get_type()
  int numBytes = numElements * sizeof(d.get_type());
  ...
}
```

#### Suspicious usage of `sizeof(this)`

The `this` keyword is evaluated to a pointer to an object of a given type. The expression `sizeof(this)` is returning the size of a pointer. The programmer most likely wanted the size of the object and not the size of the pointer.

```
class Point {
  [...]
  size_t size() { return sizeof(this); }  // should probably be sizeof(*this)
  [...]
};
```

#### Suspicious usage of `sizeof(char*)`

There is a subtle difference between declaring a string literal with `char* A = ""` and `char A[] = ""`. The first case has the type `char*` instead of the aggregate type `char[]`. Using `sizeof` on an object declared with `char*` type is returning the size of a pointer instead of the number of characters (bytes) in the string literal.

```
const char* kMessage = "Hello World!";      // const char kMessage[] = "...";
void getMessage(char* buf) {
  memcpy(buf, kMessage, sizeof(kMessage));  // sizeof(char*)
}
```

#### Suspicious usage of `sizeof(A*)`

A common mistake is to compute the size of a pointer instead of its pointee. These cases may occur because of explicit cast or implicit conversion.

```
int A[10];
memset(A, 0, sizeof(A + 0));

struct Point point;
memset(point, 0, sizeof(&point));
```

#### Suspicious usage of `sizeof(…)/sizeof(…)`

Dividing `sizeof` expressions is typically used to retrieve the number of elements of an aggregate. This check warns on incompatible or suspicious cases.

In the following example, the entity has 10-bytes and is incompatible with the type `int` which has 4 bytes.

```
char buf[] = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };  // sizeof(buf) => 10
void getMessage(char* dst) {
  memcpy(dst, buf, sizeof(buf) / sizeof(int));  // sizeof(int) => 4  [incompatible sizes]
}
```

In the following example, the expression `sizeof(Values)` is returning the size of `char*`. One can easily be fooled by its declaration, but in parameter declaration the size ‘10’ is ignored and the function is receiving a `char*`.

```
char OrderedValues[10] = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
return CompareArray(char Values[10]) {
  return memcmp(OrderedValues, Values, sizeof(Values)) == 0;  // sizeof(Values) ==> sizeof(char*) [implicit cast to char*]
}
```

#### Suspicious `sizeof` by `sizeof` expression

Multiplying `sizeof` expressions typically makes no sense and is probably a logic error. In the following example, the programmer used `*` instead of `/`.

```
const char kMessage[] = "Hello World!";
void getMessage(char* buf) {
  memcpy(buf, kMessage, sizeof(kMessage) * sizeof(char));  //  sizeof(kMessage) / sizeof(char)
}
```

This check may trigger on code using the arraysize macro. The following code is working correctly but should be simplified by using only the `sizeof` operator.

```
extern Object objects[100];
void InitializeObjects() {
  memset(objects, 0, arraysize(objects) * sizeof(Object));  // sizeof(objects)
}
```

#### Suspicious usage of `sizeof(sizeof(…))`

Getting the `sizeof` of a `sizeof` makes no sense and is typically an error hidden through macros.

```
#define INT_SZ sizeof(int)
int buf[] = { 42 };
void getInt(int* dst) {
  memcpy(dst, buf, sizeof(INT_SZ));  // sizeof(sizeof(int)) is suspicious.
}
```

#### Options

##### `WarnOnSizeOfConstant`

When true, the check will warn on an expression like `sizeof(CONSTANT)`.

Default is true.

##### `WarnOnSizeOfIntegerExpression`

When true, the check will warn on an expression like `sizeof(expr)` where the expression results in an integer.

Default is false.

##### `WarnOnSizeOfThis`

When true, the check will warn on an expression like `sizeof(this)`.

Default is true.

##### `WarnOnSizeOfCompareToConstant`

When true, the check will warn on an expression like `sizeof(expr) <= k` for a suspicious constant k while k is 0 or greater than 0x8000.

Default is true.

##### `WarnOnSizeOfPointerToAggregate`

When true, the check will warn on an expression like ``sizeof(expr)` where the expression is a pointer to aggregate.

Default is true.

### bugprone-spuriously-wake-up-functions

Finds `cnd_wait`, `cnd_timedwait`, `wait`, `wait_for`, or `wait_until` function calls when the function is not invoked from a loop that checks whether a condition predicate holds or the function has a condition parameter.

```
if (condition_predicate) {
    condition.wait(lk);
}
if (condition_predicate) {
    if (thrd_success != cnd_wait(&condition, &lock)) {
    }
}
```

This check corresponds to the CERT C++ Coding Standard rule [CON54-CPP. Wrap functions that can spuriously wake up in a loop](https://wiki.sei.cmu.edu/confluence/display/cplusplus/CON54-CPP.+Wrap+functions+that+can+spuriously+wake+up+in+a+loop). and CERT C Coding Standard rule [CON36-C. Wrap functions that can spuriously wake up in a loop](https://wiki.sei.cmu.edu/confluence/display/c/CON36-C.+Wrap+functions+that+can+spuriously+wake+up+in+a+loop).

### bugprone-standalone-empty

Warns when `empty()` is used on a range and the result is ignored. Suggests `clear()` if it is an existing member function.

The `empty()` method on several common ranges returns a Boolean indicating whether or not the range is empty, but is often mistakenly interpreted as a way to clear the contents of a range. Some ranges offer a `clear()` method for this purpose. This check warns when a call to empty returns a result that is ignored, and suggests replacing it with a call to `clear()` if it is available as a member function of the range.

For example, the following code could be used to indicate whether a range is empty or not, but the result is ignored:

```
std::vector<int> v;
...
v.empty();
```

A call to `clear()` would appropriately clear the contents of the range:

```
std::vector<int> v;
...
v.clear();
```

#### Limitations

- Doesn’t warn if `empty()` is defined and used with the ignore result in the class template definition (for example in the library implementation). These error cases can be caught with `[[nodiscard]]` attribute.

### bugprone-string-constructor

Finds string constructors that are suspicious and probably errors.

A common mistake is to swap parameters to the ‘fill’ string-constructor.

Examples:

```
std::string str('x', 50); // should be str(50, 'x')
```

Calling the string-literal constructor with a length bigger than the literal is suspicious and adds extra random characters to the string.

Examples:

```
std::string("test", 200);   // Will include random characters after "test".
std::string_view("test", 200);
```

Creating an empty string from constructors with parameters is considered suspicious. The programmer should use the empty constructor instead.

Examples:

```
std::string("test", 0);   // Creation of an empty string.
std::string_view("test", 0);
```

#### Options

##### `WarnOnLargeLength`

When true, the check will warn on a string with a length greater than [`LargeLengthThreshold`](https://clang.llvm.org/extra/clang-tidy/checks/bugprone/string-constructor.html#cmdoption-arg-LargeLengthThreshold).

Default is true.

##### `LargeLengthThreshold`

An integer specifying the large length threshold.

Default is 0x800000.

##### `StringNames`

Default is `::std::basic_string`;`::std::basic_string_view`.

Semicolon-delimited list of class names to apply this check to.

By default `::std::basic_string` applies to `std::string` and `std::wstring`.

Set to e.g. `::std::basic_string`;`llvm::StringRef`;`QString` to perform this check on custom classes.

### bugprone-string-integer-assignment

The check finds assignments of an integer to `std::basic_string<CharT>` (`std::string`, `std::wstring`, etc.).

The source of the problem is the following assignment operator of `std::basic_string<CharT>`:

```
basic_string& operator=( CharT ch );
```

Numeric types can be implicitly casted to character types.

```
std::string s;
int x = 5965;
s = 6;
s = x;
```

Use the appropriate conversion functions or character literals.

```
std::string s;
int x = 5965;
s = '6';
s = std::to_string(x);
```

In order to suppress false positives, use an explicit cast.

```
std::string s;
s = static_cast<char>(6);
```

### bugprone-string-literal-with-embedded-nul

Finds occurrences of string literal with embedded NUL character and validates their usage.

#### Invalid escaping

Special characters can be escaped within a string literal by using their hexadecimal encoding like `\x42`. A common mistake is to escape them like this `\0x42` where the `\0` stands for the NUL character.

```
const char* Example[] = "Invalid character: \0x12 should be \x12";
const char* Bytes[] = "\x03\0x02\0x01\0x00\0xFF\0xFF\0xFF";
```

#### Truncated literal

String-like classes can manipulate strings with embedded NUL as they are keeping track of the bytes and the length. This is not the case for a `char*` (NUL-terminated) string.

A common mistake is to pass a string-literal with embedded NUL to a string constructor expecting a NUL-terminated string. The bytes after the first NUL character are truncated.

```
std::string str("abc\0def");  // "def" is truncated
str += "\0";                  // This statement is doing nothing
if (str == "\0abc") return;   // This expression is always true
```

### bugprone-stringview-nullptr

Checks for various ways that the `const CharT*` constructor of `std::basic_string_view` can be passed a null argument and replaces them with the default constructor in most cases. For the comparison operators, braced initializer list does not compile so instead a call to `.empty()` or the empty string literal are used, where appropriate.

This prevents code from invoking behavior which is unconditionally undefined. The single-argument `const CharT*` constructor does not check for the null case before dereferencing its input. The standard is slated to add an explicitly-deleted overload to catch some of these cases: wg21.link/p2166

To catch the additional cases of `NULL` (which expands to `__null`) and `0`, first run the `modernize-use-nullptr` check to convert the callers to `nullptr`.

```
std::string_view sv = nullptr;

sv = nullptr;

bool is_empty = sv == nullptr;
bool isnt_empty = sv != nullptr;

accepts_sv(nullptr);

accepts_sv({{}});  // A

accepts_sv({nullptr, 0});  // B
```

is translated into…

```
std::string_view sv = {};

sv = {};

bool is_empty = sv.empty();
bool isnt_empty = !sv.empty();

accepts_sv("");

accepts_sv("");  // A

accepts_sv({nullptr, 0});  // B
```

#### Note

The source pattern with trailing comment “A” selects the `(const CharT*)` constructor overload and then value-initializes the pointer, causing a null dereference. It happens to not include the `nullptr` literal, but it is still within the scope of this ClangTidy check.

#### Note

The source pattern with trailing comment “B” selects the `(const CharT*, size_type)` constructor which is perfectly valid, since the length argument is `0`. It is not changed by this ClangTidy check.

### bugprone-suspicious-enum-usage

The checker detects various cases when an enum is probably misused (as a bitmask ).

1. When “ADD” or “bitwise OR” is used between two enum which come from different types and these types value ranges are not disjoint.

The following cases will be investigated only using [`StrictMode`](https://clang.llvm.org/extra/clang-tidy/checks/bugprone/argument-comment.html#cmdoption-arg-StrictMode). We regard the enum as a (suspicious) bitmask if the three conditions below are true at the same time:

- at most half of the elements of the enum are non pow-of-2 numbers (because of short enumerations)
- there is another non pow-of-2 number than the enum constant representing all choices (the result “bitwise OR” operation of all enum elements)
- enum type variable/enumconstant is used as an argument of a + or “bitwise OR “ operator

So whenever the non pow-of-2 element is used as a bitmask element we diagnose a misuse and give a warning.

1. Investigating the right hand side of += and |= operator.
2. Check only the enum value side of a | and + operator if one of them is not enum val.
3. Check both side of | or + operator where the enum values are from the same enum type.

Examples:

```
enum { A, B, C };
enum { D, E, F = 5 };
enum { G = 10, H = 11, I = 12 };

unsigned flag;
flag =
    A |
    H; // OK, disjoint value intervals in the enum types ->probably good use.
flag = B | F; // Warning, have common values so they are probably misused.

// Case 2:
enum Bitmask {
  A = 0,
  B = 1,
  C = 2,
  D = 4,
  E = 8,
  F = 16,
  G = 31 // OK, real bitmask.
};

enum Almostbitmask {
  AA = 0,
  BB = 1,
  CC = 2,
  DD = 4,
  EE = 8,
  FF = 16,
  GG // Problem, forgot to initialize.
};

unsigned flag = 0;
flag |= E; // OK.
flag |=
    EE; // Warning at the decl, and note that it was used here as a bitmask.
```

#### Options

##### `StrictMode`

Default value: 0.

When non-null the suspicious bitmask usage will be investigated additionally to the different enum usage check.

### bugprone-suspicious-include

The check detects various cases when an include refers to what appears to be an implementation file, which often leads to hard-to-track-down ODR violations.

Examples:

```
#include "Dinosaur.hpp"     // OK, .hpp files tend not to have definitions.
#include "Pterodactyl.h"    // OK, .h files tend not to have definitions.
#include "Velociraptor.cpp" // Warning, filename is suspicious.
#include_next <stdio.c>     // Warning, filename is suspicious.
```

#### Options

##### `HeaderFileExtensions`

Note: this option is deprecated, it will be removed in **clang-tidy** version 19. Please use the global configuration option HeaderFileExtensions.Default value: `";h;hh;hpp;hxx"` A semicolon-separated list of filename extensions of header files (the filename extensions should not contain a “.” prefix). For extension-less header files, use an empty string or leave an empty string between “;” if there are other filename extensions.

##### ImplementationFileExtensions

Note: this option is deprecated, it will be removed in **clang-tidy** version 19. Please use the global configuration option ImplementationFileExtensions.Default value: `"c;cc;cpp;cxx"` Likewise, a semicolon-separated list of filename extensions of implementation files.

### bugprone-suspicious-memory-comparison

Finds potentially incorrect calls to `memcmp()` based on properties of the arguments.

The following cases are covered:

**Case 1: Non-standard-layout type**

Comparing the object representations of non-standard-layout objects may not properly compare the value representations.

**Case 2: Types with no unique object representation**

Objects with the same value may not have the same object representation. This may be caused by padding or floating-point types.

See also: [EXP42-C. Do not compare padding data](https://wiki.sei.cmu.edu/confluence/display/c/EXP42-C.+Do+not+compare+padding+data) and [FLP37-C. Do not use object representations to compare floating-point values](https://wiki.sei.cmu.edu/confluence/display/c/FLP37-C.+Do+not+use+object+representations+to+compare+floating-point+values)

This check is also related to and partially overlaps the CERT C++ Coding Standard rules [OOP57-CPP. Prefer special member functions and overloaded operators to C Standard Library functions](https://wiki.sei.cmu.edu/confluence/display/cplusplus/OOP57-CPP.+Prefer+special+member+functions+and+overloaded+operators+to+C+Standard+Library+functions) and [EXP62-CPP. Do not access the bits of an object representation that are not part of the object’s value representation](https://wiki.sei.cmu.edu/confluence/display/cplusplus/EXP62-CPP.+Do+not+access+the+bits+of+an+object+representation+that+are+not+part+of+the+object's+value+representation)

### bugprone-suspicious-memset-usage

This check finds `memset()` calls with potential mistakes in their arguments. Considering the function as `void* memset(void* destination, int fill_value, size_t byte_count)`, the following cases are covered:

**Case 1: Fill value is a character `'0'`**

Filling up a memory area with ASCII code 48 characters is not customary, possibly integer zeroes were intended instead. The check offers a replacement of `'0'` with `0`. Memsetting character pointers with `'0'` is allowed.

**Case 2: Fill value is truncated**

Memset converts `fill_value` to `unsigned char` before using it. If `fill_value` is out of unsigned character range, it gets truncated and memory will not contain the desired pattern.

**Case 3: Byte count is zero**

Calling memset with a literal zero in its `byte_count` argument is likely to be unintended and swapped with `fill_value`. The check offers to swap these two arguments.

Corresponding cpplint.py check name: `runtime/memset`.

Examples:

```c++
void foo() {
  int i[5] = {1, 2, 3, 4, 5};
  int *ip = i;
  char c = '1';
  char *cp = &c;
  int v = 0;

  // Case 1
  memset(ip, '0', 1); // suspicious
  memset(cp, '0', 1); // OK

  // Case 2
  memset(ip, 0xabcd, 1); // fill value gets truncated
  memset(ip, 0x00, 1);   // OK

  // Case 3
  memset(ip, sizeof(int), v); // zero length, potentially swapped
  memset(ip, 0, 1);           // OK
}
```

### bugprone-suspicious-missing-comma

String literals placed side-by-side are concatenated at translation phase 6 (after the preprocessor). This feature is used to represent long string literal on multiple lines.

For instance, the following declarations are equivalent:

```c++
const char* A[] = "This is a test";
const char* B[] = "This" " is a "    "test";
```

A common mistake done by programmers is to forget a comma between two string literals in an array initializer list.

```c++
const char* Test[] = {
  "line 1",
  "line 2"     // Missing comma!
  "line 3",
  "line 4",
  "line 5"
};
```

The array contains the string “line 2line3” at offset 1 (i.e. Test[1]). Clang won’t generate warnings at compile time.

This check may warn incorrectly on cases like:

```c++
const char* SupportedFormat[] = {
  "Error %s",
  "Code " PRIu64,   // May warn here.
  "Warning %s",
};
```

#### Options

##### `SizeThreshold`

An unsigned integer specifying the minimum size of a string literal to be considered by the check.

Default is `5U`.

##### `RatioThreshold`

A string specifying the maximum threshold ratio [0, 1.0] of suspicious string literals to be considered.

Default is `".2"`.

##### `MaxConcatenatedTokens`

An unsigned integer specifying the maximum number of concatenated tokens.

Default is `5U`.

### bugprone-suspicious-realloc-usage

This check finds usages of `realloc` where the return value is assigned to the same expression as passed to the first argument: `p = realloc(p, size);` The problem with this construct is that if `realloc` fails it returns a null pointer but does not deallocate the original memory. If no other variable is pointing to it, the original memory block is not available any more for the program to use or free. In either case `p = realloc(p, size);` indicates bad coding style and can be replaced by `q = realloc(p, size);`.

The pointer expression (used at `realloc`) can be a variable or a field member of a data structure, but can not contain function calls or unresolved types.

In obvious cases when the pointer used at realloc is assigned to another variable before the `realloc` call, no warning is emitted. This happens only if a simple expression in form of `q = p` or `void *q = p` is found in the same function where `p = realloc(p, ...)` is found. The assignment has to be before the call to realloc (but otherwise at any place) in the same function. This suppression works only if `p` is a single variable.

Examples:

```c++
struct A {
  void *p;
};

A &getA();

void foo(void *p, A *a, int new_size) {
  p = realloc(p, new_size); // warning: 'p' may be set to null if 'realloc' fails, which may result in a leak of the original buffer
  a->p = realloc(a->p, new_size); // warning: 'a->p' may be set to null if 'realloc' fails, which may result in a leak of the original buffer
  getA().p = realloc(getA().p, new_size); // no warning
}

void foo1(void *p, int new_size) {
  void *p1 = p;
  p = realloc(p, new_size); // no warning
}
```

### bugprone-suspicious-semicolon

Finds most instances of stray semicolons that unexpectedly alter the meaning of the code. More specifically, it looks for `if`, `while`, `for` and `for-range` statements whose body is a single semicolon, and then analyzes the context of the code (e.g. indentation) in an attempt to determine whether that is intentional.

```
if (x < y);
{
  x++;
}
```

Here the body of the `if` statement consists of only the semicolon at the end of the first line, and x will be incremented regardless of the condition.

```
while ((line = readLine(file)) != NULL);
  processLine(line);
```

As a result of this code, processLine() will only be called once, when the `while` loop with the empty body exits with line == NULL. The indentation of the code indicates the intention of the programmer.

```
if (x >= y);
x -= y;
```

While the indentation does not imply any nesting, there is simply no valid reason to have an if statement with an empty body (but it can make sense for a loop). So this check issues a warning for the code above.

To solve the issue remove the stray semicolon or in case the empty body is intentional, reflect this using code indentation or put the semicolon in a new line. For example:

```
while (readWhitespace());
  Token t = readNextToken();
```

Here the second line is indented in a way that suggests that it is meant to be the body of the while loop - whose body is in fact empty, because of the semicolon at the end of the first line.

Either remove the indentation from the second line:

```
while (readWhitespace());
Token t = readNextToken();
```

… or move the semicolon from the end of the first line to a new line:

```
while (readWhitespace())
  ;

  Token t = readNextToken();
```

In this case the check will assume that you know what you are doing, and will not raise a warning.

### bugprone-suspicious-string-compare

Find suspicious usage of runtime string comparison functions. This check is valid in C and C++.

Checks for calls with implicit comparator and proposed to explicitly add it.

```
if (strcmp(...))       // Implicitly compare to zero
if (!strcmp(...))      // Won't warn
if (strcmp(...) != 0)  // Won't warn
```

Checks that compare function results (i.e., `strcmp`) are compared to valid constant. The resulting value is

```
<  0    when lower than,
>  0    when greater than,
== 0    when equals.
```

A common mistake is to compare the result to 1 or -1.

```
if (strcmp(...) == -1)  // Incorrect usage of the returned value.
```

Additionally, the check warns if the results value is implicitly cast to a *suspicious* non-integer type. It’s happening when the returned value is used in a wrong context.

```
if (strcmp(...) < 0.)  // Incorrect usage of the returned value.
```

#### Options

##### `WarnOnImplicitComparison`

When true, the check will warn on implicit comparison.

true by default.

##### `WarnOnLogicalNotComparison`

When true, the check will warn on logical not comparison.

false by default.

##### `StringCompareLikeFunctions`

A string specifying the comma-separated names of the extra string comparison functions.

Default is an empty string.

The check will detect the following string comparison functions: `__builtin_memcmp`, `__builtin_strcasecmp`, `__builtin_strcmp`, `__builtin_strncasecmp`, `__builtin_strncmp`, `_mbscmp`, `_mbscmp_l`, `_mbsicmp`, `_mbsicmp_l`, `_mbsnbcmp`, `_mbsnbcmp_l`, `_mbsnbicmp`, `_mbsnbicmp_l`, `_mbsncmp`, `_mbsncmp_l`, `_mbsnicmp`, `_mbsnicmp_l`, `_memicmp`, `_memicmp_l`, `_stricmp`, `_stricmp_l`, `_strnicmp`, `_strnicmp_l`, `_wcsicmp`, `_wcsicmp_l`, `_wcsnicmp`, `_wcsnicmp_l`, `lstrcmp`, `lstrcmpi`, `memcmp`, `memicmp`, `strcasecmp`, `strcmp`, `strcmpi`, `stricmp`, `strncasecmp`, `strncmp`, `strnicmp`, `wcscasecmp`, `wcscmp`, `wcsicmp`, `wcsncmp`, `wcsnicmp`, `wmemcmp`.

### bugprone-swapped-arguments

Finds potentially swapped arguments by examining implicit conversions. It analyzes the types of the arguments being passed to a function and compares them to the expected types of the corresponding parameters. If there is a mismatch or an implicit conversion that indicates a potential swap, a warning is raised.

```
void printNumbers(int a, float b);

int main() {
  // Swapped arguments: float passed as int, int as float)
  printNumbers(10.0f, 5);
  return 0;
}
```

Covers a wide range of implicit conversions, including: - User-defined conversions - Conversions from floating-point types to boolean or integral types - Conversions from integral types to boolean or floating-point types - Conversions from boolean to integer types or floating-point types - Conversions from (member) pointers to boolean

It is important to note that for most argument swaps, the types need to match exactly. However, there are exceptions to this rule. Specifically, when the swapped argument is of integral type, an exact match is not always necessary. Implicit casts from other integral types are also accepted. Similarly, when dealing with floating-point arguments, implicit casts between different floating-point types are considered acceptable.

To avoid confusion, swaps where both swapped arguments are of integral types or both are of floating-point types do not trigger the warning. In such cases, it’s assumed that the developer intentionally used different integral or floating-point types and does not raise a warning. This approach prevents false positives and provides flexibility in handling situations where varying integral or floating-point types are intentionally utilized.

### bugprone-switch-missing-default-case

Ensures that switch statements without default cases are flagged, focuses only on covering cases with non-enums where the compiler may not issue warnings.

Switch statements without a default case can lead to unexpected behavior and incomplete handling of all possible cases. When a switch statement lacks a default case, if a value is encountered that does not match any of the specified cases, the program will continue execution without any defined behavior or handling.

This check helps identify switch statements that are missing a default case, allowing developers to ensure that all possible cases are handled properly. Adding a default case allows for graceful handling of unexpected or unmatched values, reducing the risk of program errors and unexpected behavior.

Example:

```
// Example 1:
// warning: switching on non-enum value without default case may not cover all cases
switch (i) {
case 0:
  break;
}

// Example 2:
enum E { eE1 };
E e = eE1;
switch (e) { // no-warning
case eE1:
  break;
}

// Example 3:
int i = 0;
switch (i) { // no-warning
case 0:
  break;
default:
  break;
}
```

#### Note

Enum types are already covered by compiler warnings (comes under -Wswitch) when a switch statement does not handle all enum values. This check focuses on non-enum types where the compiler warnings may not be present.

#### See also

The [CppCoreGuideline ES.79](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-default) provide guidelines on switch statements, including the recommendation to always provide a default case.

### bugprone-terminating-continue

Detects `do while` loops with a condition always evaluating to false that have a `continue` statement, as this `continue` terminates the loop effectively.

```
void f() {
do {
  // some code
  continue; // terminating continue
  // some other code
} while(false);
```

### bugprone-throw-keyword-missing

Warns about a potentially missing `throw` keyword. If a temporary object is created, but the object’s type derives from (or is the same as) a class that has ‘EXCEPTION’, ‘Exception’ or ‘exception’ in its name, we can assume that the programmer’s intention was to throw that object.

Example:

```
void f(int i) {
  if (i < 0) {
    // Exception is created but is not thrown.
    std::runtime_error("Unexpected argument");
  }
}
```

### bugprone-too-small-loop-variable

Detects those `for` loops that have a loop variable with a “too small” type which means this type can’t represent all values which are part of the iteration range.

```
int main() {
  long size = 294967296l;
  for (short i = 0; i < size; ++i) {}
}
```

This `for` loop is an infinite loop because the `short` type can’t represent all values in the `[0..size]` interval.

In a real use case size means a container’s size which depends on the user input.

```
int doSomething(const std::vector& items) {
  for (short i = 0; i < items.size(); ++i) {}
}
```

This algorithm works for a small amount of objects, but will lead to freeze for a larger user input.

#### Options

##### `MagnitudeBitsUpperLimit`

Upper limit for the magnitude bits of the loop variable. If it’s set the check filters out those catches in which the loop variable’s type has more magnitude bits as the specified upper limit.

The default value is 16.

For example, if the user sets this option to 31 (bits), then a 32-bit `unsigned int` is ignored by the check, however a 32-bit `int` is not (A 32-bit `signed int` has 31 magnitude bits).

```
int main() {
  long size = 294967296l;
  for (unsigned i = 0; i < size; ++i) {} // no warning with MagnitudeBitsUpperLimit = 31 on a system where unsigned is 32-bit
  for (int i = 0; i < size; ++i) {} // warning with MagnitudeBitsUpperLimit = 31 on a system where int is 32-bit
}
```

### bugprone-unchecked-optional-access

*Note*: This check uses a flow-sensitive static analysis to produce its results. Therefore, it may be more resource intensive (RAM, CPU) than the average clang-tidy check.

This check identifies unsafe accesses to values contained in `std::optional<T>`, `absl::optional<T>`, `base::Optional<T>`, or `folly::Optional<T>` objects. Below we will refer to all these types collectively as `optional<T>`.

An access to the value of an `optional<T>` occurs when one of its `value`, `operator*`, or `operator->` member functions is invoked. To align with common misconceptions, the check considers these member functions as equivalent, even though there are subtle differences related to exceptions versus undefined behavior. See *Additional notes*, below, for more information on this topic.

An access to the value of an `optional<T>` is considered safe if and only if code in the local scope (for example, a function body) ensures that the `optional<T>` has a value in all possible execution paths that can reach the access. That should happen either through an explicit check, using the `optional<T>::has_value` member function, or by constructing the `optional<T>` in a way that shows that it unambiguously holds a value (e.g using `std::make_optional` which always returns a populated `std::optional<T>`).

Below we list some examples, starting with unsafe optional access patterns, followed by safe access patterns.

#### Unsafe access patterns

##### Access the value without checking if it exists

The check flags accesses to the value that are not locally guarded by existence check:

```
void f(std::optional<int> opt) {
  use(*opt); // unsafe: it is unclear whether `opt` has a value.
}
```

##### Access the value in the wrong branch

The check is aware of the state of an optional object in different branches of the code. For example:

```
void f(std::optional<int> opt) {
  if (opt.has_value()) {
  } else {
    use(opt.value()); // unsafe: it is clear that `opt` does *not* have a value.
  }
}
```

##### Assume a function result to be stable

The check is aware that function results might not be stable. That is, consecutive calls to the same function might return different values. For example:

```
void f(Foo foo) {
  if (foo.opt().has_value()) {
    use(*foo.opt()); // unsafe: it is unclear whether `foo.opt()` has a value.
  }
}
```

##### Rely on invariants of uncommon APIs

The check is unaware of invariants of uncommon APIs. For example:

```
void f(Foo foo) {
  if (foo.HasProperty("bar")) {
    use(*foo.GetProperty("bar")); // unsafe: it is unclear whether `foo.GetProperty("bar")` has a value.
  }
}
```

##### Check if a value exists, then pass the optional to another function

The check relies on local reasoning. The check and value access must both happen in the same function. An access is considered unsafe even if the caller of the function performing the access ensures that the optional has a value. For example:

```
void g(std::optional<int> opt) {
  use(*opt); // unsafe: it is unclear whether `opt` has a value.
}

void f(std::optional<int> opt) {
  if (opt.has_value()) {
    g(opt);
  }
}
```

#### Safe access patterns

##### Check if a value exists, then access the value

The check recognizes all straightforward ways for checking if a value exists and accessing the value contained in an optional object. For example:

```
void f(std::optional<int> opt) {
  if (opt.has_value()) {
    use(*opt);
  }
}
```

##### Check if a value exists, then access the value from a copy

The criteria that the check uses is semantic, not syntactic. It recognizes when a copy of the optional object being accessed is known to have a value. For example:

```
void f(std::optional<int> opt1) {
  if (opt1.has_value()) {
    std::optional<int> opt2 = opt1;
    use(*opt2);
  }
}
```

##### Ensure that a value exists using common macros

The check is aware of common macros like `CHECK` and `DCHECK`. Those can be used to ensure that an optional object has a value. For example:

```
void f(std::optional<int> opt) {
  DCHECK(opt.has_value());
  use(*opt);
}
```

##### Ensure that a value exists, then access the value in a correlated branch

The check is aware of correlated branches in the code and can figure out when an optional object is ensured to have a value on all execution paths that lead to an access. For example:

```
void f(std::optional<int> opt) {
  bool safe = false;
  if (opt.has_value() && SomeOtherCondition()) {
    safe = true;
  }
  // ... more code...
  if (safe) {
    use(*opt);
  }
}
```

#### Stabilize function results

Since function results are not assumed to be stable across calls, it is best to store the result of the function call in a local variable and use that variable to access the value. For example:

```
void f(Foo foo) {
  if (const auto& foo_opt = foo.opt(); foo_opt.has_value()) {
    use(*foo_opt);
  }
}
```

#### Do not rely on uncommon-API invariants

When uncommon APIs guarantee that an optional has contents, do not rely on it – instead, check explicitly that the optional object has a value. For example:

```
void f(Foo foo) {
  if (const auto& property = foo.GetProperty("bar")) {
    use(*property);
  }
}
```

instead of the HasProperty, GetProperty pairing we saw above.

#### Do not rely on caller-performed checks

If you know that all of a function’s callers have checked that an optional argument has a value, either change the function to take the value directly or check the optional again in the local scope of the callee. For example:

```
void g(int val) {
  use(val);
}

void f(std::optional<int> opt) {
  if (opt.has_value()) {
    g(*opt);
  }
}
```

and

```
struct S {
  std::optional<int> opt;
  int x;
};

void g(const S &s) {
  if (s.opt.has_value() && s.x > 10) {
    use(*s.opt);
}

void f(S s) {
  if (s.opt.has_value()) {
    g(s);
  }
}
```

#### Additional notes

##### Aliases created via `using` declarations

The check is aware of aliases of optional types that are created via `using` declarations. For example:

```
using OptionalInt = std::optional<int>;

void f(OptionalInt opt) {
  use(opt.value()); // unsafe: it is unclear whether `opt` has a value.
}
```

##### Lambdas

The check does not currently report unsafe optional accesses in lambdas. A future version will expand the scope to lambdas, following the rules outlined above. It is best to follow the same principles when using optionals in lambdas.

##### Access with `operator*()` vs. `value()`

Given that `value()` has well-defined behavior (either throwing an exception or terminating the program), why treat it the same as `operator*()` which causes undefined behavior (UB)? That is, why is it considered unsafe to access an optional with `value()`, if it’s not provably populated with a value? For that matter, why is `CHECK()` followed by `operator*()` any better than `value()`, given that they are semantically equivalent (on configurations that disable exceptions)?

The answer is that we assume most users do not realize the difference between `value()` and `operator*()`. Shifting to `operator*()` and some form of explicit value-presence check or explicit program termination has two advantages:

- Readability. The check, and any potential side effects like program shutdown, are very clear in the code. Separating access from checks can actually make the checks more obvious.
- Performance. A single check can cover many or even all accesses within scope. This gives the user the best of both worlds – the safety of a dynamic check, but without incurring redundant costs.

### bugprone-undefined-memory-manipulation

Finds calls of memory manipulation functions `memset()`, `memcpy()` and `memmove()` on non-TriviallyCopyable objects resulting in undefined behavior.

Using memory manipulation functions on non-TriviallyCopyable objects can lead to a range of subtle and challenging issues in C++ code. The most immediate concern is the potential for undefined behavior, where the state of the object may become corrupted or invalid. This can manifest as crashes, data corruption, or unexpected behavior at runtime, making it challenging to identify and diagnose the root cause. Additionally, misuse of memory manipulation functions can bypass essential object-specific operations, such as constructors and destructors, leading to resource leaks or improper initialization.

For example, when using `memcpy` to copy `std::string`, pointer data is being copied, and it can result in a double free issue.

```
#include <cstring>
#include <string>

int main() {
    std::string source = "Hello";
    std::string destination;

    std::memcpy(&destination, &source, sizeof(std::string));

    // Undefined behavior may occur here, during std::string destructor call.
    return 0;
}
```

### bugprone-undelegated-constructor

Finds creation of temporary objects in constructors that look like a function call to another constructor of the same class.

The user most likely meant to use a delegating constructor or base class initializer.

### bugprone-unhandled-exception-at-new

Finds calls to `new` with missing exception handler for `std::bad_alloc`.

Calls to `new` may throw exceptions of type `std::bad_alloc` that should be handled. Alternatively, the nonthrowing form of `new` can be used. The check verifies that the exception is handled in the function that calls `new`.

If a nonthrowing version is used or the exception is allowed to propagate out of the function no warning is generated.

The exception handler is checked if it catches a `std::bad_alloc` or `std::exception` exception type, or all exceptions (catch-all). The check assumes that any user-defined `operator new` is either `noexcept` or may throw an exception of type `std::bad_alloc` (or one derived from it). Other exception class types are not taken into account.

```
int *f() noexcept {
  int *p = new int[1000]; // warning: missing exception handler for allocation failure at 'new'
  // ...
  return p;
}
int *f1() { // not 'noexcept'
  int *p = new int[1000]; // no warning: exception can be handled outside
                          // of this function
  // ...
  return p;
}

int *f2() noexcept {
  try {
    int *p = new int[1000]; // no warning: exception is handled
    // ...
    return p;
  } catch (std::bad_alloc &) {
    // ...
  }
  // ...
}

int *f3() noexcept {
  int *p = new (std::nothrow) int[1000]; // no warning: "nothrow" is used
  // ...
  return p;
}
```

### bugprone-unhandled-self-assignment

cert-oop54-cpp redirects here as an alias for this check. For the CERT alias, the WarnOnlyIfThisHasSuspiciousField option is set to false.

Finds user-defined copy assignment operators which do not protect the code against self-assignment either by checking self-assignment explicitly or using the copy-and-swap or the copy-and-move method.

By default, this check searches only those classes which have any pointer or C array field to avoid false positives. In case of a pointer or a C array, it’s likely that self-copy assignment breaks the object if the copy assignment operator was not written with care.

See also: [OOP54-CPP. Gracefully handle self-copy assignment](https://wiki.sei.cmu.edu/confluence/display/cplusplus/OOP54-CPP.+Gracefully+handle+self-copy+assignment)

A copy assignment operator must prevent that self-copy assignment ruins the object state. A typical use case is when the class has a pointer field and the copy assignment operator first releases the pointed object and then tries to assign it:

```
class T {
int* p;

public:
  T(const T &rhs) : p(rhs.p ? new int(*rhs.p) : nullptr) {}
  ~T() { delete p; }

  // ...

  T& operator=(const T &rhs) {
    delete p;
    p = new int(*rhs.p);
    return *this;
  }
};
```

There are two common C++ patterns to avoid this problem. The first is the self-assignment check:

```
class T {
int* p;

public:
  T(const T &rhs) : p(rhs.p ? new int(*rhs.p) : nullptr) {}
  ~T() { delete p; }

  // ...

  T& operator=(const T &rhs) {
    if(this == &rhs)
      return *this;

    delete p;
    p = new int(*rhs.p);
    return *this;
  }
};
```

The second one is the copy-and-swap method when we create a temporary copy (using the copy constructor) and then swap this temporary object with `this`:

```
class T {
int* p;

public:
  T(const T &rhs) : p(rhs.p ? new int(*rhs.p) : nullptr) {}
  ~T() { delete p; }

  // ...

  void swap(T &rhs) {
    using std::swap;
    swap(p, rhs.p);
  }

  T& operator=(const T &rhs) {
    T(rhs).swap(*this);
    return *this;
  }
};
```

There is a third pattern which is less common. Let’s call it the copy-and-move method when we create a temporary copy (using the copy constructor) and then move this temporary object into `this` (needs a move assignment operator):

```
class T {
int* p;

public:
  T(const T &rhs) : p(rhs.p ? new int(*rhs.p) : nullptr) {}
  ~T() { delete p; }

  // ...

  T& operator=(const T &rhs) {
    T t = rhs;
    *this = std::move(t);
    return *this;
  }

  T& operator=(T &&rhs) {
    p = rhs.p;
    rhs.p = nullptr;
    return *this;
  }
};
```

#### Options

##### `WarnOnlyIfThisHasSuspiciousField`

When true, the check will warn only if the container class of the copy assignment operator has any suspicious fields (pointer or C array). This option is set to true by default.

### bugprone-unique-ptr-array-mismatch

Finds initializations of C++ unique pointers to non-array type that are initialized with an array.

If a pointer `std::unique_ptr<T>` is initialized with a new-expression `new T[]` the memory is not deallocated correctly. A plain `delete` is used in this case to deallocate the target memory. Instead a `delete[]` call is needed. A `std::unique_ptr<T[]>` uses the correct delete operator. The check does not emit warning if an `unique_ptr` with user-specified deleter type is used.

The check offers replacement of `unique_ptr<T>` to `unique_ptr<T[]>` if it is used at a single variable declaration (one variable in one statement).

Example:

```
std::unique_ptr<Foo> x(new Foo[10]); // -> std::unique_ptr<Foo[]> x(new Foo[10]);
//                     ^ warning: unique pointer to non-array is initialized with array
std::unique_ptr<Foo> x1(new Foo), x2(new Foo[10]); // no replacement
//                                   ^ warning: unique pointer to non-array is initialized with array

D d;
std::unique_ptr<Foo, D> x3(new Foo[10], d); // no warning (custom deleter used)

struct S {
  std::unique_ptr<Foo> x(new Foo[10]); // no replacement in this case
  //                     ^ warning: unique pointer to non-array is initialized with array
};
```

This check partially covers the CERT C++ Coding Standard rule [MEM51-CPP. Properly deallocate dynamically allocated resources](https://wiki.sei.cmu.edu/confluence/display/cplusplus/MEM51-CPP.+Properly+deallocate+dynamically+allocated+resources) However, only the `std::unique_ptr` case is detected by this check.

### bugprone-unsafe-functions

Checks for functions that have safer, more secure replacements available, or are considered deprecated due to design flaws. The check heavily relies on the functions from the **Annex K.** “Bounds-checking interfaces” of C11.

The check implements the following rules from the CERT C Coding Standard:

- Recommendation [MSC24-C. Do not use deprecated or obsolescent functions](https://wiki.sei.cmu.edu/confluence/display/c/MSC24-C.+Do+not+use+deprecated+or+obsolescent+functions).
- Rule [MSC33-C. Do not pass invalid data to the asctime() function](https://wiki.sei.cmu.edu/confluence/display/c/MSC33-C.+Do+not+pass+invalid+data+to+the+asctime()+function).

cert-msc24-c and cert-msc33-c redirect here as aliases of this check.

#### Unsafe functions

If *Annex K.* is available, a replacement from *Annex K.* is suggested for the following functions:

`asctime`, `asctime_r`, `bsearch`, `ctime`, `fopen`, `fprintf`, `freopen`, `fscanf`, `fwprintf`, `fwscanf`, `getenv`, `gets`, `gmtime`, `localtime`, `mbsrtowcs`, `mbstowcs`, `memcpy`, `memmove`, `memset`, `printf`, `qsort`, `scanf`, `snprintf`, `sprintf`, `sscanf`, `strcat`, `strcpy`, `strerror`, `strlen`, `strncat`, `strncpy`, `strtok`, `swprintf`, `swscanf`, `vfprintf`, `vfscanf`, `vfwprintf`, `vfwscanf`, `vprintf`, `vscanf`, `vsnprintf`, `vsprintf`, `vsscanf`, `vswprintf`, `vswscanf`, `vwprintf`, `vwscanf`, `wcrtomb`, `wcscat`, `wcscpy`, `wcslen`, `wcsncat`, `wcsncpy`, `wcsrtombs`, `wcstok`, `wcstombs`, `wctomb`, `wmemcpy`, `wmemmove`, `wprintf`, `wscanf`.

If *Annex K.* is not available, replacements are suggested only for the following functions from the previous list:

> - `asctime`, `asctime_r`, suggested replacement: `strftime`
> - `gets`, suggested replacement: `fgets`

The following functions are always checked, regardless of *Annex K* availability:

> - `rewind`, suggested replacement: `fseek`
> - `setbuf`, suggested replacement: `setvbuf`

If [ReportMoreUnsafeFunctions](https://clang.llvm.org/extra/clang-tidy/checks/bugprone/unsafe-functions.html#cmdoption-arg-ReportMoreUnsafeFunctions) is enabled, the following functions are also checked:

> - `bcmp`, suggested replacement: `memcmp`
> - `bcopy`, suggested replacement: `memcpy_s` if *Annex K* is available, or `memcpy`
> - `bzero`, suggested replacement: `memset_s` if *Annex K* is available, or `memset`
> - `getpw`, suggested replacement: `getpwuid`
> - `vfork`, suggested replacement: `posix_spawn`

Although mentioned in the associated CERT rules, the following functions are **ignored** by the check:

`atof`, `atoi`, `atol`, `atoll`, `tmpfile`.

The availability of *Annex K* is determined based on the following macros:

> - `__STDC_LIB_EXT1__`: feature macro, which indicates the presence of *Annex K. “Bounds-checking interfaces”* in the library implementation
> - `__STDC_WANT_LIB_EXT1__`: user-defined macro, which indicates that the user requests the functions from *Annex K.* to be defined.

Both macros have to be defined to suggest replacement functions from *Annex K.* `__STDC_LIB_EXT1__` is defined by the library implementation, and `__STDC_WANT_LIB_EXT1__` must be defined to `1` by the user **before** including any system headers.

#### Options

##### `ReportMoreUnsafeFunctions`

When true, additional functions from widely used APIs (such as POSIX) are added to the list of reported functions. See the main documentation of the check for the complete list as to what this option enables. Default is true.

#### Examples

```
#ifndef __STDC_LIB_EXT1__
#error "Annex K is not supported by the current standard library implementation."
#endif

#define __STDC_WANT_LIB_EXT1__ 1

#include <string.h> // Defines functions from Annex K.
#include <stdio.h>

enum { BUFSIZE = 32 };

void Unsafe(const char *Msg) {
  static const char Prefix[] = "Error: ";
  static const char Suffix[] = "\n";
  char Buf[BUFSIZE] = {0};

  strcpy(Buf, Prefix); // warning: function 'strcpy' is not bounds-checking; 'strcpy_s' should be used instead.
  strcat(Buf, Msg);    // warning: function 'strcat' is not bounds-checking; 'strcat_s' should be used instead.
  strcat(Buf, Suffix); // warning: function 'strcat' is not bounds-checking; 'strcat_s' should be used instead.
  if (fputs(buf, stderr) < 0) {
    // error handling
    return;
  }
}

void UsingSafeFunctions(const char *Msg) {
  static const char Prefix[] = "Error: ";
  static const char Suffix[] = "\n";
  char Buf[BUFSIZE] = {0};

  if (strcpy_s(Buf, BUFSIZE, Prefix) != 0) {
    // error handling
    return;
  }

  if (strcat_s(Buf, BUFSIZE, Msg) != 0) {
    // error handling
    return;
  }

  if (strcat_s(Buf, BUFSIZE, Suffix) != 0) {
    // error handling
    return;
  }

  if (fputs(Buf, stderr) < 0) {
    // error handling
    return;
  }
}
```

### bugprone-unused-raii

Finds temporaries that look like RAII objects.

The canonical example for this is a scoped lock.

```
{
  scoped_lock(&global_mutex);
  critical_section();
}
```

The destructor of the scoped_lock is called before the `critical_section` is entered, leaving it unprotected.

We apply a number of heuristics to reduce the false positive count of this check:

- Ignore code expanded from macros. Testing frameworks make heavy use of this.
- Ignore types with trivial destructors. They are very unlikely to be RAII objects and there’s no difference when they are deleted.
- Ignore objects at the end of a compound statement (doesn’t change behavior).
- Ignore objects returned from a call.

### bugprone-unused-return-value

Warns on unused function return values. The checked functions can be configured.

#### Options

##### `CheckedFunctions`

Semicolon-separated list of functions to check. The function is checked if the name and scope matches, with any arguments. By default the following functions are checked: `std::async, std::launder, std::remove, std::remove_if, std::unique, std::unique_ptr::release, std::basic_string::empty, std::vector::empty, std::back_inserter, std::distance, std::find, std::find_if, std::inserter, std::lower_bound, std::make_pair, std::map::count, std::map::find, std::map::lower_bound, std::multimap::equal_range, std::multimap::upper_bound, std::set::count, std::set::find, std::setfill, std::setprecision, std::setw, std::upper_bound, std::vector::at, bsearch, ferror, feof, isalnum, isalpha, isblank, iscntrl, isdigit, isgraph, islower, isprint, ispunct, isspace, isupper, iswalnum, iswprint, iswspace, isxdigit, memchr, memcmp, strcmp, strcoll, strncmp, strpbrk, strrchr, strspn, strstr, wcscmp, access, bind, connect, difftime, dlsym, fnmatch, getaddrinfo, getopt, htonl, htons, iconv_open, inet_addr, isascii, isatty, mmap, newlocale, openat, pathconf, pthread_equal, pthread_getspecific, pthread_mutex_trylock, readdir, readlink, recvmsg, regexec, scandir, semget, setjmp, shm_open, shmget, sigismember, strcasecmp, strsignal, ttyname``std::async()`. Not using the return value makes the call synchronous.`std::launder()`. Not using the return value usually means that the function interface was misunderstood by the programmer. Only the returned pointer is “laundered”, not the argument.`std::remove()`, `std::remove_if()` and `std::unique()`. The returned iterator indicates the boundary between elements to keep and elements to be removed. Not using the return value means that the information about which elements to remove is lost.`std::unique_ptr::release()`. Not using the return value can lead to resource leaks if the same pointer isn’t stored anywhere else. Often, ignoring the `release()` return value indicates that the programmer confused the function with `reset()`.`std::basic_string::empty()` and `std::vector::empty()`. Not using the return value often indicates that the programmer confused the function with `clear()`.

##### `CheckedReturnTypes`

Semicolon-separated list of function return types to check.

By default the following function return types are checked: ::std::error_code, ::std::error_condition, ::std::errc, ::std::expected, ::boost::system::error_code.

##### `AllowCastToVoid`

Controls whether casting return values to `void` is permitted.

Default: false.

[cert-err33-c](https://clang.llvm.org/extra/clang-tidy/checks/cert/err33-c.html) is an alias of this check that checks a fixed and large set of standard library functions.

### bugprone-use-after-move

Warns if an object is used after it has been moved, for example:

```
std::string str = "Hello, world!\n";
std::vector<std::string> messages;
messages.emplace_back(std::move(str));
std::cout << str;
```

The last line will trigger a warning that `str` is used after it has been moved.

The check does not trigger a warning if the object is reinitialized after the move and before the use. For example, no warning will be output for this code:

```
messages.emplace_back(std::move(str));
str = "Greetings, stranger!\n";
std::cout << str;
```

Subsections below explain more precisely what exactly the check considers to be a move, use, and reinitialization.

The check takes control flow into account. A warning is only emitted if the use can be reached from the move. This means that the following code does not produce a warning:

```
if (condition) {
  messages.emplace_back(std::move(str));
} else {
  std::cout << str;
}
```

On the other hand, the following code does produce a warning:

```
for (int i = 0; i < 10; ++i) {
  std::cout << str;
  messages.emplace_back(std::move(str));
}
```

(The use-after-move happens on the second iteration of the loop.)

In some cases, the check may not be able to detect that two branches are mutually exclusive. For example (assuming that `i` is an int):

```
if (i == 1) {
  messages.emplace_back(std::move(str));
}
if (i == 2) {
  std::cout << str;
}
```

In this case, the check will erroneously produce a warning, even though it is not possible for both the move and the use to be executed. More formally, the analysis is [flow-sensitive but not path-sensitive](https://en.wikipedia.org/wiki/Data-flow_analysis#Sensitivities).

#### Silencing erroneous warnings

An erroneous warning can be silenced by reinitializing the object after the move:

```
if (i == 1) {
  messages.emplace_back(std::move(str));
  str = "";
}
if (i == 2) {
  std::cout << str;
}
```

If you want to avoid the overhead of actually reinitializing the object, you can create a dummy function that causes the check to assume the object was reinitialized:

```
template <class T>
void IS_INITIALIZED(T&) {}
```

You can use this as follows:

```
if (i == 1) {
  messages.emplace_back(std::move(str));
}
if (i == 2) {
  IS_INITIALIZED(str);
  std::cout << str;
}
```

The check will not output a warning in this case because passing the object to a function as a non-const pointer or reference counts as a reinitialization (see section [Reinitialization](https://clang.llvm.org/extra/clang-tidy/checks/bugprone/use-after-move.html#reinitialization) below).

#### Unsequenced moves, uses, and reinitializations

In many cases, C++ does not make any guarantees about the order in which sub-expressions of a statement are evaluated. This means that in code like the following, it is not guaranteed whether the use will happen before or after the move:

```
void f(int i, std::vector<int> v);
std::vector<int> v = { 1, 2, 3 };
f(v[1], std::move(v));
```

In this kind of situation, the check will note that the use and move are unsequenced.

The check will also take sequencing rules into account when reinitializations occur in the same statement as moves or uses. A reinitialization is only considered to reinitialize a variable if it is guaranteed to be evaluated after the move and before the use.

#### Move

The check currently only considers calls of `std::move` on local variables or function parameters. It does not check moves of member variables or global variables.

Any call of `std::move` on a variable is considered to cause a move of that variable, even if the result of `std::move` is not passed to an rvalue reference parameter.

This means that the check will flag a use-after-move even on a type that does not define a move constructor or move assignment operator. This is intentional. Developers may use `std::move` on such a type in the expectation that the type will add move semantics in the future. If such a `std::move` has the potential to cause a use-after-move, we want to warn about it even if the type does not implement move semantics yet.

Furthermore, if the result of `std::move` *is* passed to an rvalue reference parameter, this will always be considered to cause a move, even if the function that consumes this parameter does not move from it, or if it does so only conditionally. For example, in the following situation, the check will assume that a move always takes place:

```
std::vector<std::string> messages;
void f(std::string &&str) {
  // Only remember the message if it isn't empty.
  if (!str.empty()) {
    messages.emplace_back(std::move(str));
  }
}
std::string str = "";
f(std::move(str));
```

The check will assume that the last line causes a move, even though, in this particular case, it does not. Again, this is intentional.

There is one special case: A call to `std::move` inside a `try_emplace` call is conservatively assumed not to move. This is to avoid spurious warnings, as the check has no way to reason about the `bool` returned by `try_emplace`.

When analyzing the order in which moves, uses and reinitializations happen (see section [Unsequenced moves, uses, and reinitializations](https://clang.llvm.org/extra/clang-tidy/checks/bugprone/use-after-move.html#unsequenced-moves-uses-and-reinitializations)), the move is assumed to occur in whichever function the result of the `std::move` is passed to.

#### Use

Any occurrence of the moved variable that is not a reinitialization (see below) is considered to be a use.

An exception to this are objects of type `std::unique_ptr`, `std::shared_ptr` and `std::weak_ptr`, which have defined move behavior (objects of these classes are guaranteed to be empty after they have been moved from). Therefore, an object of these classes will only be considered to be used if it is dereferenced, i.e. if `operator*`, `operator->` or `operator[]` (in the case of `std::unique_ptr<T []>`) is called on it.

If multiple uses occur after a move, only the first of these is flagged.

#### Reinitialization

The check considers a variable to be reinitialized in the following cases:

- The variable occurs on the left-hand side of an assignment.
- The variable is passed to a function as a non-const pointer or non-const lvalue reference. (It is assumed that the variable may be an out-parameter for the function.)
- `clear()` or `assign()` is called on the variable and the variable is of one of the standard container types `basic_string`, `vector`, `deque`, `forward_list`, `list`, `set`, `map`, `multiset`, `multimap`, `unordered_set`, `unordered_map`, `unordered_multiset`, `unordered_multimap`.
- `reset()` is called on the variable and the variable is of type `std::unique_ptr`, `std::shared_ptr` or `std::weak_ptr`.
- A member function marked with the `[[clang::reinitializes]]` attribute is called on the variable.

If the variable in question is a struct and an individual member variable of that struct is written to, the check does not consider this to be a reinitialization – even if, eventually, all member variables of the struct are written to. For example:

```
struct S {
  std::string str;
  int i;
};
S s = { "Hello, world!\n", 42 };
S s_other = std::move(s);
s.str = "Lorem ipsum";
s.i = 99;
```

The check will not consider `s` to be reinitialized after the last line; instead, the line that assigns to `s.str` will be flagged as a use-after-move. This is intentional as this pattern of reinitializing a struct is error-prone. For example, if an additional member variable is added to `S`, it is easy to forget to add the reinitialization for this additional member. Instead, it is safer to assign to the entire struct in one go, and this will also avoid the use-after-move warning.

### bugprone-virtual-near-miss

Warn if a function is a near miss (i.e. the name is very similar and the function signature is the same) to a virtual function from a base class.

Example:

```
struct Base {
  virtual void func();
};

struct Derived : Base {
  virtual void funk();
  // warning: 'Derived::funk' has a similar name and the same signature as virtual method 'Base::func'; did you mean to override it?
};
```

## cert-*

Checks related to CERT Secure Coding Guidelines.

## clang-analyzer-*

Clang Static Analyzer checks.

## concurrency-*

Checks related to concurrent programming (including threads, fibers, coroutines, etc.).

### concurrency-mt-unsafe

Checks for some thread-unsafe functions against a black list of known-to-be-unsafe functions. Usually they access static variables without synchronization (e.g. gmtime(3)) or utilize signals in a racy way. The set of functions to check is specified with the **FunctionSet** option.

>根据已知不安全函数的黑名单检查某些线程不安全函数。
>
>通常它们访问静态变量时没有同步（例如`gmtime`(3)），或者以一种活泼的方式利用信号。
>
>要检查的函数集由**FunctionSet**选项指定。

Note that using some thread-unsafe functions may be still valid in concurrent programming if only a single thread is used (e.g. setenv(3)), however, some functions may track a state in global variables which would be clobbered by subsequent (non-parallel, but concurrent) calls to a related function.

>请注意，如果只使用单个线程（例如`setenv`(3)），在并发编程中使用一些线程不安全的函数可能仍然可以，然而，一些函数可能会跟踪全局变量中的状态，这些状态将被后续（非并行，但并发）调用相关函数所破坏。

E.g. the following code suffers from unprotected accesses to a global state:

```c++
// getnetent(3) maintains global state with DB connection, etc.
// If a concurrent green thread calls getnetent(3), the global state is corrupted.
netent = getnetent();
yield();
netent = getnetent();
```

Examples:

```c++
tm = gmtime(timep); // uses a global buffer
sleep(1); // implementation may use SIGALRM
```

**FunctionSet**

Specifies which functions in libc should be considered thread-safe, possible values are `posix`, `glibc`, or `any`.

- `posix` means POSIX defined thread-unsafe functions. POSIX.1-2001 in “2.9.1 Thread-Safety” defines that all functions specified in the standard are thread-safe except a predefined list of thread-unsafe functions.
- `glibc` defines some of them as thread-safe (e.g. dirname(3)), but adds non-POSIX thread-unsafe ones (e.g. getopt_long(3)).
  - Glibc’s list is compiled from GNU web documentation with a search for MT-Safe tag<sup>[3]</sup>.
- If you want to identify thread-unsafe API for at least one libc or unsure which libc will be used, use `any` (default).

### concurrency-thread-canceltype-asynchronous

Finds `pthread_setcanceltype` function calls where a thread’s cancellation type is set to asynchronous. Asynchronous cancellation type (`PTHREAD_CANCEL_ASYNCHRONOUS`) is generally unsafe, use type `PTHREAD_CANCEL_DEFERRED` instead which is the default. Even with deferred cancellation, a cancellation point in an asynchronous signal handler may still be acted upon and the effect is as if it was an asynchronous cancellation.

>查找线程的取消类型设置为异步的`pthread_setcanceltype`函数调用。
>
>异步取消类型（`PTHREAD_CANCEL_ASYNCHRONOUS`）通常是不安全的，请使用默认类型`PTHREAD_CANCEL_DEFERRED`代替。
>
>即使使用延迟取消，异步信号处理程序中的取消点仍然可以被操作，并且效果就好像它是异步取消一样。

```c++
pthread_setcanceltype(PTHREAD_CANCEL_ASYNCHRONOUS, &oldtype);
```

This check corresponds to the CERT C Coding Standard rule<sup>[4]</sup>.

## cppcoreguidelines-*

Checks related to C++ Core Guidelines.

### cppcoreguidelines-avoid-capturing-lambda-coroutines

Flags C++20 coroutine lambdas with non-empty capture lists that may cause use-after-free errors and suggests avoiding captures or ensuring the lambda closure object has a guaranteed lifetime.

>标识出C++ 20协程lambda具有可能导致UAF（use-after-free）错误的非空捕获列表，并建议避免捕获或确保lambda闭包对象具有保证的生命周期。

This check implements [CP.51](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rcoro-capture) from the C++ Core Guidelines.

Using coroutine lambdas with non-empty capture lists can be risky, as capturing variables can lead to accessing freed memory after the first suspension point. This issue can occur even with refcounted smart pointers and copyable types. When a lambda expression creates a coroutine, it results in a closure object with storage, which is often on the stack and will eventually go out of scope. When the closure object goes out of scope, its captures also go out of scope. While normal lambdas finish executing before this happens, coroutine lambdas may resume from suspension after the closure object has been destructed, resulting in use-after-free memory access for all captures.

>使用带有非空捕获列表的协程lambdas可能有风险，因为捕获变量可能导致访问第一个挂起点之后释放的内存。即使使用引用计数的智能指针和可复制类型，也可能出现此问题。当一个lambda表达式创建一个协程时，它会产生一个带有存储的闭包对象，该闭包对象通常位于栈上，最终将超出作用域。当闭包对象超出作用域时，它的捕获也超出作用域。虽然正常的lambda会在此发生之前完成执行，但协程lambda可能在闭包对象被销毁后从挂起恢复，从而导致所有捕获都在释放后的内存访问。

Consider the following example:

```c++
int value = get_value();
std::shared_ptr<Foo> sharedFoo = get_foo();
{
    const auto lambda = [value, sharedFoo]() -> std::future<void>
    {
        co_await something();
        // "sharedFoo" and "value" have already been destroyed
        // the "shared" pointer didn't accomplish anything
    };
    lambda();
} // the lambda closure object has now gone out of scope
```

In this example, the lambda object is defined with two captures: value and `sharedFoo`. When `lambda()` is called, the lambda object is created on the stack, and the captures are copied into the closure object. When the coroutine is suspended, the lambda object goes out of scope, and the closure object is destroyed. When the coroutine is resumed, the captured variables may have been destroyed, resulting in use-after-free bugs.

>在本例中，lambda对象通过两个捕获定义：`value`和`sharedFoo`。
>
>当调用`lambda()`时，在堆栈上创建lambda对象，并将捕获复制到闭包对象中。
>
>当协程挂起时，lambda对象将超出作用域，闭包对象将被销毁。
>
>当协程被恢复时，捕获的变量可能已经被销毁，从而导致use-after-free错误。

In conclusion, the use of coroutine lambdas with non-empty capture lists can lead to use-after-free errors when resuming the coroutine after the closure object has been destroyed. This check helps prevent such errors by flagging C++20 coroutine lambdas with non-empty capture lists and suggesting avoiding captures or ensuring the lambda closure object has a guaranteed lifetime.

>总之，使用带有非空捕获列表的协程lambda会导致在闭包对象被销毁后恢复协程时出现use-after-free错误。此检查通过标记出带有非空捕获列表的C++ 20协程lambda，并建议避免捕获或确保lambda闭包对象具有保证的生命周期来帮助防止此类错误。

Following these guidelines can help ensure the safe and reliable use of coroutine lambdas in C++ code.

>遵循这些指导方针可以帮助确保在C++代码中安全可靠地使用协程lambdas。

### cppcoreguidelines-avoid-const-or-ref-data-members

This check warns when structs or classes that are copyable or movable, and have const-qualified or reference (lvalue or rvalue) data members. Having such members is rarely useful, and makes the class only copy-constructible but not copy-assignable.

>当结构体或类是可复制或可移动的，并且具有`const`限定的或引用（左值或右值）数据成员时，此检查会发出警告。
>
>拥有这样的成员很少有用，并且使类只能复制构造而不能复制赋值。

Examples:

```c++
// Bad, const-qualified member
struct Const {
  const int x;
}

// Good:
class Foo {
 public:
  int get() const { return x; }
 private:
  int x;
};

// Bad, lvalue reference member
struct Ref {
  int& x;
};

// Good:
struct Foo {
  int* x;
  std::unique_ptr<int> x;
  std::shared_ptr<int> x;
  gsl::not_null<int> x;
};

// Bad, rvalue reference member
struct RefRef {
  int&& x;
};
```

This check implements C.12<sup>[5]</sup> from the C++ Core Guidelines.

Further reading: Data members: Never const<sup>[6]</sup>.

### cppcoreguidelines-avoid-do-while

Warns when using `do-while` loops. They are less readable than plain `while` loops, since the termination condition is at the end and the condition is not checked prior to the first iteration. This can lead to subtle bugs.

>在使用`do`-`while`循环时发出警告。
>
>`do`-`while`循环的可读性不如普通的`while`循环，因为终止条件在末尾，并且在第一次迭代之前不检查条件。
>
>这可能会导致一些微妙的bug。

This check implements ES.75<sup>[7]</sup> from the C++ Core Guidelines.

Examples:

```c++
int x;
do {
    std::cin >> x;
    // ...
} while (x < 0);
```

Options - `IgnoreMacros`

Ignore the check when analyzing macros. This is useful for safely defining function-like macros:

```c++
#define FOO_BAR(x)	\
    do {			\
      foo(x);		\
      bar(x);		\
    } while(0)
```

Defaults to false.

### cppcoreguidelines-avoid-goto

The usage of `goto` for control flow is error prone and should be replaced with looping constructs. Only forward jumps in nested loops are accepted.

>对控制流使用`goto`很容易出错，应该用循环结构代替。只接受嵌套循环中的前跳。

> 注：Linux内核中使用了大量的`goto`，但不代表它是坏的代码。这条规则需要根据场景进行选择性的应用，不能一概而论。

This check implements C++ Core Guidelines ES.76<sup>[8]</sup> and High Integrity C++ Coding Standard 6.3.1<sup>[9]</sup>.

For more information on why to avoid programming with `goto` you can read the famous paper A Case against the GO TO Statement<sup>[10]</sup>.

The check diagnoses `goto` for backward jumps in every language mode. These should be replaced with C/C++ looping constructs.

> 该检查诊断在每种语言模式下的向后跳转的`goto`。这些应该用C/C++循环结构代替。

```c++
// Bad, handwritten for loop.
int i = 0;
// Jump label for the loop
loop_start:
do_some_operation();

if (i < 100) {
  ++i;
  goto loop_start;
}

// Better
for(int i = 0; i < 100; ++i)
  do_some_operation();
```

Modern C++ needs `goto` only to jump out of nested loops.

```c++
for(int i = 0; i < 100; ++i) {
  for(int j = 0; j < 100; ++j) {
    if (i * j > 500)
      goto early_exit;
  }
}

early_exit:
some_operation();
```

All other uses of `goto` are diagnosed in C++.

### cppcoreguidelines-avoid-non-const-global-variables

Finds non-const global variables as described in [I.2](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i2-avoid-non-const-global-variables) of C++ Core Guidelines. As [R.6](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-global) of C++ Core Guidelines is a duplicate of rule [I.2](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i2-avoid-non-const-global-variables) it also covers that rule.

```c++
char a;  // Warns!
const char b =  0;

namespace some_namespace
{
    char c;  // Warns!
    const char d = 0;
}

char * c_ptr1 = &some_namespace::c;  // Warns!
char *const c_const_ptr = &some_namespace::c;  // Warns!
char & c_reference = some_namespace::c;  // Warns!

class Foo  // No Warnings inside Foo, only namespace scope is covered
{
public:
    char e = 0;
    const char f = 0;
protected:
    char g = 0;
private:
    char h = 0;
};
```

The variables `a`, `c`, `c_ptr1`, `c_const_ptr` and `c_reference` will all generate warnings since they are either a non-`const` globally accessible variable, a pointer or a reference providing global access to non-const data or both.

### cppcoreguidelines-avoid-reference-coroutine-parameters

Warns when a coroutine accepts reference parameters. After a coroutine suspend point, references could be dangling and no longer valid. Instead, pass parameters as values.

>当协程接受引用参数时发出警告。在协程挂起点之后，引用可能悬空并且不再有效。相反，将参数作为值传递。

Examples:

```c++
std::future<int> someCoroutine(int& val) {
  co_await ...;
  // When the coroutine is resumed, 'val' might no longer be valid.
  if (val) ...
}
```

This check implements C++ Core Guidelines CP.53<sup>[11]</sup>.

### cppcoreguidelines-init-variables

Checks whether there are local variables that are declared without an initial value. These may lead to unexpected behavior if there is a code path that reads the variable before assigning to it.

>检查是否存在未声明初始值的局部变量。如果存在一个在对变量赋值之前读取变量的代码路径，可能导致意外行为。

This rule is part of the Type safety (Type.5)(???) profile and C++ Core Guidelines ES.20<sup>[12]</sup>.

Only integers, booleans, floats, doubles and pointers are checked. The fix option initializes all detected values with the value of zero. An exception is float and double types, which are initialized to `NaN`.

>只检查整数、布尔值、浮点数、双精度和指针。
>
>fix选项将所有检测到的值初始化为0。float和double类型是一个例外，它们初始化为NaN。

As an example a function that looks like this:

```c++
void function() {
  int x;
  char *txt;
  double d;

  // Rest of the function.
}
```

Would be rewritten to look like this:

```c++
#include <math.h>

void function() {
  int x = 0;
  char *txt = nullptr;
  double d = NAN;

  // Rest of the function.
}
```

It warns for the uninitialized enum case, but without a FixIt:

```c++
enum A {A1, A2, A3};
enum A_c : char { A_c1, A_c2, A_c3 };
enum class B { B1, B2, B3 };
enum class B_i : int { B_i1, B_i2, B_i3 };
void function() {
  A a;     // Warning: variable 'a' is not initialized
  A_c a_c; // Warning: variable 'a_c' is not initialized
  B b;     // Warning: variable 'b' is not initialized
  B_i b_i; // Warning: variable 'b_i' is not initialized
}
```

#### Options

- `IncludeStyle`
  - A string specifying which include-style is used, `llvm` or `google`.
  - Default is `llvm`.
- `MathHeader`
  - A string specifying the header to include to get the definition of `NaN`.
  - Default is `<math.h>`.

### cppcoreguidelines-interfaces-global-init

This check flags initializers of globals that access extern objects, and therefore can lead to order-of-initialization problems.

>此检查标记访问外部对象的全局变量的初始化器，因此可能导致初始化顺序问题。

This check implements C++ Core Guidelines I.22<sup>[12]</sup>.

Note that currently this does not flag calls to non-`constexpr` functions, and therefore globals could still be accessed from functions themselves.

>请注意，目前这并不标记对非`constexpr`函数的调用，因此全局变量仍然可以从函数本身访问。

> 注：demo???

### cppcoreguidelines-macro-usage

Finds macro usage that is considered problematic because better language constructs exist for the task.

>查找由于存在更好的任务语言结构而被认为有问题的宏用法。

The relevant sections in the C++ Core Guidelines are [ES.31](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es31-dont-use-macros-for-constants-or-functions), and [ES.32](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es32-use-all_caps-for-all-macro-names).

Examples:

```c++
#define C 0
#define F1(x, y) ((a) > (b) ? (a) : (b))
#define F2(...) (__VA_ARGS__)
#define COMMA ,
#define NORETURN [[noreturn]]
#define DEPRECATED attribute((deprecated))

#if LIB_EXPORTS
#define DLLEXPORTS __declspec(dllexport)
#else
#define DLLEXPORTS __declspec(dllimport)
#endif
```

results in the following warnings:

```powershell
4 warnings generated.
test.cpp:1:9: warning: macro 'C' used to declare a constant; consider using a 'constexpr' constant [cppcoreguidelines-macro-usage]
#define C 0
        ^
test.cpp:2:9: warning: function-like macro 'F1' used; consider a 'constexpr' template function [cppcoreguidelines-macro-usage]
#define F1(x, y) ((a) > (b) ? (a) : (b))
        ^
test.cpp:3:9: warning: variadic macro 'F2' used; consider using a 'constexpr' variadic template function [cppcoreguidelines-macro-usage]
#define F2(...) (__VA_ARGS__)
        ^
```

#### Options

- `AllowedRegexp`
  - A regular expression to filter allowed macros.
  - For example `DEBUG*|LIBTORRENT*|TORRENT*|UNI*` could be applied to filter `libtorrent`.
  - Default value is `^DEBUG_*`.
- `CheckCapsOnly`
  - Boolean flag to warn on all macros except those with `CAPS_ONLY` names.
  - This option is intended to ease introduction of this check into older code bases.
  - Default value is `false`.
- `IgnoreCommandLineMacros`
  - Boolean flag to toggle ignoring command-line-defined macros.
  - Default value is `true`.

### cppcoreguidelines-misleading-capture-default-by-value

Warns when lambda specify a by-value capture default and capture `this`.

> 当lambda指定按值捕获默认值并捕获`this`时发出警告。

By-value capture defaults in member functions can be misleading about whether data members are captured by value or reference. This occurs because specifying the capture default `[=]` actually captures the `this` pointer by value, not the data members themselves. As a result, data members are still indirectly accessed via the captured `this` pointer, which essentially means they are being accessed by reference. Therefore, even when using `[=]`, data members are effectively captured by reference, which might not align with the user’s expectations.

>成员函数中的按值捕获默认值可能会被误导数据成员是按值捕获还是按引用捕获。
>
>这是因为指定捕获默认值`[=]`实际上是按值捕获`this`指针，而不是数据成员本身。
>
>因此，数据成员仍然是通过捕获的`this`指针间接访问的，这实际上意味着它们是通过引用访问的。
>
>因此，即使使用`[=]`，事实上也是通过引用捕获数据成员，这可能与用户的期望不一致。

Examples:

```c++
struct AClass
{
	int member;

	void misleadingLogic() {
		int local = 0;
		member = 0;
		auto f = [=]() mutable {
			local += 1;
			member += 1;
		};
		f();
		// Here, local is 0 but member is 1
	}

	void clearLogic() {
		int local = 0;
		member    = 0;
		auto f = [this, local]() mutable {
			local  += 1;
			member += 1;
		};
		f();
		// Here, local is 0 but member is 1
	}
};
```

This check implements [F.54](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f54-when-writing-a-lambda-that-captures-this-or-any-class-data-member-dont-use--default-capture) from the C++ Core Guidelines.

### cppcoreguidelines-owning-memory

This check implements the type-based semantics of `gsl::owner<T*>`, which allows static analysis on code, that uses raw pointers to handle resources like dynamic memory, but won’t introduce RAII concepts.

>这个检查实现了`gsl::owner<T*>`的基于类型的语义，它允许对代码进行静态分析，使用原始指针来处理像动态内存这样的资源，但不会引入RAII概念。

This check implements [I.11](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i11-never-transfer-ownership-by-a-raw-pointer-t-or-reference-t), [C.33](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c33-if-a-class-has-an-owning-pointer-member-define-a-destructor), [R.3](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#r3-a-raw-pointer-a-t-is-non-owning) and [GSL.Views](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#SS-views) from the C++ Core Guidelines.

The definition of a `gsl::owner<T*>` is straight forward:

```c++
namespace gsl {
    template <typename T> owner = T;
}
```

It is therefore simple to introduce the owner even without using an implementation of the [Guideline Support Library](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-gsl).

All checks are purely type based and not (yet) flow sensitive.

>所有的检查都是纯粹基于类型的，对流程不敏感。

The following examples will demonstrate the correct and incorrect initializations of owners, assignment is handled the same way. Note that both `new` and `malloc()`-like resource functions are considered to produce resources.

>下面的示例将演示所有者的正确和错误初始化，分配的处理方式相同。
>
>请注意，`new`和`malloc()`类型的资源函数都被认为是生成资源。

```c++
// Creating an owner with factory functions is checked.
gsl::owner<int*> function_that_returns_owner() {
    return gsl::owner<int*>(new int(42));
}

// Dynamic memory must be assigned to an owner
int* Something         = new int(42); // BAD, will be caught
gsl::owner<int*> Owner = new int(42); // Good
gsl::owner<int*> Owner = new int[42]; // Good as well

// Returned owner must be assigned to an owner
int* Something         = function_that_returns_owner(); // Bad, factory function
gsl::owner<int*> Owner = function_that_returns_owner(); // Good, result lands in owner

// Something not a resource or owner should not be assigned to owners
int Stack = 42;
gsl::owner<int*> Owned = &Stack; // Bad, not a resource assigned
```

In the case of dynamic memory as resource, only `gsl::owner<T*>` variables are allowed to be deleted.

>在动态内存作为资源的情况下，只允许删除`gsl::owner<T*>`变量。

```c++
// Example Bad, non-owner as resource handle, will be caught.
int* NonOwner = new int(42); // First warning here, since new must land in an owner
delete NonOwner; // Second warning here, since only owners are allowed to be deleted

// Example Good, Ownership correctly stated
gsl::owner<int*> Owner = new int(42); // Good
delete Owner; // Good as well, statically enforced, that only owners get deleted
```

The check will furthermore ensure, that functions, that expect a `gsl::owner<T*>` as argument get called with either a `gsl::owner<T*>` or a newly created resource.

>该检查还将进一步确保以`gsl::owner<T*>`作为参数的函数被`gsl::owner<T*>`或新创建的资源调用。

```c++
void expects_owner(gsl::owner<int*> o) {
    delete o;
}

// Bad Code
int NonOwner = 42;
expects_owner(&NonOwner); // Bad, will get caught

// Good Code
gsl::owner<int*> Owner = new int(42);
expects_owner(Owner);       // Good
expects_owner(new int(42)); // Good as well, recognized created resource

// Port legacy code for better resource-safety
gsl::owner<FILE*> File = fopen("my_file.txt", "rw+"); // Good
FILE* BadFile = fopen("another_file.txt", "w");       // Bad, warned

// ... use the file

fclose(File);    // Ok, File is annotated as 'owner<>'
fclose(BadFile); // Bad, BadFile is not an 'owner<>', will be warned
```

#### Options

`LegacyResourceProducers`

Semicolon-separated list of fully qualified names of legacy functions that create resources but cannot introduce `gsl::owner<>`.

Defaults to `::malloc;::aligned_alloc;::realloc;::calloc;::fopen;::freopen;::tmpfile`.

`LegacyResourceConsumers`

Semicolon-separated list of fully qualified names of legacy functions expecting resource owners as pointer arguments but cannot introduce `gsl::owner<>`.

Defaults to `::free;::realloc;::freopen;::fclose`.

#### Limitations

Using `gsl::owner<T*>` in a typedef or alias is not handled correctly.

```c++
using heap_int = gsl::owner<int*>;
heap_int allocated = new int(42); // False positive!
```

The `gsl::owner<T*>` is declared as a templated type alias. In template functions and classes, like in the example below, the information of the type aliases gets lost. Therefore using `gsl::owner<T*>` in a heavy templated code base might lead to false positives.

Known code constructs that do not get diagnosed correctly are:

- `std::exchange`
- `std::vector<gsl::owner<T*>>`

```c++
// This template function works as expected. Type information doesn't get lost.
template <typename T>
void delete_owner(gsl::owner<T*> owned_object) {
  delete owned_object; // Everything alright
}

gsl::owner<int*> function_that_returns_owner() {
    return gsl::owner<int*>(new int(42));
}

// Type deduction does not work for auto variables.
// This is caught by the check and will be noted accordingly.
auto OwnedObject = function_that_returns_owner(); // Type of OwnedObject will be 'int*'

// Problematic function template that looses the typeinformation on owner
template <typename T>
void bad_template_function(T some_object) {
  // This line will trigger the warning, that a non-owner is assigned to an owner
  gsl::owner<T*> new_owner = some_object;
}

// Calling the function with an owner still yields a false positive.
bad_template_function(gsl::owner<int*>(new int(42)));

// The same issue occurs with templated classes like the following.
template <typename T>
class OwnedValue {
public:
  const T getValue() const {
      return _val;
  }
private:
  T _val;
};

// Code, that yields a false positive.
OwnedValue<gsl::owner<int*>> Owner(new int(42)); // Type deduction yield T -> int *
// False positive, getValue returns int* and not gsl::owner<int*>
gsl::owner<int*> OwnedInt = Owner.getValue();
```

Another limitation of the current implementation is only the type based checking.

Suppose you have code like the following:

```c++
// Two owners with assigned resources
gsl::owner<int*> Owner1 = new int(42);
gsl::owner<int*> Owner2 = new int(42);

Owner2 = Owner1; // Conceptual Leak of initial resource of Owner2!
Owner1 = nullptr;
```

The semantic of a `gsl::owner<T*>` is mostly like a `std::unique_ptr<T>`, therefore assignment of two `gsl::owner<T*>` is considered a move, which requires that the resource `Owner2` must have been released before the assignment. This kind of condition could be caught in later improvements of this check with flowsensitive analysis. Currently, the Clang Static Analyzer catches this bug for dynamic memory, but not for general types of resources.

>`gsl::owner<T*>`的语义很像`std::unique_ptr<T>`，因此两个`gsl::owner<T*>`的赋值被认为是一次移动，这要求资源`Owner2`必须在赋值之前被释放。这种情况可以在以后用流量敏感分析改进。目前，Clang静态分析器可以捕获动态内存的这个错误，但不能捕获一般类型的资源。

>注：还是用`std::unique_ptr<T>`吧。

## fuchsia-*

Checks related to Fuchsia coding conventions.

## google-*

Checks related to Google coding conventions.

### google-readability-todo

Finds TODO comments without a username or bug number.

>查找没有用户名或bug号的TODO注释。

The relevant style guide section is https://google.github.io/styleguide/cppguide.html#TODO_Comments.

Corresponding cpplint.py check: readability/todo

## hicpp-*

Checks related to High Integrity C++ Coding Standard.

## llvm-*

Checks related to the LLVM coding conventions.

## misc-*

Checks that we didn’t have a better category for.

### misc-non-private-member-variables-in-classes

cppcoreguidelines-non-private-member-variables-in-classes redirects here as an alias for this check.

Finds classes that contain non-static data members in addition to user-declared non-static member functions and diagnose all data members declared with a non-public access specifier. The data members should be declared as private and accessed through member functions instead of exposed to derived classes or class consumers.

>查找除了用户声明的非静态成员函数之外还包含非静态数据成员的类，并诊断使用非公共访问说明符声明的所有数据成员。
>
>数据成员应该声明为私有并通过成员函数访问，而不是公开给派生类或类消费者。

#### Options

`IgnoreClassesWithAllMemberVariablesBeingPublic`

Allows to completely ignore classes if all the member variables in that class a declared with a public access specifier.

>如果类中的所有成员变量都是用公共访问说明符声明的，则允许完全忽略该类。

`IgnorePublicMemberVariables`

Allows to ignore (not diagnose) all the member variables declared with a public access specifier.

>允许忽略（而不是诊断）使用公共访问说明符声明的所有成员变量。

## modernize-*

Checks that advocate usage of modern (currently “modern” means “C++11”) language constructs.

>支持使用现代（目前“现代”指的是“C++ 11”及以后版本）语言结构的检查。

### modernize-avoid-bind

The check finds uses of `std::bind` and `boost::bind` and replaces them with lambdas. Lambdas will use value-capture unless reference capture is explicitly requested with `std::ref` or `boost::ref`.

It supports arbitrary callables including member functions, function objects, and free functions, and all variations thereof. Anything that you can pass to the first argument of `bind` should be diagnosable. Currently, the only known case where a fix-it is unsupported is when the same placeholder is specified multiple times in the parameter list.

Given:

```
int add(int x, int y) { return x + y; }
```

Then:

```
void f() {
  int x = 2;
  auto clj = std::bind(add, x, _1);
}
```

is replaced by:

```
void f() {
  int x = 2;
  auto clj = [=](auto && arg1) { return add(x, arg1); };
}
```

`std::bind` can be hard to read and can result in larger object files and binaries due to type information that will not be produced by equivalent lambdas.

#### Options

##### `PermissiveParameterList`

If the option is set to true, the check will append `auto&&...` to the end of every placeholder parameter list. Without this, it is possible for a fix-it to perform an incorrect transformation in the case where the result of the `bind` is used in the context of a type erased functor such as `std::function` which allows mismatched arguments. For example:

```
int add(int x, int y) { return x + y; }
int foo() {
  std::function<int(int,int)> ignore_args = std::bind(add, 2, 2);
  return ignore_args(3, 3);
}
```

is valid code, and returns 4. The actual values passed to `ignore_args` are simply ignored.

Without `PermissiveParameterList`, this would be transformed into

```
int add(int x, int y) { return x + y; }
int foo() {
  std::function<int(int,int)> ignore_args = [] { return add(2, 2); }
  return ignore_args(3, 3);
}
```

which will not compile, since the lambda does not contain an `operator()` that accepts 2 arguments.

With permissive parameter list, it instead generates

```
int add(int x, int y) { return x + y; }
int foo() {
  std::function<int(int,int)> ignore_args = [](auto&&...) { return add(2, 2); }
  return ignore_args(3, 3);
}
```

which is correct.

This check requires using C++14 or higher to run.

### modernize-avoid-c-arrays

cppcoreguidelines-avoid-c-arrays redirects here as an alias for this check.

hicpp-avoid-c-arrays redirects here as an alias for this check.

Finds C-style array types and recommend to use `std::array<>` / `std::vector<>`. All types of C arrays are diagnosed.

However, fix-it are potentially dangerous in header files and are therefore **not emitted** right now.

```c++
int a[] = {1, 2};	// warning: do not declare C-style arrays, use std::array<> instead
int b[1];			// warning: do not declare C-style arrays, use std::array<> instead

void foo() {
  int c[b[0]]; // warning: do not declare C VLA arrays, use std::vector<> instead
}

template <typename T, int Size>
class array {
  T d[Size];	// warning: do not declare C-style arrays, use std::array<> instead
  int e[1];		// warning: do not declare C-style arrays, use std::array<> instead
};

std::array<int[4], 2> d;	// warning: do not declare C-style arrays, use std::array<> instead
using k = int[4];			// warning: do not declare C-style arrays, use std::array<> instead
```

However, the `extern "C"` code is ignored, since it is common to share such headers between C code, and C++ code.

```c++
// Some header
extern "C" {

int f[] = {1, 2}; // not diagnosed

int j[1]; // not diagnosed

inline void bar() {
  {
    int j[j[0]]; // not diagnosed
  }
}

}
```

Similarly, the `main()` function is ignored. Its second and third parameters can be either `char* argv[]` or `char** argv`, but cannot be `std::array<>`.

### modernize-concat-nested-namespaces

Checks for use of nested namespaces such as `namespace a { namespace b { ... } }` and suggests changing to the more concise syntax introduced in C++17: `namespace a::b { ... }`. Inline namespaces are not modified.

For example:

```
namespace n1 {
namespace n2 {
    void t();
}
}

namespace n3 {
namespace n4 {
namespace n5 {
    void t();
}
}
namespace n6 {
namespace n7 {
    void t();
}
}
}

// in c++20
namespace n8 {
inline namespace n9 {
    void t();
}
}
```

Will be modified to:

```
namespace n1::n2 {
    void t();
}

namespace n3 {
namespace n4::n5 {
    void t();
}
namespace n6::n7 {
    void t();
}
}

// in c++20
namespace n8::inline n9 {
    void t();
}
```

### modernize-deprecated-headers

Some headers from C library were deprecated in C++ and are no longer welcome in C++ codebases. Some have no effect in C++. For more details refer to the C++ 14 Standard [depr.c.headers] section.

This check replaces C standard library headers with their C++ alternatives and removes redundant ones.

```
// C++ source file...
#include <assert.h>
#include <stdbool.h>

// becomes

#include <cassert>
// No 'stdbool.h' here.
```

Important note: the Standard doesn’t guarantee that the C++ headers declare all the same functions in the global namespace. The check in its current form can break the code that uses library symbols from the global namespace.

- <assert.h>
- <complex.h>
- <ctype.h>
- <errno.h>
- <fenv.h> // deprecated since C++11
- <float.h>
- <inttypes.h>
- <limits.h>
- <locale.h>
- <math.h>
- <setjmp.h>
- <signal.h>
- <stdarg.h>
- <stddef.h>
- <stdint.h>
- <stdio.h>
- <stdlib.h>
- <string.h>
- <tgmath.h> // deprecated since C++11
- <time.h>
- <uchar.h> // deprecated since C++11
- <wchar.h>
- <wctype.h>

If the specified standard is older than C++11 the check will only replace headers deprecated before C++11, otherwise – every header that appeared in the previous list.

These headers don’t have effect in C++:

- <iso646.h>
- <stdalign.h>
- <stdbool.h>

The checker ignores include directives within extern “C” { … } blocks, since a library might want to expose some API for C and C++ libraries.

```
// C++ source file...
extern "C" {
#include <assert.h>  // Left intact.
#include <stdbool.h> // Left intact.
}
```

#### Options

##### `CheckHeaderFile`

clang-tidy cannot know if the header file included by the currently analyzed C++ source file is not included by any other C source files. Hence, to omit false-positives and wrong fixit-hints, we ignore emitting reports into header files. One can set this option to true if they know that the header files in the project are only used by C++ source file. Default is false.

### modernize-deprecated-ios-base-aliases

Detects usage of the deprecated member types of `std::ios_base` and replaces those that have a non-deprecated equivalent.

| Deprecated member type     | Replacement               |
| :------------------------- | :------------------------ |
| `std::ios_base::io_state`  | `std::ios_base::iostate`  |
| `std::ios_base::open_mode` | `std::ios_base::openmode` |
| `std::ios_base::seek_dir`  | `std::ios_base::seekdir`  |
| `std::ios_base::streamoff` |                           |
| `std::ios_base::streampos` |                           |

### modernize-loop-convert

This check converts `for(...; ...; ...)` loops to use the new range-based loops in C++11.

Three kinds of loops can be converted:

- Loops over statically allocated arrays.
- Loops over containers, using iterators.
- Loops over array-like containers, using `operator[]` and `at()`.

#### MinConfidence option

##### risky

In loops where the container expression is more complex than just a reference to a declared expression (a variable, function, enum, etc.), and some part of it appears elsewhere in the loop, we lower our confidence in the transformation due to the increased risk of changing semantics. Transformations for these loops are marked as risky, and thus will only be converted if the minimum required confidence level is set to risky.

```
int arr[10][20];
int l = 5;

for (int j = 0; j < 20; ++j)
  int k = arr[l][j] + l; // using l outside arr[l] is considered risky

for (int i = 0; i < obj.getVector().size(); ++i)
  obj.foo(10); // using 'obj' is considered risky
```

See [Range-based loops evaluate end() only once](https://clang.llvm.org/extra/clang-tidy/checks/modernize/loop-convert.html#incorrectriskytransformation) for an example of an incorrect transformation when the minimum required confidence level is set to risky.

##### reasonable (Default)

If a loop calls `.end()` or `.size()` after each iteration, the transformation for that loop is marked as reasonable, and thus will be converted if the required confidence level is set to reasonable (default) or lower.

```
// using size() is considered reasonable
for (int i = 0; i < container.size(); ++i)
  cout << container[i];
```

##### safe

Any other loops that do not match the above criteria to be marked as risky or reasonable are marked safe, and thus will be converted if the required confidence level is set to safe or lower.

```
int arr[] = {1,2,3};

for (int i = 0; i < 3; ++i)
  cout << arr[i];
```

#### Example

Original:

```
const int N = 5;
int arr[] = {1,2,3,4,5};
vector<int> v;
v.push_back(1);
v.push_back(2);
v.push_back(3);

// safe conversion
for (int i = 0; i < N; ++i)
  cout << arr[i];

// reasonable conversion
for (vector<int>::iterator it = v.begin(); it != v.end(); ++it)
  cout << *it;

// reasonable conversion
for (vector<int>::iterator it = begin(v); it != end(v); ++it)
  cout << *it;

// reasonable conversion
for (vector<int>::iterator it = std::begin(v); it != std::end(v); ++it)
  cout << *it;

// reasonable conversion
for (int i = 0; i < v.size(); ++i)
  cout << v[i];

// reasonable conversion
for (int i = 0; i < size(v); ++i)
  cout << v[i];
```

After applying the check with minimum confidence level set to reasonable (default):

```
const int N = 5;
int arr[] = {1,2,3,4,5};
vector<int> v;
v.push_back(1);
v.push_back(2);
v.push_back(3);

// safe conversion
for (auto & elem : arr)
  cout << elem;

// reasonable conversion
for (auto & elem : v)
  cout << elem;

// reasonable conversion
for (auto & elem : v)
  cout << elem;
```

#### Reverse Iterator Support

The converter is also capable of transforming iterator loops which use `rbegin` and `rend` for looping backwards over a container. Out of the box this will automatically happen in C++20 mode using the `ranges` library, however the check can be configured to work without C++20 by specifying a function to reverse a range and optionally the header file where that function lives.

##### `UseCxx20ReverseRanges`

When set to true convert loops when in C++20 or later mode using `std::ranges::reverse_view`. Default value is `true`.

##### `MakeReverseRangeFunction`

Specify the function used to reverse an iterator pair, the function should accept a class with `rbegin` and `rend` methods and return a class with `begin` and `end` methods that call the `rbegin` and `rend` methods respectively. Common examples are `ranges::reverse_view` and `llvm::reverse`. Default value is an empty string.

##### `MakeReverseRangeHeader`

Specifies the header file where [`MakeReverseRangeFunction`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/loop-convert.html#cmdoption-arg-MakeReverseRangeFunction) is declared. For the previous examples this option would be set to `range/v3/view/reverse.hpp` and `llvm/ADT/STLExtras.h` respectively. If this is an empty string and [`MakeReverseRangeFunction`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/loop-convert.html#cmdoption-arg-MakeReverseRangeFunction) is set, the check will proceed on the assumption that the function is already available in the translation unit. This can be wrapped in angle brackets to signify to add the include as a system include. Default value is an empty string.

##### `IncludeStyle`

A string specifying which include-style is used, llvm or google. Default is llvm.

#### Limitations

There are certain situations where the tool may erroneously perform transformations that remove information and change semantics. Users of the tool should be aware of the behavior and limitations of the check outlined by the cases below.

##### Comments inside loop headers

Comments inside the original loop header are ignored and deleted when transformed.

```
for (int i = 0; i < N; /* This will be deleted */ ++i) { }
```

##### Range-based loops evaluate end() only once

The C++11 range-based for loop calls `.end()` only once during the initialization of the loop. If in the original loop `.end()` is called after each iteration the semantics of the transformed loop may differ.

```
// The following is semantically equivalent to the C++11 range-based for loop,
// therefore the semantics of the header will not change.
for (iterator it = container.begin(), e = container.end(); it != e; ++it) { }

// Instead of calling .end() after each iteration, this loop will be
// transformed to call .end() only once during the initialization of the loop,
// which may affect semantics.
for (iterator it = container.begin(); it != container.end(); ++it) { }
```

As explained above, calling member functions of the container in the body of the loop is considered risky. If the called member function modifies the container the semantics of the converted loop will differ due to `.end()` being called only once.

```
bool flag = false;
for (vector<T>::iterator it = vec.begin(); it != vec.end(); ++it) {
  // Add a copy of the first element to the end of the vector.
  if (!flag) {
    // This line makes this transformation 'risky'.
    vec.push_back(*it);
    flag = true;
  }
  cout << *it;
}
```

The original code above prints out the contents of the container including the newly added element while the converted loop, shown below, will only print the original contents and not the newly added element.

```
bool flag = false;
for (auto & elem : vec) {
  // Add a copy of the first element to the end of the vector.
  if (!flag) {
    // This line makes this transformation 'risky'
    vec.push_back(elem);
    flag = true;
  }
  cout << elem;
}
```

Semantics will also be affected if `.end()` has side effects. For example, in the case where calls to `.end()` are logged the semantics will change in the transformed loop if `.end()` was originally called after each iteration.

```
iterator end() {
  num_of_end_calls++;
  return container.end();
}
```

##### Overloaded operator->() with side effects

Similarly, if `operator->()` was overloaded to have side effects, such as logging, the semantics will change. If the iterator’s `operator->()` was used in the original loop it will be replaced with `<container element>.<member>` instead due to the implicit dereference as part of the range-based for loop. Therefore any side effect of the overloaded `operator->()` will no longer be performed.

```
for (iterator it = c.begin(); it != c.end(); ++it) {
  it->func(); // Using operator->()
}
// Will be transformed to:
for (auto & elem : c) {
  elem.func(); // No longer using operator->()
}
```

##### Pointers and references to containers

While most of the check’s risk analysis is dedicated to determining whether the iterator or container was modified within the loop, it is possible to circumvent the analysis by accessing and modifying the container through a pointer or reference.

If the container were directly used instead of using the pointer or reference the following transformation would have only been applied at the risky level since calling a member function of the container is considered risky. The check cannot identify expressions associated with the container that are different than the one used in the loop header, therefore the transformation below ends up being performed at the safe level.

```
vector<int> vec;

vector<int> *ptr = &vec;
vector<int> &ref = vec;

for (vector<int>::iterator it = vec.begin(), e = vec.end(); it != e; ++it) {
  if (!flag) {
    // Accessing and modifying the container is considered risky, but the risk
    // level is not raised here.
    ptr->push_back(*it);
    ref.push_back(*it);
    flag = true;
  }
}
```

##### OpenMP

As range-based for loops are only available since OpenMP 5, this check should not be used on code with a compatibility requirement of OpenMP prior to version 5. It is **intentional** that this check does not make any attempts to exclude incorrect diagnostics on OpenMP for loops prior to OpenMP 5.

To prevent this check to be applied (and to break) OpenMP for loops but still be applied to non-OpenMP for loops the usage of `NOLINT` (see [Suppressing Undesired Diagnostics](https://clang.llvm.org/extra/clang-tidy/index.html#clang-tidy-nolint)) on the specific for loops is recommended.

### modernize-macro-to-enum

Replaces groups of adjacent macros with an unscoped anonymous enum. Using an unscoped anonymous enum ensures that everywhere the macro token was used previously, the enumerator name may be safely used.

This check can be used to enforce the C++ core guideline [Enum.1: Prefer enumerations over macros](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#enum1-prefer-enumerations-over-macros), within the constraints outlined below.

Potential macros for replacement must meet the following constraints:

- Macros must expand only to integral literal tokens or expressions of literal tokens. The expression may contain any of the unary operators `-`, `+`, `~` or `!`, any of the binary operators `,`, `-`, `+`, `*`, `/`, `%`, `&`, `|`, `^`, `<`, `>`, `<=`, `>=`, `==`, `!=`, `||`, `&&`, `<<`, `>>` or `<=>`, the ternary operator `?:` and its [GNU extension](https://gcc.gnu.org/onlinedocs/gcc/Conditionals.html). Parenthesized expressions are also recognized. This recognizes most valid expressions. In particular, expressions with the `sizeof` operator are not recognized.
- Macros must be defined on sequential source file lines, or with only comment lines in between macro definitions.
- Macros must all be defined in the same source file.
- Macros must not be defined within a conditional compilation block. (Conditional include guards are exempt from this constraint.)
- Macros must not be defined adjacent to other preprocessor directives.
- Macros must not be used in any conditional preprocessing directive.
- Macros must not be used as arguments to other macros.
- Macros must not be undefined.
- Macros must be defined at the top-level, not inside any declaration or definition.

Each cluster of macros meeting the above constraints is presumed to be a set of values suitable for replacement by an anonymous enum. From there, a developer can give the anonymous enum a name and continue refactoring to a scoped enum if desired. Comments on the same line as a macro definition or between subsequent macro definitions are preserved in the output. No formatting is assumed in the provided replacements, although clang-tidy can optionally format all fixes.

#### Warning

Initializing expressions are assumed to be valid initializers for an enum. C requires that enum values fit into an `int`, but this may not be the case for some accepted constant expressions. For instance `1 << 40` will not fit into an `int` when the size of an `int` is 32 bits.

#### Examples

```
#define RED   0xFF0000
#define GREEN 0x00FF00
#define BLUE  0x0000FF

#define TM_NONE (-1) // No method selected.
#define TM_ONE 1    // Use tailored method one.
#define TM_TWO 2    // Use tailored method two.  Method two
                    // is preferable to method one.
#define TM_THREE 3  // Use tailored method three.
```

becomes

```
enum {
RED = 0xFF0000,
GREEN = 0x00FF00,
BLUE = 0x0000FF
};

enum {
TM_NONE = (-1), // No method selected.
TM_ONE = 1,    // Use tailored method one.
TM_TWO = 2,    // Use tailored method two.  Method two
                    // is preferable to method one.
TM_THREE = 3  // Use tailored method three.
};
```

### modernize-make-shared

This check finds the creation of `std::shared_ptr` objects by explicitly calling the constructor and a `new` expression, and replaces it with a call to `std::make_shared`.

```
auto my_ptr = std::shared_ptr<MyPair>(new MyPair(1, 2));

// becomes

auto my_ptr = std::make_shared<MyPair>(1, 2);
```

This check also finds calls to `std::shared_ptr::reset()` with a `new` expression, and replaces it with a call to `std::make_shared`.

```
my_ptr.reset(new MyPair(1, 2));

// becomes

my_ptr = std::make_shared<MyPair>(1, 2);
```

#### Options

##### `MakeSmartPtrFunction`

A string specifying the name of make-shared-ptr function.

Default is std::make_shared.

##### `MakeSmartPtrFunctionHeader`

A string specifying the corresponding header of make-shared-ptr function.

Default is memory.

##### IncludeStyle

A string specifying which include-style is used, llvm or google.

Default is llvm.

##### `IgnoreMacros`

If set to true, the check will not give warnings inside macros.

Default is true.

##### `IgnoreDefaultInitialization`

If set to non-zero, the check does not suggest edits that will transform default initialization into value initialization, as this can cause performance regressions.

Default is 1.

### modernize-make-unique

This check finds the creation of `std::unique_ptr` objects by explicitly calling the constructor and a `new` expression, and replaces it with a call to `std::make_unique`, introduced in C++14.

```
auto my_ptr = std::unique_ptr<MyPair>(new MyPair(1, 2));

// becomes

auto my_ptr = std::make_unique<MyPair>(1, 2);
```

This check also finds calls to `std::unique_ptr::reset()` with a `new` expression, and replaces it with a call to `std::make_unique`.

```
my_ptr.reset(new MyPair(1, 2));

// becomes

my_ptr = std::make_unique<MyPair>(1, 2);
```

#### Options

##### `MakeSmartPtrFunction`

A string specifying the name of make-unique-ptr function.

Default is std::make_unique.

##### `MakeSmartPtrFunctionHeader`

A string specifying the corresponding header of make-unique-ptr function.

Default is <memory>.

##### `IncludeStyle`

A string specifying which include-style is used, llvm or google.

Default is llvm.

##### `IgnoreMacros`

If set to true, the check will not give warnings inside macros.

Default is true.

##### `IgnoreDefaultInitialization`

If set to non-zero, the check does not suggest edits that will transform default initialization into value initialization, as this can cause performance regressions.

Default is 1.

### modernize-pass-by-value

With move semantics added to the language and the standard library updated with move constructors added for many types it is now interesting to take an argument directly by value, instead of by const-reference, and then copy. This check allows the compiler to take care of choosing the best way to construct the copy.

The transformation is usually beneficial when the calling code passes an *rvalue* and assumes the move construction is a cheap operation. This short example illustrates how the construction of the value happens:

```
void foo(std::string s);
std::string get_str();

void f(const std::string &str) {
  foo(str);       // lvalue  -> copy construction
  foo(get_str()); // prvalue -> move construction
}
```

**Note**

Currently, only constructors are transformed to make use of pass-by-value. Contributions that handle other situations are welcome!

#### Pass-by-value in constructors

Replaces the uses of const-references constructor parameters that are copied into class fields. The parameter is then moved with std::move().

Since `std::move()` is a library function declared in <utility> it may be necessary to add this include. The check will add the include directive when necessary.

```
 #include <string>

 class Foo {
 public:
-  Foo(const std::string &Copied, const std::string &ReadOnly)
-    : Copied(Copied), ReadOnly(ReadOnly)
+  Foo(std::string Copied, const std::string &ReadOnly)
+    : Copied(std::move(Copied)), ReadOnly(ReadOnly)
   {}

 private:
   std::string Copied;
   const std::string &ReadOnly;
 };

 std::string get_cwd();

 void f(const std::string &Path) {
   // The parameter corresponding to 'get_cwd()' is move-constructed. By
   // using pass-by-value in the Foo constructor we managed to avoid a
   // copy-construction.
   Foo foo(get_cwd(), Path);
 }
```

If the parameter is used more than once no transformation is performed since moved objects have an undefined state. It means the following code will be left untouched:

```
#include <string>

void pass(const std::string &S);

struct Foo {
  Foo(const std::string &S) : Str(S) {
    pass(S);
  }

  std::string Str;
};
```

##### Known limitations

A situation where the generated code can be wrong is when the object referenced is modified before the assignment in the init-list through a “hidden” reference.

Example:

```
 std::string s("foo");

 struct Base {
   Base() {
     s = "bar";
   }
 };

 struct Derived : Base {
-  Derived(const std::string &S) : Field(S)
+  Derived(std::string S) : Field(std::move(S))
   { }

   std::string Field;
 };

 void f() {
-  Derived d(s); // d.Field holds "bar"
+  Derived d(s); // d.Field holds "foo"
 }
```

##### Note about delayed template parsing

When delayed template parsing is enabled, constructors part of templated contexts; templated constructors, constructors in class templates, constructors of inner classes of template classes, etc., are not transformed. Delayed template parsing is enabled by default on Windows as a Microsoft extension: [Clang Compiler User’s Manual - Microsoft extensions](https://clang.llvm.org/docs/UsersManual.html#microsoft-extensions).

Delayed template parsing can be enabled using the -fdelayed-template-parsing flag and disabled using -fno-delayed-template-parsing.

Example:

```
  template <typename T> class C {
    std::string S;

  public:
=  // using -fdelayed-template-parsing (default on Windows)
=  C(const std::string &S) : S(S) {}

+  // using -fno-delayed-template-parsing (default on non-Windows systems)
+  C(std::string S) : S(std::move(S)) {}
  };
```

**See also**

For more information about the pass-by-value idiom, read: [Want Speed? Pass by Value](https://web.archive.org/web/20140205194657/http://cpp-next.com/archive/2009/08/want-speed-pass-by-value/).

#### Options

##### `IncludeStyle`

A string specifying which include-style is used, llvm or google.

Default is llvm.

##### `ValuesOnly`

When true, the check only warns about copied parameters that are already passed by value.

Default is false.

### modernize-raw-string-literal

This check selectively replaces string literals containing escaped characters with raw string literals.

Example:

```
const char *const Quotes{"embedded \"quotes\""};
const char *const Paragraph{"Line one.\nLine two.\nLine three.\n"};
const char *const SingleLine{"Single line.\n"};
const char *const TrailingSpace{"Look here -> \n"};
const char *const Tab{"One\tTwo\n"};
const char *const Bell{"Hello!\a  And welcome!"};
const char *const Path{"C:\\Program Files\\Vendor\\Application.exe"};
const char *const RegEx{"\\w\\([a-z]\\)"};
```

becomes

```
const char *const Quotes{R"(embedded "quotes")"};
const char *const Paragraph{"Line one.\nLine two.\nLine three.\n"};
const char *const SingleLine{"Single line.\n"};
const char *const TrailingSpace{"Look here -> \n"};
const char *const Tab{"One\tTwo\n"};
const char *const Bell{"Hello!\a  And welcome!"};
const char *const Path{R"(C:\Program Files\Vendor\Application.exe)"};
const char *const RegEx{R"(\w\([a-z]\))"};
```

The presence of any of the following escapes can cause the string to be converted to a raw string literal: `\\`, `\'`, `\"`, `\?`, and octal or hexadecimal escapes for printable ASCII characters.

A string literal containing only escaped newlines is a common way of writing lines of text output. Introducing physical newlines with raw string literals in this case is likely to impede readability. These string literals are left unchanged.

An escaped horizontal tab, form feed, or vertical tab prevents the string literal from being converted. The presence of a horizontal tab, form feed or vertical tab in source code is not visually obvious.

#### Options

##### `DelimiterStem`

Custom delimiter to escape characters in raw string literals.

It is used in the following construction: `R"stem_delimiter(contents)stem_delimiter"`.

The default value is lit.

##### `ReplaceShorterLiterals`

Controls replacing shorter non-raw string literals with longer raw string literals.

Setting this option to true enables the replacement.

The default value is false (shorter literals are not replaced).

### modernize-redundant-void-arg

Find and remove redundant `void` argument lists.

Examples

| Initial code                      | Code with applied fixes   |
| --------------------------------- | ------------------------- |
| `int f(void);`                    | `int f();`                |
| `int (*f(void))(void);`           | `int (*f())();`           |
| `typedef int (*f_t(void))(void);` | `typedef int (*f_t())();` |
| `void (C::*p)(void);`             | `void (C::*p)();`         |
| `C::C(void) {}`                   | `C::C() {}`               |
| `C::~C(void) {}`                  | `C::~C() {}`              |

### modernize-replace-auto-ptr

This check replaces the uses of the deprecated class `std::auto_ptr` by `std::unique_ptr` (introduced in C++11). The transfer of ownership, done by the copy-constructor and the assignment operator, is changed to match `std::unique_ptr` usage by using explicit calls to `std::move()`.

Migration example:

```
-void take_ownership_fn(std::auto_ptr<int> int_ptr);
+void take_ownership_fn(std::unique_ptr<int> int_ptr);

 void f(int x) {
-  std::auto_ptr<int> a(new int(x));
-  std::auto_ptr<int> b;
+  std::unique_ptr<int> a(new int(x));
+  std::unique_ptr<int> b;

-  b = a;
-  take_ownership_fn(b);
+  b = std::move(a);
+  take_ownership_fn(std::move(b));
 }
```

Since `std::move()` is a library function declared in `<utility>` it may be necessary to add this include. The check will add the include directive when necessary.

#### Known Limitations

- If headers modification is not activated or if a header is not allowed to be changed this check will produce broken code (compilation error), where the headers’ code will stay unchanged while the code using them will be changed.
- Client code that declares a reference to an `std::auto_ptr` coming from code that can’t be migrated (such as a header coming from a 3rd party library) will produce a compilation error after migration. This is because the type of the reference will be changed to `std::unique_ptr` but the type returned by the library won’t change, binding a reference to `std::unique_ptr` from an `std::auto_ptr`. This pattern doesn’t make much sense and usually `std::auto_ptr` are stored by value (otherwise what is the point in using them instead of a reference or a pointer?).

```
 // <3rd-party header...>
 std::auto_ptr<int> get_value();
 const std::auto_ptr<int> & get_ref();

 // <calling code (with migration)...>
-std::auto_ptr<int> a(get_value());
+std::unique_ptr<int> a(get_value()); // ok, unique_ptr constructed from auto_ptr

-const std::auto_ptr<int> & p = get_ptr();
+const std::unique_ptr<int> & p = get_ptr(); // won't compile
```

- Non-instantiated templates aren’t modified.

```
template <typename X>
void f() {
    std::auto_ptr<X> p;
}

// only 'f<int>()' (or similar) will trigger the replacement.
```

#### Options

##### `IncludeStyle`

A string specifying which include-style is used, llvm or google.

Default is llvm.

### modernize-replace-disallow-copy-and-assign-macro

Finds macro expansions of `DISALLOW_COPY_AND_ASSIGN(Type)` and replaces them with a deleted copy constructor and a deleted assignment operator.

Before the `delete` keyword was introduced in C++11 it was common practice to declare a copy constructor and an assignment operator as private members. This effectively makes them unusable to the public API of a class.

With the advent of the `delete` keyword in C++11 we can abandon the `private` access of the copy constructor and the assignment operator and delete the methods entirely.

When running this check on a code like this:

```
class Foo {
private:
  DISALLOW_COPY_AND_ASSIGN(Foo);
};
```

It will be transformed to this:

```
class Foo {
private:
  Foo(const Foo &) = delete;
  const Foo &operator=(const Foo &) = delete;
};
```

#### Known Limitations

Notice that the migration example above leaves the `private` access specification untouched. You might want to run the check [modernize-use-equals-delete](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-equals-delete.html) to get warnings for deleted functions in private sections.

#### Options

##### `MacroName`

A string specifying the macro name whose expansion will be replaced.

Default is `DISALLOW_COPY_AND_ASSIGN`.

See: https://en.cppreference.com/w/cpp/language/function#Deleted_functions

### modernize-replace-random-shuffle

This check will find occurrences of `std::random_shuffle` and replace it with `std::shuffle`. In C++17 `std::random_shuffle` will no longer be available and thus we need to replace it.

Below are two examples of what kind of occurrences will be found and two examples of what it will be replaced with.

```
std::vector<int> v;

// First example
std::random_shuffle(vec.begin(), vec.end());

// Second example
std::random_shuffle(vec.begin(), vec.end(), randomFunc);
```

Both of these examples will be replaced with:

```
std::shuffle(vec.begin(), vec.end(), std::mt19937(std::random_device()()));
```

The second example will also receive a warning that `randomFunc` is no longer supported in the same way as before so if the user wants the same functionality, the user will need to change the implementation of the `randomFunc`.

One thing to be aware of here is that `std::random_device` is quite expensive to initialize. So if you are using the code in a performance critical place, you probably want to initialize it elsewhere. Another thing is that the seeding quality of the suggested fix is quite poor: `std::mt19937` has an internal state of 624 32-bit integers, but is only seeded with a single integer. So if you require higher quality randomness, you should consider seeding better, for example:

```
std::shuffle(v.begin(), v.end(), []() {
  std::mt19937::result_type seeds[std::mt19937::state_size];
  std::random_device device;
  std::uniform_int_distribution<typename std::mt19937::result_type> dist;
  std::generate(std::begin(seeds), std::end(seeds), [&] { return dist(device); });
  std::seed_seq seq(std::begin(seeds), std::end(seeds));
  return std::mt19937(seq);
}());
```

### modernize-return-braced-init-list

Replaces explicit calls to the constructor in a return with a braced initializer list. This way the return type is not needlessly duplicated in the function definition and the return statement.

>用带括号的初始化列表替换返回函数中对构造函数的显式调用。这样，返回类型就不会在函数定义和返回语句中不必要地重复。

```c++
Foo bar() {
  Baz baz;
  return Foo(baz);
}

// transforms to:

Foo bar() {
  Baz baz;
  return {baz};
}
```

### modernize-shrink-to-fit

Replace copy and swap tricks on shrinkable containers with the `shrink_to_fit()` method call.

The `shrink_to_fit()` method is more readable and more effective than the copy and swap trick to reduce the capacity of a shrinkable container. Note that, the `shrink_to_fit()` method is only available in C++11 and up.

### modernize-type-traits

Converts standard library type traits of the form `traits<...>::type` and `traits<...>::value` into `traits_t<...>` and `traits_v<...>` respectively.

For example:

```
std::is_integral<T>::value
std::is_same<int, float>::value
typename std::add_const<T>::type
std::make_signed<unsigned>::type
```

Would be converted into:

```
std::is_integral_v<T>
std::is_same_v<int, float>
std::add_const_t<T>
std::make_signed_t<unsigned>
```

#### Options

##### `IgnoreMacros`

If true don’t diagnose traits defined in macros.

Note: Fixes will never be emitted for code inside of macros.`#define IS_SIGNED(T) std::is_signed<T>::value `.

Defaults to false.

### modernize-unary-static-assert

The check diagnoses any `static_assert` declaration with an empty string literal and provides a fix-it to replace the declaration with a single-argument `static_assert` declaration.

The check is only applicable for C++17 and later code.

The following code:

```
void f_textless(int a) {
  static_assert(sizeof(a) <= 10, "");
}
```

is replaced by:

```
void f_textless(int a) {
  static_assert(sizeof(a) <= 10);
}
```

### modernize-use-auto

This check is responsible for using the `auto` type specifier for variable declarations to *improve code readability and maintainability*. For example:

```
std::vector<int>::iterator I = my_container.begin();

// transforms to:

auto I = my_container.begin();
```

The `auto` type specifier will only be introduced in situations where the variable type matches the type of the initializer expression. In other words `auto` should deduce the same type that was originally spelled in the source. However, not every situation should be transformed:

```
int val = 42;
InfoStruct &I = SomeObject.getInfo();

// Should not become:

auto val = 42;
auto &I = SomeObject.getInfo();
```

In this example using `auto` for builtins doesn’t improve readability. In other situations it makes the code less self-documenting impairing readability and maintainability. As a result, `auto` is used only introduced in specific situations described below.

#### Iterators

Iterator type specifiers tend to be long and used frequently, especially in loop constructs. Since the functions generating iterators have a common format, the type specifier can be replaced without obscuring the meaning of code while improving readability and maintainability.

```
for (std::vector<int>::iterator I = my_container.begin(),
                                E = my_container.end();
     I != E; ++I) {
}

// becomes

for (auto I = my_container.begin(), E = my_container.end(); I != E; ++I) {
}
```

The check will only replace iterator type-specifiers when all of the following conditions are satisfied:

- The iterator is for one of the standard containers in `std` namespace:
  - `array`
  - `deque`
  - `forward_list`
  - `list`
  - `vector`
  - `map`
  - `multimap`
  - `set`
  - `multiset`
  - `unordered_map`
  - `unordered_multimap`
  - `unordered_set`
  - `unordered_multiset`
  - `queue`
  - `priority_queue`
  - `stack`
- The iterator is one of the possible iterator types for standard containers:
  - `iterator`
  - `reverse_iterator`
  - `const_iterator`
  - `const_reverse_iterator`
- In addition to using iterator types directly, typedefs or other ways of referring to those types are also allowed. However, implementation-specific types for which a type like `std::vector<int>::iterator` is itself a typedef will not be transformed. Consider the following examples:

```
// The following direct uses of iterator types will be transformed.
std::vector<int>::iterator I = MyVec.begin();
{
  using namespace std;
  list<int>::iterator I = MyList.begin();
}

// The type specifier for J would transform to auto since it's a typedef
// to a standard iterator type.
typedef std::map<int, std::string>::const_iterator map_iterator;
map_iterator J = MyMap.begin();

// The following implementation-specific iterator type for which
// std::vector<int>::iterator could be a typedef would not be transformed.
__gnu_cxx::__normal_iterator<int*, std::vector> K = MyVec.begin();
```

- The initializer for the variable being declared is not a braced initializer list. Otherwise, use of `auto` would cause the type of the variable to be deduced as `std::initializer_list`.

#### New expressions

Frequently, when a pointer is declared and initialized with `new`, the pointee type is written twice: in the declaration type and in the `new` expression. In this case, the declaration type can be replaced with `auto` improving readability and maintainability.

```
TypeName *my_pointer = new TypeName(my_param);

// becomes

auto *my_pointer = new TypeName(my_param);
```

The check will also replace the declaration type in multiple declarations, if the following conditions are satisfied:

- All declared variables have the same type (i.e. all of them are pointers to the same type).
- All declared variables are initialized with a `new` expression.
- The types of all the new expressions are the same than the pointee of the declaration type.

```
TypeName *my_first_pointer = new TypeName, *my_second_pointer = new TypeName;

// becomes

auto *my_first_pointer = new TypeName, *my_second_pointer = new TypeName;
```

#### Cast expressions

Frequently, when a variable is declared and initialized with a cast, the variable type is written twice: in the declaration type and in the cast expression. In this case, the declaration type can be replaced with `auto` improving readability and maintainability.

```
TypeName *my_pointer = static_cast<TypeName>(my_param);

// becomes

auto *my_pointer = static_cast<TypeName>(my_param);
```

The check handles `static_cast`, `dynamic_cast`, `const_cast`, `reinterpret_cast`, functional casts, C-style casts and function templates that behave as casts, such as `llvm::dyn_cast`, `boost::lexical_cast` and `gsl::narrow_cast`. Calls to function templates are considered to behave as casts if the first template argument is explicit and is a type, and the function returns that type, or a pointer or reference to it.

#### Known Limitations

- If the initializer is an explicit conversion constructor, the check will not replace the type specifier even though it would be safe to do so.
- User-defined iterators are not handled at this time.

#### Options

##### `MinTypeNameLength`

If the option is set to non-zero (default 5), the check will ignore type names having a length less than the option value. The option affects expressions only, not iterators. Spaces between multi-lexeme type names (`long int`) are considered as one. If the [`RemoveStars`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-auto.html#cmdoption-arg-RemoveStars) option (see below) is set to true, then `*s` in the type are also counted as a part of the type name.

```
// MinTypeNameLength = 0, RemoveStars=0

int a = static_cast<int>(foo());            // ---> auto a = ...
// length(bool *) = 4
bool *b = new bool;                         // ---> auto *b = ...
unsigned c = static_cast<unsigned>(foo());  // ---> auto c = ...

// MinTypeNameLength = 5, RemoveStars=0

int a = static_cast<int>(foo());                 // ---> int  a = ...
bool b = static_cast<bool>(foo());               // ---> bool b = ...
bool *pb = static_cast<bool*>(foo());            // ---> bool *pb = ...
unsigned c = static_cast<unsigned>(foo());       // ---> auto c = ...
// length(long <on-or-more-spaces> int) = 8
long int d = static_cast<long int>(foo());       // ---> auto d = ...

// MinTypeNameLength = 5, RemoveStars=1

int a = static_cast<int>(foo());                 // ---> int  a = ...
// length(int * * ) = 5
int **pa = static_cast<int**>(foo());            // ---> auto pa = ...
bool b = static_cast<bool>(foo());               // ---> bool b = ...
bool *pb = static_cast<bool*>(foo());            // ---> auto pb = ...
unsigned c = static_cast<unsigned>(foo());       // ---> auto c = ...
long int d = static_cast<long int>(foo());       // ---> auto d = ...
```

##### `RemoveStars`

If the option is set to true (default is false), the check will remove stars from the non-typedef pointer types when replacing type names with `auto`. Otherwise, the check will leave stars.

For example:

```
TypeName *my_first_pointer = new TypeName, *my_second_pointer = new TypeName;

// RemoveStars = 0

auto *my_first_pointer = new TypeName, *my_second_pointer = new TypeName;

// RemoveStars = 1

auto my_first_pointer = new TypeName, my_second_pointer = new TypeName;
```

### modernize-use-bool-literals

Finds integer literals which are cast to `bool`.

```
bool p = 1;
bool f = static_cast<bool>(1);
std::ios_base::sync_with_stdio(0);
bool x = p ? 1 : 0;

// transforms to

bool p = true;
bool f = true;
std::ios_base::sync_with_stdio(false);
bool x = p ? true : false;
```

#### Options

##### `IgnoreMacros`

If set to true, the check will not give warnings inside macros.

Default is true.

### modernize-use-constraints

Replace `std::enable_if` with C++20 requires clauses.

`std::enable_if` is a SFINAE mechanism for selecting the desired function or class template based on type traits or other requirements. `enable_if` changes the meta-arity of the template, and has other [adverse side effects](https://open-std.org/JTC1/SC22/WG21/docs/papers/2016/p0225r0.html) in the code. C++20 introduces concepts and constraints as a cleaner language provided solution to achieve the same outcome.

This check finds some common `std::enable_if` patterns that can be replaced by C++20 requires clauses. The tool can replace some of these patterns automatically, otherwise, the tool will emit a diagnostic without a replacement. The tool can detect the following `std::enable_if` patterns

1. `std::enable_if` in the return type of a function
2. `std::enable_if` as the trailing template parameter for function templates

Other uses, for example, in class templates for function parameters, are not currently supported by this tool. Other variants such as `boost::enable_if` are not currently supported by this tool.

Below are some examples of code using `std::enable_if`.

```
// enable_if in function return type
template <typename T>
std::enable_if_t<T::some_trait, int> only_if_t_has_the_trait() { ... }

// enable_if in the trailing template parameter
template <typename T, std::enable_if_t<T::some_trait, int> = 0>
void another_version() { ... }

template <typename T>
typename std::enable_if<T::some_value, Obj>::type existing_constraint() requires (T::another_value) {
  return Obj{};
}

template <typename T, std::enable_if_t<T::some_trait, int> = 0>
struct my_class {};
```

The tool will replace the above code with,

```
// warning: use C++20 requires constraints instead of enable_if [modernize-use-constraints]
template <typename T>
int only_if_t_has_the_trait() requires T::some_trait { ... }

// warning: use C++20 requires constraints instead of enable_if [modernize-use-constraints]
template <typename T>
void another_version() requires T::some_trait { ... }

// The tool will emit a diagnostic for the following, but will
// not attempt to replace the code.
// warning: use C++20 requires constraints instead of enable_if [modernize-use-constraints]
template <typename T>
typename std::enable_if<T::some_value, Obj>::type existing_constraint() requires (T::another_value) {
  return Obj{};
}

// The tool will not emit a diagnostic or attempt to replace the code.
template <typename T, std::enable_if_t<T::some_trait, int> = 0>
struct my_class {};
```

### modernize-use-default-member-init

This check converts constructors’ member initializers into the new default member initializers in C++11. Other member initializers that match the default member initializer are removed. This can reduce repeated code or allow use of ‘= default’.

```
struct A {
  A() : i(5), j(10.0) {}
  A(int i) : i(i), j(10.0) {}
  int i;
  double j;
};

// becomes

struct A {
  A() {}
  A(int i) : i(i) {}
  int i{5};
  double j{10.0};
};
```

Note

Only converts member initializers for built-in types, enums, and pointers. The readability-redundant-member-init check will remove redundant member initializers for classes.

#### Options

##### `UseAssignment`

If this option is set to true (default is false), the check will initialize members with an assignment.

For example:

```
struct A {
  A() {}
  A(int i) : i(i) {}
  int i = 5;
  double j = 10.0;
};
```

##### `IgnoreMacros`

If this option is set to true (default is true), the check will not warn about members declared inside macros.

### modernize-use-emplace

The check flags insertions to an STL-style container done by calling the `push_back`, `push`, or `push_front` methods with an explicitly-constructed temporary of the container element type. In this case, the corresponding `emplace` equivalent methods result in less verbose and potentially more efficient code. Right now the check doesn’t support `insert`. It also doesn’t support `insert` functions for associative containers because replacing `insert` with `emplace` may result in [speed regression](https://htmlpreview.github.io/?https://github.com/HowardHinnant/papers/blob/master/insert_vs_emplace.html), but it might get support with some addition flag in the future.

The [`ContainersWithPushBack`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-ContainersWithPushBack), [`ContainersWithPush`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-ContainersWithPush), and [`ContainersWithPushFront`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-ContainersWithPushFront) options are used to specify the container types that support the `push_back`, `push`, and `push_front` operations respectively. The default values for these options are as follows:

- [`ContainersWithPushBack`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-ContainersWithPushBack): `std::vector`, `std::deque`, and `std::list`.
- [`ContainersWithPush`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-ContainersWithPush): `std::stack`, `std::queue`, and `std::priority_queue`.
- [`ContainersWithPushFront`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-ContainersWithPushFront): `std::forward_list`, `std::list`, and `std::deque`.

This check also reports when an `emplace`-like method is improperly used, for example using `emplace_back` while also calling a constructor. This creates a temporary that requires at best a move and at worst a copy. Almost all `emplace`-like functions in the STL are covered by this, with `try_emplace` on `std::map` and `std::unordered_map` being the exception as it behaves slightly differently than all the others. More containers can be added with the [`EmplacyFunctions`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-EmplacyFunctions) option, so long as the container defines a `value_type` type, and the `emplace`-like functions construct a `value_type` object.

Before:

```
std::vector<MyClass> v;
v.push_back(MyClass(21, 37));
v.emplace_back(MyClass(21, 37));

std::vector<std::pair<int, int>> w;

w.push_back(std::pair<int, int>(21, 37));
w.push_back(std::make_pair(21L, 37L));
w.emplace_back(std::make_pair(21L, 37L));
```

After:

```
std::vector<MyClass> v;
v.emplace_back(21, 37);
v.emplace_back(21, 37);

std::vector<std::pair<int, int>> w;
w.emplace_back(21, 37);
w.emplace_back(21L, 37L);
w.emplace_back(21L, 37L);
```

By default, the check is able to remove unnecessary `std::make_pair` and `std::make_tuple` calls from `push_back` calls on containers of `std::pair` and `std::tuple`. Custom tuple-like types can be modified by the [`TupleTypes`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-TupleTypes) option; custom make functions can be modified by the [`TupleMakeFunctions`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-TupleMakeFunctions) option.

The other situation is when we pass arguments that will be converted to a type inside a container.

Before:

```
std::vector<boost::optional<std::string> > v;
v.push_back("abc");
```

After:

```
std::vector<boost::optional<std::string> > v;
v.emplace_back("abc");
```

In some cases the transformation would be valid, but the code wouldn’t be exception safe. In this case the calls of `push_back` won’t be replaced.

```
std::vector<std::unique_ptr<int>> v;
v.push_back(std::unique_ptr<int>(new int(0)));
auto *ptr = new int(1);
v.push_back(std::unique_ptr<int>(ptr));
```

This is because replacing it with `emplace_back` could cause a leak of this pointer if `emplace_back` would throw exception before emplacement (e.g. not enough memory to add a new element).

For more info read item 42 - “Consider emplacement instead of insertion.” of Scott Meyers “Effective Modern C++”.

The default smart pointers that are considered are `std::unique_ptr`, `std::shared_ptr`, `std::auto_ptr`. To specify other smart pointers or other classes use the [`SmartPointers`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-SmartPointers) option.

Check also doesn’t fire if any argument of the constructor call would be:

- a bit-field (bit-fields can’t bind to rvalue/universal reference)
- a `new` expression (to avoid leak)
- if the argument would be converted via derived-to-base cast.

This check requires C++11 or higher to run.

#### Options

##### `ContainersWithPushBack`

Semicolon-separated list of class names of custom containers that support `push_back`.

##### `ContainersWithPush`

Semicolon-separated list of class names of custom containers that support `push`.

##### `ContainersWithPushFront`

Semicolon-separated list of class names of custom containers that support `push_front`.

##### `IgnoreImplicitConstructors`

When true, the check will ignore implicitly constructed arguments of `push_back`, e.g.

```c++
std::vector<std::string> v;
v.push_back("a"); // Ignored when IgnoreImplicitConstructors is `true`.
```

Default is false.

##### `SmartPointers`

Semicolon-separated list of class names of custom smart pointers.

##### `TupleTypes`

Semicolon-separated list of `std::tuple`-like class names.

##### `TupleMakeFunctions`

Semicolon-separated list of `std::make_tuple`-like function names.

Those function calls will be removed from `push_back` calls and turned into `emplace_back`.

##### `EmplacyFunctions`

Semicolon-separated list of containers without their template parameters and some `emplace`-like method of the container. Example: `vector::emplace_back`. Those methods will be checked for improper use and the check will report when a temporary is unnecessarily created.

#### Example

```
std::vector<MyTuple<int, bool, char>> x;
x.push_back(MakeMyTuple(1, false, 'x'));
x.emplace_back(MakeMyTuple(1, false, 'x'));
```

transforms to:

```
std::vector<MyTuple<int, bool, char>> x;
x.emplace_back(1, false, 'x');
x.emplace_back(1, false, 'x');
```

when [`TupleTypes`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-TupleTypes) is set to `MyTuple`, [`TupleMakeFunctions`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-TupleMakeFunctions) is set to `MakeMyTuple`, and [`EmplacyFunctions`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-emplace.html#cmdoption-arg-EmplacyFunctions) is set to `vector::emplace_back`.

### modernize-use-equals-default

This check replaces default bodies of special member functions with `= default;`. The explicitly defaulted function declarations enable more opportunities in optimization, because the compiler might treat explicitly defaulted functions as trivial.

>这个检查用`= default;`代替特殊成员函数的默认体。
>
>显式默认的函数声明为优化提供了更多的机会，因为编译器可能会将显式默认的函数视为trivial。

Note: Move-constructor and move-assignment operator are not supported yet.

#### Options

- `IgnoreMacros`
  - If set to `true`, the check will not give warnings inside macros and will ignore special members with bodies contain macros or preprocessor directives.
    - 如果设置为`true`，检查将不会在宏中给出警告，并且会忽略主体中包含宏或预处理器指令的特殊成员。
  - Default is `true`.

### modernize-use-equals-delete

Identifies unimplemented private special member functions, and recommends using `= delete` for them. Additionally, it recommends relocating any deleted member function from the `private` to the `public` section.

Before the introduction of C++11, the primary method to effectively “erase” a particular function involved declaring it as `private` without providing a definition. This approach would result in either a compiler error (when attempting to call a private function) or a linker error (due to an undefined reference).

However, subsequent to the advent of C++11, a more conventional approach emerged for achieving this purpose. It involves flagging functions as `= delete` and keeping them in the `public` section of the class.

To prevent false positives, this check is only active within a translation unit where all other member functions have been implemented. The check will generate partial fixes by introducing `= delete`, but the user is responsible for manually relocating functions to the `public` section.

```
// Example: bad
class A {
 private:
  A(const A&);
  A& operator=(const A&);
};

// Example: good
class A {
 public:
  A(const A&) = delete;
  A& operator=(const A&) = delete;
};
```

#### Options

##### `IgnoreMacros`

If this option is set to true (default is true), the check will not warn about functions declared inside macros.

### modernize-use-nodiscard

Adds `[[nodiscard]]` attributes (introduced in C++17) to member functions in order to highlight at compile time which return values should not be ignored.

Member functions need to satisfy the following conditions to be considered by this check:

- no `[[nodiscard]]`, `[[noreturn]]`, `__attribute__((warn_unused_result))`, `[[clang::warn_unused_result]]` nor `[[gcc::warn_unused_result]]` attribute,
- non-void return type,
- non-template return types,
- const member function,
- non-variadic functions,
- no non-const reference parameters,
- no pointer parameters,
- no template parameters,
- no template function parameters,
- not be a member of a class with mutable member variables,
- no Lambdas,
- no conversion functions.

Such functions have no means of altering any state or passing values other than via the return type. Unless the member functions are altering state via some external call (e.g. I/O).

#### Example

```
bool empty() const;
bool empty(int i) const;
```

transforms to:

```
[[nodiscard]] bool empty() const;
[[nodiscard]] bool empty(int i) const;
```

#### Options

##### `ReplacementString`

Specifies a macro to use instead of `[[nodiscard]]`.

This is useful when maintaining source code that needs to compile with a pre-C++17 compiler.

#### Example

```
bool empty() const;
bool empty(int i) const;
```

transforms to:

```
NO_DISCARD bool empty() const;
NO_DISCARD bool empty(int i) const;
```

if the [`ReplacementString`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-nodiscard.html#cmdoption-arg-ReplacementString) option is set to NO_DISCARD.

**Note**

If the [`ReplacementString`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-nodiscard.html#cmdoption-arg-ReplacementString) is not a C++ attribute, but instead a macro, then that macro must be defined in scope or the fix-it will not be applied.

Note

For alternative `__attribute__` syntax options to mark functions as `[[nodiscard]]` in non-c++17 source code. See https://clang.llvm.org/docs/AttributeReference.html#nodiscard-warn-unused-result

### modernize-use-noexcept

This check replaces deprecated dynamic exception specifications with the appropriate noexcept specification (introduced in C++11). By default this check will replace `throw()` with `noexcept`, and `throw(<exception>[,...])` or `throw(...)` with `noexcept(false)`.

#### Example

```
void foo() throw();
void bar() throw(int) {}
```

transforms to:

```
void foo() noexcept;
void bar() noexcept(false) {}
```

#### Options

##### `ReplacementString`

Users can use [`ReplacementString`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-nodiscard.html#cmdoption-arg-ReplacementString) to specify a macro to use instead of `noexcept`. This is useful when maintaining source code that uses custom exception specification marking other than `noexcept`. Fix-it hints will only be generated for non-throwing specifications.

##### Example

```
void bar() throw(int);
void foo() throw();
```

transforms to:

```
void bar() throw(int);  // No fix-it generated.
void foo() NOEXCEPT;
```

if the [`ReplacementString`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-nodiscard.html#cmdoption-arg-ReplacementString) option is set to NOEXCEPT.

##### `UseNoexceptFalse`

Enabled by default, disabling will generate fix-it hints that remove throwing dynamic exception specs, e.g., `throw(<something>)`, completely without providing a replacement text, except for destructors and delete operators that are `noexcept(true)` by default.

##### Example

```
void foo() throw(int) {}

struct bar {
  void foobar() throw(int);
  void operator delete(void *ptr) throw(int);
  void operator delete[](void *ptr) throw(int);
  ~bar() throw(int);
}
```

transforms to:

```
void foo() {}

struct bar {
  void foobar();
  void operator delete(void *ptr) noexcept(false);
  void operator delete[](void *ptr) noexcept(false);
  ~bar() noexcept(false);
}
```

if the [`UseNoexceptFalse`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-noexcept.html#cmdoption-arg-UseNoexceptFalse) option is set to false.

### modernize-use-nullptr

The check converts the usage of null pointer constants (e.g. `NULL`, `0`) to use the new C++11 `nullptr` keyword.

#### Example

```
void assignment() {
  char *a = NULL;
  char *b = 0;
  char c = 0;
}

int *ret_ptr() {
  return 0;
}
```

transforms to:

```
void assignment() {
  char *a = nullptr;
  char *b = nullptr;
  char c = 0;
}

int *ret_ptr() {
  return nullptr;
}
```

#### Options

##### `IgnoredTypes`

Semicolon-separated list of regular expressions to match pointer types for which implicit casts will be ignored. Default value: std::_CmpUnspecifiedParam::;^std::__cmp_cat::__unspec.

##### `NullMacros`

Comma-separated list of macro names that will be transformed along with `NULL`. By default this check will only replace the `NULL` macro and will skip any similar user-defined macros.

#### Example

```
#define MY_NULL (void*)0
void assignment() {
  void *p = MY_NULL;
}
```

transforms to:

```
#define MY_NULL NULL
void assignment() {
  int *p = nullptr;
}
```

if the [`NullMacros`](https://clang.llvm.org/extra/clang-tidy/checks/modernize/use-nullptr.html#cmdoption-arg-NullMacros) option is set to `MY_NULL`.

### modernize-use-override

Adds `override` (introduced in C++11) to overridden virtual functions and removes `virtual` from those functions as it is not required.

`virtual` on non base class implementations was used to help indicate to the user that a function was virtual. C++ compilers did not use the presence of this to signify an overridden function.

In C++ 11 `override` and `final` keywords were introduced to allow overridden functions to be marked appropriately. Their presence allows compilers to verify that an overridden function correctly overrides a base class implementation.

This can be useful as compilers can generate a compile time error when:

> - The base class implementation function signature changes.
> - The user has not created the override with the correct signature.

#### Options

##### `IgnoreDestructors`

If set to true, this check will not diagnose destructors.

Default is false.

##### `IgnoreTemplateInstantiations`

If set to true, instructs this check to ignore virtual function overrides that are part of template instantiations.

Default is false.

##### `AllowOverrideAndFinal`

If set to true, this check will not diagnose `override` as redundant with `final`. This is useful when code will be compiled by a compiler with warning/error checking flags requiring `override` explicitly on overridden members, such as `gcc -Wsuggest-override`/`gcc -Werror=suggest-override`.

Default is false.

##### `OverrideSpelling`

Specifies a macro to use instead of `override`.

This is useful when maintaining source code that also needs to compile with a pre-C++11 compiler.

##### `FinalSpelling`

Specifies a macro to use instead of `final`.

This is useful when maintaining source code that also needs to compile with a pre-C++11 compiler.

#### Note

For more information on the use of `override` see https://en.cppreference.com/w/cpp/language/override

### modernize-use-std-print

Converts calls to `printf`, `fprintf`, `absl::PrintF` and `absl::FPrintf` to equivalent calls to C++23’s `std::print` or `std::println` as appropriate, modifying the format string appropriately. The replaced and replacement functions can be customised by configuration options. Each argument that is the result of a call to `std::string::c_str()` and `std::string::data()` will have that now-unnecessary call removed in a similar manner to the readability-redundant-string-cstr check.

In other words, it turns lines like:

```
fprintf(stderr, "The %s is %3d\n", description.c_str(), value);
```

into:

```
std::println(stderr, "The {} is {:3}", description, value);
```

If the ReplacementPrintFunction or ReplacementPrintlnFunction options are left, or assigned to their default values then this check is only enabled with -std=c++23 or later.

The check doesn’t do a bad job, but it’s not perfect. In particular:

- It assumes that the format string is correct for the arguments. If you get any warnings when compiling with -Wformat then misbehaviour is possible.
- At the point that the check runs, the AST contains a single `StringLiteral` for the format string and any macro expansion, token pasting, adjacent string literal concatenation and escaping has been handled. Although it’s possible for the check to automatically put the escapes back, they may not be exactly as they were written (e.g. `"\x0a"` will become `"\n"` and `"ab" "cd"` will become `"abcd"`.) This is helpful since it means that the `PRIx` macros from `<inttypes.h>` are removed correctly.
- It supports field widths, precision, positional arguments, leading zeros, leading `+`, alignment and alternative forms.
- Use of any unsupported flags or specifiers will cause the entire statement to be left alone and a warning to be emitted. Particular unsupported features are:
  - The `%'` flag for thousands separators.
  - The glibc extension `%m`.
- `printf` and similar functions return the number of characters printed. `std::print` does not. This means that any invocations that use the return value will not be converted. Unfortunately this currently includes explicitly-casting to `void`. Deficiencies in this check mean that any invocations inside `GCC` compound statements cannot be converted even if the resulting value is not used.

If conversion would be incomplete or unsafe then the entire invocation will be left unchanged.

If the call is deemed suitable for conversion then:

- `printf`, `fprintf`, `absl::PrintF`, `absl::FPrintF` and any functions specified by the PrintfLikeFunctions option or FprintfLikeFunctions are replaced with the function specified by the ReplacementPrintlnFunction option if the format string ends with `\n` or ReplacementPrintFunction otherwise.
- the format string is rewritten to use the `std::formatter` language. If a `\n` is found at the end of the format string not preceded by `r` then it is removed and ReplacementPrintlnFunction is used rather than ReplacementPrintFunction.
- any arguments that corresponded to `%p` specifiers that `std::formatter` wouldn’t accept are wrapped in a `static_cast` to `const void *`.
- any arguments that corresponded to `%s` specifiers where the argument is of `signed char` or `unsigned char` type are wrapped in a `reinterpret_cast<const char *>`.
- any arguments where the format string and the parameter differ in signedness will be wrapped in an appropriate `static_cast` if StrictMode is enabled.
- any arguments that end in a call to `std::string::c_str()` or `std::string::data()` will have that call removed.

#### Options

##### `StrictMode`

When true, the check will add casts when converting from variadic functions like `printf` and printing signed or unsigned integer types (including fixed-width integer types from `<cstdint>`, `ptrdiff_t`, `size_t` and `ssize_t`) as the opposite signedness to ensure that the output matches that of `printf`. This does not apply when converting from non-variadic functions such as `absl::PrintF` and `fmt::printf`. For example, with StrictMode enabled:`int i = -42; unsigned int u = 0xffffffff; printf("%d %u\n", i, u); `would be converted to:`std::print("{} {}\n", static_cast<unsigned int>(i), static_cast<int>(u)); `to ensure that the output will continue to be the unsigned representation of -42 and the signed representation of 0xffffffff (often 4294967254 and -1 respectively.) When false (which is the default), these casts will not be added which may cause a change in the output.

##### `PrintfLikeFunctions`

A semicolon-separated list of (fully qualified) extra function names to replace, with the requirement that the first parameter contains the printf-style format string and the arguments to be formatted follow immediately afterwards. If neither this option nor FprintfLikeFunctions are set then the default value for this option is printf; absl::PrintF, otherwise it is empty.

##### `FprintfLikeFunctions`

A semicolon-separated list of (fully qualified) extra function names to replace, with the requirement that the first parameter is retained, the second parameter contains the printf-style format string and the arguments to be formatted follow immediately afterwards. If neither this option nor PrintfLikeFunctions are set then the default value for this option is fprintf; absl::FPrintF, otherwise it is empty.

##### `ReplacementPrintFunction`

The function that will be used to replace `printf`, `fprintf` etc. during conversion rather than the default `std::print` when the originalformat string does not end with `\n`. It is expected that the function provides an interface that is compatible with `std::print`. A suitable candidate would be `fmt::print`.

##### `ReplacementPrintlnFunction`

The function that will be used to replace `printf`, `fprintf` etc. during conversion rather than the default `std::println` when the original format string ends with `\n`. It is expected that the function provides an interface that is compatible with `std::println`. A suitable candidate would be `fmt::println`.

##### `PrintHeader`

The header that must be included for the declaration of ReplacementPrintFunction so that a `#include` directive can be added if required. If ReplacementPrintFunction is `std::print` then this option will default to `<print>`, otherwise this option will default to nothing and no `#include` directive will be added.











### modernize-use-trailing-return-type

Rewrites function signatures to use a trailing return type (introduced in C++11). This transformation is purely stylistic. The return type before the function name is replaced by auto and inserted after the function parameter list (and qualifiers).

>重写函数签名以使用尾随返回类型（在C++ 11中引入）。
>
>这种转变纯粹是格式上的。函数名之前的返回类型被`auto`替换，并插入到函数参数列表（和限定符）之后。

Example

```c++
int f1();
inline int f2(int arg) noexcept;
virtual float f3() const && = delete;
```

transforms to:

```c++
auto f1() -> int;
inline auto f2(int arg) -> int noexcept;
virtual auto f3() const && -> float = delete;
```

#### Known Limitations

The following categories of return types cannot be rewritten currently:

- function pointers
- member function pointers
- member pointers

Unqualified names in the return type might erroneously refer to different entities after the rewrite. Preventing such errors requires a full lookup of all unqualified names present in the return type in the scope of the trailing return type location. This location includes e.g. function parameter names and members of the enclosing class (including all inherited classes). Such a lookup is currently not implemented.

>在重写之后，返回类型中的非限定名可能会错误地引用不同的实体。
>
>要防止此类错误，需要在返回类型的末尾返回类型位置的作用域中完整查找返回类型中出现的所有非限定名。这个位置包括函数参数名和外围类的成员（包括所有继承类）。目前还没有实现这样的查找。

Given the following piece of code

```c++
struct S { long long value; };
S f(unsigned S) { return {S * 2}; }
class CC {
  int S;
  struct S m();
};
S CC::m() { return {0}; }
```

a careless rewrite would produce the following output:

```c++
struct S { long long value; };
auto f(unsigned S) -> S { return {S * 2}; } // error
class CC {
  int S;
  auto m() -> struct S;
};
auto CC::m() -> S { return {0}; } // error
```

This code fails to compile because the `S` in the context of f refers to the equally named function parameter. Similarly, the S in the context of m refers to the equally named class member. The check can currently only detect and avoid a clash with a function parameter name.

>这段代码无法编译，因为`f`上下文中的`S`引用了同名的函数参数。类似地，`m`上下文中的`S`指的是同名的类成员。
>
>该检查目前只能检测并避免与函数参数名冲突。

### modernize-use-transparent-functors

Prefer transparent functors to non-transparent ones. When using transparent functors, the type does not need to be repeated. The code is easier to read, maintain and less prone to errors. It is not possible to introduce unwanted conversions.

```
// Non-transparent functor
std::map<int, std::string, std::greater<int>> s;

// Transparent functor.
std::map<int, std::string, std::greater<>> s;

// Non-transparent functor
using MyFunctor = std::less<MyType>;
```

It is not always a safe transformation though. The following case will be untouched to preserve the semantics.

```
// Non-transparent functor
std::map<const char *, std::string, std::greater<std::string>> s;
```

#### Options

##### `SafeMode`

If the option is set to true, the check will not diagnose cases where using a transparent functor cannot be guaranteed to produce identical results as the original code. The default value for this option is false.

This check requires using C++14 or higher to run.

### modernize-use-uncaught-exceptions

This check will warn on calls to `std::uncaught_exception` and replace them with calls to `std::uncaught_exceptions`, since `std::uncaught_exception` was deprecated in C++17.

Below are a few examples of what kind of occurrences will be found and what they will be replaced with.

```
#define MACRO1 std::uncaught_exception
#define MACRO2 std::uncaught_exception

int uncaught_exception() {
  return 0;
}

int main() {
  int res;

  res = uncaught_exception();
  // No warning, since it is not the deprecated function from namespace std

  res = MACRO2();
  // Warning, but will not be replaced

  res = std::uncaught_exception();
  // Warning and replaced

  using std::uncaught_exception;
  // Warning and replaced

  res = uncaught_exception();
  // Warning and replaced
}
```

After applying the fixes the code will look like the following:

```
#define MACRO1 std::uncaught_exception
#define MACRO2 std::uncaught_exception

int uncaught_exception() {
  return 0;
}

int main() {
  int res;

  res = uncaught_exception();

  res = MACRO2();

  res = std::uncaught_exceptions();

  using std::uncaught_exceptions;

  res = uncaught_exceptions();
}
```

### modernize-use-using

The check converts the usage of `typedef` with `using` keyword.

Before:

```
typedef int variable;

class Class{};
typedef void (Class::* MyPtrType)() const;

typedef struct { int a; } R_t, *R_p;
```

After:

```
using variable = int;

class Class{};
using MyPtrType = void (Class::*)() const;

using R_t = struct { int a; };
using R_p = R_t*;
```

This check requires using C++11 or higher to run.

#### Options

##### `IgnoreMacros`

If set to true, the check will not give warnings inside macros.

Default is true.

## performance-*

Checks that target performance-related issues.

### performance-avoid-endl

Checks for uses of `std::endl` on streams and suggests using the newline character `'\n'` instead.

Rationale: Using `std::endl` on streams can be less efficient than using the newline character `'\n'` because `std::endl` performs two operations: it writes a newline character to the output stream and then flushes the stream buffer. Writing a single newline character using `'\n'` does not trigger a flush, which can improve performance. In addition, flushing the stream buffer can cause additional overhead when working with streams that are buffered.

Example:

Consider the following code:

```
#include <iostream>

int main() {
  std::cout << "Hello" << std::endl;
}
```

Which gets transformed into:

```
#include <iostream>

int main() {
  std::cout << "Hello" << '\n';
}
```

This code writes a single newline character to the `std::cout` stream without flushing the stream buffer.

Additionally, it is important to note that the standard C++ streams (like `std::cerr`, `std::wcerr`, `std::clog` and `std::wclog`) always flush after a write operation, unless `std::ios_base::sync_with_stdio` is set to `false`. regardless of whether `std::endl` or `'\n'` is used. Therefore, using `'\n'` with these streams will not result in any performance gain, but it is still recommended to use `'\n'` for consistency and readability.

If you do need to flush the stream buffer, you can use `std::flush` explicitly like this:

```
#include <iostream>

int main() {
  std::cout << "Hello\n" << std::flush;
}
```

### performance-enum-size

Recommends the smallest possible underlying type for an `enum` or `enum` class based on the range of its enumerators. Analyzes the values of the enumerators in an `enum` or `enum` class, including signed values, to recommend the smallest possible underlying type that can represent all the values of the `enum`. The suggested underlying types are the integral types `std::uint8_t`, `std::uint16_t`, and `std::uint32_t` for unsigned types, and `std::int8_t`, `std::int16_t`, and `std::int32_t` for signed types. Using the suggested underlying types can help reduce the memory footprint of the program and improve performance in some cases.

For example:

```
// BEFORE
enum Color {
    RED = -1,
    GREEN = 0,
    BLUE = 1
};

std::optional<Color> color_opt;
```

The Color `enum` uses the default underlying type, which is `int` in this case, and its enumerators have values of -1, 0, and 1. Additionally, the `std::optional<Color>` object uses 8 bytes due to padding (platform dependent).

```
// AFTER
enum Color : std:int8_t {
    RED = -1,
    GREEN = 0,
    BLUE = 1
}

std::optional<Color> color_opt;
```

In the revised version of the Color `enum`, the underlying type has been changed to `std::int8_t`. The enumerator RED has a value of -1, which can be represented by a signed 8-bit integer.

By using a smaller underlying type, the memory footprint of the Color `enum` is reduced from 4 bytes to 1 byte. The revised version of the `std::optional<Color>` object would only require 2 bytes (due to lack of padding), since it contains a single byte for the Color `enum` and a single byte for the `bool` flag that indicates whether the optional value is set.

Reducing the memory footprint of an `enum` can have significant benefits in terms of memory usage and cache performance. However, it’s important to consider the trade-offs and potential impact on code readability and maintainability.

Enums without enumerators (empty) are excluded from analysis.

Requires C++11 or above. Does not provide auto-fixes.

#### Options

##### `EnumIgnoreList`

Option is used to ignore certain enum types. It accepts a semicolon-separated list of (fully qualified) enum type names or regular expressions that match the enum type names.

The default value is an empty string, which means no enums will be ignored.

### performance-faster-string-find

Optimize calls to `std::string::find()` and friends when the needle passed is a single character string literal. The character literal overload is more efficient.

Examples:

```
str.find("A");

// becomes

str.find('A');
```

#### Options

##### `StringLikeClasses`

Semicolon-separated list of names of string-like classes.

By default only `::std::basic_string` and `::std::basic_string_view` are considered.

The check will only consider member functions named `find`, `rfind`, `find_first_of`, `find_first_not_of`, `find_last_of`, or `find_last_not_of` within these classes.

### performance-for-range-copy

Finds C++11 for ranges where the loop variable is copied in each iteration but it would suffice to obtain it by const reference.

The check is only applied to loop variables of types that are expensive to copy which means they are not trivially copyable or have a non-trivial copy constructor or destructor.

To ensure that it is safe to replace the copy with a const reference the following heuristic is employed:

1. The loop variable is const qualified.
2. The loop variable is not const, but only const methods or operators are invoked on it, or it is used as const reference or value argument in constructors or function calls.

#### Options

##### `WarnOnAllAutoCopies`

When true, warns on any use of auto as the type of the range-based for loop variable.

Default is false.

##### `AllowedTypes`

A semicolon-separated list of names of types allowed to be copied in each iteration. Regular expressions are accepted, e.g. [Rr]ef(erence)?$ matches every type with suffix Ref, ref, Reference and reference.

The default is empty.

If a name in the list contains the sequence :: it is matched against the qualified typename (i.e. namespace::Type, otherwise it is matched against only the type name (i.e. Type).

### performance-implicit-conversion-in-loop

This warning appears in a range-based loop with a loop variable of const ref type where the type of the variable does not match the one returned by the iterator. This means that an implicit conversion happens, which can for example result in expensive deep copies.

Example:

```
map<int, vector<string>> my_map;
for (const pair<int, vector<string>>& p : my_map) {}
// The iterator type is in fact pair<const int, vector<string>>, which means
// that the compiler added a conversion, resulting in a copy of the vectors.
```

The easiest solution is usually to use `const auto&` instead of writing the type manually.

### performance-inefficient-algorithm

Warns on inefficient use of STL algorithms on associative containers.

Associative containers implement some of the algorithms as methods which should be preferred to the algorithms in the algorithm header. The methods can take advantage of the order of the elements.

```c++
std::set<int> s;
auto it = std::find(s.begin(), s.end(), 43);

// becomes

auto it = s.find(43);
```

```c++
std::set<int> s;
auto c = std::count(s.begin(), s.end(), 43);

// becomes

auto c = s.count(43);
```

### performance-inefficient-string-concatenation

This check warns about the performance overhead arising from concatenating strings using the `operator+`, for instance:

```
std::string a("Foo"), b("Bar");
a = a + b;
```

Instead of this structure you should use `operator+=` or `std::string`’s (`std::basic_string`) class member function `append()`. For instance:

```
std::string a("Foo"), b("Baz");
for (int i = 0; i < 20000; ++i) {
    a = a + "Bar" + b;
}
```

Could be rewritten in a greatly more efficient way like:

```
std::string a("Foo"), b("Baz");
for (int i = 0; i < 20000; ++i) {
    a.append("Bar").append(b);
}
```

And this can be rewritten too:

```
void f(const std::string&) {}
std::string a("Foo"), b("Baz");
void g() {
    f(a + "Bar" + b);
}
```

In a slightly more efficient way like:

```
void f(const std::string&) {}
std::string a("Foo"), b("Baz");
void g() {
    f(std::string(a).append("Bar").append(b));
}
```

#### Options

##### `StrictMode`

When false, the check will only check the string usage in `while`, `for` and `for-range` statements.

Default is false.

### performance-inefficient-vector-operation

Finds possible inefficient `std::vector` operations (e.g. `push_back`, `emplace_back`) that may cause unnecessary memory reallocations.

It can also find calls that add element to protobuf repeated field in a loop without calling Reserve() before the loop. Calling Reserve() first can avoid unnecessary memory reallocations.

Currently, the check only detects following kinds of loops with a single statement body:

- Counter-based for loops start with 0:

```
std::vector<int> v;
for (int i = 0; i < n; ++i) {
  v.push_back(n);
  // This will trigger the warning since the push_back may cause multiple
  // memory reallocations in v. This can be avoid by inserting a 'reserve(n)'
  // statement before the for statement.
}

SomeProto p;
for (int i = 0; i < n; ++i) {
  p.add_xxx(n);
  // This will trigger the warning since the add_xxx may cause multiple memory
  // reallocations. This can be avoid by inserting a
  // 'p.mutable_xxx().Reserve(n)' statement before the for statement.
}
```

- For-range loops like `for (range-declaration : range_expression)`, the type of `range_expression` can be `std::vector`, `std::array`, `std::deque`, `std::set`, `std::unordered_set`, `std::map`, `std::unordered_set`:

```
std::vector<int> data;
std::vector<int> v;

for (auto element : data) {
  v.push_back(element);
  // This will trigger the warning since the 'push_back' may cause multiple
  // memory reallocations in v. This can be avoid by inserting a
  // 'reserve(data.size())' statement before the for statement.
}
```

#### Options

##### `VectorLikeClasses`

Semicolon-separated list of names of vector-like classes.

By default only `::std::vector` is considered.

##### `EnableProto`

When true, the check will also warn on inefficient operations for proto repeated fields.

Otherwise, the check only warns on inefficient vector operations.

Default is false.

### performance-move-const-arg

The check warns

- if `std::move()` is called with a constant argument,
- if `std::move()` is called with an argument of a trivially-copyable type,
- if the result of `std::move()` is passed as a const reference argument.

In all three cases, the check will suggest a fix that removes the `std::move()`.

Here are examples of each of the three cases:

```
const string s;
return std::move(s);  // Warning: std::move of the const variable has no effect

int x;
return std::move(x);  // Warning: std::move of the variable of a trivially-copyable type has no effect

void f(const string &s);
string s;
f(std::move(s));  // Warning: passing result of std::move as a const reference argument; no move will actually happen
```

#### Options

##### `CheckTriviallyCopyableMove`

If true, enables detection of trivially copyable types that do not have a move constructor.

Default is true.

##### `CheckMoveToConstRef`

If true, enables detection of std::move() passed as a const reference argument.

Default is true.

### performance-move-constructor-init

“cert-oop11-cpp” redirects here as an alias for this check.

The check flags user-defined move constructors that have a ctor-initializer initializing a member or base class through a copy constructor instead of a move constructor.

### performance-no-automatic-move

Finds local variables that cannot be automatically moved due to constness.

Under [certain conditions](https://en.cppreference.com/w/cpp/language/return#automatic_move_from_local_variables_and_parameters), local values are automatically moved out when returning from a function. A common mistake is to declare local `lvalue` variables `const`, which prevents the move.

Example [[1\]](https://godbolt.org/z/x7SYYA):

```
StatusOr<std::vector<int>> Cool() {
  std::vector<int> obj = ...;
  return obj;  // calls StatusOr::StatusOr(std::vector<int>&&)
}

StatusOr<std::vector<int>> NotCool() {
  const std::vector<int> obj = ...;
  return obj;  // calls `StatusOr::StatusOr(const std::vector<int>&)`
}
```

The former version (`Cool`) should be preferred over the latter (`NotCool`) as it will avoid allocations and potentially large memory copies.

#### Semantics

In the example above, `StatusOr::StatusOr(T&&)` have the same semantics as long as the copy and move constructors for `T` have the same semantics. Note that there is no guarantee that `S::S(T&&)` and `S::S(const T&)` have the same semantics for any single `S`, so we’re not providing automated fixes for this check, and judgement should be exerted when making the suggested changes.

#### -Wreturn-std-move

Another case where the move cannot happen is the following:

```
StatusOr<std::vector<int>> Uncool() {
  std::vector<int>&& obj = ...;
  return obj;  // calls `StatusOr::StatusOr(const std::vector<int>&)`
}
```

In that case the fix is more consensual: just return std::move(obj). This is handled by the -Wreturn-std-move warning.

### performance-no-int-to-ptr

Diagnoses every integer to pointer cast.

While casting an (integral) pointer to an integer is obvious - you just get the integral value of the pointer, casting an integer to an (integral) pointer is deceivingly different. While you will get a pointer with that integral value, if you got that integral value via a pointer-to-integer cast originally, the new pointer will lack the provenance information from the original pointer.

So while (integral) pointer to integer casts are effectively no-ops, and are transparent to the optimizer, integer to (integral) pointer casts are *NOT* transparent, and may conceal information from optimizer.

While that may be the intention, it is not always so. For example, let’s take a look at a routine to align the pointer up to the multiple of 16: The obvious, naive implementation for that is:

```
char* src(char* maybe_underbiased_ptr) {
  uintptr_t maybe_underbiased_intptr = (uintptr_t)maybe_underbiased_ptr;
  uintptr_t aligned_biased_intptr = maybe_underbiased_intptr + 15;
  uintptr_t aligned_intptr = aligned_biased_intptr & (~15);
  return (char*)aligned_intptr; // warning: avoid integer to pointer casts [performance-no-int-to-ptr]
}
```

The check will rightfully diagnose that cast.

But when provenance concealment is not the goal of the code, but an accident, this example can be rewritten as follows, without using integer to pointer cast:

```
char*
tgt(char* maybe_underbiased_ptr) {
    uintptr_t maybe_underbiased_intptr = (uintptr_t)maybe_underbiased_ptr;
    uintptr_t aligned_biased_intptr = maybe_underbiased_intptr + 15;
    uintptr_t aligned_intptr = aligned_biased_intptr & (~15);
    uintptr_t bias = aligned_intptr - maybe_underbiased_intptr;
    return maybe_underbiased_ptr + bias;
}
```

### performance-noexcept-destructor

The check flags user-defined destructors marked with `noexcept(expr)` where `expr` evaluates to `false` (but is not a `false` literal itself).

When a destructor is marked as `noexcept`, it assures the compiler that no exceptions will be thrown during the destruction of an object, which allows the compiler to perform certain optimizations such as omitting exception handling code.

### performance-noexcept-move-constructor

The check flags user-defined move constructors and assignment operators not marked with `noexcept` or marked with `noexcept(expr)` where `expr` evaluates to `false` (but is not a `false` literal itself).

Move constructors of all the types used with STL containers, for example, need to be declared `noexcept`. Otherwise STL will choose copy constructors instead. The same is valid for move assignment operations.

### performance-noexcept-swap

The check flags user-defined swap functions not marked with `noexcept` or marked with `noexcept(expr)` where `expr` evaluates to `false` (but is not a `false` literal itself).

When a swap function is marked as `noexcept`, it assures the compiler that no exceptions will be thrown during the swapping of two objects, which allows the compiler to perform certain optimizations such as omitting exception handling code.

### performance-trivially-destructible

Finds types that could be made trivially-destructible by removing out-of-line defaulted destructor declarations.

```
struct A: TrivialType {
  ~A(); // Makes A non-trivially-destructible.
  TrivialType trivial_fields;
};
A::~A() = default;
```

### performance-type-promotion-in-math-fn

Finds calls to C math library functions (from `math.h` or, in C++, `cmath`) with implicit `float` to `double` promotions.

For example, warns on `::sin(0.f)`, because this function’s parameter is a double. You probably meant to call `std::sin(0.f)` (in C++), or `sinf(0.f)` (in C).

```
float a;
asin(a);

// becomes

float a;
std::asin(a);
```

### performance-unnecessary-copy-initialization

Finds local variable declarations that are initialized using the copy constructor of a non-trivially-copyable type but it would suffice to obtain a const reference.

The check is only applied if it is safe to replace the copy by a const reference. This is the case when the variable is const qualified or when it is only used as a const, i.e. only const methods or operators are invoked on it, or it is used as const reference or value argument in constructors or function calls.

Example:

```
const string& constReference();
void Function() {
  // The warning will suggest making this a const reference.
  const string UnnecessaryCopy = constReference();
}

struct Foo {
  const string& name() const;
};
void Function(const Foo& foo) {
  // The warning will suggest making this a const reference.
  string UnnecessaryCopy1 = foo.name();
  UnnecessaryCopy1.find("bar");

  // The warning will suggest making this a const reference.
  string UnnecessaryCopy2 = UnnecessaryCopy1;
  UnnecessaryCopy2.find("bar");
}
```

#### Options

##### `AllowedTypes`

A semicolon-separated list of names of types allowed to be initialized by copying. Regular expressions are accepted, e.g. [Rr]ef(erence)?$ matches every type with suffix Ref, ref, Reference and reference. The default is empty. If a name in the list contains the sequence :: it is matched against the qualified typename (i.e. namespace::Type, otherwise it is matched against only the type name (i.e. Type).

##### `ExcludedContainerTypes`

A semicolon-separated list of names of types whose methods are allowed to return the const reference the variable is copied from. When an expensive to copy variable is copy initialized by the return value from a type on this list the check does not trigger. This can be used to exclude types known to be const incorrect or where the lifetime or immutability of returned references is not tied to mutations of the container. An example are view types that don’t own the underlying data. Like for AllowedTypes above, regular expressions are accepted and the inclusion of :: determines whether the qualified typename is matched or not.

### performance-unnecessary-value-param

Flags value parameter declarations of expensive to copy types that are copied for each invocation but it would suffice to pass them by const reference.

The check is only applied to parameters of types that are expensive to copy which means they are not trivially copyable or have a non-trivial copy constructor or destructor.

To ensure that it is safe to replace the value parameter with a const reference the following heuristic is employed:

1. the parameter is const qualified;
2. the parameter is not const, but only const methods or operators are invoked on it, or it is used as const reference or value argument in constructors or function calls.

Example:

```
void f(const string Value) {
  // The warning will suggest making Value a reference.
}

void g(ExpensiveToCopy Value) {
  // The warning will suggest making Value a const reference.
  Value.ConstMethd();
  ExpensiveToCopy Copy(Value);
}
```

If the parameter is not const, only copied or assigned once and has a non-trivial move-constructor or move-assignment operator respectively the check will suggest to move it.

Example:

```
void setValue(string Value) {
  Field = Value;
}
```

Will become:

```
#include <utility>

void setValue(string Value) {
  Field = std::move(Value);
}
```

#### Options

##### `IncludeStyle`

A string specifying which include-style is used, llvm or google.

Default is llvm.

##### `AllowedTypes`

A semicolon-separated list of names of types allowed to be passed by value. Regular expressions are accepted, e.g. [Rr]ef(erence)?$ matches every type with suffix Ref, ref, Reference and reference.

The default is empty.

If a name in the list contains the sequence :: it is matched against the qualified typename (i.e. namespace::Type, otherwise it is matched against only the type name (i.e. Type).





## portability-*

Checks that target portability-related issues that don’t relate to any particular coding style.

>针对与任何特定编码风格无关的可移植性相关问题的检查。

### portability-restrict-system-includes

Checks to selectively allow or disallow a configurable list of system headers.

For example:

In order to **only** allow zlib.h from the system you would set the options to -*,zlib.h.

```
#include <curses.h>       // Bad: disallowed system header.
#include <openssl/ssl.h>  // Bad: disallowed system header.
#include <zlib.h>         // Good: allowed system header.
#include "src/myfile.h"   // Good: non-system header always allowed.
```

In order to allow everything **except** zlib.h from the system you would set the options to *,-zlib.h.

```
#include <curses.h>       // Good: allowed system header.
#include <openssl/ssl.h>  // Good: allowed system header.
#include <zlib.h>         // Bad: disallowed system header.
#include "src/myfile.h"   // Good: non-system header always allowed.
```

Since the options support globbing you can use wildcarding to allow groups of headers.

-*,openssl/*.h will allow all openssl headers but disallow any others.

```
#include <curses.h>       // Bad: disallowed system header.
#include <openssl/ssl.h>  // Good: allowed system header.
#include <openssl/rsa.h>  // Good: allowed system header.
#include <zlib.h>         // Bad: disallowed system header.
#include "src/myfile.h"   // Good: non-system header always allowed.
```

#### Options

##### `Includes`

A string containing a comma separated glob list of allowed include filenames. Similar to the -checks glob list for running clang-tidy itself, the two wildcard characters are * and -, to include and exclude globs, respectively.

The default is `*`, which allows all includes.

### portability-simd-intrinsics

Finds SIMD intrinsics calls and suggests `std::experimental::simd` ([P0214](https://wg21.link/p0214)) alternatives.

If the option [`Suggest`](https://clang.llvm.org/extra/clang-tidy/checks/portability/simd-intrinsics.html#cmdoption-arg-Suggest) is set to true, for

```
_mm_add_epi32(a, b); // x86
vec_add(a, b);       // Power
```

the check suggests an alternative: `operator+` on `std::experimental::simd` objects.

Otherwise, it just complains the intrinsics are non-portable (and there are [P0214](https://wg21.link/p0214) alternatives).

Many architectures provide SIMD operations (e.g. x86 SSE/AVX, Power AltiVec/VSX, ARM NEON). It is common that SIMD code implementing the same algorithm, is written in multiple target-dispatching pieces to optimize for different architectures or micro-architectures.

The C++ standard proposal [P0214](https://wg21.link/p0214) and its extensions cover many common SIMD operations. By migrating from target-dependent intrinsics to [P0214](https://wg21.link/p0214) operations, the SIMD code can be simplified and pieces for different targets can be unified.

Refer to [P0214](https://wg21.link/p0214) for introduction and motivation for the data-parallel standard library.

#### Options

##### `Suggest`

If this option is set to true (default is false), the check will suggest [P0214](https://wg21.link/p0214) alternatives, otherwise it only points out the intrinsic function is non-portable.

##### `Std`

The namespace used to suggest [P0214](https://wg21.link/p0214) alternatives.

If not specified, std:: for -std=c++20 and std::experimental:: for -std=c++11.

### portability-std-allocator-const

Report use of `std::vector<const T>` (and similar containers of const elements). These are not allowed in standard C++, and should usually be `std::vector<T>` instead.”

Per C++ `[allocator.requirements.general]`: “T is any cv-unqualified object type”, `std::allocator<const T>` is undefined. Many standard containers use `std::allocator` by default and therefore their `const T` instantiations are undefined.

libc++ defines `std::allocator<const T>` as an extension which will be removed in the future.

libstdc++ and MSVC do not support `std::allocator<const T>`:

```c++
// libstdc++ has a better diagnostic since https://gcc.gnu.org/bugzilla/show_bug.cgi?id=48101
std::deque<const int> deque; // error: static assertion failed: std::deque must have a non-const, non-volatile value_type
std::set<const int> set; // error: static assertion failed: std::set must have a non-const, non-volatile value_type
std::vector<int* const> vector; // error: static assertion failed: std::vector must have a non-const, non-volatile value_type

// MSVC
// error C2338: static_assert failed: 'The C++ Standard forbids containers of const elements because allocator<const T> is ill-formed.'
```

Code bases only compiled with libc++ may accrue such undefined usage. This check finds such code and prevents backsliding while clean-up is ongoing.

## readability-*

Checks that target readability-related issues that don’t relate to any particular coding style.

>针对与任何特定编码风格无关的与可读性相关的问题进行检查。

### readability-convert-member-functions-to-static

Finds non-static member functions that can be made `static` because the functions don’t use `this`.

>查找可以设置为`static`的非静态成员函数，因为这些函数不使用`this`。

After applying modifications as suggested by the check, running the check again might find more opportunities to mark member functions `static`.

>在按照检查建议应用修改后，再次运行检查可能会发现更多将成员函数标记为`static`的机会。

After making a member function `static`, you might want to run the check `readability-static-accessed-through-instance` to replace calls like `Instance.method()` by `Class::method()`.

>在将成员函数设置为`static`之后，您可能需要运行检查`readable -static-accessed-through-instance`，以`Class::method()`取代`Instance.method()`之类的调用。

### readability-named-parameter

Find functions with unnamed arguments.

>查找带有未命名参数的函数。

The check implements the following rule originating in the [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html#Function_Declarations_and_Definitions).

All parameters should be named, with identical names in the declaration and implementation.

>所有参数都应该命名，在声明和实现中使用相同的名称。

Corresponding cpplint.py check name: readability/function.

### readability-qualified-auto

Adds pointer qualifications to `auto`-typed variables that are deduced to pointers.

>将指针限定条件添加到演绎为指针的`auto`类型变量。

[LLVM Coding Standards](https://llvm.org/docs/CodingStandards.html#beware-unnecessary-copies-with-auto) advises to make it obvious if a `auto` typed variable is a pointer.

This check will transform `auto` to `auto *` when the type is deduced to be a pointer.

>当类型被推断为指针时，此检查将把`auto`转换为`auto *`。

```c++
for (auto Data : MutatablePtrContainer) {
  change(*Data);
}

for (auto Data : ConstantPtrContainer) {
  observe(*Data);
}
```

Would be transformed into:

```c++
for (auto *Data : MutatablePtrContainer) {
  change(*Data);
}

for (const auto *Data : ConstantPtrContainer) {
  observe(*Data);
}
```

Note `const` `volatile` qualified types will retain their `const` and `volatile` qualifiers. Pointers to pointers will not be fully qualified.

>注意`const` `volatile`限定类型将保留它们的`const`和`volatile`限定符。
>
>指向指针的指针将不是完全限定的。

```c++
   const auto Foo    = cast<int *>      (Baz1);
   const auto Bar    = cast<const int *>(Baz2);
volatile auto FooBar = cast<int *>      (Baz3);
         auto BarFoo = cast<int **>     (Baz4);
```

Would be transformed into:

```c++
      auto *const    Foo    = cast<int *>      (Baz1);
const auto *const    Bar    = cast<const int *>(Baz2);
      auto *volatile FooBar = cast<int *>      (Baz3);
      auto *         BarFoo = cast<int **>     (Baz4);
```

#### Options

`AddConstToQualified`

When set to `true`, the check will add `const` qualifiers variables defined as `auto *` or `auto &` when applicable.

Default value is `true`.

>当设置为`true`时，检查将为`auto *`或`auto &`变量添加`const`限定符。
>
>默认值为`true`。

```c++
auto  Foo1 = cast<const int *>(Bar1);
auto *Foo2 = cast<const int *>(Bar2);
auto &Foo3 = cast<const int &>(Bar3);
```

If `AddConstToQualified` is set to `false`, it will be transformed into:

```c++
const auto *Foo1 = cast<const int *>(Bar1);
      auto *Foo2 = cast<const int *>(Bar2);
      auto &Foo3 = cast<const int &>(Bar3);
```

Otherwise it will be transformed into:

```c++
const auto *Foo1 = cast<const int *>(Bar1);
const auto *Foo2 = cast<const int *>(Bar2);
const auto &Foo3 = cast<const int &>(Bar3);
```

Note in the LLVM alias, the default value is `false`.

### readability-static-accessed-through-instance

Checks for member expressions that access static members through instances, and replaces them with uses of the appropriate qualified-id.

>检查通过实例访问静态成员的成员表达式，并用适当的限定id替换它们。

Example:

The following code:

```c++
struct C {
    static void foo();
    static int x;
    enum { E1 };
    enum E { E2 };
};

C *c1 = new C();
c1->foo();
c1->x;
c1->E1;
c1->E2;
```

is changed to:

```c++
C *c1 = new C();
C::foo();
C::x;
C::E1;
C::E2;
```

## Reference

[1] [Clang-Tidy Checks](https://clang.llvm.org/extra/clang-tidy/checks/list.html)

[2] [Class template argument deduction (CTAD)](https://en.cppreference.com/w/cpp/language/class_template_argument_deduction)

[3] [POSIX Safety Concepts](https://www.gnu.org/software/libc/manual/html_node/POSIX-Safety-Concepts.html)

[4] [POS47-C. Do not use threads that can be canceled asynchronously](https://wiki.sei.cmu.edu/confluence/display/c/POS47-C.+Do+not+use+threads+that+can+be+canceled+asynchronously)

[5] [C++ Core Guidelines C.12](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-constref)

[6] [Data members: Never const](https://quuxplusone.github.io/blog/2022/01/23/dont-const-all-the-things/#data-members-never-const)

[7] [C++ Core Guidelines ES.75](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-do)

[8] [C++ Core Guidelines ES.76](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es76-avoid-goto)

[9] [High Integrity C++ Coding Standard 6.3.1](https://www.perforce.com/resources/qac/high-integrity-cpp-coding-standard/statements)

[10] [A Case against the GO TO Statement](https://www.cs.utexas.edu/users/EWD/ewd02xx/EWD215.PDF)

[11] [C++ Core Guidelines CP.53](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rcoro-reference-parameters)

[12] [C++ Core Guidelines ES.20](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-always)

[13] [C++ Core Guidelines I.22](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ri-global-init)
