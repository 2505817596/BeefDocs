+++
title = "代码生成"
+++

## 生成器概览

Beef IDE 允许通过源代码中的生成器来生成源文件。最简单的例子是 ``NewClassGenerator``。在工作区面板中右键并选择 “Generate File...” 即可访问。继承自 ``Compiler.Generator`` 的类会自动显示在 “Generate File” 面板的 “Generator” 下拉框中。生成器方法由编译器直接执行，无需手动编译。

生成器既可用于创建自定义文件模板，也可生成包含完整源码的整份文件。

```C#
public class NewClassGenerator : Compiler.Generator
{
	public override String Name => "New Class"; // This is was the generator will show up as in the "Generator" dropdown

	public override void InitUI()
	{
		AddEdit("name", "Class Name", "");
	}

	public override void Generate(String outFileName, String outText, ref Flags generateFlags)
	{
		var name = mParams["name"];
		if (name.EndsWith(".bf", .OrdinalIgnoreCase))
		  name.RemoveFromEnd(3);

    outFileName.Append(name);
    outText.AppendF(
  		$"""
  		namespace {Namespace}
  		{{
  			class {name}
  			{{
  			}}
  		}}
  		""");
	}
}
```

可在上述任一方法中调用 ``Fail(...)`` 以返回错误。

## 生成器 UI

当在 “Generator” 下拉框中选择生成器时，会调用 ``InitUI()``，用于创建生成器输入所需的 UI。传给这些方法的第一个参数是参数名，稍后可由 ``Generate`` 方法用于读取传入值。

```C#
AddEdit(...); // Adds an input field
AddCombo(...); // Adds a dropdown selection
AddCheckbox(...); // Adds a checkbox

// For example, a dropdown selection with the options A, B and C, where B is the default selection.
AddCombo("type", "Type", "B", StringView[?]("A", "B", "C"));

// ... in Generate, we can retrieve this input from mParams
let type = mParams["type"];
```

## 源码生成

在 ``Generate()`` 中，通过向方法传入的 ``outFileName`` 与 ``outFile`` 字符串追加内容来输出结果。``outFileName`` 不应包含 ".bf" 扩展名。

所有输入都是 ``StringView``，通过 ``mParams[...]``（使用在 ``InitUI`` 中定义的参数名）获取。注意：对于复选框，返回字符串为 ``bool.TrueString`` 或 ``bool.FalseString``。也可使用 ``GetString(...)`` 将 ``StringView`` 追加到传入字符串。

```C#
// For example...
let type = mParams["type"];
bool option = (mParams["option"] == bool.TrueString);
```

可通过 getter 访问若干内置参数。例如，上面的示例使用 ``Namespace`` 根据项目和文件位置获取命名空间。可用参数如下：
- `ProjectName` - 当前项目名称
- `ProjectDir` - 当前项目路径
- `FolderDir` - 文件生成到的目录路径
- `Namespace` - 文件命名空间（包含文件相对路径作为子命名空间）
- `DefaultNamespace` - 当前项目的默认命名空间
- `WorkspaceName` - 当前工作区名称
- `WorkspaceDir` - 当前工作区路径
- `DateTime` - 生成时的当前时间
- `IsRegenerating` - 指示生成器是否用于重新生成文件

## 重新生成

生成器可在 ``Generate`` 调用期间，通过在 ``generateFlags`` 上设置 ``.AllowRegenerate`` 来允许文件重新生成。当生成完整可用的源代码时，更新生成器后即可轻松重新生成之前的文件。注意：重新生成会从头重写文件内容，所有手动改动都会被丢弃。

标记为可重新生成的生成器，会将配置保存在重新生成文件顶部的注释中。该注释的存在是文件可被识别为可重新生成的必要条件。注释内容可编辑，例如添加或修改新的生成器输入。``GetString(...)`` 可用于判断是否提供了某个输入（旧文件并不了解新增的生成器参数），从而在缺失时使用默认值。``mParams[...]`` 是字典，传入未知键时会直接报错。

可右键文件并选择 “Regenrate” 选项进行重新生成。
