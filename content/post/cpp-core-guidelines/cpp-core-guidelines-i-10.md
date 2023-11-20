---
title: "C++ Core Guidelines I.10 注解"
date: 2023-11-19T00:41:11+08:00
description: "C++ Core Guidelines I.10 注解"
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

# I.10: Use exceptions to signal a failure to perform a required task

>使用异常来表示执行所需任务失败。

## Reason

It should not be possible to ignore an error because that could leave the system or a computation in an undefined (or unexpected) state. This is a major source of errors.

>不应该忽略错误，因为这可能使系统或计算处于未定义（或意外）状态。这是错误的主要来源。

## Example

### Bad

```c++
int printf(const char* ...);	// bad: return negative number if output fails
```

### Good

```c++
template<class F, class ...Args>
explicit thread(F&& f, Args&&... args); // good: throw system_error if unable to start the new thread
```

## Note

What is an error?

An error means that the function cannot achieve its advertised purpose (including establishing postconditions). Calling code that ignores an error could lead to wrong results or undefined systems state. For example, not being able to connect to a remote server is not by itself an error: the server can refuse a connection for all kinds of reasons, so the natural thing is to return a result that the caller should always check. However, if failing to make a connection is considered an error, then a failure should throw an exception.

>错误意味着函数不能达到它宣称的目的（包括建立后置条件）。调用忽略错误的代码可能导致错误的结果或未定义的系统状态。
>
>例如，无法连接到远程服务器本身并不是错误，服务器可以出于各种原因拒绝连接，因此返回一个调用者应该始终检查的结果是很自然的事情。但是，如果连接失败被认为是错误，则失败应该抛出异常。

## Exception

Many traditional interface functions (e.g., UNIX signal handlers) use error codes (e.g., `errno`) to report what are really status codes, rather than errors. You don’t have a good alternative to using such, so calling these does not violate the rule.

> 许多传统的接口函数（例如，UNIX信号处理程序）使用错误代码（例如，`errno`）来报告真正的状态代码，而不是错误。
>
> 你没有很好的替代方法，因此调用这些并不违反规则。

## Alternative

If you can’t use exceptions (e.g., because your code is full of old-style raw-pointer use or because there are hard-real-time constraints), consider using a style that returns a pair of values:

```c++
int val;
int error_code;
tie(val, error_code) = do_something();
if (error_code) {
    // ... handle the error or exit ...
}
// ... use val ...
```

This style unfortunately leads to uninitialized variables.

Since C++17 the “**structured bindings**” feature can be used to initialize variables directly from the return value:

```c++
auto [val, error_code] = do_something();
if (error_code) {
    // ... handle the error or exit ...
}
// ... use val ...
```

## Note

We don’t consider “performance” a valid reason not to use exceptions.

>我们不认为“性能”是不使用异常的正当理由。

- Often, explicit error checking and handling consume as much time and space as exception handling.

  - > 通常，显式错误检查和处理消耗的时间和空间与异常处理一样多。

- Often, cleaner code yields better performance with exceptions (simplifying the tracing of paths through the program and their optimization).

  - > 通常情况下，更简洁的代码会产生更好的异常性能（简化程序中的路径跟踪及其优化）。

- A good rule for performance critical code is to move checking outside the critical part of the code.

  - > 对于性能关键代码，一个好的规则是将检查移到代码的关键部分之外。

- In the longer term, more regular code gets better optimized.

  - > 从长远来看，更常规的代码会得到更好的优化。

- Always carefully measure before making performance claims.

  - 在做出性能声明之前，一定要仔细测量。

## Reference

[1] [C++ Core Guidelines I.10: Use exceptions to signal a failure to perform a required task](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#i10-use-exceptions-to-signal-a-failure-to-perform-a-required-task)
