+++
title = "运算符"
weight=75
+++

## 运算符概览
以下运算符分组按优先级从高到低排列。

### 主运算符
* `x.y` - 成员访问
* `x..y(<method args>)` - 成员访问级联。将方法 `y` 的结果替换为 `x`，用于将多个方法调用串联到同一目标上。例如：``string..Append("name: ").Append(name);`` 尽管 ``Append`` 返回 ``void``，但两个 ``Append`` 都作用于 ``string``。
* `x?.y` - 空条件成员访问。若 `x` 为 null，则结果为 null。
* `f(x)` - 方法调用
* `a[x]` - 数组索引

### 一元运算符 {#unary}
* `x++` - 后缀自增，先返回自增前结果再递增 x
* `x--` - 后缀自减，先返回自减前结果再递减 x
* `++x` - 前缀自增，先递增 x 再返回新值
* `--x` - 前缀自减，先递减 x 再返回新值
* `~x` - 按位取反
* `!x` - 逻辑取反
* `(T)x` - 将 `x` 转换为类型 `T`
* `&x` - 取 `x` 的地址
* `*x` - 解引用指针 `x`
* `x(..y, <other method args>)` - 参数级联。将方法 `x` 的结果替换为 `y`，用于复用参数。例如参见 [字符串易用性]({{< ref "strings.md#ease" >}})。

#### 乘除运算符 {#binary}
* `x * y` - 乘法
* `x &* y` - 禁用溢出检查的乘法
* `x / y` - 除法。若 `x` 与 `y` 为整数，则结果为向零截断的整数。
* `x % y` - 取余。若 `x` 与 `y` 为整数，则结果为 `x / y` 的余数。

#### 加减运算符
* `x + y` - 加法
* `x &+ y` - 禁用溢出检查的加法
* `x - y` - 减法
* `x &- y` - 禁用溢出检查的减法。允许两个无符号整数相减而不产生有符号整数。

### 移位运算符
* `x << y` - 左移
* `x >> y` - 右移。若 `x` 为有符号数，则左侧用符号位填充。

### 太空船运算符
* `x <=> y` - 若 `x < y` 返回负数，`x == y` 返回 0，`x > y` 返回正数

### 比较运算符
* `x < y`
* `x > y`
* `x <= y`
* `x >= y`
* `x is T` - 若 `x` 可转换为类型 `T` 则结果为 true
* `x as T` - 若可转换则将 `x` 转为 `T`，否则结果为 null

### 逻辑 AND 运算符
* `x & y` - 按位 AND

### 逻辑 XOR 运算符
* `x ^ y` - 按位 XOR

### 逻辑 OR 运算符
* `x | y` - 按位 OR

### 相等运算符

* `x == y`
* `x === y` - 严格相等
* `x != y`
* `x !== y` - 严格不等

严格相等运算符可用于检查引用相等，跳过所有相等运算符重载。对于结构体或元组等值类型，严格相等会执行逐成员的严格相等检查。

### 条件 AND 运算符
* `x && y`

### 条件 OR 运算符
* `x || y`

### 空合并运算符
* `x ?? y` - 若 `x` 非空则结果为 `x`，否则为 `y`

### 条件运算符 {#conditional}
* `x ? y : z` - 当 `x` 为 true 时结果为 `y`，否则为 `z`

### 赋值运算符 {#assignment}

赋值结果为 `x` 的新值。

* `x = y`
* `x += y`
* `x -= y`
* `x *= y`
* `x /= y`
* `x %= y`
* `x |= y`
* `x ^= y`
* `x <<= y`
* `x >>= y`
* `x ??= y`
* `=> x` - 方法绑定运算符

#### 类型属性运算符 {#typeattr}
* `sizeof(T)` - `T` 的大小。注意引用类型总是返回原生指针大小
* `alignof(T)` - `T` 的对齐
* `strideof(T)` - `T` 的步长（按 `T` 的对齐方式对齐）
* `alloctype(T)` - `new T()` 的结果，若 `T` 为引用类型则为 `T`，若为值类型则为 `T*`
* `comptype(x)` - 反射类型 `x` 的类型
* `decltype(x)` - 表达式 `x` 的结果类型。允许任意表达式（包括方法调用），但仅用于确定类型，不生成可执行代码。
* `nullable(T)` - 若 `T` 已是可空类型则为 `T`，否则为 `T?`
* `rettype(T)` - 委托或函数的返回类型
* `typeof(T)` - `T` 的反射类型
* `offsetof(T, field)` - `field` 在 `T` 中的字节偏移
* `nameof(T, field)` - `field` 在 `T` 中的名称

