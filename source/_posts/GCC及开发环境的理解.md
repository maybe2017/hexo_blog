---
title: GCC及开发环境的理解
date: 2023-07-21 17:22:16
comments: true
layout: post
toc: true
tags:
  - C/C++
categories: 开发语言基础
---

### 一、不同的操作系统

1. 计算机世界里存在各种各样的操作系统，目前通用操作系统有主流的三大类：
   - UNIX，通用操作系统鼻祖，发展出特别多衍生系统；
   - Windows，微软家根基，桌面市场霸主；
   - GNU/Linux，UNIX 近亲，有各种发行版如 Ubuntu、CentOS 等。

​ 这些系统有各自的内核，出于系统稳定性考虑是不允许用户程序直接操作内核，同时也将内核开发和应用软件开发隔离开来，系统将**必要的函数封装成函数库**供应用软件调用，约定的规范即为应用软件接口(Application Program Interface，**API**)。

​ <img src="https://raw.githubusercontent.com/maybe2017/image/main/img/API%E5%8F%8A%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8.png" alt="API及系统调用" style="zoom: 50%;" />

2. API 函数库是连接用户软件和系统内核桥梁，或者是“协议”，操作系统厂商写好函数库说明书，应用软件开发者不必关心其内部是如何实现的，用的时候对照着 API 手册查询就够了；应用软件也可以越过系统函数库通过 system call（系统调用）直接调用 os 内核函数，如图中红色虚线所示，这种方式是进入到内核空间操作，并不被推荐，一般是调用系统函数 API 在用户空间操作。

3. 如果各系统平台都能提供相同的系统函数库，那么开发者在这个系统函数库基础之上编写软件代码，那么就很容易将软件移植到各个系统平台。**然而，这只是个美好的愿望**，IEEE 就是为了达成这样的愿望才牵头制定 POSIX 标准。POSIX 标准主要就是针对 UNIX API 而制订，**不管函数如何包装、功能如何实现，但 API 按照标准约定来（比如函数变量等符号名称、数据结构、参数类型与个数、基本工具命令名称等）**，Linux 完全兼容 POSIX 标准（部分函数符合 POSIX，部分函数是 Linux 专有，即是 POSIX 的超集），微软只是声称 Windows 部分兼容 POSIX 标准。

### 二、什么是 GNU？

1. GNU 计划，又称革奴计划，GNU is Not Unix，发音革奴；由于 GNU 将要实现 UNIX 系统（UNIX 是一种广泛使用的商业操作系统的名称）的接口标准，因此 GNU 计划可以分别开发不同的操作系统部件。
2. **GNU**是一个操作系统，但是其内核发展尚未成熟，在实际使用上，多半使用`Linux内核`、`FreeBSD`等作为替代系统核心。其中主要的操作系统是 Linux 的发行版。Linux 操作系统包涵了`Linux内核`与`其他自由软件项目中的GNU组件和软件`，可以被称为`GNU/Linux`。
3. 该系统的基本组成包括`GNU编译器套装GCC`、GNU 的 C 库 `glibc`、以及 GNU 核心工具组 `coreutils`等。

### 三、GCC 是什么？Cygwin、MinGW 又是什么

1. GUN Compiler Collection ，**类 UNIX 系统下**，编译器工具集合，用于给 C++、C、Java、Python 等语言进行编译，输出可执行文件的编译器套件。
2. 我们平时使用的 IDE = 代码编辑器 + 编译器 + 调试器 + .. 相当于 IDE 已经完成了这些工具的集成，不需要我们用敲命令的方式去调用编译器编译源码，而可以用一个按钮点击取代。
3. 那么 Windows 环境下，如何使用 UNIX 的编译器呢？

   - **MinGW**：为 Windows 环境提供的 最小的一套 GNU 的工具集合（不仅仅包含了 GNU 编译器）。一般可以狭义的理解为 **Win 版的 GCC**。它是一些<span style="color:red;">头文件和端口库的集合</span>，允许你在 Windows 平台生成本地的 Windows 程序而不需要第三方 C 运行时(C Runtime)库。

- Cygwin: 是一个可原生运行于 Windows 系统上的 符合 POSIX 接口规范 的兼容环境（子系统） ；效率相对低，较臃肿，对于日常开发来说，用 MinGW 就够了。

4. 运行时库：支持程序运行的基本函数的集合，一般是静态库 lib 或动态库 dll。

### 四、用 MingW 和 Cygwin 编译出来的程序的区别

1. 首先 MingW 和 Cygwin 都不能让 Linux 下的程序直接运行在 Windows 上，必需通过源代码重新编译。
2. Windows 和 Linux 的基本设计概念像进程线程地址空间虚拟内存这些都是大同小异的，但是为啥程序不能兼容？是因为在一些具体实现上存在差异，比如:
   - 首先是可执行文件的格式，Window 使用 PE 的格式，并且要求以 .EXE 为后缀名；Linux 则使用 Elf。
   - 其次操作系统的 API 也不一样，如 Windows 用 `CreateProcess()` 创建进程，而 Linux 使用 `fork()`。
