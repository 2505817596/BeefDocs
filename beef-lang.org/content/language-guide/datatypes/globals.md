+++
title = "全局"
+++

### 全局
	
全局变量可视为匿名类型的静态字段。也可以定义全局方法与 mixin。

```C#
static
{
	public static int gGlobalVal = 0;
}
```

通过 `using static` 可以在文件级别实现类似全局变量的简洁性，使得可以在当前类型外直接使用静态字段。

```C#
class Image
{
	public static int sImageCount;
}

using static Image;

class Program
{
	public void Use()
	{
		// 这种静态使用通常需要完全限定名 "Image.sImageCount";
		int imgCount = sImageCount;
	}
}

```
