---
title: "Clang-Tidy"
date: 2023-11-21T02:56:25+08:00
description: "Clang-Tidy"
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

# Clang-Tidy

clang-tidy is a clang-based C++ “linter” tool. Its purpose is to provide an extensible framework for diagnosing and fixing typical programming errors, like style violations, interface misuse, or bugs that can be deduced via static analysis. clang-tidy is modular and provides a convenient interface for writing new checks.

>clang-tidy是一个基于clang的C++“linter”工具。它的目的是提供一个可扩展的框架，用于诊断和修复典型的编程错误，如样式违反、接口误用或可以通过静态分析推断出来的错误。clang-tidy是模块化的，并为编写新检查提供了方便的接口。

## Using Clang-Tidy

clang-tidy is a LibTooling-based tool, and it’s easier to work with if you set up a compile command database for your project (for an example of how to do this, see How To Setup Tooling For LLVM).

>clang-tidy是一个基于LibTooling的工具，如果你给你的项目设置**编译命令**数据库，它会更容易使用（有关如何这样做的示例，请参阅如何为LLVM设置工具）。

You can also specify compilation options on the command line after `--`:

```powershell
clang-tidy test.cpp -- -Imy_project/include -DMY_DEFINES ...
```

clang-tidy has its own checks and can also run Clang Static Analyzer checks. Each check has a name and the checks to run can be chosen using the `-checks=` option, which specifies a comma-separated list of positive and negative (prefixed with `-`) globs. Positive globs add subsets of checks, and negative globs remove them.

>clang-tidy有自己的检查，也可以运行Clang静态分析器检查。
>
>每个检查都有一个名称，可以使用`-checks=`选项选择要运行的检查，该选项指定一个逗号分隔的阳性和阴性列表（前缀为`-`）。正globs添加检查的子集，负globs删除它们。

For example,

```c++
clang-tidy test.cpp -checks=-*,clang-analyzer-*,-clang-analyzer-cplusplus*
```

will **disable** all default checks (`-*`) and **enable** all `clang-analyzer-*` checks **except** for `clang-analyzer-cplusplus*` ones.

The `-list-checks` option lists all the enabled checks. When used without `-checks=`, it shows checks enabled by default. Use `-checks=*` to see all available checks or with any other value of `-checks=` to see which checks are enabled by this value.

>`-list-checks`选项列出所有已启用的检查。当不使用`-checks=`时，它显示默认启用的检查。
>
>使用`-checks=*`查看所有可用的检查，或者使用`-checks=`的任何其他值查看该值启用了哪些检查。

There are currently the following groups of checks:

| Name prefix          | Description                                                  | Intro |
| :------------------- | :----------------------------------------------------------- | :---: |
| `abseil-`            | Checks related to Abseil library.                            |       |
| `altera-`            | Checks related to OpenCL programming for FPGAs.              |       |
| `android-`           | Checks related to Android.                                   |       |
| `boost-`             | Checks related to Boost library.                             |   *   |
| `bugprone-`          | Checks that target bug-prone code constructs.                |   *   |
| `cert-`              | Checks related to CERT Secure Coding Guidelines.             |       |
| `clang-analyzer-`    | Clang Static Analyzer checks.                                |   *   |
| `concurrency-`       | Checks related to concurrent programming (including threads, fibers, coroutines, etc.). |   *   |
| `cppcoreguidelines-` | Checks related to C++ Core Guidelines.                       |   *   |
| `darwin-`            | Checks related to Darwin coding conventions.                 |       |
| `fuchsia-`           | Checks related to Fuchsia coding conventions.                |       |
| `google-`            | Checks related to Google coding conventions.                 |       |
| `hicpp-`             | Checks related to High Integrity C++ Coding Standard.        |       |
| `linuxkernel-`       | Checks related to the Linux Kernel coding conventions.       |       |
| `llvm-`              | Checks related to the LLVM coding conventions.               |       |
| `llvmlibc-`          | Checks related to the LLVM-libc coding standards.            |       |
| `misc-`              | Checks that we didn’t have a better category for.            |   *   |
| `modernize-`         | Checks that advocate usage of modern (currently “modern” means “C++11”) language constructs. |   *   |
| `mpi-`               | Checks related to MPI (Message Passing Interface).           |       |
| `objc-`              | Checks related to Objective-C coding conventions.            |       |
| `openmp-`            | Checks related to OpenMP API.                                |       |
| `performance-`       | Checks that target performance-related issues.               |   *   |
| `portability-`       | Checks that target portability-related issues that don’t relate to any particular coding style. |   *   |
| `readability-`       | Checks that target readability-related issues that don’t relate to any particular coding style. |   *   |
| `zircon-`            | Checks related to Zircon kernel coding conventions.          |       |

