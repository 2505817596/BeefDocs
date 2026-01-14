+++
title = "反射"
weight=75
+++

## 反射
Beef 支持运行时反射，可枚举并访问类型、字段、方法和属性。默认情况下，为了减小可执行文件，仅包含最少的反射信息。可以通过代码特性标注需要额外输出的类成员信息。

```C#
public struct Options
{
	[Reflect]
	public bool mFlag;
}

void Use(ref Options options)
{
	/* 注意此处使用 &options，以装箱指向 'options' 的指针，而不是装箱 'options' 的副本 */
	options.GetType().GetField("mFlag").Value.SetValue(&options, true);
}
```

## 反射构造

可以通过反射信息创建值。需要确保相关类型确实被包含在构建产物中。

```C#
/* 由于我们仅通过反射实例化该类型，需要强制包含所需数据 */
/* 如果在已包含的代码中创建 TestClass 实例，则此处不一定需要 AlwaysInclude 特性 */
[Reflect(.DefaultConstructor), AlwaysInclude(AssumeInstantiated=true)]
class TestClass
{
}

void DynamicCreate()
{
	/* CreateObject() 返回 Result<Object>，可这样处理 */
	if (Object obj = typeof(TestClass).CreateObject())
	{
		Console.WriteLine("Successfully created TestClass instance");
		UseObject(obj);

		/* 通过反射创建的对象和值在堆上分配，需要 delete */
		delete obj;
	}
}
```

### 反射自定义特性
```C#
/* 该特性会出现在用户的反射信息中，使用该特性的类型会导出其使用的方法的反射信息 */
[AttributeUsage(.Class | .Struct, .ReflectAttribute, ReflectUser=.Methods)]
struct ScriptableAttribute : Attribute
{
	public String mName;
	public this(String name)
	{
		mName = name;
	}
}

/* 该类将提供可反射的默认构造函数，且所有方法都会被包含在构建中，即使它们未被直接调用 */
[Scriptable("Main Test Class"), AlwaysInclude(AssumeInstantiated=true, IncludeAllMethods=true)]
class TestClass
{
	public static void Test()
	{
		Console.WriteLine("TestClass.Test");
	}
}

class Program
{
	public static void Main()
	{
		for (let type in Type.Types)
		{
			if (let scriptableAttribute = type.GetCustomAttribute<ScriptableAttribute>())
			{
				for (let method in type.GetMethods(.Static))
				{
					Console.WriteLine("Calling method {} on {}", method.Name, scriptableAttribute.mName);
					method.Invoke(null);
				}
			}
		}
	}
}
```

或者也可以把 `AlwaysInclude` 的效果直接写到 `Scriptable` 特性上。使用下列结构体时，`TestClass` 只需要标注 `Scriptable` 特性即可。

```C#
[AttributeUsage(.Class | .Struct, .ReflectAttribute, ReflectUser=.Methods, AlwaysIncludeUser=.AssumeInstantiated | .IncludeAllMethods)]
struct ScriptableAttribute : Attribute
{
	public String mName;
	public this(String name)
	{
		mName = name;
	}
}
```

### 调用反射方法
```C#
[AlwaysInclude(IncludeAllMethods=true), Reflect(.Methods)]
class MethodHolder
{
	public int mIdentifier;

	static int GiveMeFive()
	{
		return 5;
	}

	public static void Print(int num, String message)
	{
		Console.WriteLine(scope $"{num}: {message}");
	}

	public void ChangeIdentifier(int newIdent)
	{
		mIdentifier = newIdent;
		Console.WriteLine(scope $"I am now number {newIdent}");
	}
}

class Program
{
	static void InvokeFuncs()
	{
		/* 调用成员方法 */
		{
			let mh = scope MethodHolder();

			/* 将 'mh' 作为 target，并传入方法参数。注意此处未处理错误 */
			typeof(MethodHolder).GetMethod("ChangeIdentifier").Get().Invoke(mh, 14);

			Runtime.Assert(mh.mIdentifier == 14);
		}

		/* 调用所有静态方法 */
		int passInt = 8;
		for (let m in typeof(MethodHolder).GetMethods(.Static))
		PARAMS:
		{
			/* 根据函数参数传入参数 */
			let methodParams = scope Object[m.ParamCount];
			for (let i < m.ParamCount)
			{
				Object param;
				switch (m.GetParamType(i)) /* 覆盖此示例中的所有情况 */
				{
				case typeof(String):
					param = "A nice string message";
				case typeof(int):

					/* 需要手动装箱，确保调用方法时不会因离开作用域而被释放 */
					/* param = passInt; 会隐式装箱 passInt，但会在离开 'for (let i < m.ParamCount)' 循环后被删除 */
					/* 而我们需要其在外层 "method" 循环的周期内保持有效，以便 Invoke() 调用 */
					param = scope:PARAMS box passInt;
				default:
					param = null;
				}

				methodParams[i] = param;
			}

			/* 调用方法并处理结果/返回值。静态方法没有 target */
			/* 注意 'Invoke(null, methodParams)' 会将 Object[] 作为唯一参数传入 */
			switch (m.Invoke(null, params methodParams))
			{
			case .Ok(let val):

				/* 处理返回的 int variant */
				if (val.VariantType == typeof(int))
				{
					let num = val.Get<int>();
					Console.WriteLine(scope $"Method {m.Name} returned {num}");
				}

			case .Err:
				Console.WriteLine(scope $"无法调用方法 {m.Name}");
			}
		}
	}
}

/* 输出：
	I am now number 14
	Method GiveMeFive returned 5
	8: A nice string message
*/
```

