+++
title = "数据类型"
weight = 40
alwaysopen = true
+++

## 基本数据类型
Beef 提供以下基础类型：

#### 整数类型

* int
* int8
* int16
* int32
* int64
* uint
* uint8
* uint16
* uint32
* uint64

int 与 uint 使用平台原生整数宽度，并被视为独立类型，而非显式大小类型（如 int64/uint64）的别名。

#### 浮点类型

* float
* double

#### 字符类型

字符类型只指定大小而不指定编码。例如 char8 可以表示 UTF8 的一个字节或一个 ASCII 字符，具体取决于上下文。

* char8
* char16
* char32

#### 无值类型

* void

#### 布尔类型

* bool

## 结构体
结构体是用户定义的值集合，类似于 C 中的 struct。结构体可包含字段、属性、方法，并可定义内部类型。对 C++ 程序员而言，这些结构体类似 C++ POD，不支持虚方法或拷贝构造函数。

```C#
struct Vector
{
	public float x;
	public float y;

	/* 默认构造函数 - 构造函数必须设置所有字段
	 "this = default;" 等价于 "x = 0; y = 0;" */
	public this()
	{
		this = default;
	}

	/* 带参数构造函数 */
	public this(float x, float y)
	{
		this.x = x;
		this.y = y;
	}

	/* 计算长度的属性 */
	public float Length
	{
		get
		{
			return Math.Sqrt(x*x + y*y);
		}
	}

	/* 结构体的方法需声明为 'mut' 才能修改结构体 */
	public void SetZero() mut
	{
		x = 0;
		y = 0;
	}
}
```

结构体可以为零大小，这使某些泛型模式得以高效实现，而这些在不允许零大小结构体的 C++ 中需要其他方法。另外，Beef 的结构体设计为比 C 的对应结构体产生更少的“对齐空洞”：字段按对齐大小排序，次级再按声明顺序排序；在 Beef 中结构体大小与步长是不同概念，而 C 中结构体大小总是按最大成员对齐，因此等同于步长。字段重排可通过 [Ordered] 特性禁用，结构体可通过 [CRepr] 标记为完全 C 互操作。字段对齐打包可通过 [Packed] 禁用。可用 [Union] 创建联合体。

```C#
struct StructA
{
	int32 i;
	int64 j;
}

struct StructB : StructA
{
	int8 k;
}

struct StructC : StructB
{
	int8 l;
}

/* 在 Beef 中，StructC 仅占 14 字节，而在 C 中则为 24 或 32 字节
（取决于实现）。

 Beef:
 sizeof(StructA) = 12 strideof(StructA) = 16
 sizeof(StructB) = 13 strideof(StructB) = 16
 sizeof(StructC) = 14 strideof(StructC) = 16

 C/C++:
 sizeof(StructA) = 16
 sizeof(StructB) = 24
 sizeof(StructB) = 32（或 24）

 在 Beef 中，由于字段重排消除了对齐填充，数据大小更小。C 中，StructB 新增的 'k' 字节
会导致额外 7 字节填充。StructC 新增的 'l' 字节在某些编译器（VC）上会再增加 7 字节填充，
而其他编译器会将 'l' 填入之前的填充中（Clang/GCC）。 */
```

```C#
/* 自动构造函数为简单数据类型提供简写。
 以下两种类型定义等价 */
struct Vector3 : this(float x, float y, float z);

struct Vector3
{
	public float x, y, z;
	public this(float x, float y, float z)
	{
		this.x = x;
		this.y = y;
		this.z = z;
	}
}
```

通过省略结构体主体可创建不透明结构体定义，用于互操作且不允许直接分配（因为大小未知）。

## 元组类型 {#tuples}

元组是一种特殊的结构体。其语法可更简洁地表达某些代码模式，但不能定义属性或方法。

```C#
let tup = (1, 2); // 无名称成员
int sum = tup.0 + tup.1; // 按位置访问
let (first, second) = tup; // 解构到新变量
let coords = (x: 2, y: 3); // 有名称成员
let len = Math.Sqrt(coords.x*coords.x + coords.y*coords.y); // 按名称访问

(uint, uint) utup = (1, 2); // 显式类型、无名称成员
(int index, Type type) entry = (first, null); // 显式类型、有名称成员
 ```

元组允许隐式转换：逐字段类型相同，且一方字段有名称而另一方无名称。

## 类
类是引用类型（类似 C#、Swift、Java），概念上与结构体相似，但总是带有类型类指针，用于虚方法调用、动态类型和动态接口派发。所有用户类最终都继承自 System.Object。

