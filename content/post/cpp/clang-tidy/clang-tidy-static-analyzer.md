---
title: "Clang-Tidy Static Analyzer"
date: 2023-12-22T14:58:27+08:00
description: "Clang-Tidy Static Analyzer"
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

The analyzer performs checks that are categorized into families or “checkers”.

The default set of checkers covers a variety of checks targeted at finding security and API usage bugs, dead code, and other logic errors. See the Default Checkers checkers list below.

默认的检查器集涵盖了各种旨在查找安全性和API使用错误、死代码和其他逻辑错误的检查。

In addition to these, the analyzer contains a number of Experimental Checkers (aka alpha checkers). These checkers are under development and are switched off by default. They may crash or emit a higher number of false positives.

除此之外，分析器还包含许多实验检查器（又名Alpha检查器）。

这些检查器正在开发中，默认情况下是关闭的。它们可能会崩溃或产生更多的误报。

The debug package contains checkers for analyzer developers for debugging purposes.

# Default Checkers

## `core`

Models core language features and contains general-purpose checkers such as division by zero, null pointer dereference, usage of uninitialized values, etc. These checkers must be always switched on as other checker rely on them.

对核心语言特性进行建模，并包含通用检查器，如除零、空指针解引用、未初始化值的使用等。

当其他检查器依赖这些检查器时，必须始终打开这些检查器。

### `core.BitwiseShift` (C, C++)

Finds undefined behavior caused by the bitwise left- and right-shift operator operating on integer types.

查找由在整数类型上操作的按位左移和右移操作符引起的未定义行为。

By default, this checker only reports situations when the right operand is either negative or larger than the bit width of the type of the left operand; these are logically unsound.

默认情况下，此检查器仅报告右操作数为负或大于左操作数类型的位宽的情况，这些在逻辑上是站不住脚的。

Moreover, if the pedantic mode is activated by `-analyzer-config core.BitwiseShift:Pedantic=true`, then this checker also reports situations where the `_left_` operand of a shift operator is negative or overflow occurs during the right shift of a signed value. (Most compilers handle these predictably, but the C standard and the C++ standards before C++20 say that they’re undefined behavior. In the C++20 standard these constructs are well-defined, so activating pedantic mode in C++20 has no effect.)

此外，如果通过`-analyzer-config core.bitwisshift:Pedantic=true`激活迂腐模式，那么这个检查器还会报告移位操作符的`_left_`操作数为负或在有符号值的右移期间发生溢出的情况。（大多数编译器可以预测地处理这些行为，但是C标准和C++ 20之前的标准将它们定义为未定义行为。在c++ 20标准中，这些构造是定义良好的，因此在C++ 20中激活迂腐模式没有效果。）

**Examples**

```c++
static_assert(sizeof(int) == 4, "assuming 32-bit int");

void basic_examples(int a, int b) {
    if (b < 0) {
        b = a << b;  // warn: right operand is negative in left shift
    }
    else if (b >= 32) {
        b = a >> b;  // warn: right shift overflows the capacity of 'int'
    }
}

int pedantic_examples(int a, int b) {
    if (a < 0) {
        return a >> b;  // warn: left operand is negative in right shift
    }
    a = 1000u << 31;  // OK, overflow of unsigned value is well-defined, a == 0
    if (b > 10) {
        a = b << 31;  // this is undefined before C++20, but the checker doesn't
                      // warn because it doesn't know the exact value of b
    }
    return 1000 << 31;  // warn: this overflows the capacity of 'int'
}
```

**Solution**

Ensure the shift operands are in proper range before shifting.

移位前确保移位操作数在合适的范围内。

### `core.CallAndMessage` (C, C++, ObjC)

Check for logical errors for function calls and Objective-C message expressions (e.g., uninitialized arguments, null function pointers).

检查函数调用和Objective-C消息表达式的逻辑错误（例如，未初始化的参数，空函数指针）。

```c++
// C
void test() {
    void (*foo)(void);
    foo = 0;
    foo();  // warn: function pointer is null
}

// C++
class C {
public:
    void f();
};

void test() {
    C* pc;
    pc->f();  // warn: object pointer is uninitialized
}

// C++
class C {
public:
    void f();
};

void test() {
    C* pc = 0;
    pc->f();  // warn: object pointer is null
}
```

### `core.DivideZero` (C, C++, ObjC)

Check for division by zero.

```c++
void test(int z) {
  if (z == 0) {
    int x = 1 / z; // warn
  }
}

void test() {
  int x = 1;
  int y = x % 0; // warn
}
```

### `core.NonNullParamChecker` (C, C++, ObjC)

Check for null pointers passed as arguments to a function whose arguments are references or marked with the ‘nonnull’ attribute.

检查作为参数传递给函数的空指针，其参数是引用或标记为`nonnull`属性。

```c++
int f(int *p) __attribute__((nonnull));

void test(int *p) {
  if (!p) {
    f(p); // warn
  }
}
```

`core.NullDereference` (C, C++, ObjC)

Check for dereferences of null pointers.

检查空指针的解引用。

This checker specifically does not report null pointer dereferences for x86 and x86-64 targets when the address space is 256 (x86 GS Segment), 257 (x86 FS Segment), or 258 (x86 SS segment). See X86/X86-64 Language Extensions for reference.

The `SuppressAddressSpaces` option suppresses warnings for null dereferences of all pointers with address spaces. You can disable this behavior with the option `-analyzer-config core.NullDereference:SuppressAddressSpaces=false`. Defaults to true.

```c++
// C
void test(int *p) {
  if (p) {
    return;
  }

  int x = p[0]; // warn
}

// C
void test(int *p) {
  if (!p) {
    *p = 0; // warn
  }
}

// C++
class C {
public:
  int x;
};

void test() {
  C *pc = 0;
  int k = pc->x; // warn
}
```

### `core.StackAddressEscape` (C)

Check that addresses to stack memory do not escape the function.

检查堆栈内存的地址（是否）没有逃离函数。

```c++
char const *p;

void test() {
  char const str[] = "string";
  p = str; // warn
}

void* test() {
   return __builtin_alloca(12); // warn
}

void test() {
  static int *x;
  int y;
  x = &y; // warn
}
```

### `core.UndefinedBinaryOperatorResult` (C)

Check for undefined results of binary operators.

检查二元运算符的未定义结果。

```c++
void test() {
  int x;
  int y = x + 1; // warn: left operand is garbage
}
```

### `core.VLASize` (C)

Check for declarations of Variable Length Arrays of undefined or zero size.

检查未定义或大小为零的可变长度数组的声明。

```c++
void test() {
  int x;
  int vla1[x]; // warn: garbage as size
}

void test() {
  int x = 0;
  int vla2[x]; // warn: zero size
}
```

### `core.uninitialized.ArraySubscript` (C)

Check for uninitialized values used as array subscripts.

检查用作数组下标的未初始化值。

```c++
void test() {
  int i, a[10];
  int x = a[i]; // warn: array subscript is undefined
}
```

### `core.uninitialized.Assign` (C)

Check for assigning uninitialized values.

检查是否分配了未初始化的值。

```c++
void test() {
  int x;
  x |= 1; // warn: left expression is uninitialized
}
```

### `core.uninitialized.Branch` (C)

Check for uninitialized values used as branch conditions.

检查用作分支条件的未初始化值。

```c++
void test() {
  int x;
  if (x) { // warn
    return;
  }
}
```

### `core.uninitialized.CapturedBlockVariable` (C)

Check for blocks that capture uninitialized values.

检查捕获未初始化值的块。

```c++
void test() {
  int x;
  ^{ int y = x; }(); // warn // WTF????????????????
}
```

### `core.uninitialized.UndefReturn` (C)

Check for uninitialized values being returned to the caller.

检查是否有未初始化的值返回给调用者。

```c++
int test() {
  int x;
  return x; // warn
}
```

### `core.uninitialized.NewArraySize` (C++)

Check if the element count in `new[]` is garbage or undefined.

检查`new[]`中的元素计数是否为垃圾或未定义。

```c++
void test() {
  int n;
  int *arr = new int[n]; // warn: Element count in new[] is a garbage value
  delete[] arr;
}
```

