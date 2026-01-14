+++
title = "互操作"
weight = 65
+++

### 互操作（FFI）
Beef 允许零开销地链接静态与动态库。声明为 `extern` 的方法会创建外部符号引用，必须在链接时被满足。当缺少 C 链接库时，Beef 也可通过 `[Import(...)]` 特性导入 DLL 方法。默认的符号重整规则类似于 C++，在许多情况下可匹配，但可通过 [CLink] 或 [LinkName] 覆盖方法的链接名。
 
```C#
/* 链接外部库中的方法，使用以下特性：
  Import：使用该方法时链接 'wsock32.lib' 库
  CLink：不使用 C++ 重整，改用 C 链接名
  StdCall：使用 stdcall 调用约定（WINAPI） */
[Import("wsock32.lib"), CLink, StdCall]
static extern int32 WSAGetLastError(); 

/* 该方法链接到 C++ 中定义为 'const float& GetVal(const int32& a)'' 的外部函数 */
[return: MangleConst, LinkName(.CPP)]
public static extern ref float GetVal([MangleConst]ref int32 a);
```

由于字段重排（减少对齐空隙）以及 Beef 将结构体大小与步长分离（Beef 结构体末尾不包含对齐填充），默认情况下结构体布局并不与 C 的结构体布局一致。可使用 `[CRepr]` 特性创建与 C 匹配的结构体以便互操作。

默认情况下 Beef 使用系统的 `cdecl` 调用约定，除非方法标记为 `[StdCall]`，此时使用系统的 `stdcall` 约定。

若 FFI 需要 CRT 分配器，请注意工作区可定义全局分配器，因此可能无法使用 CRT 的分配器。此时可通过 `System.Internal.StdMalloc` 和 `System.Internal.StdFree` 管理 FFI 内存，或使用 `System.gCRTAlloc` 自定义分配器。

Beef 也支持通过 corlib 的 System.FFI 进行手动与动态外部函数接口（FFI）。

### 互操作类型映射

|C/C++|System.Interop TypeAlias|Beef Primitive|
|-----|------|------|
|short|c_short|int16|
|unsigned short|c_ushort|uint16|
|int|c_int|int32|
|unsigned int|c_uint|uint32|
|long (Windows)|c_long|int32|
|long (Linux/macOS/others)|c_long|int64|
|unsigned long (Windows)|c_ulong|uint32|
|unsigned long (Linux/macOS/others)|c_ulong|uint64|
|long long|c_longlong|int64|
|unsigned long long|c_ulonglong|int64|
|intptr_t|c_intptr|int|
|uintptr_t|c_uintptr|uint|
|char|c_char|char8|
|unsigned char|c_uchar|uint8|
|wchar_t (Windows)|c_wchar|char16|
|wchar_t (Linux/macOS/others)|c_wchar|char32|

### ABI 稳定性

Beef 不提供通用 ABI 稳定性，除了由通用的 FFI C 互操作带来的那部分——即便是完全相同代码的独立编译，也不保证生成稳定的 ABI 边界。以下是一些可能在 ABI 边界库未改动代码时仍会出现的 ABI 破坏例子：

- 工作区调试设置会影响对象的大小与内容。这些设置包括 “Object Debug Flags”、“Realtime Leak Check”、“Enable Hot Compilation”、“Large Strings” 和 “Large Collections”。
- 用户程序与库可添加类型扩展，改变系统类型的数据布局
- 快速动态转换依赖生成工作区范围的继承顺序类型 ID
- 去虚拟化可能发生，并依赖调用点对完整继承信息的了解
- 反射数据在 ABI 边界之间不兼容
- 字符串字面量地址相等保证会被破坏
- 按需编译也会影响 vtable 布局，因为被省略的方法不会占用 vtable 项
- 全局分配器的选择发生在工作区级别

尽管 C++ 等语言并未“官方”提供稳定 ABI，其独立“编译模块”的方式有时可形成一种“软”ABI：只要能保证边界两侧（库与库使用者）以“兼容设置”编译，包括编译器版本、兼容头文件、相关预处理器标志、谨慎的内存管理等，便可形成可用的 ABI 边界。虽然这比 Beef 的 ABI 稳定性稍强，但仍不足以满足 ABI 稳定性的总体目标——包括允许库作者发布包含安全或性能改进的二进制库更新而无需用户程序重新编译，以及提供系统级库使其二进制足迹无需随用户程序一并打包（这也是 Swift ABI 稳定性的主要目标之一）。

<sup>进一步阅读：https://gankra.github.io/blah/swift-abi/</sup>