```C#
abstract class Shape
{
	float x;
	float y;

	public abstract void Draw();
}

class Circle : Shape
{
	float radius;

	public override void Draw()
	{
		DrawCircle(x, y, radius);
	}

	/* 与结构体不同，修改类的成员方法不需要声明为 'mut' */
	public void DoubleSize()
	{
		radius *= 2;
	}
}
```

类可以定义析构函数，单个字段也可以定义字段析构函数，用于更紧密地将销毁与初始化关联起来。

销毁顺序与构造相反。构造时，先构造基类，再按声明顺序执行字段构造，最后执行类构造函数。销毁时，先执行类析构函数，再按声明逆序执行字段析构，最后销毁基类。

```C#
public Button : Widget
{
	String mLabel = new String() ~ delete _; // "_" 在此处是 "mLabel" 的别名

	public ~this()
	{
		RemoveWidget(this);

		/* 字段析构在此之后执行，顺序与初始化相反 */
	}
}
```

### 数组

Beef 支持多种数组形式：数组类、定长数组类型、Span 以及原始指针。

```C#
/* 分配 float 数组类 */
float[] floatArr = new float[3];

/* 分配二维 float 数组类 */
float[,] floatArr2D = new float[3, 2];

/* 这是定长数组，类似包含四个值的元组 */
float[4] sizedFloatArr = .(100, 200, 300, 400);
int[?] inferredSizeArr = .(500, 600);
let inferredSizeArr2 = int[?](700, 800, 900);

/* Span 是指针/大小的值类型对 */
Span<float> floatSpan = floatArr;

/* 原始指针。末尾的 "*" 表示原始数组分配而非 float[] 对象 */
float* floatPtr = new float[3]*;
```

### 枚举
Beef 的枚举类型可表示一组命名的整数常量。除非显式指定，枚举的底层类型将是能容纳所有指定值的最小整数类型。

```C#
enum Direction
{
	North,
	East,
	South,
	West
}

/* 注意无需写成 "Direction.South"，因为初始化器的期望类型是 "Direction"，因此已可推断 */
Direction facing = .South;

/* 特殊值 '_' 会求值为上一个字段的值，便于定义位标志 */
enum MultiHue
{
	Black = 0,
	Red = 1,
	Green = _*2,
	Blue = _*2
}
```

枚举也可使用更冗长的语法定义，以便像值类型结构体一样添加方法和属性。

```C#
enum Direction
{
	case North;
	case East;
	case South;
	case West;

	public Direction Opposite
	{
		get
		{
			switch (this)
			{
			case .North: return .South;
			case .East: return .West;
			case .South: return .North;
			case .West: return .East;
			}
		}
	}
}
```
枚举提供一些操作以便检查和枚举其值。

```C#

/* 从最小枚举值遍历到最大枚举值 */
for (var direction = typeof(Direction).MinValue; direction <= typeof(Direction).MaxValue; direction++)
{
}

/* 转换为底层整数表示（此处为 int8） */
let val = direction.Underlying;
/* 类似上面，但结果是可赋值的整数引用 */
let valRef = ref direction.UnderlyingRef;


```

枚举还可让多个值作为一组标志使用，支持类型安全的按位二元操作，并提供便捷的 "HasFlag" 方法用于检查是否设置了某组标志。

枚举也可为每个 case 定义关联数据，使其表现为类型安全的“可辨识联合体”。

```C#
enum Shape
{
	case None;
	case Square(int x, int y, int width, int height);
	case Circle(int x, int y, int radius);
}
Shape drawShape = .Circle(10, 20, 5);
...
switch (drawShape)
{
case .None:
case .Square(let x, let y, let width, let height): DrawSquare(x, y, width, height);
case .Circle(let x, let y, let radius): DrawCircle(x, y, radius);
}
....
if (drawShape case .Square)
	Console.WriteLine("We drew a square");

if (drawShape not case .Square)
	Console.WriteLine("We did not draw a square");

/* 若值不是 Circle，则 radius 仍保持 -1 */
int radius = -1;
if (drawShape case .Circle(?, ?, ref radius)) {}

```

### 可空类型

可空类型是值类型的枚举包装（System.Nullable<T>），使值类型能够使用通常只适用于指针与引用类型的 null 语义。

```C#
int? val = null;
int i = val ?? 21;
if (val == null)
	val = 42;
int? val2 = val + 123;
```

### 类型别名

Beef 的类型别名允许创建直接映射到另一个类型的类型名。

```C#
typealias Size = int;
typealias Collection<T> = List<T>;
typealias StringLookup = Dictionary<String, String>;
```
