+++

title = "语言特性"
weight = 10
widget = "blank"
headless = true  # This file represents a page section.
active = true  # Activate this widget? true/false

[design]
columns = "1"

+++

{{< rawhtml >}}

<style>
.hljs {	
	background-color:#e0e0e0;
}

div.Example {
	display: none;
	position: relative; 
	top: 4px;
	width: 400px;
}

.hljs-title, .hljs-section, .hljs-selector-id {
    color: #458;
    font-weight: bold;
}

</style>

<table>
<tr>
<td width=400px style="vertical-align: top;">
<ul>
<li>与静态/动态库的 C/C++ 互操作</li>
<li>自定义分配器</li>
<li>批量分配</li>
<li>编译期函数求值</li>
<li>编译期代码生成</li>
<li>标签联合（Tagged union）</li>
<li>泛型</li>
<li>元组</li>
<li>反射</li>
<li>属性</li>
<li>Lambda 表达式</li>
<li>无值方法引用</li>
<li>defer 语句</li>
<li>SIMD 支持</li>
<li>类型别名</li>
<li>类型扩展</li>
<li>模式匹配</li>
<li>范围</li>
</ul>
</td>
<td width=300px bgcolor=#aaaa80 style="vertical-align: top;">
<ul>
<li>字符串插值</li>
<li>参数级联</li>
<li>Mixin 方法</li>
<li>接口</li>
<li>自定义特性</li>
<li>不可变值</li>
<li>运算符重载</li>
<li>命名空间</li>
<li>位集</li>
<li>原子操作</li>
<li>带检查的算术</li>
<li>值装箱</li>
<li>动态 FFI</li>
<li>局部方法</li>
<li>预处理器</li>
<li>保证内联</li>
<li>增量编译</li>
<li>内置测试</li>
</ul>
</td>
<td style="vertical-align: top;">
<select id="ExampleSelect" style="width: 100%;">
</select>

<!-------------------------------------------------------------------------------------->
<div id="Hello World" class="Example" data-label="你好，世界">
{{< /rawhtml >}}
```C#
using System;

class Program
{
	public static void Main()
	{
		Console.WriteLine("Hello, World!");
	}
}
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="File IO and error handling" class="Example" data-label="文件 IO 与错误处理">
{{< /rawhtml >}}
```C#
// Try! propagates file and parsing errors down the call stack
static Result<void> Parse(StringView filePath, List<float> outValues)
{
	var fs = scope FileStream();
	Try!(fs.Open(filePath));
	for (var lineResult in scope StreamReader(fs).Lines)
	{
		for (let elem in Try!(lineResult).Split(','))
			outValues.Add(Try!(float.Parse(elem)));
	}
	return .Ok;
}
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Tuples" class="Example" data-label="元组">
{{< /rawhtml >}}
```C#
// Method returning a tuple
(float x, float y) GetCoords => (X, Y);

var tup = GetCoords;
if (tup != (0, 0))
	Draw(tup.x, tup.y);

// Decompose tuple
var (x, y) = GetCoords;
Draw(x, y);
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Ranges" class="Example" data-label="范围">
{{< /rawhtml >}}
```C#
for (let i in 10...20)
	Console.WriteLine($"Value: {i}");

let range = 1..<10;
Debug.Assert(range.Contains(3));

Span<int> GetLast10(List<int> list) => list[...^10];
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Strings" class="Example" data-label="字符串">
{{< /rawhtml >}}
```C#
// Allocate a string with a 4096-byte internal UTF8 buffer, all on the stack
var str = scope String(4096);

// String interpolation, formatting in 'x' and 'y' values
var str2 = scope $"x:{x} y:{y}";

// Create a view into str2 without the first and last character
StringView sv = str2[1...^1];

// Get a pointer to a null-terminated C string
char8* strPtr = str2;
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Tagged unions (aka enums with payloads)" class="Example" data-label="标签联合（即带载荷的枚举）">
{{< /rawhtml >}}
```C#
enum Shape
{
	case None;
	case Square(int x, int y, int width, int height);
	case Circle(int x, int y, int radius);
}

Shape drawShape = .Circle(10, 20, 5);

switch (drawShape)
{
case .Circle(0, 0, ?):
	HandleCircleAtOrigin();
case .Circle(let x, let y, let radius):
	HandleCircle(x, y, radius);
default:
}

if (drawShape case .Square)
	HandleSquare();
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Local methods" class="Example" data-label="局部方法">
{{< /rawhtml >}}
```C#
void Draw(List<Variant> values)
{
	int idx = 0;
	float NextFloat()
	{
		return values[idx++].Get<float>();
	}
	DrawCircle(NextFloat(), NextFloat(), NextFloat());
}
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Batched allocations (aka append allocations)" class="Example" data-label="批量分配（即追加分配）">
{{< /rawhtml >}}
```C#
// This class uses 'append' allocations, which allows a single "batch" 
//  allocation which can accomodate the size of the 'Record' object, the 
//  'mName' String (including character data), and the 'mList' list, 
//  including storage for up to 'count' number of ints
class Record
{
	public String mName;
	public List<int> mList;

