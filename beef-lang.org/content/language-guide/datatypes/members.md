+++
title = "类型成员"
weight = 10
+++

## 字段

字段可以是 const、static 或实例字段。const 字段表示值，但不占用内存；在简单的常量算术求值下，const 必须在编译期求值为常量。使用 const 等同于使用其对应的字面量值。static 值是在定义类型命名空间内的单个“全局变量”。既非 static 也非 const 的字段为实例字段，表示定义类型实例内部的数据。

```C#
class Widget
{
	/* 静态“共享”值 */
	static int totalCount;

	const int MaxWidgets = 64 * 1024;

	/* 普通“实例”数据字段 */
	int x;
	int y;
}
```

通过 `using` 字段支持匿名字段访问。标记为 `using` 的字段，其成员字段可在不使用字段名限定的情况下直接访问。这种用法对命名字段本身与匿名使用有不同的访问级别。例如 `public using private Vector2 mPosition` 中，`mPosition` 为 private，但通过匿名访问时其 `X` 与 `Y` 成员是 public 可见。

匿名字段访问可以通过组合提供一些类似继承的优势。

```C#
struct Vector2
{
	public float X;
	public float Y;
}

struct Entity
{
	/* 可直接访问 'mX' 或通过 'mPosition.mX' 访问。注意匿名访问未直接指定访问级别，
	因此采用命名字段的访问级别（public） */
	using public Vector2 mPosition;
	public int32 mHealth;
}
```

追加字段是一种优化，允许引用类型字段的数据静态包含在所属对象内部。这样字段在语义上仍表现为引用类型，但无需引用类型的间接访问（数据偏移已静态确定），也无需单独分配。

```C#
/* 分配 'User' 时会包含两个成员字符串的数据，以及它们内部缓冲区的指定空间 */
class User
{
	public append String mName = .(256);
	public append String mPassword = .(32);
}
```

## 方法

Beef 支持方法重载。对于泛型重载，若约束不同，允许存在同名泛型方法的多个版本。若多个泛型方法同时匹配，当其中一个的约束是另一个的超集时，可视为“更好”的匹配；否则重载选择将产生歧义。

参数值默认为不可变，除非使用 `ref`、`out` 或 `mut` 标记。`in` 标记可显式要求以不可变引用传参，但这会禁止在更高效的情况下以值传递小型结构体，因此只有在确需该语义时才应使用 `in`。

```C#
bool GetInt(out int outVal)
{
	outVal = 92;
	return true;
}

/* 'mut' 可用于可能是值类型或引用类型的泛型，但需要可变引用。
'mut' 对引用类型无影响，但对值类型会被视为 'ref'。 */
bool DisposeVal<T>(mut T val) where T : IDisposable
{
	val.Dispose();
}

/* 方法可以返回 'ref' 或 'readonly ref' 值 */
readonly ref int Get()
{
	return ref mVal;
}
```

方法可使用参数名调用。命名参数可按任意顺序出现，并可与普通“位置”参数混用，但有一定限制。

```C#
static void Method(int a, int b, int c) {}

/* 合法 */
Method(1, 2, 3);
Method(a:1, b:2, c:3);
Method(a:1, 2, c:3);
Method(c:3, b:2, a:1);

/* 非法 */
Method(a:1, a:2, b:3, c:4);
Method(b:2, 1, 3);
```

当需要将参数视为可修改的“初始值”时，可使用变量遮蔽功能：语义上会创建参数的可修改副本，并生成同名的新遮蔽变量。

```C#
void Write(char8 c)
{
	var c;
	if (c == '\n')
		c = ' ';
	file.Write(c);

	/* 原始参数仍可通过 "@c" 访问，并可在调试构建的调试器中查看 */
}
```

默认情况下，“较大”的结构体以不可变引用传递，“较小”的结构体以不可变值传递。诸如 `Vector3` 的结构体可在 x86 上直接通过 XMM 寄存器传递。

