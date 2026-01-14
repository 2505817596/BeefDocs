+++
title = "内存管理"
weight = 30
+++

## 内存分配 {#allocating}
分配可以发生在栈、全局分配器或自定义分配器上。栈分配使用 "scope" 关键字，可指定从当前作用域（如代码块）到整个方法的作用域（即使在循环中）。

```C#
static void Test(StreamReader fs)
{
	let strList = scope List<String>();
	for (let line in fs.Lines)
	{
		/* 该字符串的作用域为整个方法 */
		let lineStr = scope:: String(line);
		strList.Add(lineStr);
	}
	strList.Sort();
}

static void Test(StreamReader fs)
{
	Sort:
	{
		let strList = scope List<String>();
		for (let line in fs.Lines)
		{
			/* 该字符串的作用域为 "Sort" 作用域 */
			let lineStr = scope:Sort String(line);
			strList.Add(lineStr);
		}
		strList.Sort();
	}
}
```

作用域分配可能动态增加栈大小，需要确保为给定计算提供足够的栈空间，就像递归方法必须保证递归深度不会耗尽栈一样。

通过全局分配器分配使用 "new" 关键字。

```C#
String AllocGlobalString(int len)
{
	return new String(len);
}
```

通过自定义分配器分配时，使用带自定义分配器实例的 "new" 关键字。
```C#
String AllocCustomString(int len)
{
	return new:customAllocator String(len);
}
```

自定义分配器最少只需实现一个 `Alloc` 方法，但可额外实现 `AllocTyped` 方法以添加类型特定的分配逻辑。内存通过 `Free` 方法释放。
```C#
struct ArenaAlloc
{
	public void* Alloc(int size, int align)
	{
		return Internal.StdMalloc(size);
	}

	public void* AllocTyped(Type type, int size, int align)
	{
		void* data = Alloc(size, align);
		if (type.HasDestructor)
			MarkRequiresDeletion(data);
		return data;
	}

	public void Free(void* ptr)
	{
		Internal.StdFree(ptr);
	}

	public void MarkRequiresDeletion(void* obj)
	{
		/* TODO: 当分配器释放时调用该对象的析构函数 */
	}
}
```

注意：若启用了实时泄漏检查，而自定义分配器使用的内存未被泄漏检查器追踪，则分配器需要报告其内存以便扫描对象引用。可参考 corlib 的 BumpAllocator，特别是其中的 'GCMarkMembers' 方法，了解如何与泄漏检查器协作。

自定义分配也可通过 [mixin]({{< ref "language-guide/datatypes/members.md#mixins" >}}) 进行，甚至可以根据条件在栈上分配。例如 ScopedAlloc mixin 会在栈上进行小规模分配，在堆上分配较大对象。
```C#
static mixin ScopedAlloc(int size, int align)
{
	void* data;
	if (size <= 128)
	{
		data = scope:mixin [Align(align)] uint8[size]* { ? };
	}
	else
	{
		data = new [Align(align)] uint8[size]* { ? };
		defer:mixin delete data;
	}
	data
}

void ReadString(int reserveLen)
{
	String str = new:ScopedAlloc! String(reserveLen);
	defer delete:null str;
	UseString(str);
}
```

注意上述示例中使用了 `delete:null`。`ScopedAlloc!` 调用会释放实际分配的内存，但不会调用 String 的析构函数。如果 `UseString` 向 `str` 追加了超过 `reserveLen` 的数据，则会发生堆分配，需要由 `String` 析构函数释放。`delete:null` 允许你执行析构而不请求释放任何内存。

许多 corlib 类（如 [System.String](../../doxygen/corlib/html/class_system_1_1_string.html) 和 [System.Collections.List<T>](../../doxygen/corlib/html/class_system_1_1_collections_1_1_list.html)）需要动态分配内存。按惯例，这些类从全局分配器分配，并通过 `String.Alloc`、`String.Free` 等虚方法重写支持其内部使用自定义分配器。

### 全局分配器
全局分配器以工作区为单位选择。默认使用 CRT malloc/free 分配器，但也可使用任意 C 风格全局分配器，如 TCMalloc 或 JEMalloc。此外，Beef 还提供特殊的调试分配器，用于实时泄漏检查与热编译等功能。

Beef 的分配是 C 风格的：不可移动，且没有垃圾回收器。

### 释放内存
作用域分配会在作用域结束时自动释放，但手动分配必须使用 "delete" 关键字手动释放。与自定义分配器分配类似，delete 也可以指定自定义分配器目标来释放该分配器的内存。

### 追加分配 {#append}
追加分配是一类可放在构造函数中的特殊分配，可在分配所属对象时手动请求额外内存。这在 corlib 中用于接受 "size" 参数的字符串等场景。

```C#
class FloatArray
{
	int mLength;
	float* mPtr;

	[AllowAppend]
	public this(int length)
	{
		let ptr = append float[length]*;
		mPtr = ptr;
		mLength = length;
	}
}

/* 追加分配保证紧随对象自身内存之后（考虑对齐）。我们可以利用这一点计算数组存储位置，而不需要在内部保存指针。
注意这里使用了 "ZeroGap" 属性，确保没有类能扩展该类型并添加字段，从而破坏追加位置的假设。 */
class FloatArray
{
	int mLength;

	[AllowAppend(ZeroGap=true)]
	public this(int length)
	{
		let ptr = append float[length]*;
		mLength = length;
	}

	public float* Ptr
	{
		get
		{
			return (float*)(&mLength + 1);
		}
	}
}

```

在内部，追加分配通过在分配发生前调用尺寸计算函数来实现。编译器会尝试对该函数及其在调用点的相关参数进行常量求值，从而得到固定大小的分配而非动态大小分配，这样可省去额外调用，并可能提升某些栈分配的性能。

追加分配的内存不需要显式释放，但仍可通过 `delete:append obj` 语句调用对象析构函数。

### 装箱 {#boxing}
所有值类型（基础类型、结构体、元组、指针、枚举）都可“装箱”为 Object，这对于动态类型处理与接口派发很有用。基础类型都对应库内定义的结构体包装，用于装箱（例如 int32 会被 System.Int32 包装）。装箱是分配操作，转换为 System.Object 时会隐式进行临时栈分配，但也可显式指定长期装箱与更长生命周期的栈分配。装箱值类型时，会静态生成一个包裹该值类型的特殊“盒类型”。这会带来一定代码膨胀，因此这些盒类型按需为每个值类型生成。

```C#
// 格式化调用依赖装箱以处理传入类型
Console.WriteLine("a + b = {}", a + b);
Object a = 1.2f; // 隐式在栈上装箱到当前作用域
Object b = scope box:: 2.3f; // 显式在栈上装箱到方法作用域
Object c = new:allok box 4.5f; // 通过自定义分配器 'allok' 显式装箱
```

### Variant
`System.Variant` 是装箱的替代方案。Variant 不是对象类型，因此无法进行动态接口派发，但其优势是可在不分配的情况下存储小型数据类型，且不会产生装箱代码膨胀。Variant 可通过 `Variant.GetBoxed` 转换为堆分配的装箱对象，但如果编译器尚未为存储的值类型生成按需盒类型，该操作会失败。可通过 [反射选项]({{< ref "reflection.md" >}}) 明确请求生成盒类型。
