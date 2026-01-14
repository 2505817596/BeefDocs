+++
title = "语句"
weight = 42
+++

### break/continue
我们可以使用 "break" 终止执行 "for"、"while" 或 "do" 语句。使用 "continue" 可跳到 "for" 循环的下一次迭代；默认情况下，这些语句作用于当前作用域中最深层的适用语句，但可以使用标签引用更外层的语句。这在跳出嵌套循环等场景中很有用。

```C#
Y: for (int y < height)
	for (int x < width)
		if (Check(x, y))
			break Y;
```

### defer
defer 语句用于将方法调用或代码块的执行推迟到退出某个作用域时。当延迟的是方法调用时，其参数（包括 `this`）会立刻求值。

```#C
/* 以下将输出 "End:2 B:1 A:0"。注意执行顺序与 defer 的声明顺序相反。 */
{
	int i = 0;
	defer Console.WriteLine("A:{}", i);
	i++;
	defer Console.WriteLine("B:{}", i);
	i++;
	Console.WriteLine("End:{}", i);
}

/* 以下将输出 "End:2 B:2 A:2"。defer 位置没有参数需要求值，所以 WriteLine 使用的是作用域结束时的当前 i 值。 */
{
	int i = 0;
	defer
	{
		Console.WriteLine("A:{}", i);
	}
	i++;
	defer
	{
		Console.WriteLine("B:{}", i);
	}
	i++;
	Console.WriteLine("End:{}", i);
}

/* defer 语句可指定作用域目标。以下将在退出包含方法时打印 9 到 0。 */
{
	for (int i < 10)
	{
		defer:: Console.WriteLine("i={}", i);
	}
}
```

### delete

`delete` 语句释放已分配的内存。（参见 [内存管理]({{< ref "memory.md" >}})）

* `delete x` - 释放由全局分配器分配、由 `x` 引用的内存。
* `delete:a x` - 释放由自定义分配器 `a` 分配的内存。

当 `x` 为对象时，会调用析构函数。

### do
允许创建一个可被 break 跳出的非循环代码块，可在某些代码模式中减少 `if` 嵌套。

```C#
do
{
	c = NextChar();
	if (c == 0)
		break;
	op = NextChar();
	if (c != '+')
		break;
	c2 = NextChar();
}
```

### for
for 循环通常用于遍历集合或数列。可用形式如下。

```C#
/* 经典 C 风格循环，包含初始化、条件和迭代器 */
for (int i = 0; i < count; i++)
{
}

/* 上述形式的简写 */
for (int i < count)
{
}

/* 遍历 List<int> 中的元素 */
for (let val in intList)
{
	if (val == 0)
		@val.Remove();
}

/* 上述等价于 */
var enumerator = intList.GetEnumerator();
while (enumerator.GetNext() case .Ok(let val))
{
	if (val == 0)
		enumerator.Remove();
}
enumerator.Dispose();

/* 我们还可以按引用遍历而非按值遍历 */
for (var valRef in ref intList)
	valRef += 100;


/* 常见错误：这不会改变 intList 的值，只会改变 enumerator.GetNext() 返回的值 */
for (var val in intList)
	val += 100;
```

### if

```C#
if (i > 0)
	Use(i);
else if (i == 0)
	Use(1);
else
	break;

/* Some variable declarations can be used as a condition */

/* Use 'not null' as a condition */
if (var str = obj as String)
	Console.WriteLine(str);

/* Unwrap a Result<int> */
if (int i = intResult)
	Use(i);

/* Unwrap an int? */
if ((int i = intNullable) && (i != 0))
	Use(i);

```

### return
从方法返回一个值。

```C#
int GetSize()
{
	return mSize;
}
```

### repeat while
先执行一次语句，只要条件为真就继续重复执行。

```C#
bool wantsMore;
repeat
{
	wantsMore = Write();
}
while (wantsMore);
```

### switch

```C#
/* Note that 'break' is not required after a case */
switch (c)
{
case ' ', '\t':
	Space();
case '\n'
	NewLine();
default:
	Other();  
}

switch (c)
{
/* Cases can contain additional 'when' conditions */
case '\n' when isLastChar:
	SendCommand();
case 'A':
	WasA();
	/* 'fallthrough' continues into the next case block*/
	fallthrough;
case 'E', 'I', 'O', 'U':
	WasVowel();
}

switch (shape)
{
/* Pattern match, with '?' discard */
case .Circle(0, 0, ?):
	HandleOriginCircle();
case .Circle(let x, let y, let radius):
	HandleCircle(x, y, radius);
default:
}
```

Switches can be used for [pattern matching]({{<ref "pattern.md">}}) (see for further examples) for enums and tuples.
```C#
/* Note that switches over enums that do not handle every case and have no 'default' case will give a "not exhaustive" compile error. Thus, if we added a new entry to the Shape definition, we would ensure that all switches will make modifications to handle it */
switch (shape)
{
case Square(let x, let y, let width, let height):
	DrawSquare(x, y, width, height);
case Circle:
	IgnoreShape();
}
```

Switches can operate on non-integral types, as well.

```C#
switch (str)
{
case "TEST":
	PerformTest();
case "EXIT":
	Exit();
default:
	Fail();
}
```

### using
`using` 语句表示对值的作用域使用。

```C#
using (g.Open("Header"))
{
	g.Write("Hello");
}

/* These are equivalent */
{
	var res = g.Open("Header");
	g.Write("Hello");
	res.Dispose();
}

/* Or */
{
	defer g.Open("Header").Dispose();
	g.Write("Hello");
}
```

### 变量声明

```C#
// 定义可变变量，显式类型且不赋值
int val;
// 定义多个变量，显式类型
int a, b;
// 定义可变变量，显式类型并初始化
int val2 = 123;
// 定义可变变量，隐式类型并初始化
var val3 = 1.2f;
// 定义不可变变量，隐式类型并初始化
let val4 = 2.3;
```

### while
当条件为真时重复执行语句。

```C#
while (i >= 0)
{
	i--;
}
```
