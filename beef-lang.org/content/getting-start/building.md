+++
title = "从源码构建"
weight = 2
+++

## 构建概览

对于已有二进制发行版的平台（如 Windows），无需从源码构建。

源码位于 https://github.com/beefytech/Beef。

### 引导构建

Beef 编译器核心使用 C++ 编写，而 IDE 与命令行构建系统 BeefBuild 使用 Beef 本身编写。为实现引导构建，Beef 提供了一个最小引导编译器，其唯一职责是先构建一次 BeefBuild，随后由 BeefBuild 再进行“正式”的自举构建。

---

### 在 Windows 上构建

#### 要求

* Visual Studio 2017 或更高版本的 Microsoft C++ 构建工具。可仅安装 Microsoft Visual C++ Build Tools，也可安装完整的 Visual Studio 套件：https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022。
* Platform Toolset 141 (VS2017)
* Windows SDK 10.0.17763.0
* CMake 3.15 or newer
* Python 3.6 or newer
* Git command line tools

#### 构建步骤
* 执行 bin/build.bat

注意：该过程会先下载并构建 LLVM，需要一些时间。
构建产物位于 IDE/dist。

---

### 在 Linux 和 macOS 上构建

#### 要求

* CMake 3.15 or newer
* LLVM-18
* Git

### 构建步骤

* 使用 bin/build.sh 构建 Beef

构建产物位于 IDE/dist。

请注意，这些平台支持 BeefBuild 等 CLI 工具，但 IDE 目前仅支持 Windows。