Clang diagnostics are treated in a similar way as check diagnostics. Clang diagnostics are displayed by clang-tidy and can be filtered out using the `-checks=` option. However, the `-checks=` option does not affect compilation arguments, so it cannot turn on Clang warnings which are not already turned on in the build configuration. The `-warnings-as-errors=` option upgrades any warnings emitted under the `-checks=` flag to errors (but it does not enable any checks itself).

>Clang诊断的处理方式与检查诊断类似。Clang诊断信息由clang-tidy显示，可以使用`-checks=`选项过滤掉。但是，`-checks=`选项不会影响编译参数，因此它不能打开在构建配置中尚未打开的Clang警告。`-warnings-as-errors=`选项将`-checks=`标记下发出的任何警告升级为错误（但它本身不启用任何检查）。

Clang diagnostics have check names starting with `clang-diagnostic-`. Diagnostics which have a corresponding warning option, are named `clang-diagnostic-<warning-option>`, e.g. Clang warning controlled by `-Wliteral-conversion` will be reported with check name `clang-diagnostic-literal-conversion`.

The `-fix` flag instructs clang-tidy to fix found errors if supported by corresponding checks.

## Suppressing Undesired Diagnostics

clang-tidy diagnostics are intended to call out code that does not adhere to a coding standard, or is otherwise problematic in some way. However, if the code is known to be correct, it may be useful to silence the warning. Some clang-tidy checks provide a check-specific way to silence the diagnostics, e.g. bugprone-use-after-move can be silenced by re-initializing the variable after it has been moved out, bugprone-string-integer-assignment can be suppressed by explicitly casting the integer to char, readability-implicit-bool-conversion can also be suppressed by using explicit casts, etc.

If a specific suppression mechanism is not available for a certain warning, or its use is not desired for some reason, clang-tidy has a generic mechanism to suppress diagnostics using `NOLINT`, `NOLINTNEXTLINE`, and `NOLINTBEGIN … NOLINTEND` comments.

The NOLINT comment instructs clang-tidy to ignore warnings on the same line (it doesn’t apply to a function, a block of code or any other language construct; it applies to the line of code it is on). If introducing the comment on the same line would change the formatting in an undesired way, the `NOLINTNEXTLINE` comment allows suppressing clang-tidy warnings on the next line. The `NOLINTBEGIN` and `NOLINTEND` comments allow suppressing clang-tidy warnings on multiple lines (affecting all lines between the two comments).

All comments can be followed by an optional list of check names in parentheses (see below for the formal syntax). The list of check names supports globbing, with the same format and semantics as for enabling checks. Note: negative globs are ignored here, as they would effectively re-activate the warning.

For example:

