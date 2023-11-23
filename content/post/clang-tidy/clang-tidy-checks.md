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

> 注：个人认为，Clang-Tidy是C++最佳实践之集大成者，规则众多，文档清晰，实践性强，易于使用，还支持定制规则，十分强大。

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

## bugprone-*

Checks that target bug-prone code constructs.

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

## modernize-*

Checks that advocate usage of modern (currently “modern” means “C++11”) language constructs.

>支持使用现代（目前“现代”指的是“C++ 11”及以后版本）语言结构的检查。

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

## performance-*

Checks that target performance-related issues.

## portability-*

Checks that target portability-related issues that don’t relate to any particular coding style.

>针对与任何特定编码风格无关的可移植性相关问题的检查。

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
