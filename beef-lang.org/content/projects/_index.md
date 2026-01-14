+++
title = "项目与工作区"
alwaysopen = true
weight = 41
+++

## 通用宏

- $(BeefPath) - Beef 安装路径
- $(Slash str) - 将输入字符串转换为斜杠路径
- $(Var varName) - 变量 'varName' 的内容
- $(VSToolPath) - Visual Studio 工具路径
- $(VSToolPath_x64) - x64 的 Visual Studio 工具路径
- $(VSToolPath_x86) - x86 的 Visual Studio 工具路径

## 工作区宏

- $(Configuration) - 当前选中的构建配置
- $(Platform) - 当前选中的构建平台
- $(WorkspaceDir) - 工作区目录

## 项目宏

- $(Arguments) - 调试时的命令行参数
- $(BuildDir) - 构建目录，包含对象文件等构建产物
- $(LinkFlags) - 默认链接参数
- $(ProjectDir) - 包含该项目的目录
- $(ProjectName) - 项目名称
- $(TargetDir) - 目标二进制目录
- $(TargetPath) - 该配置构建的目标二进制路径
- $(WorkingDir) - 调试时的工作目录

项目宏可指定项目名，例如 "$(BuildDir LibA)"。

## 预构建/后构建命令
- CopyFilesIfNewer(srcPath, destPath) - 仅当源文件更新时复制文件（递归，支持通配符）
- CopyToDependents(srcPath) - 对依赖该库的项目，将这些文件复制到其目标目录
- CreateFile(path, text) - 将 'text' 写入文件 'path'
- DeleteFile(path) - 删除文件 'path'
- DelTree(path) - 递归删除路径
- Echo(text) - 将 'text' 输出到 Output 窗口或控制台
- ReadFile(path, varName) - 读取文件 'path' 内容并赋值给变量 'varName'
- RenameFile(srcPath, destPath) - 重命名文件 'srcPath' 为 'destPath'
- SetVal(varName, value) - 设置变量 'varName' 为 'value'
- ShowFile(path) - 在 IDE 编辑器中打开 'path'
- Sleep(timeMS) - 延迟 timeMS 毫秒
