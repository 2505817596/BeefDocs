+++
title = "命名空间"
weight = 30
+++

## 命名空间概览
命名空间用于在 Beef 中组织类型并防止命名冲突。注意，命名空间中的元素始终是 public。

```C#
namespace Gfx
{
	// Defines "Gfx.Window" class
	class Window
	{

	}

	namespace Resources
	{
		// Defines "Gfx.Resources.Image" class
		class Image
		{

		}
	}
}

namespace Gfx.Resources
{
	// Defines "Gfx.Resources.Shader" class
	class Shader
	{

	}
}

```

### 使用命名空间

虽然类型可以使用完全限定名来引用，但如果在该文件中通过 `using` 指令列出了其所属命名空间，就可使用更短的未限定名。

```C#
using Gfx.Resources;

class Program
{
	void Use()
	{
		// Shader refers to Gfx.Resources.Shader;
		let s = new Shader();
	}
}
```
