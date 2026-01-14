+++
title = "错误处理"
weight=75
+++

## 错误处理

一些语言如 C# 使用异常处理，但 Beef 不支持异常。按照约定，错误处理使用 System.Result<T> 和 System.Result<T, TError> 枚举类型，它们包含 .Ok(T) 与 .Err(TError) 两个值。带 TError 参数的版本支持返回显式错误值，否则错误类型未指定。泛型可用 "void" 作为参数，因此 Result<void> 用于可能返回错误但没有返回值的方法。

若未处理返回的错误，将导致运行时致命错误。
```C#
static Result<uint> GetMinusOne(uint i)
{
	if (i == 0)
		return .Err;
	return .Ok(i - 1);
}

void Use()
{
	/* Handle result via a switch */
	switch (GetMinusOne(i))
	{
		case .Ok(let newVal): Console.WriteLine("Val: {}", newVal);
		case .Err: Console.WriteLine("Failed");
	}

	/* This invokes an implicit conversion operator, which will be fatal at runtime if an error is returned */
	let newVal = GetMinusOne(i);

	/* Result<T> contains a special "ReturnValueDiscarded" method which is invoked to facilitate failing fatally on ignored returned errors here */
	GetMinusOne(i);

	/* "ReturnValueDiscarded" will not be called */
	GetMinusOne(i).IgnoreError();
}
```

Result<T> 也可通过 if 语句处理。使用 [case]({{< ref "pattern.md#enum" >}}) 匹配 .Err 或 .Ok 枚举值。

```C#
void Use()
{
	/* Handle result via an if. Note that Result<T> returns are matched with 'case', not compared with '==' */
	if (GetMinusOne(i) case .Ok(let newVal))
		Console.WriteLine("Val: {}", newVal);
}
```

## 断言

断言通过 `Debug.Assert()` 与 `Runtime.Assert()` 实现。`Debug.Assert(cond)` 在调试模式下若 `cond` 为 `false` 会触发致命错误，但在发布模式下不会生成任何指令（即使 `cond` 含有方法调用或复杂表达式）。断言通常用于“快速失败”以确保程序状态合法，而不用于处理可合法发生的错误（例如用户输入错误、超时等）。

## 致命错误

当检测到不可恢复的错误时，可使用 `Runtime.FatalError` 手动“崩溃”程序。

## 崩溃处理

默认情况下，GUI 程序会显示包含回溯的崩溃对话框，控制台程序则输出崩溃报告到控制台。可通过 `Runtime.SetCrashReportKind` 更改崩溃处理方式。