## `cplusplus`

C++ Checkers.

### `cplusplus.InnerPointer` (C++)

Check for inner pointers of C++ containers used after re/deallocation.

检查reallocation/deallocation之后使用的C++容器的内部指针。

Many container methods in the C++ standard library are known to invalidate “references” (including actual references, iterators and raw pointers) to elements of the container. Using such references after they are invalidated causes undefined behavior, which is a common source of memory errors in C++ that this checker is capable of finding.

众所周知，C++标准库中的许多容器方法会使对容器元素的“引用”（包括实际引用、迭代器和原始指针）无效。在这些引用无效之后使用它们会导致未定义的行为，这是C++中内存错误的常见来源，该检查器能够找到这些错误。

The checker is currently limited to `std::string` objects and doesn’t recognize some of the more sophisticated approaches to passing unowned pointers around, such as `std::string_view`.

检查器目前仅限于`std::string`对象，并且不能识别一些更复杂的传递无主指针的方法，例如`std::string_view`。

```c++
void deref_after_assignment() {
  std::string s = "llvm";
  
  // note: pointer to inner buffer of 'std::string' obtained here
  const char *c = s.data();
  
  // note: inner buffer of 'std::string' reallocated by call to 'operator='
  s = "clang";
  
  // warn: inner pointer of container used after reallocation/deallocation
  consume(c);
}

const char *return_temp(int x) {
  // warn: inner pointer of container used after reallocation/deallocation
  // note: pointer to inner buffer of 'std::string' obtained here
  // note: inner buffer of 'std::string' deallocated by call to destructor
  return std::to_string(x).c_str();
}
```

### `cplusplus.NewDelete` (C++)

Check for double-free and use-after-free problems. Traces memory managed by `new`/`delete`.

检查是否有重复释放和释放后使用（UAF）的问题。跟踪由`new`/`delete`管理的内存。

```c++
void f(int *p);

void testUseMiddleArgAfterDelete(int *p) {
  delete p;
  f(p); // warn: use after free
}

class SomeClass {
public:
  void f();
};

void test() {
  SomeClass *c = new SomeClass;
  delete c;
  c->f(); // warn: use after free
}

void test() {
  int *p = (int *)__builtin_alloca(sizeof(int));
  delete p; // warn: deleting memory allocated by alloca
}

void test() {
  int *p = new int;
  delete p;
  delete p; // warn: attempt to free released
}

void test() {
  int i;
  delete &i; // warn: delete address of local
}

void test() {
  int *p = new int[1];
  delete[] (++p);
    // warn: argument to 'delete[]' is offset by 4 bytes
    // from the start of memory allocated by 'new[]'
}
```

### `cplusplus.NewDeleteLeaks` (C++)

Check for memory leaks. Traces memory managed by new/delete.

检查内存泄漏。跟踪由`new`/`delete`管理的内存。

```c++
void test() {
  int *p = new int;
} // warn
```

### `cplusplus.PlacementNew` (C++)

Check if default `placement new` is provided with pointers to sufficient storage capacity.

检查默认`placement new`是否提供了指向足够存储容量的指针。

```c++
#include <new>

void f() {
  short s;
  long *lp = ::new (&s) long; // warn
}
```

### `cplusplus.SelfAssignment` (C++)

Checks C++ copy and move assignment operators for self assignment.

检查C++的复制赋值操作符和移动赋值操作符的自赋值。

### `cplusplus.StringChecker` (C++)

Checks `std::string` operations.

Checks if the `cstring` pointer from which the `std::string` object is constructed is `NULL` or not. If the checker cannot reason about the nullness of the pointer it will assume that it was non-null to satisfy the precondition of the constructor.

检查构造`std::string`对象的`cstring`指针是否为`NULL`。

如果检查器不能推断指针的空性，它将假定它是非空的，以满足构造函数的先决条件。

