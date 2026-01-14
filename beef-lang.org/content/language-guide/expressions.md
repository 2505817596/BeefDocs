+++
title = "表达式"
weight = 43
+++

### 分配

`new` 和 `scope` 关键字用于分配（参见 [内存管理]({{< ref "memory.md" >}})）

### append

`append` 表达式会在对象已分配内存的紧后位置继续分配，只能在构造函数中使用。（参见 [内存管理]({{< ref "memory.md#append" >}})）

`append` 分配可用于 `new` 分配能使用的任何场景。（参见 [new]({{< ref "#new" >}})）

### 赋值操作

参见 [赋值运算符]({{< ref "operators.md#assignment" >}})

### 二元操作

参见 [二元运算符]({{< ref "operators.md#binary" >}})

### 绑定表达式 =>

`=>` 表达式用于方法绑定（参见 [方法引用]({{< ref "datatypes/methodrefs.md" >}})）

### box

`box` 表达式分配一个对象来包装值类型。（参见 [内存管理（装箱）]({{< ref "memory.md#boxing" >}})）

* `scope box x` - 在当前作用域中装箱 `x`
* `scope:s box x` - 在作用域 `s` 中装箱 `x`
* `new box x` - 在全局分配器中装箱 `x`
* `new:a box x` - 在自定义分配器 `a` 中装箱 `x`  

### case

`case` 表达式可在 switch 之外用于模式匹配。（参见 [模式匹配]({{< ref "pattern.md" >}})）

### 类型转换表达式

* `(T)x` 将值 `x` 转换为类型 `T`

### 条件运算符
* `x ? y : z` - 当 `x` 为 true 时结果为 `y`，否则为 `z`

### 条件变量声明
可空类型的变量声明可在 `if` 语句中用作布尔表达式。在某些二元操作中，当整体 `if` 条件为 true 能确保条件变量声明也被求值且结果为 true 时，这种写法是允许的。

```C#
/* 简单的条件变量声明 */
if (let name = GetName())
{

}

/* “复杂”条件变量声明 */
if ((let name = GetName()) && (isEnabled))
{

}

/* 这是非法的，因为 "force" 可能导致即使条件变量声明失败也进入该块 */
if ((let name == GetName()) || (force))
{

}
```

### default

每种类型都有一个 “default” 值，始终为零初始化。

```C#
// Default 可指定类型并返回该类型的默认初始化值
var str = default(String);
// 若未显式指定 default 类型，Default 会使用“期望类型”
String str2 = default;
```

### 表达式块
表达式块以不带分号的表达式结尾。
```C#
Console.WriteLine("Result={}",
	{
		GetByRef(let val);
		val
	});
```

### 索引表达式

* `y = x[i]` - 用索引 `i` 访问 `x`。若 `x` 为指针，则等价于 `y = *(x + i)`；否则调用 `this[int]` 索引器属性的 `get` 方法。
* `x[i] = y` - 用索引 `i` 访问 `x` 并赋值。若 `x` 为指针，则等价于 `*(x + i) = y`；否则若 `this[int]` 索引器存在 `set` 方法则调用它，否则若 `get` 返回 `ref` 则调用 `get` 方法。

### 字面量

* `123` - 数字
* `0x1234` - 十六进制数字
* `0x1234'5678` - 带分隔符的数字，可放在任意位置
* `0x1234L` - int64 十六进制数字
* `0x1234UL` - uint64 十六进制数字
* `'c'` - char8
* `'😃'` - char32
* 1.2f - float
* 2.3 - double
* "Hello" - String

### new {#new}

`new` 表达式会在全局分配器或自定义分配器中分配内存。（参见 [内存管理]({{< ref "memory.md#allocating" >}})）

* `new T(...)` - 在全局分配器中分配 `T` 实例。若 `T` 为引用类型则结果为 `T`，否则结果为 `T*`
* `new T[i]` - 分配数组大小为 `i` 的 `T[]`
* `new T[i] (...)` - 分配数组大小为 `i` 的 `T[]` 并进行初始化
* `new T[] (...)` - 分配 `T[]`，大小由初始化项数量决定
* `new T[i]*` - 在全局分配器中分配 `i` 个连续的 `T` 实例，并返回指向首元素的 `T*` 指针。注意分配大小为 typeof(T).InstanceStride*i 以便使用，但严格来说这会在最后一个元素末尾分配额外填充。
* `new box x` - 在全局分配器中装箱 `x`。（参见 [内存管理（装箱）]({{< ref "memory.md#boxing" >}})）

所有 `new` 操作也可接受自定义分配器参数。

* `new:a T(....)` 在自定义分配器 `a` 中分配 `T` 实例，其中 `a` 为标识符。
* `new:(a) T(...)` 在自定义分配器 `a` 中分配 `T` 实例，其中 `a` 可为任意表达式。

### 空条件运算符

空条件运算符 `val?.field` 和 `val?[index]` 在 `val` 为 null 时结果为 `null`。空条件运算符可链式使用，会在遇到第一个 `null` 时短路返回 `null`。

```C#
int? a = val?.intField;
int? nameLength = val?.name?.Length;
```

### 括号表达式

在表达式外加括号可改变复杂表达式的运算顺序。

```C#
int a = 1 + 2 * 3; // The multiply happens before the add here, resulting in 7
int b = (1 + 2) * 3; // The add happens before the multiply here, resulting in 9
```

### 范围表达式 {#range}

范围由起始与结束整数值组成，主要用于循环迭代和范围索引。可创建为包含式或排除式范围。索引范围可有一侧开放。（参见 [范围运算符]({{< ref "operators.md#range" >}})）

```C#
let list = scope List<int>() { 5, 1, 0 };

/* 由于在范围上迭代，创建范围时只会调用一次 list.Count */
/* 因此列表计数只是翻倍，而不会产生无限循环 */
for (let i in 0 ..< list.Count)
	list.Add(list[i]);

// list 现在为：{ 5, 1, 0, 5, 1, 0 }

for (let i in (0 ..< list.Count).Reversed)
	list.Add(list[i]);

// list 现在为：{ 5, 1, 0, 5, 1, 0, 0, 1, 5, 0, 1, 5 }

var subset = list[...2]; // 等价于 0...2、..<3 和 ...^10（^ 从末尾计数，起点为 Count）
subset = list[4...]; // 等价于 4...11、4...^1 和 4..<^0
```

### scope

`scope` 表达式在栈上分配内存，作用域位于正在执行的方法之内。（参见 [内存管理]({{< ref "memory.md#allocating" >}})）

`scope` 分配可用于 `new` 分配能使用的任何场景。（参见 [new]({{< ref "#new" >}})）

### this

`this` 是一个特殊变量名，表示实例方法中的当前实例。若定义类型为结构体，则 `this` 为值类型（非指针），在 "mut" 方法中可变，其他情况下不可变。

### 元组表达式

元组表达式是一个括号包裹的表达式，包含多个逗号分隔的值，并可选字段名。（参见 [数据类型（元组）]({{< ref "datatypes/_index.md#tuples" >}})）

### 一元操作

参见 [一元运算符]({{< ref "operators.md#unary" >}})

### 未初始化 '?'

当赋值给变量或字段时，`?` 会让该值被视为已赋值，但不一定执行任何实际操作。这在“缓冲区”类型数组等不需要在使用前清零的场景中很有用。

在与 `out` 参数一起使用时，`?` 作为丢弃值。

在构造函数中使用时，未初始化构造调用 `this(...) : this(?)` 与 `this(...) : base(?)` 会丢弃相应的初始化器与构造函数。参见 [初始化]({{< ref "datatypes/initialization.md" >}})。
