+++
title = "泛型"
weight = 70
+++

## 泛型概览

泛型允许在编译期编写可应用于不同类型的抽象代码。例如 List<T> 是一个基础列表抽象，可安全地存储不同类型的值。使用 "List<int32>" 类型引用会创建特化的 int32 列表类型。

方法也可以拥有泛型参数，从而根据调用点的参数类型进行显式或隐式特化。

```C#
public static T GetFirst<T>(List<T> list)
{
	return list[0];
}
...
let intList = new List<int32>();
intList.Add(123);
let firstVal = GetFirst(intList);
```

可指定泛型约束，用于描述泛型代码所期望的类型“形状”。

- 接口类型 - 可为泛型参数指定任意数量的接口，传入类型必须实现这些接口。
- 类/结构体类型 - 可指定一个具体类型，传入类型必须从其派生。
- 委托类型 - 传入类型可为该委托类型实例，或为签名符合该委托的方法引用（参见方法引用）。
- `operator T <op> T2` - 类型必须由指定类型间的二元运算得到
- `operator <op> T` - 类型必须由指定类型的一元运算得到
- `operator implicit T` - 类型必须可从指定类型隐式转换得到
- `operator explicit T` - 类型必须可从指定类型显式转换得到
- `class` - 类型必须为类
- `struct` - 类型必须为值类型
- `enum` - 类型必须为枚举
- `interface` - 类型必须为接口
- `struct*` - 类型必须为值类型指针
- `new` - 类型必须定义可访问的默认构造函数
- `delete` - 类型必须定义可访问的析构函数
- `const` - 类型必须为常量值 - 参见“常量泛型”
- `var` - 类型不受约束。对某些“鸭子类型”场景有用，可生成类似 C++ 模板的模式，但通常会产生不够友好的错误与较差的开发体验

```C#
public static T Abs<T>(T value) where T : IOpComparable, IOpNegatable
{
    if (value < default)
        return -value;
    else
		return value;
}
```

```C#
/* This method can eliminate runtime branching by specializing at compile time by incoming array size */
public static float GetSum<TCount>(float[TCount] vals) where TCount : const int
{
	if (vals.Count == 0)
	{
		return 0;
	}
	else if (vals.Count == 1)
	{
		return vals[0];
	}
	else
	{
		float total = 0;
		for (let val in vals)
			total += val;
		return total;
	}
}
```

```C#
static TTo Convert<TTo, TFrom>(TFrom val) where TTo : operator explicit TFrom
{
	return (TTo)val;
}

/* We use partial explicit generic args to allow inference of 'TFrom' */
var val = Convert<int...>(1.2f);
```

支持泛型构造函数。

```C#
class WriteValue
{
	public this<T>(T val)
	{
		Console.WriteLine($"Value: {val}");
	}
}

/* Implicitly determine 'T' for constructor */
var val = scope WriteVal(1.0f);

/* Explicitly specify 'T' for constructor */
var val = scope WriteVal.this<float>(1);

```
