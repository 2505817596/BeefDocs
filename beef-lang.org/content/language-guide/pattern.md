+++
title = "模式匹配"
weight=75
+++

### 模式匹配概览

模式匹配既可用于简单的相等性检查，也可在 switch 与 case 表达式中对元组和枚举值组合使用成员相等检查、成员捕获与成员“忽略”。

### 元组模式匹配

```C#

void TupleCaseExpr()
{
	let tup = (1.2, "Abc");

	// This performs one equality check and one member capture
	if (tup case (1.2, let str))
	{
		UseStr(str);
	}
}

void TupleSwitch()
{
	let tup = (x: 10, y: 20, str:"Abc");

	switch (tup)
	{
	case (0, 0, let drawStr):
		DrawAtOrigin(drawStr);
	case (var drawX, var drawY, let drawStr):
		/* Since drawX and drawY are 'var' the are mutable, whereas drawStr is immutable */
		drawX += 10;
		drawY += 20;
		Draw(x, y, drawStr);
	}
}
```

### 枚举模式匹配 {#enum}

```C#
enum Shape
{
	case Rectangle(int x, int y, int width, int height);
	case Circle(int x, int y, int radius);
}

void EnumCase()
{
	Shape shape = .Circle(10, 20, 30);

	/* One enum discriminator check, two member equality checks, and one member captures */
	if (shape case .Circle(0, 0, let radius))
	{
		DrawCircleAtOrigin(radius);
	}

	/* One enum discriminator check, and three member captures */
	else if (shape case .Circle(let x, let y, let radius))
	{
		DrawCircle(x, y, radius);
	}

	/* One enum discriminator check, and two member captures, one member ignore */
	else if (shape case .Circle(let x, let y, ?))
	{
		DrawPoint(x, y);
	}

	/* Enum discriminator check, no member checks or captures*/
	if (shape case .Rectangle)
	{
		IgnoreRectangle();
	}
}

/* This is equivalent to the EnumCase method */
void EnumSwitch()
{
	Shape shape = .Circle(10, 20, 30);
	switch (shape)
	{
	case .Circle(0, 0, let radius):
		DrawCircleAtOrigin(radius);
	case .Circle(let x, let y, let radius):
		DrawCircle(x, y, radius);
	case .Rectangle:
		IgnoreRectangle();
	}
}

```

枚举也可以匹配到 `ref`，从而修改其内部值。

``` C#

void Enlarge(ref Shape shape)
{
	switch (shape)
	{
	case .Circle(?, ?, var ref radius):
		radius++;
	case .Rectangle(?, ?, var ref width, var ref height):
		width++;
		height++;
	}
}

```