3. MingW 有专门的 W32api 头文件，来把代码中 Linux 方式的`系统调用`替换为对应的 Windows 方式系统调用。而 Cygwin 则通过 cygwin1.dll 这个文件来实现这种 API 的转换，并模拟一个 Linux 系统调用接口给程序，程序依然以 Linux 的方式调用系统 API，只不过这个 API 在 cygwin1.dll 上，cygwin1.dll 再调用 Windows 对应的实现，来把结果返回给程序。
4. 二者生成的程序都是能在 Windows 上运行的 EXE 文件，显然都是 PE 格式，用一个 PE 格式查看工具检查一下就能发现，Cygwin 生成的程序依然有 `fork()` 这样的 Linux 系统调用，但目标库是 cygwin1。而 MingW 生成的程序，则全部使用从 KERNEL32 导出的标准 Windows 系统 API。

### 五、MinGW64 的下载安装

1. 地址：https://www.mingw-w64.org/ ，下载 x86_64_seh-ucrt.7z
2. 解压后不要忘记配置系统环境变量。

### 六、MSVC 是什么？

1. Microsoft Visual C++（简称 Visual C++、MSVC、VS 或 VC）是微软公司的免费 C++开发工具，具有集成开发环境，可提供编辑 C 语言，C++以及 C++/CLI 等。VC++集成了便利的除错工具，特别是集成了微软 Windows 视窗操作系统应用程序接口`Windows API`、三维动画 DirectX API，Microsoft .NET 框架（说白了就是一些运行时库）

2. 与 Linux 系列操作系统不同，Windows 原生环境不提供类似 GCC，Clang 的 C/C++语言源程序编译运行工具链。运行在 Windows 上的 IDE（集成开发环境），比如 VSCode、CLion 之类，一般都使用 MinGW 配置，模拟 Linux 下的开发环境来在 Windows 环境下进行开发。
3. 但是在 Windows 下，与开发环境以及 code 编辑器协同更好的还是**MSVC**（Microsoft Visual C/C++）编译器。说白了，最好用将该编译器集成到代码编辑器中（Visual Studio IDE 已集成）。
4. MSVC 编译器工具链主要由**cl.exe**与**link.exe**构成；**cl.exe**用于控制在 Microsoft C/C++的编译器和链接器，**link.exe** 将通用对象文件格式 (COFF) 对象文件和库链接起来，以创建可执行 (.exe) 文件或动态链接库 (DLL)。

5. 一般来说，获取 MSVC 要通过 Microsoft Visual Studio 来实现。对于一般的应用场景来说，我们不需要下载完整的 IDE 而是只需要下载单个组件下的 MSVC C++ 生成工具，以及 Windows 10 SDK 即可。（**MSVC C++生成工具包含了全部编译链接工具链以及 <span style="color:red;">大部分函数库与头文件</span>**；Windows 10 SDK 只是提供其它的一些必要库文件与头文件）

### 七、应该用什么编译器环境呢？

1. 编译方式有多种，就拿 MinGW 与 MSVC 来说，它们都是编译工具，但是两者的**兼容性并不好**；当你的项目使用 MinGW 编译的使用，想要用一个 MSVC 编译生成的库时就会有问题。使用 MinGW 编译项目的时候，所使用的 Lib 也要是 MinGW 编译的。
2. 如果你只是开发 Window 平台的软件时，最好用 Qt MSVC 组合，这样可以使用大量的微软第三方 lib，还有很多的构建指令；若考虑后期跨平台的问题，则使用 MinGw。

### 八、gcc 与 g++的区别

1. gcc 和 g++都是 GNU(组织)的一个编译器；更准确的说法是：gcc 调用了 C compiler，而 g++调用了 C++ compiler

2. 重要区别例如：

   - 源文件后缀为.c 的, gcc 把它当作是 c 源代码处理，而 g++当作是 c++源代码处理；
   - 使用 g++编译文件时，**g++会自动链接标准库 STL，而 gcc 不会自动链接 STL**;
   - 在用 gcc 编译 c++文件时，为了能够使用 STL，需要加参数 –lstdc++ ，但这并不代表 gcc –lstdc++ 和 g++等价，它们的区别不仅仅是这个。

3. 总结:

   - 编译可以用 gcc/g++，而链接可以用 g++或者 gcc -lstdc++；
   - gcc 命令不能自动和 C++程序使用的库联接．所以通常使用 g++来完成联接。但在编译阶段， gcc 会自动调用 g++，二者等价；

4. 编译器对源码操作的阶段示下：

   <img src="http://rxa6ee5rh.hn-bkt.clouddn.com/GCC%E9%98%B6%E6%AE%B5.png?e=1688490335&token=B8_UN8ODkL2O4SfPVX70gtgnZ5FOGg9QeRVOOgB0:XB_8vRcXj3V39NfDDsj7qzTKsAk" style="zoom: 200%;" />
