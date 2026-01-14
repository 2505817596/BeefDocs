+++
title = "特性"
weight = 11
+++

## 类型特性
- [CRepr] - 将结构体标记为 C 兼容
- [Ordered] - 禁用字段重排（字段重排用于减少对齐填充）
- [Packed] - 省略字段对齐填充
- [Union] - 对结构体创建联合体

## 成员特性
- [Reflect] - 强制为该成员生成反射数据
- [NoShow] - 在自动补全中隐藏该字段

## 静态字段或方法特性
- [CLink] - 使用 C 链接名而非 C++ 风格重整
- [LinkName] - 显式覆盖链接名

## 构造函数特性
- [AllowAppend] - 允许追加分配

## 方法特性
- [AlwaysInclude] - 表示即使按需编译会跳过该方法，也应将其包含在构建中。适用于需要通过反射调用的方法。
- [Checked] - 表示该方法执行运行时检查（如参数校验/越界检查）
- [Commutable] - 为该方法生成一个参数前两位交换的额外版本，可用于可交换运算符
- [Comptime] - 标记方法仅可在编译期求值，不可在运行期调用
- [DisableChecks] - 表示该方法内部调用尽可能使用 Unchecked 版本的方法（优化）
- [DisableObjectAccessChecks] - 表示在该方法内禁用对象访问检查（优化）
- [Error] - 调用该方法时抛出编译错误（[Obsolete] 的更通用形式）
- [Export] - 导出该方法
- [Import] - 从指定 DLL 导入方法，可用于没有 DLL 的 lib 文件时
- [Inline] - 即使在非优化构建中也内联函数
- [Intrinsic] - 将方法绑定为 intrinsic（通常仅用于系统库）
- [NoDiscard] - 若返回值未使用，则在调用点发出警告
- [NoReturn] - 表示该方法不会返回
- [Obsolete] - 标记方法过时，抛出警告或错误
- [Optimized] - 在启用优化的情况下编译方法
- [SkipCall] - 禁止生成对该方法调用以及参数求值的代码
- [StdCall] - 使用 stdcall 调用约定而非默认 cdecl
- [Test] - 标记方法为测试方法
- [Unchecked] - 表示该方法省略运行时检查（通常用于性能）
- [Warn] - 调用该方法时抛出编译警告（[Obsolete] 的更通用形式）

## 静态字段特性
- [ThreadStatic] - 将该字段标记为线程本地静态字段

## 成员访问特性
- [Friend] - 允许访问私有成员
- [SkipAccessCheck] - 对该成员访问目标省略对象访问检查（优化）

## 代码块特性
- [IgnoreErrors] - 静默忽略该代码块中的错误
