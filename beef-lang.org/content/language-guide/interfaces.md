+++
title = "接口"
weight = 41
+++

## 接口概览

接口定义了类或结构体需要实现的属性与方法要求。接口可用于动态派发，也可作为泛型约束使用（参见“泛型”部分）。

```C#
interface IDrawable
{
	void Draw();
}

struct Circle : IDrawable
{
	float x;
	float y;
	float radius;

	/* Implements IDrawable.Draw 
	If we wanted to keep the Draw method as a private implementation, we could have declared it as 
	"private void IDrawable.Draw()" */
	public void Draw()
	{
		DrawCircle(x, y, radius);
	}
}

/* Calling the following method with an instance of Circle will first cause boxing to occur at the
 callsite, then Draw will be called via dynamic dispatch (method table) */
public static void DrawDynamic(IDrawable val)
{
	val.Draw();
}

/* Calling the following method with an instance of Circle will create a specialized instance of 
the DrawGeneric method which accepts a Circle argument and statically calls the Circle.Draw
 method, which is faster */
public static void DrawGeneric<T>(T val) where T : IDrawable
{
	val.Draw();
}
```

### 默认方法实现
接口可通过提供方法体来定义默认实现。这对于向既有接口添加方法而不破坏已有使用者非常重要。

### 具体返回类型
接口方法可声明为返回某个接口的具体实现类型。由于未定义具体返回类型，此类方法无法用于动态派发。这有助于返回值类型实例，并避免动态方法派发——如果实现方法返回接口实例引用则必须动态派发。

```C#
concrete interface IEnumerable<T>
{
    concrete IEnumerator<T> GetEnumerator();
}
```
