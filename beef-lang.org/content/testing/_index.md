+++
title = "测试"
description = ""
weight = 90
+++

每个工作区默认都包含一个 “Test” 配置，可以通过向 BeefBuild 传入 “-test” 或在 IDE 的 “Test” 菜单中执行。测试通过在任意静态方法上添加 `[Test]` 特性来定义。

```C#
[Test]
public static void TestAPI()
{
	Test.Assert(API_Init());
	Test.Assert(API_GetVersion() == 12);
}
```

在某些情况下，测试应当失败。可使用 `[Test]` 特性进行标记。

```C#
[Test(ShouldFail=true)]
public static void TestBoundsCheck()
{
	let array = scope int8[5];

	/* The following will fail a Runtime.Assert in the array indexer, thus failing the test */
	let num = array[5];
}
```

在工作区属性中，确保 “Projects” 中需要运行测试的项目都指定了 “Test” 配置。默认情况下，工作区中列出的第一个项目会被用于测试。

Beef 测试适用于 Jenkins 等 CI 系统：BeefBuild 会输出包含耗时统计的测试文本，并通过标准返回码报告成功或失败。
