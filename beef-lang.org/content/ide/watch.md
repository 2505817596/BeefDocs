+++
title = "调试器表达式"
+++

## 调试器监视表达式

调试器监视表达式求值器可评估大多数常规 Beef 表达式，包括访问属性与调用方法。
除这些表达式外，还支持一些特殊监视表达式。

|表达式|说明|
|----|------|
|{&lt;number>} &lt;expr> or {^&lt;number>} &lt;expr>|在调用栈向上第 &lt;number> 层处求值表达式|
|{MethodName} &lt;expr>|在调用栈向上第一个名为 MethodName 的方法中求值表达式|
|{MethodName^&lt;number>} &lt;expr>|在调用栈向上第 &lt;number> 个名为 MethodName 的方法中求值表达式|
|{*} &lt;expr>|在调用栈中第一个表达式有效的作用域里求值|

## 格式标志

以下标志可加在监视表达式之后，用逗号分隔。

|标志|说明|
|----|------|
|this=&lt;expr>|为表达式显式设置 'this' 值|
|arraysize=&lt;number>|将表达式显示为包含 &lt;number> 个元素的数组|
|&lt;number>|将表达式显示为包含 &lt;number> 个元素的数组|
|d|十进制|
|s|ASCII|
|s8|UTF8|
|s16|UTF16|
|s32|UTF32|
|na|隐藏指针|
|nv|不使用可视化器|
|x|十六进制（小写）|
|X|十六进制（大写）|
