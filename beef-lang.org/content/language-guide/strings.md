+++
title = "字符串"
weight=80
+++

### 字符串概览
Beef 的 String 类型是可变对象，带有可调整的“小字符串优化”缓冲区，以 UTF8 存储字符数据。`StringView` 是一个 `Span<char8>` 的指针+长度结构体。按惯例，方法通过 `StringView` 参数接收字符串输入，需返回字符串数据的方法则使用 `String` 参数并向其追加数据。

在同一工作区内值相同的字符串字面量会被合并，并拥有相同的对象地址；通过 `char8*` 转换或 `CStr()` 方法得到的以 null 结尾的 C 字符串指针也保证地址相同。生成的字符串若与某个字面量值相同，经过 `String.Intern()` 会返回该字面量的地址。

```C#
String str = "This is a string";
char8* cStr = "This is a C string";
String str2 = "This string contains\n Two lines";
String str3 = @"C:\Path\Literal\NoSlashing";
String str4 =
	"""
	Multiline string literal
		with a tabbed second line;
	""";
```

更多字符串字面量信息参见 [字面量]({{< ref "literals.md#string" >}})。

### 字符串插值

字符串插值可用于字符串分配或字符串参数展开。

```C#
/* This is equivalent to scope String()..AppendF("X = {} Y = {}", GetX(), GetY()) */
String str = scope $"X = {GetX()} Y = {GetY()}";

/* This is equivalent to Console.WriteLine("X = {} Y = {}", GetX(), GetY()) */
Console.WriteLine($"X = {GetX()} Y = {GetY()}");
```

### 字符串对比

|名称           |可设定的非动态缓冲区                 |自定义动态分配器        |编码              |
|---------------|-------------------------------------|------------------------|------------------|
|C 字符数组     | 是（用户指定数组大小）              | 不适用（无分配）       | char/wchar*      |
|C++ std::string| 否（固定小缓冲优化）                | 模板参数               | char/wchar*      |
|C# StringBuffer| 否                                  | 否                     | UTF16            |
|Beef           | 是                                  | 虚方法重写             | UTF8             |

除 C 字符数组外，Beef 是唯一允许完全在栈上创建非小字符串（> 32 字节）的字符串实现：例如 `var str = scope String(1024)` 在栈上构建了一个内部缓冲为 1024 字符的 String。若需要超过内部缓冲的大小，可通过继承 String 并重写 Alloc/Free 与自定义分配器集成。在 C++ 中，字符串自定义分配器通过 `std::basic_string` 的模板参数提供，这意味着自定义分配器的字符串无法传给期望 `std::string` 的方法，因为类型不再匹配。就字符编码而言，C/C++ 不定义字符串字符的编码，因此编码问题完全交给用户处理。C# 的字符串出于历史原因使用 UTF16，这在很多方面是编码与大小的“双输”：用户仍需处理 UTF16 代理对（一个 Unicode 字符拆成两个 UTF16 字符），而 UTF16 字符串几乎总是比 UTF8 更大（即便仅处理亚洲语言）。

### 易用性 {#ease}

在处理字符串时，[参数级联运算符]({{< ref "operators.md#unary" >}}) 尤其有用。为了让用户掌控分配，字符串通常以参数形式传入方法并在其中被修改。这类方法多返回 void，因此不会损失返回值，但返回 Result<T> 的方法无法很好地使用这种方式。

```C#
void ToString(String strBuffer)
{
	strBuffer.Append("Count: ");
	count.ToString(strBuffer);
}

{
	let printString = scope String();
	ToString(printString);
	Console.WriteLine(printString);

	/* 与上面等价。ToString 被设计为返回传入的 String */
	let printString2 = ToString(.. scope String());
	Console.WriteLine(printString2);

	/* 等价的一行写法。'.' 可用于推断 'String'，因为传入 ToString 的类型明确 */
	Console.WriteLine(ToString(.. scope .()));
}

{
	let filePath = Path.InternalCombine(.. scope .(), rootDir, "Folder", "file.bin");
}
```

### 常见字符串错误

```C#
/* 错误 - 字符串字面量位于只读内存，不能修改 */
String newString = "Hello, ";
newString.Append(name);

/* 正确 */
String newString = scope String()..AppendF("Hello, {}", name);
/* 正确 - 与上面的代码等价 */
String newString = scope $"Hello, {name}";
```

```C#
/* 错误 - 剥夺了用户对分配的控制，并将释放内存的负担交给调用方 */
String GetName()
{
	return new String("Brian");
}

/* 更错误 - 返回时该内存已离开作用域，不能将栈分配内存返回给调用方 */
String GetName()
{
	return scope String("Brian");
}

/* 正确 - 注意使用 Append，可避免创建需要后续拼接的临时字符串 */
void GetName(String outName)
{
	outName.Append("Brian");
}
```

```C#
/* 错误 - 字符串值不能直接相加 */
String strC = strA + strB;
/* 正确 - 已提供分配 */
String strC = scope $"{strA}{strB}";
/* 正确 - 字符串常量可相加，因为结果仍是字符串常量，运行时无需分配 */
String strC = "A" + "B";
/* 正确 - 等价于 strC.Append(strA) */
strC += strA;
```
