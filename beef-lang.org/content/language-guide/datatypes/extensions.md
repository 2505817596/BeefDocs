+++
title = "扩展"
weight = 20
+++

## 类型扩展

类型定义可以被扩展，用户项目甚至可以扩展核心库中定义的类型，并添加额外数据字段。

```C#
/* 为线程创建添加时间戳。可在用户代码任意位置包含。 */
namespace System.Threading
{
	extension Thread
	{
		DateTime mCreateTime = DateTime.Now;
	}
}
```


泛型类型可以基于匹配的接口约束进行条件扩展。

```C#
namespace System.Collections
{
	extension List<T> where T : IOpComparable
	{
		public T GetMax()
		{
			if (mSize == 0)
				return default;
			T max = this[0];
			for (let val in this)
			{
				// 没有 IOpComparable 约束，这个 '>' 检查会失败
				if (val > max)
					max = val;
			}
			return max;
		}
	}
}
```

扩展可用于为不受你控制的类型（如系统类型或其他库中的类型）添加接口一致性。

注意扩展中定义的方法遵循依赖可见性规则：只有当调用点所在项目依赖于定义该方法的项目，或者某个泛型参数所在项目依赖于定义该方法的项目时，方法才可被调用。这同样适用于运算符重载——例如为 `System.Collections.List<T>` 提供 `operator==` 重载时，该重载不会在 `corlib` 或其他库内部对 `List<T>` 的相等比较中被调用。

扩展可以提供构造函数、析构函数和字段初始化器。

```C#
namespace System.Collections
{
	extension List<T>
	{		
		public int mID = GetId();

		/* 此构造函数的可见性遵循上述规则。 */
		/* 仍需调用根定义的构造函数进行初始化，因此在此调用它。注意若没有 `[NoExtension]` 将会递归调用自身 */
		public this() : [NoExtension]this()
		{

		}
		
		/* 无论调用哪个构造函数，该初始化块都会执行 */
		this
		{
			RegisterList(this);
		}

		/* 该析构函数在根定义的析构函数之前运行 */
		public ~this()
		{
			UnregisterList(this);
		}
	}
```

扩展还可通过提供方法声明并由依赖项目实现的方法来反转项目间依赖关系。这种技术提供了虚方法之外的静态派发方案。

```C#
/* In project 'Engine' */
class Platform
{
	public extern Texture CreateTexture();
}

/* In project 'DirectXEngine' */
extension Platform
{
	public override Texture CreateTexture()
	{
		return new DirectXTexture();
	}
}
```

我们还可以使用扩展在不需要子类的情况下重写虚方法。其优点是不需要重载，但缺点是仍为动态派发。

```C#
/* In project 'Engine' */
class Platform
{
	public virtual Texture CreateTexture() => null;
}

/* In project 'DirectXEngine' */
extension Platform
{
	public override Texture CreateTexture()
	{
		return new DirectXTexture();
	}
}
```

## 扩展方法

扩展方法可在不修改原始类型的情况下为现有类型“虚拟地”添加方法。扩展方法是静态方法，但调用方式与被扩展类型的实例方法类似，或用于满足一组泛型约束的类型。当你希望将方法范围限制在特定命名空间或工具方法中，或该方法面向符合特定泛型约束的广泛类型时，扩展方法比类型扩展更合适。

```C#


/* 在全局命名空间为 String 提供 CharCount 方法 */
static
{
	public static int CharCount(this String str, char8 c)
	{
		int total = 0;
		for (let checkC in str.RawChars)
			if (checkC == c)
				total++;
		return total;
	}
}

/* 为任何可相加的 T 提供 List<T>.Total 方法。
 该方法仅在使用 'using static' 显式导入该类型时可见 */
static class ListUtils
{
	public static T Total<T>(this List<T> list) where T : IOpAddable
	{
		T total = default;
		for (let val in list)
			total += val;
		return total;
	}
}

/*************************************************/

int charCount = "Test string".CharCount('s');

using static ListUtils;
int GetListTotal(List<int> list) => list.Total();


```
