+++
title = "初始化"
weight = 20
+++

## 初始化
类的初始化语义如下：先将对象清零，然后按声明顺序执行根类字段初始化器，接着按声明顺序执行初始化块，然后运行根类构造函数，之后依次执行派生类的字段初始化器、初始化块和构造函数。即使在基类构造函数中调用虚方法，也会派发到最派生类型，这可能导致在某个类型构造函数尚未执行时就调用了该类型的方法。

```C#

class Person
{
	public String mFirstName = GetFirstName();
	public String mLastName = GetLastName();

	public this()
	{
		AddPerson();
	}
}

class Student : Person
{
	public School mSchool = GetSchool();
	public int? mAge;

	/* 初始化块无论调用哪个构造函数都会执行 */
	this
	{
		RegisterStudent();
	}

	public this()
	{

	}

	public this(int age)
	{
		mAge = age;
	}
}

/* 构造 Student 时，初始化顺序如下：
	mFirstName = GetFirstName()
	mLastName = GetLastName()
	AddPerson();
	mSchool = GetSchool()
	RegisterStudent();
 */

/* 类或结构体及其继承者或扩展也可以选择忽略所有初始化器，保留清零后的类/结构体 */
extension Person
{
	/* 为 Person 添加一个构造函数，因使用 'this(?)' 而不调用 GetFirstName()/GetLastName() */
	/* 派生类也可通过 base(?) 达到同样效果 */
	public this(String firstName, String lastName) : this(?)
	{
		/* 此时 mFirstName 与 mLastName 仍为 null */
		mFirstName = firstName;
		mLastName = lastName;
		AddPerson();
	}
}
```

对于结构体初始化语义，结构体不会自动清零——字段初始化器与构造函数必须共同完成所有字段的初始化。结构体的使用者也可以选择不执行构造函数，而是直接手动初始化所有字段。未完全初始化的结构体使用会在简单的静态分析中被禁止，但可通过显式的 "?" 未初始化表达式覆盖。

```C#
/* 在此示例中，我们知道 "UseVec" 会初始化 Vector2，因此使用 '?' 以避免“未初始化”错误 */
Vector2 vec = ?;
UseVec(&vec);
```

分配时也支持数组初始化。

```C#
/* 将末尾 7 个元素清零初始化 */
int[] iArr = new int[10] (1, 2, 3, );

/* 末尾 7 个元素保持未初始化 */
int[] iArr = new int[10] (1, 2, 3, ?);

/* 抛出初始化大小不匹配错误 - 若希望清零初始化，需要结尾逗号 */
int[] iArr = new int[10] (1, 2, 3);
```

值初始化器允许在创建时为字段与属性赋值。

```C#
/* 构造 Cat，调用默认构造函数，然后赋值 Age 与 Name 属性 */
var cat = new Cat() { Age = 10, Name = "Fluffy" };
```

对于结构体，值初始化器也可以在不调用任何构造函数或初始化块的情况下使用。

```C#
struct WindowInit
{
	public width = 1280;
	public height;

	this
	{
		height = 720;
	}
}

/* 默认构造函数 + 值初始化器：height 为 720，width 为 1280，然后被设为 1920 */
var init = WindowInit() { width = 1920 }

/* 等价于 */
var init = WindowInit();
init.width = 1920;

/* 仅值初始化器：height 为 0，width 为 0，然后被设为 1920 */
var init = WindowInit { width = 1920 }

/* 等价于 */
WindowInit init = default; // Struct is zeroed
init.width = 1920;
```

值初始化器还允许在创建时向集合添加项。可以提供表达式列表，它们会分别传给适用的 `Add` 方法。

```C#
var list = scope List<int>() {1, 2, 3, 4};
var weightDict = scope Dictionary<String, float>() { ("Roger", 212.3f), ("Sam", 110.2f) };
```

类型也可以定义静态字段与静态构造函数。静态初始化顺序由完全限定类型名的字母数字顺序决定。可使用 [StaticInitPriority(...)] 与 [StaticInitAfter(...)] 等类型特性覆盖该顺序。若静态初始化引用了其他类型的静态字段，会在访问前按需执行该类型的静态初始化器。不过，这种按需初始化可能导致循环依赖，处理方式是直接跳过重入的初始化调用。

## 析构

类可以定义析构函数。通常析构顺序与初始化顺序相反。

```C#

class Person
{
	public String mFirstName = GetFirstName() ~ delete _;
	public String mLastName = GetLastName() ~ delete _;

	public this()
	{
		AddPerson();
	}

	public ~this()
	{
		RemovePerson();
	}
}

class Student : Person
{
	public School mSchool = GetSchool() ~ ReleaseSchool(_);

	public this()
	{
		RegisterStudent();
	}

	public ~this()
	{
		UnregisterStudent();
	}
}

/* 删除 Student 时，反初始化顺序如下：
	UnregisterStudent()
	ReleaseSchool(mSchool)
	RemovePerson()
	delete mLastName
	delete mFirstName
*/

```

结构体不能定义析构函数，但可定义包含反初始化代码的 `Dispose` 方法。Dispose 可与 `using` 或 `defer` 语句配合，以 “RAII 风格”（作用域退出时调用）使用。
