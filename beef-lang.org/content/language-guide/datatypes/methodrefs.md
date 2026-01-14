+++
title = "方法引用"
+++

## 函数指针
函数指针类型可指向静态方法、带显式 `this` 参数的非静态方法、外部函数，或由库动态生成的函数。函数指针通过 `=>` 运算符生成。
```C#
/* 局部变量 `funcPtr` */
function void() funcPtr = => StaticMethod;
funcPtr();
/* 作为参数传递给另一个方法 */
void UseFuncPtr(function void() funcPtrB) => funcPtrB();
UseFuncPtr(=> StaticMethodB);
/* 注意此处未捕获 'ca'。也可以写成 'ClassA.MemberMethod' */
ClassA ca = new ClassA();
function void(ClassA this, float f) funcPtr2 = => ca.MemberMethod;
funcPtr2(ca, 1.2f);
/* 注意结构体需要区分 mut/非 mut */
StructA sa = StructA();
function void(mut StructA this, float f) funcPtr3 = => sa.MemberMethod; 
funcPtr3(mut sa, 2.3f);
/* 注意：在互操作中，所有 [CRepr] 结构体都会以指针传递，无论是否指定 mut，调用约定相同 */
```

## 委托
委托更为通用，定义为类类型，不仅能引用函数指针可引用的对象，还能引用实例方法，并在指向局部方法或 lambda 时捕获局部变量。委托通过带分配标记的 `=>` 运算符分配。
```C#
delegate void() delegateVal = scope => MemberMethod;
```

## 事件
事件可视为多播委托。`System.Event<T>` 结构体包装了委托类型，可包含零个或多个委托引用。
```C#
Event<delegate void(int)> evt = default;
/* 注意使用 'new =>'，因为 Event 会接管委托的所有权 */
evt.Add(new => obj.MethodA);
evt.Add(new => obj.MethodB);
/* 将按添加顺序调用委托 */
evt(intVal);
/* 移除单个委托。注意使用 'scope'，因为该参数仅用于比较，不会转移所有权 */
evt.Remove(scope => obj.MethodA, true);
/* Dispose 将删除所有剩余委托 */
evt.Dispose();
```

## Lambda
Lambda 是创建局部方法并分配委托指向它的简写形式，同时允许定义 lambda 析构函数，用于释放 lambda 生命周期内所需的资源。

```C#
static void Test(StringView str)
{
	int i = 0;

	/* 本地定义的方法，按引用捕获 'i' */
	char8 GetNext()
	{
		if (i >= str.Length)
			return 0;
		return str[i++];
	}

	/* 本地定义的方法，不捕获任何变量 */
	char8 GetEmpty()
	{
		return 0;
	}

	/* 分配绑定到 GetNext() 的委托 */
	delegate char8() strDlg = scope => GetNext;

	/* 将 emptyFunc 绑定到 GetEmpty()。绑定到 GetNext() 会失败，因为函数指针无法持有捕获 */
	function char8() emptyFunc = => GetEmpty;

	/* 分配 lambda */
	strDlg = scope () =>
	{
		return 'A';
	};

	/* 分配按引用捕获的 lambda，使 GetNext 调用能够捕获 'i' */
	strDlg = scope [&] () =>
	{
		return GetNext();
	};

	/* 该 lambda 持有一个字符串，在 lambda 作用域结束后清理 */
	String tempStr = new String(str);
	tempStr.EnsureNullTerminator();
	/* 按引用捕获 'i'，按值捕获 str 与 tempStr */
	strDlg = scope [&i, =str, =tempStr] () =>
	{
		return tempStr[i];
	}
	~
	{
		delete tempStr;
	};
}
```

## 无值方法引用
无值方法引用可用于特化某些泛型方法，使其直接调用目标方法而非通过委托间接调用。在某些关键路径上可提升性能。

```C#
static T Max<T, TFunc>(T lhs, T rhs, TFunc func) where TFunc : delegate int(T lhs, T rhs)
{
	return (func(lhs, rhs) >= 0) ? lhs : rhs;
}

int CmpVec(Vector2 lhs, Vector2 rhs)
{
	return lhs.x <=> rhs.x;
}

/* 该调用的 'func' 参数为无值引用 - 会生成特化的 'Max' 方法直接调用 CmpVec */
Vector2 max = Max(vec0, vec1, => CmpVec);
```