	[AllowAppend]
	public this(StringView name, int count)
	{
		var nameStr = append String(name);
		var list = append List<int>(count);

		mName = nameStr;
		mList = list;
	}
}

// The following line results in a single allocation of 80080 bytes
var record = new Record("Record name", 10000);
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Interop" class="Example" data-label="互操作">
{{< /rawhtml >}}
```C#
[CRepr]
struct FileInfo
{
	c_short version;
	c_long size;
	c_char[256] path;
}

/* Link to external C++ library method */
[CallingConvention(.Cdecl), LinkName(.CPP)]
extern c_long GetFileHash(FileInfo fileInfo);

/* Import optional dynamic method - may be null */
[Import("Optional.dll"), LinkName("Optional_GetVersion")]
static function int32 (char8* args) GetOptionalVersion;
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Reflection" class="Example" data-label="反射">
{{< /rawhtml >}}
```C#
void Serialize(SerializeContext ctx, Object obj)
{
 	for (let field in obj.GetType().GetFields())
	{
		Variant v = field.GetValue(obj);
		ctx.Serialize(field.Name, v);
		if (let attr = field.GetCustomAttribute<OnSerializeAttribute>())
		{
			var m = attr.mSerializeType.GetMethod("SerializeField").Value;
			m.Invoke(null, obj, field);
		}
	}
}
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Mixins" class="Example" data-label="Mixin">
{{< /rawhtml >}}
```C#
static mixin Inc(var val)
{
    if (val == null)
        return false;
    (*val)++;
}

static bool Inc3(int* a, int* b, int* c)
{
	// "return false" from mixin is injected into the Inc3 callsite
    Inc!(a);
    Inc!(b);
    Inc!(c);
    return true;
}
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Type Extensions" class="Example" data-label="类型扩展">
{{< /rawhtml >}}
```C#
// Declare List<T>.DisposeAll() for all disposable types
namespace System.Collections
{
	extension List<T> where T : IDisposable
	{
		public void DiposeAll()
		{
			for (var val in this)
				val.Dispose();
		}
	}
}
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Compile-time function evaluation" class="Example" data-label="编译期函数求值">
{{< /rawhtml >}}
```C#
static int32 Factorial(int32 n) => n <= 1 ? 1 : (n * Factorial(n - 1));
const int fac = Factorial(8); // Evaluates to 40320 at compile-time

public static Span<int32> GetSorted(String numList)
{
	List<int32> list = scope .();
	for (var sv in numList.Split(','))
		list.Add(int32.Parse(sv..Trim()));		
	return list..Sort();
}
const int32[?] iArr = GetSorted("8, 1, 3, 7"); // Results in int32[4](1, 3, 7, 8)
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
<div id="Compile-time code generation" class="Example" data-label="编译期代码生成">
{{< /rawhtml >}}
```C#
// Create serialization method at compile-time on types with [Serialize]
// No runtime reflection required
[AttributeUsage(.Types)]
struct SerializableAttribute : Attribute, IComptimeTypeApply
{
	[Comptime]
	public void ApplyToType(Type type)
	{
		let code = scope String();
		code.Append("void ISerializable.Serialize(SerializationContext ctx)\n{");
		for (let field in type.GetFields())
			code.AppendF($"\n\tctx.Serialize(\"{field.Name}\", {field.Name});");
		code.Append($"\n}");
		Compiler.EmitAddInterface(type, typeof(ISerializable));
		Compiler.EmitTypeBody(type, code);
	}
}
```
{{< rawhtml >}}
</div>

<!-------------------------------------------------------------------------------------->
</td>
</tr>
</table>

<script>
	function Check(node, value)
	{		
		node.style.display = (value == node.id) ? "block" : "none";		
	}

	function ExampleSelected()
	{  				
		const nodes = document.getElementsByClassName("Example");
		for (let i = 0; i < nodes.length; i++)
		{			
			Check(nodes[i], this.value);
			nodes[i].style.width = "700px";
		}
	}

	select = document.getElementById("ExampleSelect");
	const nodes = document.getElementsByClassName("Example");
	for (let i = 0; i < nodes.length; i++)
	{
		var option = document.createElement("option");		
		option.value = nodes[i].id;
		option.text = nodes[i].dataset.label || nodes[i].id;
		if (nodes[i].id == "Hello World")
			option.selected = true;
		select.appendChild(option);		
	}

	select.onchange = ExampleSelected;	
	Check(document.getElementById("Hello World"), "Hello World");	
</script>

{{< /rawhtml >}}
