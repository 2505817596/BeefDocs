+++
title = "平台"
alwaysopen = true
weight = 42
+++

### 一级平台：
以下平台名称被 Beef 原生识别，Beef 可作为这些平台的完整构建系统。这些平台之间不支持交叉编译，因为它们都需要访问系统原生的链接器与库。

- Win32 - Windows x86
- Win64 - Windows x64
- Linux32 - Linux x86
- Linux64 - Linux x64
- macOS - macOS x64

### 二级平台：
这些是由 LLVM 目标三元组表示的跨平台目标。在这些平台上，Beef 编译器由外部构建流程调用，生成可被外部构建流程链接进最终目标的 lib 文件。Beef 运行时需要事先为该平台编译好，并通过外部构建流程一并链接。

- iOS - (early alpha) Supported architectures include AArch64 and x64-based simulators. The Beef runtime can be built on macOS via bin/build_ios.sh.
- Android - (early alpha) Supported architectures include AArch64, ARM7, and x86/x64-based simulators. The Beef runtime can be built on Windows via bin/build_android.bat.

### 三级平台：
与二级平台类似，这些平台也由 LLVM 目标三元组表示，但没有内置的 Beef 运行时支持。尽管 Beef 编译器可以为这些架构生成机器码，但根据 corlib 的使用情况，运行时可能会缺少某些符号。此列表可轻松扩展为更多 LLVM 支持的目标，但相比扩展一级与二级平台，收益有限。

- arm7 - arm7-*
- aarch64 - aarch64-*, arm64-*
- x86 - x86-*, i686-*
- x64 - x86_64-* 