```c++
class Foo {
  // Suppress all the diagnostics for the line
  Foo(int param); // NOLINT

  // Consider explaining the motivation to suppress the warning
  Foo(char param); // NOLINT: Allow implicit conversion from `char`, because <some valid reason>

  // Silence only the specified checks for the line
  Foo(double param); // NOLINT(google-explicit-constructor, google-runtime-int)

  // Silence all checks from the `google` module
  Foo(bool param); // NOLINT(google*)

  // Silence all checks ending with `-avoid-c-arrays`
  int array[10]; // NOLINT(*-avoid-c-arrays)

  // Silence only the specified diagnostics for the next line
  // NOLINTNEXTLINE(google-explicit-constructor, google-runtime-int)
  Foo(bool param);

  // Silence all checks from the `google` module for the next line
  // NOLINTNEXTLINE(google*)
  Foo(bool param);

  // Silence all checks ending with `-avoid-c-arrays` for the next line
  // NOLINTNEXTLINE(*-avoid-c-arrays)
  int array[10];

  // Silence only the specified checks for all lines between the BEGIN and END
  // NOLINTBEGIN(google-explicit-constructor, google-runtime-int)
  Foo(short param);
  Foo(long param);
  // NOLINTEND(google-explicit-constructor, google-runtime-int)

  // Silence all checks from the `google` module for all lines between the BEGIN and END
  // NOLINTBEGIN(google*)
  Foo(bool param);
  // NOLINTEND(google*)

  // Silence all checks ending with `-avoid-c-arrays` for all lines between the BEGIN and END
  // NOLINTBEGIN(*-avoid-c-arrays)
  int array[10];
  // NOLINTEND(*-avoid-c-arrays)
};
```

The formal syntax of `NOLINT`, `NOLINTNEXTLINE`, and `NOLINTBEGIN` … `NOLINTEND` is the following:

```
lint-comment:
  lint-command
  lint-command lint-args

lint-args:
  ( check-name-list )

check-name-list:
  check-name
  check-name-list , check-name

lint-command:
  NOLINT
  NOLINTNEXTLINE
  NOLINTBEGIN
  NOLINTEND
```

Note that whitespaces between `NOLINT`/`NOLINTNEXTLINE`/`NOLINTBEGIN`/`NOLINTEND` and the opening parenthesis are not allowed (in this case the comment will be treated just as `NOLINT`/`NOLINTNEXTLINE`/`NOLINTBEGIN`/`NOLINTEND`), whereas in the check names list (inside the parentheses), whitespaces can be used and will be ignored.

All `NOLINTBEGIN` comments must be paired by an equal number of `NOLINTEND` comments. Moreover, a pair of comments must have matching arguments – for example, `NOLINTBEGIN(check-name)` can be paired with `NOLINTEND(check-name)` but not with `NOLINTEND` (zero arguments). clang-tidy will generate a `clang-tidy-nolint` error diagnostic if any `NOLINTBEGIN`/`NOLINTEND` comment violates these requirements.

## Clang-Tidy Checks

refer to [Clang-Tidy Checks]({{< ref "clang-tidy-checks.md" >}}).

## Clang-Tidy IDE/Editor Integrations

Apart from being a standalone tool, clang-tidy is integrated into various IDEs, code analyzers, and editors. We recommend using clangd which integrates clang-tidy and is available in most major editors through plugins (Vim, Emacs, Visual Studio Code, Sublime Text and more).

>除了作为一个独立的工具，clang-tidy还被集成到各种IDE、代码分析器和编辑器中。
>
>我们推荐使用clangd，它集成了clang-tidy，并且可以通过插件（Vim, Emacs, Visual Studio Code, Sublime Text等）在大多数主流编辑器中使用。

> 注：VSCode本身自己集成了Clang-Tidy功能，无需安装clangd插件。

The following table shows the most well-known clang-tidy integrations in detail.

|                                     |        Feature        |                                |                         |                                       |                          |
| :---------------------------------- | :-------------------: | :----------------------------: | :---------------------: | :-----------------------------------: | :----------------------: |
| **Tool**                            | On-the-fly inspection | Check list configuration (GUI) | Options to checks (GUI) | Configuration via `.clang-tidy` files | Custom clang-tidy binary |
| A.L.E. for Vim                      |           +           |               -                |            -            |                   -                   |            +             |
| Clang Power Tools for Visual Studio |           -           |               +                |            -            |                   +                   |            -             |
| **Clangd**                          |           +           |               -                |            -            |                   +                   |            -             |
| CLion IDE                           |           +           |               +                |            +            |                   +                   |            +             |
| CodeChecker                         |           -           |               -                |            -            |                   -                   |            +             |
| CPPCheck                            |           -           |               -                |            -            |                   -                   |            -             |
| CPPDepend                           |           -           |               -                |            -            |                   -                   |            -             |
| Flycheck for Emacs                  |           +           |               -                |            -            |                   +                   |            +             |
| KDevelop IDE                        |           -           |               +                |            +            |                   +                   |            +             |
| Qt Creator IDE                      |           +           |               +                |            -            |                   +                   |            +             |
| ReSharper C++ for Visual Studio     |           +           |               +                |            -            |                   +                   |            +             |
| Syntastic for Vim                   |           +           |               -                |            -            |                   -                   |            +             |
| Visual Assist for Visual Studio     |           +           |               +                |            -            |                   -                   |            -             |
| **Visual Studio Code**              |           ?           |               ?                |            ?            |                   +                   |            ?             |