### 从接口反射
```C#
/* 实现该接口的所有类型都会启用动态装箱 */
[Reflect(.None, ReflectImplementer=.DynamicBoxing)]
interface ISerializable
{
	void Serialize(Stream stream);
}

namespace System
{
	extension StringView : ISerializable
	{
		void ISerializable.Serialize(Stream stream)
		{
			stream.Write(mLength);
			stream.TryWrite(.((uint8*)mPtr, mLength));
		}
	}
}

class Serializer
{
	public void Serialize(Variant v, Stream stream)
	{
		ISerializable iSerializable;
		if (v.IsObject)
			iSerializable = v.Get<Object>() as ISerializable;
		else
		{
			/* 由于 'ReflectImplementer=.DynamicBoxing' 特性，'v.GetBoxed' 可用于实现 ISerializable 的类型 */
			iSerializable = v.GetBoxed().GetValueOrDefault() as ISerializable;
			defer:: delete iSerializable;
		}
		iSerializable?.Serialize(stream);
	}
}
```

### 编译期类型枚举

在编译期通过 `Type.Types` 枚举类型是不允许的，因为在编译完全结束之前无法得知所有类型的集合。不过可以在编译期枚举工作区的类型*声明*。类型声明不包含泛型实例、数组、元组、可空类型或其他按需类型，也不包含只有在类型闭合后才知道的信息（如大小或成员列表）。类型声明仅在编译期可用，运行时不可用。

```C#
[Comptime]
int GetSerializableCount()
{
	int count = 0;
	for (var typeDecl in Type.TypeDeclarations)
	{
		if (typeDecl.HasCustomAttribute<SerializableAttribute>())
			count++;
	}
	return count;
}
```

### Distinct Build Options

反射信息可在工作区与项目的 Distinct Build Options 中配置。例如，如果你需要为所有 `System.Collection.List<T>` 实例反射 `Add` 与 `Remove` 方法，可以在 Distinct Build Options 下添加 `System.Collections.List<*>`：
	* 将 "Reflect\Method Filter" 设为 "Add;Remove"，确保设置仅应用于这些方法
	* 将 "Reflect\Always Include" 设为 "Include All"，确保指定方法即使未被显式使用也会编译进构建
	* 将 "Reflect\Non-Static Methods" 设为 "Yes"，确保指定的非静态方法添加反射信息

Distinct Build Options 过滤器支持：
	* 类型名匹配（如 `System.Collections.List<*>`）
	* 类型特性匹配（如 `[System.Optimize]`）
	* 接口实现匹配（如 `:System.IDisposable`）

### 动态装箱

`Variant.GetBoxed` 可用于动态创建堆分配的装箱对象。如果编译器未通过按需编译或反射选项生成存储值类型的盒类型（如为值类型标注 `[Reflect(.DynamicBoxing)]` 或在 Distinct Build Options 中设置 "Dynamic Boxing"），则调用会失败。

### 常见反射问题

Beef 致力于生成尽可能小的可执行文件——一个 “Hello World” 程序理想情况下只应包含打印 “Hello World” 所需的最小机器码和数据。如果你为该程序增加功能，让用户传入类型名与方法名，并期望基于反射信息构造该类型并调用该方法，那么除非可执行文件包含 corlib 中每个方法的机器码和反射信息，否则显然无法实现，这将违背“最小二进制”的目标。

首先，Beef 按需包含类型，因此未被程序直接使用的类型不会包含在构建中。可添加 `[AlwaysInclude]` 特性强制在所有构建中包含该类型。如果需要动态构造该类型，使用 `[AlwaysInclude(AssumeInstantiated=true)]`。单个方法也会按需编译，但可通过 `[AlwaysInclude(IncludeAllMethods=true)]` 强制将所有方法编译进构建。