### 可变参数数量 {#params}

可以通过在最后一个参数上使用 "params" 标记来定义可变参数方法。这在 Console.WriteLine 等字符串格式化方法中很常见。"params" 类型可声明为数组或 Span 类型。"params" 也可用于委托或函数类型，从而将参数声明展开为该委托/函数的参数列表——这对于泛型参数转发很有用，如 System.Event<T> 的场景。

```C#
	/* 可变参数 */
void Draw(String format, params Object[] args)
{
	let str = scope String();
	/* 使用 'params' */
	str.AppendF(format, params args);
}

/* 委托参数转发 */
public static rettype(T) SafeInvoke<T>(T dlg, params T p) where T : Delegate
{
	if (dlg == null)
		return default;
	return dlg(params p);
}
```

泛型元组 params 允许按参数进行泛型特化。通常会使用编译期代码生成，为传入参数类型生成特化的方法体。高级示例参见 https://github.com/beefytech/Beef/blob/master/IDEHelper/Tests/src/Params.bf 的快速字符串格式化测试实现。

```C#

void HandleArgs<TArgs>(params TArgs args) where TArgs : Tuple
{
}

HandleArgs("String", 1, 2.3f);

```

Beef 支持与 C 兼容的可变参数。这比 `params` 更不安全，因为参数数量与类型未知，但对 C 互操作很有用。

```C#
/* 外部 C 方法 'void Log_External(char*, va_list)' */
[CLink]
static extern void Log_External(char8* format, void* varArgs);

public static void Log(String format, ...)
{
	VarArgs vaArgs = .();
	vaArgs.Start!();
	Log_External(format, vaArgs.ToVAList());
	vaArgs.End!();
	return result;
}

public static void HandleVarArgInts(int count, ...)
{
	VarArgs vaArgs = .();
	vaArgs.Start!();
	for (int i < count)
		Dispay(vaArgs.Get!<int>())
	vaArgs.End!();
	return result;
}
```

### 内联

后端优化会执行内联，但你也可以通过 [Inline] 特性显式内联方法。这会进行直接内联，即使在调试构建中也会内联（不影响可调试性），并确保方法在可能无法自动内联的场景下也能内联，例如非 LTO 的跨模块调用。

```C#
class Collection
{
	[Inline]
	int GetLength()
	{
		return mLength;
	}

	void Clear()
	{
		mLength = 0;
	}
}

void Use(Collection c)
{
	c.GetLength();

	/* 可在调用点请求 Inline，会创建一个局部的“总是内联”版本来调用 */
	c.[Inline]Clear();
}
```

### 丢弃的返回值

方法提供一定的安全机制以防止丢弃返回值。这有助于确保错误被处理，或避免把返回修改值的方法误认为是原地修改。可通过为方法添加 [NoDiscard] 特性，在丢弃结果时生成静态警告。对于未处理错误的场景，可在返回类型上添加 `ReturnValueDiscarded()` 方法，编译器会在运行时调用——内置的 `Result` 类型就使用了这一机制（参见 [错误处理]({{< ref "errors.md" >}})）。

```C#

[NoDiscard]
char8 ToUpperCase(char8 c);

void ToUpperCase(String str)
{
	for (var c in ref str.RawChars)
	{
		/* 糟糕 - 这里会抛出“返回值被丢弃”的警告。原意是写 'c = ToUpperCase(c)' */
		ToUpperCase(c);
	}
}

```

## Mixin
Mixin 是一种会被直接“混入”调用点的方法，而不是被“调用”。这不仅能消除调用开销，还具有不同语义：mixin 中的 `break` 可直接跳出调用者的结构块，`return` 也会从调用者返回。

```C#
static mixin Inc(int* val)
{
	if (val == null)
		return false;
	(*val)++;
}

static bool Inc3(int* a, int* b, int* c)
{
	Inc!(a);
	Inc!(b);
	Inc!(c);
	return true;
}
```