### 引用运算符
* `ref x` - 传入 `ref` 参数或需要 `ref` 的值时必需
* `out x` - 传入 `out` 参数时必需
* `var x` - 从 `out` 参数创建新变量 `x`
* `let x` - 从 `out` 参数创建新常量/只读变量 `x`

### Params 运算符
* params x - 当 x 为可变参数时，将这些参数传递给另一个可变参数。当 x 为委托或函数的 params 时，会在原地展开。

示例参见 [可变参数数量]({{< ref "datatypes/members.md#params" >}})。

### 范围运算符 {#range}
* `x...y` - 创建包含式范围，从 x 到 y（含 y）
* `x..<y` - 创建排除式范围，从 x 到 y（不含 y）

* `x...` - 创建索引范围，从 x 到集合末尾（含末尾）
* `...y` - 创建索引范围，从集合起始到 y（含 y）
* `x...^y` - 创建索引范围，从 x 到 y（含 y），但 y 从末尾计数（见下方 Index 运算符）。例如从列表第 `x` 个元素到从后往前数的第 `y` 个元素。

示例参见 [范围表达式]({{< ref "expressions.md#range" >}})。

### 末尾索引运算符
* `^x` - 创建一个从末尾计数的 Index（`.FromEnd`），其中最后一个元素为 ^1（^0 等于集合的数量/长度）

`Index` 主要用于集合索引与范围构造。

```C#
let list = scope List<int>() { 5, 1, 0 };

let first = list[0];
// first == 5

// 从末尾索引起点是 Count（此处为 3），超出范围，因此向前一位得到最后元素 -> ^1
let last = list[^1];
// last == 0
```

## 类型转换

`(T)x` 类型转换运算符可直接执行多种类型转换，但有一些特殊情况：
* 反装箱。`(T)obj` 其中 `obj` 为 `Object`、`T` 为值类型，会执行反装箱。该反装箱在 Debug 模式下（启用动态转换检查时）可能在运行时失败。可使用 `obj is T` 检查或 `obj as T?` 表达式安全反装箱。
* 获取对象地址：`(void*)obj`（`obj` 为 Object 类型）实际上是反装箱请求，而非类型转换。`System.Internal.UnsafeCastToPtr` 可返回 Object 的地址作为 `void*`。
* 转换为不相关类型。有时可使用双重转换实现原本非法的转换。例如 `(void*)handle` 中 `handle` 是带类型的基础类型 `struct Handle : int`，直接转为 `void*` 不允许，但 `(void*)(int)handle` 允许。

## 运算符重载

结构体与类可提供运算符重载。比较运算符的选择较灵活，不需要定义 <、<=、==、!=、>、>= 的全部组合。若存在“逆运算符”，会优先调用；或仅定义 <=> 运算符时，也可用于所有比较类型。

```C#
struct Vector2
{
	float x;
	float y;

	public this(float x, float y)
	{
		this.x = x;
		this.y = y;
	}

	/* 二元 + 运算符 */
	public static Vector2 operator+(Vector2 lhs, Vector2 rhs)
	{
		return .(lhs.x + rhs.x, lhs.y + rhs.y);
	}

	/* 一元 '-' 运算符 */
	public static Vector2 operator-(Vector2 val)
	{
		return .(-val.x, -val.y);
	}

	/* 一元 '++' 运算符 */
	public static Vector2 operator++(Vector2 val)
	{
		return .(val.x + 1, val.y + 1);
	}

	/* 非静态一元 '--' 运算符 */
	public void operator--() mut
	{
		x--;
		y--;
	}

	/* 赋值 '+=' 运算符 */
	public void operator+=(Vector2 rhs) mut
	{
		x += rhs.x;
		y += rhs.y;
	}

	/* 比较运算符 */
	public static int operator<=>(Vector2 lhs, Vector2 rhs)
	{
		/* 先比较 X，若 X 相等则比较 Y */
		int cmp = lhs.x <=> rhs.x;
		if (cmp != 0)
			return cmp;
		return lhs.y <=> rhs.y;
	}

	/* 来自 float[2] 的转换运算符 */
	public static operator Vector2(float[2] val)
	{
		return .(val[0], val[1]);
	}
}
```

二元运算符可标记为 `[Commutable]` 特性，从而允许某些运算符变换。例如，可交换的 "A < B" 运算符可用于 "B > A"、"!(A >= B)" 和 "!(B <= A)"。可交换的 "A == B" 运算符可用于 "B == A"、"!(A != B)" 与 "!(B != A)"。
