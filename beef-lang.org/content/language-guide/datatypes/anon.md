+++
title = "匿名类型"
weight=30
+++

## 匿名类型声明

匿名类型声明与普通类型声明类似，但它们没有名称，并放在通常写类型引用的位置。

```C#

[Union]
struct Vector3
{
	public float[3] vals;

    /* 带匿名字段的匿名结构体声明。
    下面等价于：
    public using struct
	{
		public float mX;
		public float mY;
		public float mZ;
	} _UNUSED_NAME_;
     */
	public struct
	{
		public float x;
		public float y;
		public float z;
	};

    /* 匿名枚举声明 */
    public enum { Left, Center, Right } GetXDirection() => (mX < 0) ? .Left : (mX > 0) ? .Right : .Center;
}

Vector3 vec = .();
vec.mX = 1;
vec.mY = 2;
vec.mZ = 3;

/* 类型可以匿名扩展与重写

*/
var buttonWidget = new ButtonWidget("OK")
{
    bool wasClicked;

    public override void OnClick()
    {
        base.OnClick();
        wasClicked = true;
        CloseDialog();
    }
};

```