Mixin 参数类型也可通过声明为 `var` 来“解除约束”，有助于创建更通用的类宏辅助。Mixin 也可使用带约束的泛型参数实现，这有助于生成更有用的错误或在存在多个重载版本时进行选择，但不影响代码生成。使用 `var` 不会带来性能损耗，因为 mixin 在调用点的展开方式相同——区别仅在于错误发生在调用点表达式本身还是展开后的 mixin 内部。

当指定 mixin 参数类型时，如果调用者传入的可变值与类型匹配，该参数将是可变的；否则会发生类型转换，参数将不可变。

Mixin 不声明返回类型，因为它们不“返回”，但可使用“表达式块”语法产生值：mixin 定义块以不带分号的表达式结尾。

```C#
static mixin Max(var a, var b)
{
	(a > b) ? a : b
}
```

### Mixin 目标

在 mixin 定义中，对于允许指定作用域目标的操作（如 break、continue、scope 分配），可使用特殊的 `mixin` 目标，使作用域目标可在 mixin 调用点指定。

```C#
static mixin AllocString()
{
	scope:mixin String()
}

void Use()
{
	String outerStr = null;

	while (IsRunning())
	WhileBody:
	{
		String whileStr = null;

		if (Check())
		{
			// 在当前作用域分配字符串
			let localStr = AllocString!();

			// 在 'while' 体作用域中分配字符串
			whileStr = AllocString!:WhileBody();

			// 在方法作用域中分配字符串
			outerStr = AllocString!::();
		}
	}
}
```

## 属性

属性用于向类型添加具名值，并定义 getter 和/或 setter 方法。

```C#
struct Square
{
	int x;
	int y;
	int width;
	int height;

	public int Right
	{
		get
		{
			return x + width;
		}

		/* 注意这里要求使用 'mut' */
		set mut
		{
			width = value - x;
		}
	}

	/* 允许按引用返回 'Left'，从而可通过赋值设置其值，
	同时也可按引用传递给其他方法 */
	ref int Left
	{
		get
		{
			return ref x;
		}
	}

	/* 该属性定义会隐式创建成员变量以及相应的 get/set 方法 */
	uint32 Color { get; set; } = 0xBFBF;
}

struct IntRef
{
	int* mValue;

	public ref int Ref
	{
		get
		{
			return ref *mValue;
		}

		/* 若没有 'ref' 标记，set 方法会接收 'int' 而非 'ref int' */
		set ref mut
		{
			mValue = &value;
		}
	}

	public ref int Value => ref *mValue;
}
```

索引运算符是带一个或多个索引参数的属性。

```C#
public ReadOnlyList<T>
{
	public T this[int idx]
	{
		get
		{
			return mList[idx];
		}
	}
}
```

## 成员访问
默认情况下，结构体与类成员为 'private'，仅能在该类型内部访问。'protected' 成员可被派生类型访问，'public' 成员可被任何人访问。'internal' 成员可在指定 `using internal <namespace>` 的文件中访问。'protected internal' 是 'protected' 与 'internal' 的最宽松组合。注意即便位于同一命名空间的类型，也需要显式指定 `using internal` 才能访问彼此的 internal 成员。

```C#
/* 允许我们在 'GameEngine' 命名空间内的任意位置访问 internal 成员 */
using internal GameEngine;

class Widget
{
	private int32 id;
	protected Rect pos;
	public uint32 color;
	internal void* impl;
}

class Button : Widget
{
	/* 该类可访问 'pos' 和 'color'，但不能访问 'id' */
}

static void Main()
{
	var button = new Button();
	/* 我们只能访问 'button.color' */

	/* [Friend] 特性允许访问通常被隐藏的成员 */
	int32 id = button.[Friend]id;
	/* 注意这里的 'friend' 关系与 C++ 相反 —— 使用者承诺“友好”，
	而不是由定义类型声明其友元 */
}
```
