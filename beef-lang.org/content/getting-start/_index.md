+++
title = "入门指南"
description = ""
weight = 2
alwaysopen = true
+++

## 欢迎使用 Beef

Beef 主要以 IDE 方式使用，但也支持命令行构建。第一步可以 [安装 Beef]({{< ref "getting-start/installation.md" >}}) 或 [从源码构建]({{< ref "getting-start/building.md" >}})。目前仅提供 Windows 的二进制包，且 Beef IDE 也只支持 Windows。

### 支持的平台

Windows 提供二进制包，Windows、Linux 和 macOS 支持从源码构建。包含 Android 和 iOS 在内的目标平台 [交叉编译]({{< ref "platforms/_index.md" >}}) 正在开发中。

### 使用 IDE 创建“Hello World”

* 启动 Beef IDE
* 选择 File/New/Project 新建项目，创建名为“Hello”的 Console 项目
* 右键新项目，选择“New Class...”，名称输入“Program”
* 在新建文件中输入以下内容

```C#
using System;

namespace Hello
{
	class Program
	{
    	static void Main()
    	{
	        Console.WriteLine("Hello, world!");
    	}
	}
}
```
* 按 F5 编译并运行

### 使用命令行创建“Hello World”

* 在任意位置创建名为“Hello”的目录
* 在终端中进入该目录
* 运行 `BeefBuild -new` 在该目录初始化新的工作区和项目
* 创建 `src/Program.bf` 文本文件，并粘贴上方代码
* 运行 `BeefBuild -run` 编译并执行
