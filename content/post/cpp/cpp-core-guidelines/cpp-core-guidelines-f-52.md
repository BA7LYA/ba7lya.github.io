---
title: "C++ Core Guidelines F.52 注解"
date: 2023-12-22T21:56:10+08:00
description: "C++ Core Guidelines F.52 注解"
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

# F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms

## Reason

For efficiency and correctness, you nearly always want to capture by reference when using the lambda locally. This includes when writing or calling parallel algorithms that are local because they join before returning.

为了提高效率和正确性，在本地使用lambda时，几乎总是希望通过引用进行捕获。

这包括在编写或调用本地并行算法时，因为它们在返回之前连接在一起。

## Discussion

The efficiency consideration is that most types are cheaper to pass by reference than by value.

效率方面的考虑是，大多数类型通过引用传递比通过值传递更便宜。

The correctness consideration is that many calls want to perform side effects on the original object at the call site (see example below). Passing by value prevents this.

正确性方面的考虑是，许多调用都希望对调用站点上的原始对象执行副作用（见下面的示例）。按值传递可以防止这种情况。

## Note

Unfortunately, there is no simple way to capture by reference to `const` to get the efficiency for a local call but also prevent side effects.

不幸的是，没有简单的方法可以通过引用`const`来获取本地调用的效率，同时还可以防止副作用。

## Example

Here, a large object (a network message) is passed to an iterative algorithm, and it is not efficient or correct to copy the message (which might not be copyable):

这里，一个大对象（网络消息）被传递给迭代算法，复制消息（可能不可复制）是不高效或不正确的：

```c++
std::for_each(
    begin(sockets),
    end(sockets),
    [&message](auto& socket) {
        socket.send(message);
    }
);
```

## Example

This is a simple three-stage parallel pipeline. Each `stage` object encapsulates a worker thread and a queue, has a `process` function to enqueue work, and in its destructor automatically blocks waiting for the queue to empty before ending the thread.

这是一个简单的三级并行管道。每个`stage`对象封装了一个工作线程和一个队列，有一个`process`函数来对工作进行排队，并在其析构函数中自动阻塞等待队列清空，然后才结束线程。

```c++
void send_packets(buffers& bufs)
{
    stage encryptor([](buffer& b) { encrypt(b); });
    stage compressor([&](buffer& b) { compress(b); encryptor.process(b); });
    stage decorator([&](buffer& b) { decorate(b); compressor.process(b); });
    for (auto& b : bufs) { decorator.process(b); }
}  // automatically blocks waiting for pipeline to finish
```

## Enforcement

Flag a lambda that captures by reference, but is used other than locally within the function scope or passed to a function by reference. (Note: This rule is an approximation, but does flag passing by pointer as those are more likely to be stored by the callee, writing to a heap location accessed via a parameter, returning the lambda, etc. The Lifetime rules will also provide general rules that flag escaping pointers and references including via lambdas.)
