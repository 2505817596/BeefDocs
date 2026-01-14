+++
title = "主题"
+++

## 主题

IDE 支持主题文件，可用于自定义外观。主题文件应放置在 `bin/themes` 目录下的子目录中。用户可在 `File\Preferences\Settings` 的 `UI\Theme` 中设置主题。该值可以是相对于 `themes` 目录的目录名，或相对于 `theme` 目录的主题 `.toml` 文件。

替换图像需要提供包含图像分片的 PSD 文件，以覆盖标准的 `bin/images/DarkUI.png`、`bin/images/DarkUI_2.png` 和 `bin/images/DarkUI_4.png`。主题文件应分别命名为 `UI.psd`、`UI_2.psd` 和 `UI_4.psd`，它们对应不同的缩放等级。

并非所有图像文件都必须存在，也不必在每个文件中填充所有图像分片。缺失的部分会用其他主题图像文件的缩放分片或默认分片补齐。例如，仅提供一个 `UI_4.psd` 文件并只替换复选框分片也是可以的。但对于需要清晰边缘的分片，一般建议提供 1x 缩放的 `UI.png` 图像，因为缩放可能导致边缘对不齐或模糊。

当用户在 IDE 中选择你的主题时，会执行 `bin/images/ImgCreate.exe`，通过缩放并合并 PSD 生成 `UI.png`、`UI_2.png` 和 `UI_4.png`。这些文件应视为缓存文件，当源文件变化时会自动重建，从而既能更新主题，也能兼容未来 IDE 版本新增的图像分片。

主题还支持通过 `theme.toml` 覆盖颜色设置。

```
[Colors]
Text = 0xFFFFFFFF
Window = 0xFF595962
Background = 0xFF26262A
SelectedOutline = 0xFFCFAE11
MenuFocused = 0xFFE5A910
MenuSelected = 0xFFCB9B80
Code = 0xFFFFFFFF
Keyword = 0xFFE1AE9A
Literal = 0xFFC8A0FF
Identifier = 0xFFFFFFFF
Comment = 0xFF75715E
Method = 0xFFA6E22A
Type = 0xFF66D9EF
RefType = 0xFF66D9EF
Interface = 0xFF66D9EF
Namespace = 0xFF7BEEB7
DisassemblyText = 0xFFB0B0B0
DisassemblyFileName = 0XFFFF0000
Error = 0xFFFF0000
BuildError = 0xFFFF8080
BuildWarning = 0xFFFFFF80
VisibleWhiteSpace = 0xFF9090C0
```

用户可在主题目录中提供 `user.toml` 文件，该文件会在 `theme.toml` 之后处理，用于覆盖一个或多个主题设置。