### IDEs

#### CLion

CLion 2017.2 and later integrates clang-tidy as an extension to the built-in code analyzer. Starting from 2018.2 EAP, CLion allows using clang-tidy via Clangd. Inspections and applicable quick-fixes are performed on the fly, and checks can be configured in standard command line format. In this integration, you can switch to the clang-tidy binary different from the bundled one, pass the configuration in .clang-tidy files instead of using the IDE settings, and configure options for particular checks.

#### KDevelop

KDevelop with the kdev-clang-tidy plugin, starting from version 5.1, performs static analysis using clang-tidy. The plugin launches the clang-tidy binary from the specified location and parses its output to provide a list of issues.

#### QtCreator

QtCreator 4.6 integrates clang-tidy warnings into the editor diagnostics under the Clang Code Model. To employ clang-tidy inspection in QtCreator, you need to create a copy of one of the presets and choose the checks to be performed. Since QtCreator 4.7 project-wide analysis is possible with the Clang Tools analyzer.

#### Visual Studio

MS Visual Studio can integrate clang-tidy by means of three different tools. The ReSharper C++ extension, version 2017.3 and later, provides seamless clang-tidy integration: checks and quick-fixes run alongside native inspections. Apart from that, ReSharper C++ incorporates clang-tidy as a separate step of its code clean-up process. Visual Assist build 2210 includes a subset of clang-tidy checklist to inspect the code as you edit. Another way to bring clang-tidy functionality to Visual Studio is the Clang Power Tools plugin, which includes most of the clang-tidy checks and runs them during compilation or as a separate step of code analysis.

### Editors

#### GNU Emacs

Emacs24, when expanded with the Flycheck plugin, incorporates the clang-tidy inspection into the syntax analyzer.

#### Vim

For Vim, you can use Syntastic, which includes clang-tidy, or A.L.E., a lint engine that applies clang-tidy along with other linters.

#### Visual Studio Code

在Visual Studio Code配置使用Clang-Tidy十分简单。

**STEP 01**

以Windows为例，安装LLVM，并将clang-tidy所在路径添加到系统变量路径。

**STEP 02**

在`vscode/setting.json`中添加如下配置：

```json
"C_Cpp.codeAnalysis.clangTidy.enabled": true,
"C_Cpp.codeAnalysis.clangTidy.useBuildPath": true,
"C_Cpp.codeAnalysis.runAutomatically": false, // optional, but recommended
```

**STEP 03**

在项目的根目录创建`.clang-tidy`文件，参考Github其他项目的配置或者自己编辑，录入检查的规则。

**STEP 04**

最好重启下Visual Studio Code，首先将项目编译一下，然后通过Ctrl+P调出VSCode命令提示框，选择“C/C++: Run Code Analysis on All Files”，即可对项目中的全部文件进行规则检查。根据项目的大小，这可能会不同长度的一段时间。

**STEP 05**

Clang-Tidy检查完成后，会给出问题列表，根据列表修改自己的源文件，或者视情况打开或者屏蔽某些规则。

### Analyzers

#### CPPDepend

clang-tidy is integrated in CPPDepend starting from version 2018.1 and CPPCheck 1.82. CPPCheck integration lets you import Visual Studio solutions and run the clang-tidy inspection on them. The CodeChecker application of version 5.3 or later, which also comes as a plugin for Eclipse, supports clang-tidy as a static analysis instrument and allows to use a custom clang-tidy binary.