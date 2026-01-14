+++
title = "预处理器"
weight=75
+++

## 预处理器
Beef 预处理器用于在解析器运行前按条件包含文本块并修改警告选项。它比 C 预处理器灵活性低，不能用于实现宏或其他“代码生成”用途。

* #define <X> - 将符号 "X" 设为 true
* #endif - 结束 #if、#else 或 #elif 块
* #else - 若前一 #if/#elif 为 false，则处理直到下一个 #endif
* #elif <X> - 若前一 #if/#elif 为 false 且 "X" 为 true，则处理直到下一个 #endif
* #error <Message> - 在解析时创建错误
* #if <X> - 若 "X" 为 true，则处理直到下一个 #endif
* #pragma format disable - 禁用格式化
* #pragma format restore - 恢复格式化
* #pragma warning disable <X> - 禁用编号为 X 的警告
* #pragma warning restore <X> - 恢复编号为 X 的警告
* #undef <X> - 将符号 "X" 设为 false
* #unwarn - 禁用下一行源代码的警告
* #warn <Message> - 在解析时创建警告

## 内置预处理器符号

* BF_32_BIT - 目标为 32 位
* BF_64_BIT - 目标为 64 位
* BF_ALLOW_HOT_SWAPPING - 启用热代码替换
* BF_DEBUG_ALLOC - 使用调试分配器
* BF_DYNAMIC_CAST_CHECK - 启用动态转换检查
* BF_ENABLE_OBJECT_DEBUG_FLAGS - 对象头包含调试标志
* BF_ENABLE_REALTIME_LEAK_CHECK - 启用实时泄漏检查
* BF_HAS_VDATA_EXTENDER - 类带有 vdata 扩展器（用于热替换期间扩展 vtable）
* BF_LARGE_COLLECTIONS - 启用大集合（>1GB）
* BF_LARGE_STRINGS - 启用大字符串（>1GB）
* BF_LITTLE_ENDIAN - 目标为小端
* BF_PLATFORM_IOS - iOS 目标
* BF_PLATFORM_LINUX - Linux 目标
* BF_PLATFORM_MACOS - macOS 目标
* BF_PLATFORM_WINDOWS - Windows 目标
* BF_RUNTIME_CHECKS - 启用运行时检查（如越界检查）
* BF_TEST_BUILD - 当前构建为 'test' 构建
* DEBUG - 当前构建为 'debug' 构建
