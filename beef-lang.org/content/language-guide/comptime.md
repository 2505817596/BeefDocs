+++
title = "编译期（Comptime）"
weight = 80
+++

## 编译期（Comptime）

Beef 提供编译期特性，可用于执行能求值为常量的代码或生成代码。

### 编译期方法求值

```C#
/* 使用 Vector3 构造函数的编译期求值来初始化常量 */
const Vector3 vec = .(1, 2, 3);

/* 普通方法也可在编译期使用 */
static int32 Factorial(int32 n)
{
    return n <= 1 ? 1 : (n * Factorial(n - 1));
}
const int fac = Factorial(8);
var fac2 = [ConstEval]Factorial(9); /* 调用特性强制编译期求值 */

/* 该方法只能在编译期调用。'var' 返回类型允许它根据输入在编译期返回不同类型 */
[Comptime(ConstEval=true)]
static var StrToValue(String str)
{
	if (str.Contains('.'))
		return float.Parse(str).Value;
	return int.Parse(str).Value;
}
public const let cVal0 = StrToValue("123"); /* 求值为 'int' */
public const let cVal1 = StrToValue("1.23"); /* 求值为 'float' */

/* 注意：返回作用域内存在编译期是合法的，但在运行期是不合法的 */
static String GenerateString(String str, int a, int b) => scope $"{str}:{a}:{b}";
const String cStr = GenerateString("Prefix", 12, 34); /* 求值为字符串字面量 "Prefix:12:34" */

/* Span 结果可用于初始化定长数组 */
public static Span<int32> GetSorted(String numList)
{
	List<int32> list = scope .();
	for (var sv in numList.Split(','))
	{
		sv.Trim();
		if (int32.Parse(sv) case .Ok(let val))
			list.Add(val);
	}
	list.Sort();
	return list;
}
const int32[?] iArr = GetSorted("8, 1, 3, 7"); /* 求值为 int32[4](1, 3, 7, 8) */
```

每次编译期求值都是隔离执行的——在一次方法求值过程中修改的静态值不会对后续方法求值可见。编译期求值限制某些副作用，例如文件 IO 与访问外部库。

### 编译期代码生成

代码生成在编译期方法求值特性的基础上扩展，允许在编译过程中某些触发点修改类型。

```C#
/* 常量字符串可在编译期注入到调用点。该字符串可由编译期方法生成。 */
{
  /* 在此例中等同于直接把字符串粘贴到代码中。 */
  Compiler.Mixin("int val = 99;");

  Console.WriteLine(val);
}

/* OnCompile 特性允许代码生成 */
class ClassA
{
	public int mA = 123;

	[OnCompile(.TypeInit), Comptime]
	public static void Generate()
	{
		Compiler.EmitTypeBody(typeof(Self), """
			public int32 mB = 234;
			public int32 GetValB() => mB;
			""");
	}
}

/* 该方法会在调用点注入运行时作用域计时器。 */
[Comptime]
public static void TimeScope(String scopeName)
{
	let nameHash = (uint)scopeName.GetHashCode();

  /* MixinRoot 会注入到最外层的非编译期调用者，而不是注入到该编译期方法中 */
	Compiler.MixinRoot(scope $"""
		let __timer_{nameHash} = scope System.Diagnostics.Stopwatch(true);
		defer
		{{
			System.Console.WriteLine($"Scope {scopeName} took {{__timer_{nameHash}.ElapsedMilliseconds}}ms.");
		}}
		""");
}

/* 给类型添加该特性会通过编译期反射生成 'ToString' 方法 */
[AttributeUsage(.Types)]
struct IFancyToStringAttribute : Attribute, IOnTypeInit
{
    [Comptime]
    public void OnTypeInit(Type type, Self* prev)
    {
        Compiler.EmitTypeBody(type, "public override void ToString(String str)\n{\n");
		int strElementIdx = 0;
        for (var fieldInfo in type.GetFields())
        {
            if ((!fieldInfo.IsInstanceField) ||
				((fieldInfo.IsPrivate) && (fieldInfo.DeclaringType != type)))
                continue;
            if (strElementIdx > 0)
                Compiler.EmitTypeBody(type, "\tstr.Append(\", \");\n");
            Compiler.EmitTypeBody(type, scope $"\tstr.AppendF($\"{fieldInfo.Name}={{ {fieldInfo.Name} }}\");\n");
			strElementIdx++;
        }
        Compiler.EmitTypeBody(type, "}");
    }
}

/* 给方法添加该特性会记录方法入口，并记录返回的 Result<T> 错误 */
[AttributeUsage(.Method)]
struct LogAttribute : Attribute, IOnMethodInit
{
	[Comptime]
	public void OnMethodInit(MethodInfo method, Self* prev)
	{
		String emit = scope $"Logger.Log($\"Called {method}";
		for (var fieldIdx < method.ParamCount)
			emit.AppendF($" {{ {method.GetParamName(fieldIdx)} }}");
		emit.Append("\\n\");");
		Compiler.EmitMethodEntry(method, emit);

		if (var genericType = method.ReturnType as SpecializedGenericType)
		{
			if ((genericType.UnspecializedType == typeof(Result<>)) || (genericType.UnspecializedType == typeof(Result<,>)))
			{
				Compiler.EmitMethodExit(method, """
					if (@return case .Err)
					Logger.Log($"Error: {@return}");
					""");
			}
		}
	}
}

interface ISerializable
{
	public void Serialize(Serializer s);
}

/* 给类型添加该特性会添加并实现 ISerializable 接口 */
struct SerializableAttribute : Attribute, IOnTypeInit
{
	[Comptime]
	public void OnTypeInit(Type type, Self* prev)
	{
		Compiler.EmitAddInterface(type, typeof(ISerializable));

		Compiler.EmitTypeBody(type, """
			public void ISerializable.Serialize(Serializer serializer)
			{
			
			""");

		Compiler.EmitTypeBody(type, scope $"\tserializer.StartType(typeof({type.GetName(.. scope .())}));\n");
		for (let field in type.GetFields())
		{
			if (!field.IsInstanceField || field.DeclaringType != type)
				continue;

			Compiler.EmitTypeBody(type, scope $"\tserializer.Store(\"{field.Name}\", {field.Name});\n");
		}
		Compiler.EmitTypeBody(type, "\tserializer.EndType();\n}");
	}
}

[AttributeUsage(.Field | .StaticField)]
struct RangedAccessorAttribute : this(int minVal, int maxVal), Attribute, IOnFieldInit
{
	[Comptime]
	public void OnFieldInit(FieldInfo fieldInfo, Self* prev) mut
	{
		Compiler.EmitTypeBody(fieldInfo.DeclaringType, scope $"""
			public {(fieldInfo.IsStatic ? "static" : "")} {fieldInfo.FieldType} Ranged{fieldInfo.Name}
			{{
				get => {fieldInfo.Name};
				set
				{{
					System.Runtime.Assert((value >= {minVal}) && (value <= {maxVal}));
					{fieldInfo.Name} = value;
				}}
			}}
			""");
	}
}

/* 创建一个包装 'Val' 的 'RangedVal' 属性，仅允许 10 到 20 的值 */
[RangedAccessor(10, 20)]
static int Val;

```