This checker is capable of checking the [SEI CERT C++ coding rule STR51-CPP. Do not attempt to create a `std::string` from a null pointer](https://wiki.sei.cmu.edu/confluence/x/E3s-BQ).

```c++
#include <string>

void f(const char *p) {
  if (!p) {
    std::string msg(p); // warn: The parameter must not be null
  }
}
```

## `deadcode`

Dead Code Checkers.

### `deadcode.DeadStores` (C)

Check for values stored to variables that are never read afterwards.

检查存储在变量中的值是否在之后永远不会被读取。

```c++
void test() {
  int x;
  x = 1; // warn
}
```

The `WarnForDeadNestedAssignments` option enables the checker to emit warnings for nested dead assignments. You can disable with the `-analyzer-config deadcode.DeadStores:WarnForDeadNestedAssignments=false`. Defaults to `true`.

Would warn for this e.g.: `if ((y = make_int())) { }`.

## `nullability`

Objective C checkers that warn for null pointer passing and dereferencing errors.

对空指针传递和解引用错误发出警告的Objective C检查器。

## `optin`

Checkers for portability, performance or coding style specific rules.

检查可移植性、性能或编码风格特定规则。

### `optin.core.EnumCastOutOfRange` (C, C++)

Check for integer to enumeration casts that would produce a value with no corresponding enumerator. This is not necessarily undefined behavior, but can lead to nasty surprises, so projects may decide to use a coding standard that disallows these “unusual” conversions.

检查是否有整数到枚举的类型转换会产生一个没有相应枚举数的值。

这不一定是未定义的行为，但可能会导致令人讨厌的意外，因此项目可能决定使用不允许这些“不寻常”转换的编码标准。

Note that no warnings are produced when the enum type (e.g. `std::byte`) has no enumerators at all.

```c++
enum WidgetKind { A=1, B, C, X=99 };

void foo() {
  WidgetKind c = static_cast<WidgetKind>(3);  // OK
  WidgetKind x = static_cast<WidgetKind>(99); // OK
  WidgetKind d = static_cast<WidgetKind>(4);  // warn
}
```

**Limitations**

This checker does not accept the coding pattern where an enum type is used to store combinations of flag values:

```c++
enum AnimalFlags {
    HasClaws   = 1,
    CanFly     = 2,
    EatsFish   = 4,
    Endangered = 8
};

AnimalFlags operator|(AnimalFlags a, AnimalFlags b) {
    return static_cast<AnimalFlags>(static_cast<int>(a) | static_cast<int>(b));
}

auto flags = HasClaws | CanFly;
```

Projects that use this pattern should not enable this `optin` checker.

### `optin.cplusplus.UninitializedObject` (C++)

This checker reports uninitialized fields in objects created after a constructor call. It doesn’t only find direct uninitialized fields, but rather makes a deep inspection of the object, analyzing all of its fields’ subfields. The checker regards inherited fields as direct fields, so one will receive warnings for uninitialized inherited data members as well.

此检查器报告在构造函数调用后创建的对象中未初始化的字段。

它不仅查找直接的未初始化字段，而且对对象进行深入检查，分析其字段的所有子字段。

检查器将继承字段视为直接字段，因此也会收到未初始化继承数据成员的警告。

```c++
// With Pedantic and CheckPointeeInitialization set to true

struct A {
  struct B {
    int x; // note: uninitialized field 'this->b.x'
    	   // note: uninitialized field 'this->bptr->x'
    int y; // note: uninitialized field 'this->b.y'
    	   // note: uninitialized field 'this->bptr->y'
  };
  int *iptr;  // note: uninitialized pointer 'this->iptr'
  B b;
  B *bptr;
  char *cptr; // note: uninitialized pointee 'this->cptr'

  A (B *bptr, char *cptr) : bptr(bptr), cptr(cptr) {}
};

void f() {
  A::B b;
  char c;
  A a(&b, &c); // warning: 6 uninitialized fields after the constructor call
}

// With Pedantic set to false and CheckPointeeInitialization set to true
// (every field is uninitialized)

struct A {
  struct B {
    int x;
    int y;
  };
  int *iptr;
  B b;
  B *bptr;
  char *cptr;

  A (B *bptr, char *cptr)
    : bptr(bptr)
    , cptr(cptr)
  {}
};

void f() {
  A::B b;
  char c;
  A a(&b, &c); // no warning
}

// With Pedantic set to true and CheckPointeeInitialization set to false
// (pointees are regarded as initialized)

struct A {
  struct B {
    int x; // note: uninitialized field 'this->b.x'
    int y; // note: uninitialized field 'this->b.y'
  };
  int *iptr; // note: uninitialized pointer 'this->iptr'
  B b;
  B *bptr;
  char *cptr;

  A (B *bptr, char *cptr)
    : bptr(bptr)
    , cptr(cptr)
  {}
};

void f() {
  A::B b;
  char c;
  A a(&b, &c); // warning: 3 uninitialized fields after the constructor call
}
```

**Options**

This checker has several options which can be set from command line (e.g. `-analyzer-config optin.cplusplus.UninitializedObject:Pedantic=true`):

- `Pedantic` (boolean).
  - If to false, the checker won’t emit warnings for objects that don’t have at least one initialized field.
  - Defaults to false.
- `NotesAsWarnings` (boolean).
  - If set to true, the checker will emit a warning for each uninitialized field, as opposed to emitting one warning per constructor call, and listing the uninitialized fields that belongs to it in notes.
  - Defaults to false.
- `CheckPointeeInitialization` (boolean).
  - If set to false, the checker will not analyze the pointee of pointer/reference fields, and will only check whether the object itself is initialized.
  - Defaults to false.
- `IgnoreRecordsWithField` (string).
  - If supplied, the checker will not analyze structures that have a field with a name or type name that matches the given pattern.
  - Defaults to "".

### `optin.cplusplus.VirtualCall` (C++)

Check virtual function calls during construction or destruction.

在构造或销毁过程中检查虚函数调用。

```c++
class A {
public:
  A() {
    f(); // warn
  }
  virtual void f();
};

class A {
public:
  ~A() {
    this->f(); // warn
  }
  virtual void f();
};
```

### `optin.mpi.MPI-Checker` (C)

Checks MPI code.

```c++
void test() {
  double buf = 0;
  MPI_Request sendReq1;
  MPI_Ireduce(
      MPI_IN_PLACE, &buf, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD, &sendReq1
  );
} // warn: request 'sendReq1' has no matching wait.

void test() {
  double buf = 0;
  MPI_Request sendReq;
  MPI_Isend(&buf, 1, MPI_DOUBLE, 0, 0, MPI_COMM_WORLD, &sendReq);
  MPI_Irecv(&buf, 1, MPI_DOUBLE, 0, 0, MPI_COMM_WORLD, &sendReq); // warn
  MPI_Isend(&buf, 1, MPI_DOUBLE, 0, 0, MPI_COMM_WORLD, &sendReq); // warn
  MPI_Wait(&sendReq, MPI_STATUS_IGNORE);
}

void missingNonBlocking() {
  int rank = 0;
  MPI_Comm_rank(MPI_COMM_WORLD, &rank);
  MPI_Request sendReq1[10][10][10];
  MPI_Wait(&sendReq1[1][7][9], MPI_STATUS_IGNORE); // warn
}
```

### `optin.performance.GCDAntipattern`

Check for performance anti-patterns when using Grand Central Dispatch.

### `optin.performance.Padding`

Check for excessively padded structs.

### `optin.portability.UnixAPI`

Finds implementation-defined behavior in UNIX/POSIX functions.

## `security`

Security related checkers.

### `security.cert.env.InvalidPtr`

Corresponds to SEI CERT Rules [ENV31-C](https://wiki.sei.cmu.edu/confluence/display/c/ENV31-C.+Do+not+rely+on+an+environment+pointer+following+an+operation+that+may+invalidate+it) and [ENV34-C](https://wiki.sei.cmu.edu/confluence/display/c/ENV34-C.+Do+not+store+pointers+returned+by+certain+functions).

ENV31-C: Rule is about the possible problem with main function’s third argument, environment pointer, “envp”. When environment array is modified using some modification function such as `putenv`, `setenv` or others, It may happen that memory is reallocated, however “envp” is not updated to reflect the changes and points to old memory region.

ENV34-C: Some functions return a pointer to a statically allocated buffer. Consequently, subsequent call of these functions will invalidate previous pointer. These functions include: `getenv`, `localeconv`, `asctime`, `setlocale`, `strerror`.

```c++
int main(int argc, const char *argv[], const char *envp[]) {
  if (setenv("MY_NEW_VAR", "new_value", 1) != 0) {
    // setenv call may invalidate 'envp'
    /* Handle error */
  }
  if (envp != NULL) {
    for (size_t i = 0; envp[i] != NULL; ++i) {
      puts(envp[i]);
      // envp may no longer point to the current environment
      // this program has unanticipated behavior, since envp
      // does not reflect changes made by setenv function.
    }
  }
  return 0;
}

void previous_call_invalidation() {
  char *p, *pp;

  p = getenv("VAR");
  setenv("SOMEVAR", "VALUE", /*overwrite = */1);
  // call to 'setenv' may invalidate p

  *p;
  // dereferencing invalid pointer
}
```

The `InvalidatingGetEnv` option is available for treating `getenv` calls as invalidating. When enabled, the checker issues a warning if `getenv` is called multiple times and their results are used without first creating a copy. This level of strictness might be considered overly pedantic for the commonly used `getenv` implementations.

To enable this option, use: `-analyzer-config security.cert.env.InvalidPtr:InvalidatingGetEnv=true`.

By default, this option is set to false.

When this option is enabled, warnings will be generated for scenarios like the following:

```c++
char* p  = getenv("VAR");
char* pp = getenv("VAR2"); // assumes this call can invalidate `env`
strlen(p); // warns about accessing invalid ptr
```

### `security.FloatLoopCounter` (C)

Warn on using a floating point value as a loop counter (CERT: FLP30-C, FLP30-CPP).

```c++
void test() {
  for (float x = 0.1f; x <= 1.0f; x += 0.1f) {} // warn
}
```

### `security.insecureAPI.UncheckedReturn` (C)

Warn on uses of functions whose return values must be always checked.

```c++
void test() {
  setuid(1); // warn
}
```

### `security.insecureAPI.bcmp` (C)

Warn on uses of the `bcmp` function.

```c++
void test() {
  bcmp(ptr0, ptr1, n); // warn
}
```

### `security.insecureAPI.bcopy` (C)

Warn on uses of the `bcopy` function.

```c++
void test() {
  bcopy(src, dst, n); // warn
}
```

### `security.insecureAPI.bzero` (C)

Warn on uses of the `bzero` function.

```c++
void test() {
  bzero(ptr, n); // warn
}
```

### `security.insecureAPI.getpw` (C)

Warn on uses of the `getpw` function.

```c++
void test() {
  char buff[1024];
  getpw(2, buff); // warn
}
```

### `security.insecureAPI.gets` (C)

Warn on uses of the `gets` function.

```c++
void test() {
  char buff[1024];
  gets(buff); // warn
}
```

### `security.insecureAPI.mkstemp` (C)

Warn when `mkstemp` is passed fewer than 6 X’s in the format string.

```c++
void test() {
  mkstemp("XX"); // warn
}
```

### `security.insecureAPI.mktemp` (C)

Warn on uses of the `mktemp` function.

```c++
void test() {
  char *x = mktemp("/tmp/zxcv"); // warn: insecure, use mkstemp
}
```

### `security.insecureAPI.rand` (C)

Warn on uses of inferior random number generating functions (only if arc4random function is available): `drand48, erand48, jrand48, lcong48, lrand48, mrand48, nrand48, random, rand_r`.

```c++
void test() {
  random(); // warn
}
```

### `security.insecureAPI.strcpy` (C)

Warn on uses of the `strcpy` and `strcat` functions.

```c++
void test() {
  char x[4];
  char *y = "abcd";

  strcpy(x, y); // warn
}
```

### `security.insecureAPI.vfork` (C)

Warn on uses of the `vfork` function.

```c++
void test() {
  vfork(); // warn
}
```

### `security.insecureAPI.DeprecatedOrUnsafeBufferHandling` (C)

Warn on occurrences of unsafe or deprecated buffer handling functions, which now have a secure variant: `sprintf, vsprintf, scanf, wscanf, fscanf, fwscanf, vscanf, vwscanf, vfscanf, vfwscanf, sscanf, swscanf, vsscanf, vswscanf, swprintf, snprintf, vswprintf, vsnprintf, memcpy, memmove, strncpy, strncat, memset`

```c++
void test() {
  char buf [5];
  strncpy(buf, "a", 1); // warn
}
```

## `unix`

POSIX/Unix checkers.

### `unix.API` (C)

Check calls to various UNIX/Posix functions: `open, pthread_once, calloc, malloc, realloc, alloca`.

```c++
// Currently the check is performed for apple targets only.
void test(const char *path) {
  int fd = open(path, O_CREAT);
    // warn: call to 'open' requires a third argument when the
    // 'O_CREAT' flag is set
}

void f();

void test() {
  pthread_once_t pred = {0x30B1BCBA, {0}};
  pthread_once(&pred, f);
    // warn: call to 'pthread_once' uses the local variable
}

void test() {
  void *p = malloc(0); // warn: allocation size of 0 bytes
}

void test() {
  void *p = calloc(0, 42); // warn: allocation size of 0 bytes
}

void test() {
  void *p = malloc(1);
  p = realloc(p, 0); // warn: allocation size of 0 bytes
}

void test() {
  void *p = alloca(0); // warn: allocation size of 0 bytes
}

void test() {
  void *p = valloc(0); // warn: allocation size of 0 bytes
}
```

### `unix.Errno` (C)

Check for improper use of `errno`. This checker implements partially CERT rule [ERR30-C. Set errno to zero before calling a library function known to set errno, and check errno only after the function returns a value indicating failure](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152351). The checker can find the first read of `errno` after successful standard function calls.

The C and POSIX standards often do not define if a standard library function may change value of `errno` if the call does not fail. Therefore, `errno` should only be used if it is known from the return value of a function that the call has failed. There are exceptions to this rule (for example `strtol`) but the affected functions are not yet supported by the checker. The return values for the failure cases are documented in the standard Linux man pages of the functions and in the [POSIX standard](https://pubs.opengroup.org/onlinepubs/9699919799/).

```c++
int unsafe_errno_read(int sock, void *data, int data_size) {
  if (send(sock, data, data_size, 0) != data_size) {
    // 'send' can be successful even if not all data was sent
    if (errno == 1) { // An undefined value may be read from 'errno'
      return 0;
    }
  }
  return 1;
}
```

The checker [unix.StdCLibraryFunctions (C)](https://clang.llvm.org/docs/analyzer/checkers.html#unix-stdclibraryfunctions) must be turned on to get the warnings from this checker. The supported functions are the same as by [unix.StdCLibraryFunctions (C)](https://clang.llvm.org/docs/analyzer/checkers.html#unix-stdclibraryfunctions). The `ModelPOSIX` option of that checker affects the set of checked functions.

**Parameters**

The `AllowErrnoReadOutsideConditionExpressions` option allows read of the errno value if the value is not used in a condition (in `if` statements, loops, conditional expressions, `switch` statements). For example `errno` can be stored into a variable without getting a warning by the checker.

```c++
int unsafe_errno_read(int sock, void *data, int data_size) {
  if (send(sock, data, data_size, 0) != data_size) {
    int err = errno;
    // warning if 'AllowErrnoReadOutsideConditionExpressions' is false
    // no warning if 'AllowErrnoReadOutsideConditionExpressions' is true
  }
  return 1;
}
```

Default value of this option is `true`. This allows save of the errno value for possible later error handling.

**Limitations**

- Only the very first usage of `errno` is checked after an affected function call. Value of `errno` is not followed when it is stored into a variable or returned from a function.

- Documentation of function `lseek` is not clear about what happens if the function returns different value than the expected file position but not -1. To avoid possible false-positives `errno` is allowed to be used in this case.

## `osx`

macOS checkers.

## `fuchsia`

Fuchsia is an open source capability-based operating system currently being developed by Google. This section describes checkers that can find various misuses of Fuchsia APIs.

### `fuchsia.HandleChecker`

Handles identify resources. Similar to pointers they can be leaked, double freed, or use after freed. This check attempts to find such problems.

句柄标识资源。与指针类似，它们可以泄漏、双重释放或在释放后使用。此检查试图查找此类问题。

## `WebKit`

WebKit is an open-source web browser engine available for macOS, iOS and Linux. This section describes checkers that can find issues in WebKit codebase.

WebKit是一个开源的Web浏览器引擎，可用于macOS, iOS和Linux。本节描述可以在WebKit代码库中发现问题的检查器。

Most of the checkers focus on memory management for which WebKit uses custom implementation of reference counted smart pointers.

- Checkers are formulated in terms related to ref-counting: Ref-counted type is either `Ref<T>` or `RefPtr<T>`.
- Ref-countable type is any type that implements `ref()` and `deref()` methods as `RefPtr<>` is a template (i. e. relies on duck typing).

- Uncounted type is ref-countable but not ref-counted type.


### `webkit.RefCntblBaseVirtualDtor`

All uncounted types used as base classes must have a virtual destructor.

Ref-counted types hold their ref-countable data by a raw pointer and allow implicit upcasting from ref-counted pointer to derived type to ref-counted pointer to base type. This might lead to an object of (dynamic) derived type being deleted via pointer to the base class type which C++ standard defines as UB in case the base class doesn’t have virtual destructor `[expr.delete]`.

```c++
struct RefCntblBase {
  void ref() {}
  void deref() {}
};

struct Derived : RefCntblBase { }; // warn
```

### `webkit.NoUncountedMemberChecker`

Raw pointers and references to uncounted types can’t be used as class members. Only ref-counted types are allowed.

```c++
struct RefCntbl {
  void ref() {}
  void deref() {}
};

struct Foo {
  RefCntbl * ptr; // warn
  RefCntbl & ptr; // warn
  // ...
};
```

### `webkit.UncountedLambdaCapturesChecker`

Raw pointers and references to uncounted types can’t be captured in lambdas. Only ref-counted types are allowed.

```c++
struct RefCntbl {
  void ref() {}
  void deref() {}
};

void foo(RefCntbl* a, RefCntbl& b) {
  [&, a]() { // warn about 'a'
    do_something(b); // warn about 'b'
  };
};
```

# Experimental Checkers

These are checkers with known issues or limitations that keep them from being on by default. They are likely to have false positives. Bug reports and especially patches are welcome.

这些检查器具有已知的问题或限制，它们在默认情况下处于关闭状态。他们很可能有假阳性。

## `alpha.clone`

### `alpha.clone.CloneChecker` (C, C++, ObjC)

Reports similar pieces of code.

```c++
void log();

int max(int a, int b) { // warn
  log();
  if (a > b) {
    return a;
  }
  return b;
}

int maxClone(int x, int y) { // similar code here
  log();
  if (x > y) {
    return x;
  }
  return y;
}
```

## `alpha.core`

### `alpha.core.C11Lock`

Similarly to `alpha.unix.PthreadLock`, checks for the locking/unlocking of `mtx_t` mutexes.

```c++
mtx_t mtx1;

void bad1(void) {
  mtx_lock(&mtx1);
  mtx_lock(&mtx1); // warn: This lock has already been acquired
}
```

### `alpha.core.CallAndMessageUnInitRefArg` (C,C++, ObjC)

Check for logical errors for function calls and Objective-C message expressions (e.g., uninitialized arguments, null function pointers, and pointer to undefined variables).

```c++
void test(void) {
  int t;
  int &p = t;
  int &s = p;
  int &q = s;
  foo(q); // warn
}

void test(void) {
  int x;
  foo(&x); // warn
}
```

### `alpha.core.CastSize` (C)

Check when casting a malloced type `T`, whether the size is a multiple of the size of `T`.

```c++
void test() {
  int *x = (int *) malloc(11); // warn
}
```

### `alpha.core.CastToStruct` (C, C++)

Check for cast from non-struct pointer to struct pointer.

```c++
// C
struct s {};

void test(int *p) {
  struct s *ps = (struct s *) p; // warn
}

// C++
class c {};

void test(int *p) {
  c *pc = (c *) p; // warn
}
```

### `alpha.core.Conversion` (C, C++, ObjC)

Loss of sign/precision in implicit conversions.

```c++
void test(unsigned U, signed S) {
  if (S > 10) {
    if (U < S) {}
  }
  if (S < -10) {
    if (U < S) {} // warn (loss of sign)
  }
}

void test() {
  long long A = 1LL << 60;
  short X = A; // warn (loss of precision)
}
```

### `alpha.core.FixedAddr` (C)

Check for assignment of a fixed address to a pointer.

```c++
void test() {
  int *p;
  p = (int *) 0x10000; // warn
}
```

### `alpha.core.IdenticalExpr` (C, C++)

Warn about unintended use of identical expressions in operators.

```c++
// C
void test() {
  int a = 5;
  int b = a | 4 | a; // warn: identical expr on both sides
}

// C++
bool f(void);

void test(bool b) {
  int i = 10;
  if (f()) { // warn: true and false branches are identical
    do {
      i--;
    } while (f());
  } else {
    do {
      i--;
    } while (f());
  }
}
```

### `alpha.core.PointerArithm` (C)

Check for pointer arithmetic on locations other than array elements.

```c++
void test() {
  int x;
  int *p;
  p = &x + 1; // warn
}
```

### `alpha.core.PointerSub` (C)

Check for pointer subtractions on two pointers pointing to different memory chunks.

```c++
void test() {
  int x, y;
  int d = &y - &x; // warn
}
```

### `alpha.core.SizeofPtr` (C)

Warn about unintended use of `sizeof()` on pointer expressions.

```c++
struct s {};

int test(struct s *p) {
  return sizeof(p);
    // warn: sizeof(ptr) can produce an unexpected result
}
```

### `alpha.core.StackAddressAsyncEscape` (C)

Check that addresses to stack memory do not escape the function that involves dispatch_after or dispatch_async. This checker is a part of `core.StackAddressEscape`, but is temporarily disabled until some false positives are fixed.

```c++
dispatch_block_t test_block_inside_block_async_leak() {
  int x = 123;
  void (^inner)(void) = ^void(void) {
    int y = x;
    ++y;
  };
  void (^outer)(void) = ^void(void) {
    int z = x;
    ++z;
    inner();
  };
  return outer; // warn: address of stack-allocated block is captured by a
                //       returned block
}
```

### `alpha.core.TestAfterDivZero` (C)

Check for division by variable that is later compared against 0. Either the comparison is useless or there is division by zero.

```c++
void test(int x) {
  var = 77 / x;
  if (x == 0) { } // warn
}
```

## `alpha.cplusplus`

### `alpha.cplusplus.ArrayDelete` (C++)

Reports destructions of arrays of polymorphic objects that are destructed as their base class.

This checker corresponds to the CERT rule [EXP51-CPP: Do not delete an array through a pointer of the incorrect type](https://wiki.sei.cmu.edu/confluence/display/cplusplus/EXP51-CPP.+Do+not+delete+an+array+through+a+pointer+of+the+incorrect+type).

```c++
class Base {
  virtual ~Base() {}
};
class Derived : public Base {}

Base *create() {
  Base *x = new Derived[10]; // note: Casting from 'Derived' to 'Base' here
  return x;
}

void foo() {
  Base *x = create();
  
  // warn: Deleting an array of 'Derived' objects as their base class 'Base' is undefined
  delete[] x;
}
```

### `alpha.cplusplus.DeleteWithNonVirtualDtor` (C++)

Reports destructions of polymorphic objects with a non-virtual destructor in their base class.

```c++
class NonVirtual {};
class NVDerived : public NonVirtual {};

NonVirtual *create() {
  NonVirtual *x = new NVDerived(); // note: Casting from 'NVDerived' to 'NonVirtual' here
  return x;
}

void foo() {
  NonVirtual *x = create();
  delete x; // warn: destruction of a polymorphic object with no virtual destructor
}
```

### `alpha.cplusplus.InvalidatedIterator` (C++)

Check for use of invalidated iterators.

```c++
void bad_copy_assign_operator_list1(
	std::list &L1,
	const std::list &L2)
{
  auto i0 = L1.cbegin();
  L1 = L2;
  *i0; // warn: invalidated iterator accessed
}
```

### `alpha.cplusplus.IteratorRange` (C++)

Check for iterators used outside their valid ranges.

```c++
void simple_bad_end(const std::vector &v) {
  auto i = v.end();
  *i; // warn: iterator accessed outside of its range
}
```

### `alpha.cplusplus.MismatchedIterator` (C++)

Check for use of iterators of different containers where iterators of the same container are expected.

```c++
void bad_insert3(std::vector &v1, std::vector &v2) {
  // warn: container accessed using foreign iterator argument
  v2.insert(v1.cbegin(), v2.cbegin(), v2.cend());
  
  // warn: iterators of different containers used where the same container is expected
  v1.insert(v1.cbegin(), v1.cbegin(), v2.cend());
  
  // warn: iterators of different containers used where the same container is expected
  v1.insert(v1.cbegin(), v2.cbegin(), v1.cend());
}
```

### `alpha.cplusplus.MisusedMovedObject` (C++)

Method calls on a moved-from object and copying a moved-from object will be reported.

```c++
 struct A {
  void foo() {}
};

void f() {
  A a;
  A b = std::move(a); // note: 'a' became 'moved-from' here
  a.foo();            // warn: method call on a 'moved-from' object 'a'
}
```

### `alpha.cplusplus.SmartPtr` (C++)

Check for dereference of null smart pointers.

```c++
void deref_smart_ptr() {
  std::unique_ptr<int> P;
  *P; // warn: dereference of a default constructed smart unique_ptr
}
```

## `alpha.deadcode`

### `alpha.deadcode.UnreachableCode` (C, C++)

Check unreachable code.

```c++
// C
int test() {
  int x = 1;
  while(x);
  return x; // warn
}

// C++
void test() {
  int a = 2;

  while (a > 1) {
    a--;
  }

  if (a > 1) {
    a++; // warn
  }
}
```

## `alpha.fuchsia`

### `alpha.fuchsia.Lock`

Similarly to `alpha.unix.PthreadLock`, checks for the locking/unlocking of fuchsia mutexes.

```c++
spin_lock_t mtx1;

void bad1(void) {
  spin_lock(&mtx1);
  spin_lock(&mtx1); // warn: This lock has already been acquired
}
```

## `alpha.llvm`

### `alpha.llvm.Conventions`

Check code for LLVM codebase conventions:

- A StringRef should not be bound to a temporary std::string whose lifetime is shorter than the StringRef’s.
- Clang AST nodes should not have fields that can allocate memory.

## `alpha.security`

### `alpha.security.ArrayBound` (C)

Warn about buffer overflows (older checker).

```c++
void test() {
  char *s = "";
  char c = s[1]; // warn
}

struct seven_words {
  int c[7];
};

void test() {
  struct seven_words a, *p;
  p = &a;
  p[0] = a;
  p[1] = a;
  p[2] = a; // warn
}

// note: requires unix.Malloc or
// alpha.unix.MallocWithAnnotations checks enabled.
void test() {
  int *p = malloc(12);
  p[3] = 4; // warn
}

void test() {
  char a[2];
  int *b = (int*)a;
  b[1] = 3; // warn
}
```

### `alpha.security.ArrayBoundV2` (C)

Warn about buffer overflows (newer checker).

```c++
void test() {
  char *s = "";
  char c = s[1]; // warn
}

void test() {
  int buf[100];
  int *p = buf;
  p = p + 99;
  p[1] = 1; // warn
}

// note: compiler has internal check for this.
// Use -Wno-array-bounds to suppress compiler warning.
void test() {
  int buf[100][100];
  buf[0][-1] = 1; // warn
}

// note: requires alpha.security.taint check turned on.
void test() {
  char s[] = "abc";
  int x = getchar();
  char c = s[x]; // warn: index is tainted
}
```

### `alpha.security.MallocOverflow` (C)

Check for overflows in the arguments to `malloc()`. It tries to catch `malloc(n * c)` patterns, where:

> - `n`: a variable or member access of an object
> - `c`: a constant foldable integral

This checker was designed for code audits, so expect false-positive reports. One is supposed to silence this checker by ensuring proper bounds checking on the variable in question using e.g. an `assert()` or a branch.

```c++
void test(int n) {
  void *p = malloc(n * sizeof(int)); // warn
}

void test2(int n) {
  if (n > 100) // gives an upper-bound
    return;
  void *p = malloc(n * sizeof(int)); // no warning
}

void test3(int n) {
  assert(n <= 100 && "Contract violated.");
  void *p = malloc(n * sizeof(int)); // no warning
}
```

Limitations:

> - The checker won’t warn for variables involved in explicit casts, since that might limit the variable’s domain. E.g.: `(unsigned char)int x` would limit the domain to `[0,255]`. The checker will miss the true-positive cases when the explicit cast would not tighten the domain to prevent the overflow in the subsequent multiplication operation.
> - It is an AST-based checker, thus it does not make use of the path-sensitive taint-analysis.

### `alpha.security.MmapWriteExec` (C)

Warn on `mmap()` calls that are both writable and executable.

```c++
void test(int n) {
  // warn:
  // Both PROT_WRITE and PROT_EXEC flags are set.
  // This can lead to exploitable memory regions, which could be overwritten with malicious code.
  void *c = mmap(
      NULL,
      32,
      PROT_READ | PROT_WRITE | PROT_EXEC, MAP_PRIVATE | MAP_ANON,
      -1,
      0
  );
}
```

### `alpha.security.ReturnPtrRange` (C)

Check for an out-of-bound pointer being returned to callers.

```c++
static int A[10];

int *test() {
  int *p = A + 10;
  return p; // warn
}

int test(void) {
  int x;
  return x; // warn: undefined or garbage returned
}
```

### `alpha.security.cert`

SEI CERT checkers which tries to find errors based on their [C coding rules](https://wiki.sei.cmu.edu/confluence/display/c/2+Rules).

### `alpha.security.cert.pos`

SEI CERT checkers of [POSIX C coding rules](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152405).

### `alpha.security.cert.pos.34c`

Finds calls to the `putenv` function which pass a pointer to an automatic variable as the argument.

```c++
int func(const char *var) {
  char env[1024];
  int retval = snprintf(env, sizeof(env),"TEST=%s", var);
  if (retval < 0 || (size_t)retval >= sizeof(env)) {
      /* Handle error */
  }

  return putenv(env); // putenv function should not be called with auto variables
}
```

Limitations:

Technically, one can pass automatic variables to `putenv`, but one needs to ensure that the given environment key stays alive until it’s removed or overwritten. Since the analyzer cannot keep track of which envvars get overwritten and when, it needs to be slightly more aggressive and warn for such cases too, leading in some cases to false-positive reports like this:
```c++
void baz() {
  char env[] = "NAME=value";
  putenv(env); // false-positive warning: putenv function should not be called...
  // More code...
  putenv((char *)"NAME=anothervalue");
  // This putenv call overwrites the previous entry, thus that can no longer dangle.
} // 'env' array becomes dead only here.
```

### `alpha.security.cert.env`

SEI CERT checkers of [Environment C coding rules](https://wiki.sei.cmu.edu/confluence/x/JdcxBQ).

### `alpha.security.taint`

Checkers implementing [taint analysis](https://en.wikipedia.org/wiki/Taint_checking).

### `alpha.security.taint.TaintPropagation (C, C++)`

Taint analysis identifies potential security vulnerabilities where the attacker can inject malicious data to the program to execute an attack (privilege escalation, command injection, SQL injection etc.).

The malicious data is injected at the taint source (e.g. `getenv()` call) which is then propagated through function calls and being used as arguments of sensitive operations, also called as taint sinks (e.g. `system()` call).

One can defend against this type of vulnerability by always checking and sanitizing the potentially malicious, untrusted user input.

The goal of the checker is to discover and show to the user these potential taint source-sink pairs and the propagation call chain.

The most notable examples of taint sources are:

> - data from network
> - files or standard input
> - environment variables
> - data from databases

Let us examine a practical example of a Command Injection attack.

```c++
// Command Injection Vulnerability Example
int main(int argc, char** argv) {
  char cmd[2048] = "/bin/cat ";
  char filename[1024];
  printf("Filename:");
  scanf (" %1023[^\n]", filename); // The attacker can inject a shell escape here
  strcat(cmd, filename);
  system(cmd); // Warning: Untrusted data is passed to a system call
}
```

The program prints the content of any user specified file. Unfortunately the attacker can execute arbitrary commands with shell escapes. For example with the following input the ls command is also executed after the contents of /etc/shadow is printed. Input: /etc/shadow ; ls /

The analysis implemented in this checker points out this problem.

One can protect against such attack by for example checking if the provided input refers to a valid file and removing any invalid user input.

```c++
// No vulnerability anymore, but we still get the warning
void sanitizeFileName(char* filename){
  if (access(filename,F_OK)){// Verifying user input
    printf("File does not exist\n");
    filename[0]='\0';
    }
}
int main(int argc, char** argv) {
  char cmd[2048] = "/bin/cat ";
  char filename[1024];
  printf("Filename:");
  scanf (" %1023[^\n]", filename); // The attacker can inject a shell escape here
  sanitizeFileName(filename);// filename is safe after this point
  if (!filename[0])
    return -1;
  strcat(cmd, filename);
  system(cmd); // Superfluous Warning: Untrusted data is passed to a system call
}
```

Unfortunately, the checker cannot discover automatically that the programmer have performed data sanitation, so it still emits the warning.

One can get rid of this superfluous warning by telling by specifying the sanitation functions in the taint configuration file (see [Taint Analysis Configuration](https://clang.llvm.org/docs/analyzer/user-docs/TaintAnalysisConfiguration.html)).

```
Filters:
- Name: sanitizeFileName
  Args: [0]
```

The clang invocation to pass the configuration file location:

```shell
clang  \
	--analyze \
	-Xclang \
	-analyzer-config \
	-Xclang alpha.security.taint.TaintPropagation:Config=`pwd`/taint_config.yml \
	...
```

If you are validating your inputs instead of sanitizing them, or don’t want to mention each sanitizing function in our configuration, you can use a more generic approach.

Introduce a generic no-op csa_mark_sanitized(..) function to tell the Clang Static Analyzer that the variable is safe to be used on that analysis path.

```c++
// Marking sanitized variables safe.
// No vulnerability anymore, no warning.

// User csa_mark_sanitize function is for the analyzer only
#ifdef __clang_analyzer__
  void csa_mark_sanitized(const void *);
#endif

int main(int argc, char** argv) {
  char cmd[2048] = "/bin/cat ";
  char filename[1024];
  printf("Filename:");
  scanf (" %1023[^\n]", filename);
  if (access(filename,F_OK)){// Verifying user input
    printf("File does not exist\n");
    return -1;
  }
  #ifdef __clang_analyzer__
    csa_mark_sanitized(filename); // Indicating to CSA that filename variable is safe to be used after this point
  #endif
  strcat(cmd, filename);
  system(cmd); // No warning
}
```

Similarly to the previous example, you need to define a Filter function in a YAML configuration file and add the `csa_mark_sanitized` function.

```
Filters:
- Name: csa_mark_sanitized
  Args: [0]
```

Then calling `csa_mark_sanitized(X)` will tell the analyzer that X is safe to be used after this point, because its contents are verified. It is the responsibility of the programmer to ensure that this verification was indeed correct. Please note that `csa_mark_sanitized` function is only declared and used during Clang Static Analysis and skipped in (production) builds.

Further examples of injection vulnerabilities this checker can find.

```c++
void test() {
  char x = getchar(); // 'x' marked as tainted
  system(&x); // warn: untrusted data is passed to a system call
}

// note: compiler internally checks if the second param to
// sprintf is a string literal or not.
// Use -Wno-format-security to suppress compiler warning.
void test() {
  char s[10], buf[10];
  fscanf(stdin, "%s", s); // 's' marked as tainted

  sprintf(buf, s); // warn: untrusted data used as a format string
}

void test() {
  size_t ts;
  scanf("%zd", &ts); // 'ts' marked as tainted
  int *p = (int *)malloc(ts * sizeof(int));
    // warn: untrusted data used as buffer size
}
```

There are built-in sources, propagations and sinks even if no external taint configuration is provided.

- Default sources:

  `_IO_getc`, `fdopen`, `fopen`, `freopen`, `get_current_dir_name`, `getch`, `getchar`, `getchar_unlocked`, `getwd`, `getcwd`, `getgroups`, `gethostname`, `getlogin`, `getlogin_r`, `getnameinfo`, `gets`, `gets_s`, `getseuserbyname`, `readlink`, `readlinkat`, `scanf`, `scanf_s`, `socket`, `wgetch`

- Default propagations rules:

  `atoi`, `atol`, `atoll`, `basename`, `dirname`, `fgetc`, `fgetln`, `fgets`, `fnmatch`, `fread`, `fscanf`, `fscanf_s`, `index`, `inflate`, `isalnum`, `isalpha`, `isascii`, `isblank`, `iscntrl`, `isdigit`, `isgraph`, `islower`, `isprint`, `ispunct`, `isspace`, `isupper`, `isxdigit`, `memchr`, `memrchr`, `sscanf`, `getc`, `getc_unlocked`, `getdelim`, `getline`, `getw`, `memcmp`, `memcpy`, `memmem`, `memmove`, `mbtowc`, `pread`, `qsort`, `qsort_r`, `rawmemchr`, `read`, `recv`, `recvfrom`, `rindex`, `strcasestr`, `strchr`, `strchrnul`, `strcasecmp`, `strcmp`, `strcspn`, `strncasecmp`, `strncmp`, `strndup`, `strndupa`, `strpbrk`, `strrchr`, `strsep`, `strspn`, `strstr`, `strtol`, `strtoll`, `strtoul`, `strtoull`, `tolower`, `toupper`, `ttyname`, `ttyname_r`, `wctomb`, `wcwidth`

- Default sinks:

  `printf`, `setproctitle`, `system`, `popen`, `execl`, `execle`, `execlp`, `execv`, `execvp`, `execvP`, `execve`, `dlopen`, `memcpy`, `memmove`, `strncpy`, `strndup`, `malloc`, `calloc`, `alloca`, `memccpy`, `realloc`, `bcopy`

Please note that there are no built-in filter functions.

One can configure their own taint sources, sinks, and propagation rules by providing a configuration file via checker option `alpha.security.taint.TaintPropagation:Config`. The configuration file is in [YAML](http://llvm.org/docs/YamlIO.html#introduction-to-yaml) format. The taint-related options defined in the config file extend but do not override the built-in sources, rules, sinks. The format of the external taint configuration file is not stable, and could change without any notice even in a non-backward compatible way.

For a more detailed description of configuration options, please see the [Taint Analysis Configuration](https://clang.llvm.org/docs/analyzer/user-docs/TaintAnalysisConfiguration.html). For an example see [Example configuration file](https://clang.llvm.org/docs/analyzer/user-docs/TaintAnalysisConfiguration.html#clangsa-taint-configuration-example).

**Configuration**

- Config Specifies the name of the YAML configuration file. The user can define their own taint sources and sinks.

**Related Guidelines**

- [CWE Data Neutralization Issues](https://cwe.mitre.org/data/definitions/137.html)
- [SEI Cert STR02-C. Sanitize data passed to complex subsystems](https://wiki.sei.cmu.edu/confluence/display/c/STR02-C.+Sanitize+data+passed+to+complex+subsystems)
- [SEI Cert ENV33-C. Do not call system()](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152177)
- [ENV03-C. Sanitize the environment when invoking external programs](https://wiki.sei.cmu.edu/confluence/display/c/ENV03-C.+Sanitize+the+environment+when+invoking+external+programs)

**Limitations**

- The taintedness property is not propagated through function calls which are unknown (or too complex) to the analyzer, unless there is a specific propagation rule built-in to the checker or given in the YAML configuration file. This causes potential true positive findings to be lost.

## `alpha.unix`

### `alpha.unix.BlockInCriticalSection` (C)

Check for calls to blocking functions inside a critical section. Applies to: `lock, unlock, sleep, getc, fgets, read, recv, pthread_mutex_lock,` `` pthread_mutex_unlock, mtx_lock, mtx_timedlock, mtx_trylock, mtx_unlock, lock_guard, unique_lock``

```c++
void test() {
  std::mutex m;
  m.lock();
  sleep(3); // warn: a blocking function sleep is called inside a critical
            //       section
  m.unlock();
}
```

### `alpha.unix.Chroot` (C)

Check improper use of chroot.

```c++
void f();

void test() {
  chroot("/usr/local");
  f(); // warn: no call of chdir("/") immediately after chroot
}
```

### `alpha.unix.PthreadLock` (C)

Simple lock -> unlock checker. Applies to: `pthread_mutex_lock, pthread_rwlock_rdlock, pthread_rwlock_wrlock, lck_mtx_lock, lck_rw_lock_exclusive` `lck_rw_lock_shared, pthread_mutex_trylock, pthread_rwlock_tryrdlock, pthread_rwlock_tryrwlock, lck_mtx_try_lock, lck_rw_try_lock_exclusive, lck_rw_try_lock_shared, pthread_mutex_unlock, pthread_rwlock_unlock, lck_mtx_unlock, lck_rw_done`.

```c++
pthread_mutex_t mtx;

void test() {
  pthread_mutex_lock(&mtx);
  pthread_mutex_lock(&mtx);
    // warn: this lock has already been acquired
}

lck_mtx_t lck1, lck2;

void test() {
  lck_mtx_lock(&lck1);
  lck_mtx_lock(&lck2);
  lck_mtx_unlock(&lck1);
    // warn: this was not the most recently acquired lock
}

lck_mtx_t lck1, lck2;

void test() {
  if (lck_mtx_try_lock(&lck1) == 0)
    return;

  lck_mtx_lock(&lck2);
  lck_mtx_unlock(&lck1);
    // warn: this was not the most recently acquired lock
}
```

### `alpha.unix.SimpleStream` (C)

Check for misuses of stream APIs.

Check for misuses of stream APIs: `fopen, fclose` (demo checker, the subject of the demo ([Slides](https://llvm.org/devmtg/2012-11/Zaks-Rose-Checker24Hours.pdf) , [Video](https://youtu.be/kdxlsP5QVPw)) by Anna Zaks and Jordan Rose presented at the [2012 LLVM Developers’ Meeting](https://llvm.org/devmtg/2012-11/)).

```c++
void test() {
  FILE *F = fopen("myfile.txt", "w");
} // warn: opened file is never closed

void test() {
  FILE *F = fopen("myfile.txt", "w");

  if (F)
    fclose(F);

  fclose(F); // warn: closing a previously closed file stream
}
```

### `alpha.unix.Stream` (C)

Check stream handling functions: `fopen, tmpfile, fclose, fread, fwrite, fseek, ftell, rewind, fgetpos,` `fsetpos, clearerr, feof, ferror, fileno`.

```c++
void test() {
  FILE *p = fopen("foo", "r");
} // warn: opened file is never closed

void test() {
  FILE *p = fopen("foo", "r");
  fseek(p, 1, SEEK_SET); // warn: stream pointer might be NULL
  fclose(p);
}

void test() {
  FILE *p = fopen("foo", "r");

  if (p)
    fseek(p, 1, 3);
     // warn: third arg should be SEEK_SET, SEEK_END, or SEEK_CUR

  fclose(p);
}

void test() {
  FILE *p = fopen("foo", "r");
  fclose(p);
  fclose(p); // warn: already closed
}

void test() {
  FILE *p = tmpfile();
  ftell(p); // warn: stream pointer might be NULL
  fclose(p);
}
```

### `alpha.unix.cstring.BufferOverlap` (C)

Checks for overlap in two buffer arguments.

Applies to: `memcpy, mempcpy, wmemcpy, wmempcpy`.

```c++
void test() {
  int a[4] = {0};
  memcpy(a + 2, a + 1, 8); // warn
}
```

### `alpha.unix.cstring.NotNullTerminated` (C)

Check for arguments which are not null-terminated strings.

Applies to: `strlen, strnlen, strcpy, strncpy, strcat, strncat, wcslen, wcsnlen`.

```c++
void test() {
  int y = strlen((char *)&test); // warn
}
```

### `alpha.unix.cstring.OutOfBounds` (C)

Check for out-of-bounds access in string functions, such as: `memcpy, bcopy, strcpy, strncpy, strcat, strncat, memmove, memcmp, memset` and more.

This check also works with string literals, except there is a known bug in that the analyzer cannot detect embedded NULL characters when determining the string length.

```c++
void test1() {
  const char str[] = "Hello world";
  char buffer[] = "Hello world";
  memcpy(buffer, str, sizeof(str) + 1); // warn
}

void test2() {
  const char str[] = "Hello world";
  char buffer[] = "Helloworld";
  memcpy(buffer, str, sizeof(str)); // warn
}
```

### `alpha.unix.cstring.UninitializedRead` (C)

Check for uninitialized reads from common memory copy/manipulation functions such as:

`memcpy, mempcpy, memmove, memcmp, strcmp, strncmp, strcpy, strlen, strsep` and many more.

```c++
void test() {
 char src[10];
 char dst[5];
 memcpy(dst,src,sizeof(dst)); // warn: Bytes string function accesses uninitialized/garbage values
}
```

Limitations:

Due to limitations of the memory modeling in the analyzer, one can likely observe a lot of false-positive reports like this:

> ```c++
> void false_positive() {
>   int src[] = {1, 2, 3, 4};
>   int dst[5] = {0};
>   memcpy(dst, src, 4 * sizeof(int)); // false-positive:
>   // The 'src' buffer was correctly initialized, yet we cannot conclude
>   // that since the analyzer could not see a direct initialization of the
>   // very last byte of the source buffer.
> }
> ```

More details at the corresponding [GitHub issue](https://github.com/llvm/llvm-project/issues/43459).

### `alpha.nondeterminism.PointerIteration` (C++)

Check for non-determinism caused by iterating unordered containers of pointers.

```c++
void test() {
 int a = 1, b = 2;
 std::unordered_set<int *> UnorderedPtrSet = {&a, &b};

 for (auto i : UnorderedPtrSet) // warn
   f(i);
}
```

### `alpha.nondeterminism.PointerSorting` (C++)

Check for non-determinism caused by sorting of pointers.

```c++
void test() {
 int a = 1, b = 2;
 std::vector<int *> V = {&a, &b};
 std::sort(V.begin(), V.end()); // warn
}
```

## `alpha.WebKit`

### `alpha.webkit.UncountedCallArgsChecker`

The goal of this rule is to make sure that lifetime of any dynamically allocated ref-countable object passed as a call argument spans past the end of the call. This applies to call to any function, method, lambda, function pointer or functor. Ref-countable types aren’t supposed to be allocated on stack so we check arguments for parameters of raw pointers and references to uncounted types.

Here are some examples of situations that we warn about as they might be potentially unsafe. The logic is that either we’re able to guarantee that an argument is safe or it’s considered if not a bug then bug-prone.

```c++
RefCountable* provide_uncounted();
void consume(RefCountable*);

// In these cases we can't make sure callee won't directly or indirectly call `deref()` on the argument which could make it unsafe from such point until the end of the call.
 
void foo1() {
  consume(provide_uncounted()); // warn
}

void foo2() {
  RefCountable* uncounted = provide_uncounted();
  consume(uncounted); // warn
}
```

Although we are enforcing member variables to be ref-counted by `webkit.NoUncountedMemberChecker` any method of the same class still has unrestricted access to these. Since from a caller’s perspective we can’t guarantee a particular member won’t get modified by callee (directly or indirectly) we don’t consider values obtained from members safe.

Note: It’s likely this heuristic could be made more precise with fewer false positives - for example calls to free functions that don’t have any parameter other than the pointer should be safe as the callee won’t be able to tamper with the member unless it’s a global variable.

```c++
struct Foo {
  RefPtr<RefCountable> member;
  void consume(RefCountable*) { /* ... */ }
  void bugprone() {
    consume(member.get()); // warn
  }
};
```

The implementation of this rule is a heuristic - we define a whitelist of kinds of values that are considered safe to be passed as arguments. If we can’t prove an argument is safe it’s considered an error.

Allowed kinds of arguments:

- values obtained from ref-counted objects (including temporaries as those survive the call too)

  ```c++
  RefCountable* provide_uncounted();
  void consume(RefCountable*);
  
  void foo() {
    RefPtr<RefCountable> rc = makeRef(provide_uncounted());
    consume(rc.get()); // ok
    consume(makeRef(provide_uncounted()).get()); // ok
  }
  ```

- forwarding uncounted arguments from caller to callee

  ```c++
  void foo(RefCountable& a) {
    bar(a); // ok
  }
  ```

  Caller of `foo()` is responsible for `a`’s lifetime.

- `this` pointer

  ```c++
  void Foo::foo() {
    baz(this);  // ok
  }
  ```

  Caller of `foo()` is responsible for keeping the memory pointed to by `this` pointer safe.

- constants

  ```c++
  foo(nullptr, NULL, 0); // ok
  ```

We also define a set of safe transformations which if passed a safe value as an input provide (usually it’s the return value) a safe value (or an object that provides safe values). This is also a heuristic.

- constructors of ref-counted types (including factory methods)
- getters of ref-counted types
- member overloaded operators
- casts
- unary operators like `&` or `*`

### `alpha.webkit.UncountedLocalVarsChecker`

The goal of this rule is to make sure that any uncounted local variable is backed by a ref-counted object with lifetime that is strictly larger than the scope of the uncounted local variable. To be on the safe side we require the scope of an uncounted variable to be embedded in the scope of ref-counted object that backs it.

These are examples of cases that we consider safe:

```c++
void foo1() {
  RefPtr<RefCountable> counted;
  // The scope of uncounted is EMBEDDED in the scope of counted.
  {
    RefCountable* uncounted = counted.get(); // ok
  }
}

void foo2(RefPtr<RefCountable> counted_param) {
  RefCountable* uncounted = counted_param.get(); // ok
}

void FooClass::foo_method() {
  RefCountable* uncounted = this; // ok
}
```

Here are some examples of situations that we warn about as they might be potentially unsafe. The logic is that either we’re able to guarantee that an argument is safe or it’s considered if not a bug then bug-prone.

```c++
void foo1() {
  RefCountable* uncounted = new RefCountable; // warn
}

RefCountable* global_uncounted;
void foo2() {
  RefCountable* uncounted = global_uncounted; // warn
}

void foo3() {
  RefPtr<RefCountable> counted;
  // The scope of uncounted is not EMBEDDED in the scope of counted.
  RefCountable* uncounted = counted.get(); // warn
}
```

We don’t warn about these cases - we don’t consider them necessarily safe but since they are very common and usually safe we’d introduce a lot of false positives otherwise: - variable defined in condition part of an ``if`` statement - variable defined in init statement condition of a ``for`` statement.

For the time being we also don’t warn about uninitialized uncounted local variables.

# Debug Checkers

## `debug`

Checkers used for debugging the analyzer. Debug Checks page contains a detailed description.

### `debug.AnalysisOrder`

Print callbacks that are called during analysis in order.

### `debug.ConfigDumper`

Dump config table.

### `debug.DumpCFG Display`

Control-Flow Graphs.

### `debug.DumpCallGraph`

Display Call Graph.

### `debug.DumpCalls`

Print calls as they are traversed by the engine.

### `debug.DumpDominators`

Print the dominance tree for a given CFG.

### `debug.DumpLiveVars`

Print results of live variable analysis.

### `debug.DumpTraversal`

Print branch conditions as they are traversed by the engine.

### `debug.ExprInspection`

Check the analyzer’s understanding of expressions.

### `debug.Stats`

Emit warnings with analyzer statistics.

### `debug.TaintTest`

Mark tainted symbols as such.

### `debug.ViewCFG`

View Control-Flow Graphs using GraphViz.

### `debug.ViewCallGraph`

View Call Graph using GraphViz.

### `debug.ViewExplodedGraph`

View Exploded Graphs using GraphViz.
