+++
# Hero widget.
widget = "hero"  # See https://sourcethemes.com/academic/docs/page-builder/
headless = true  # This file represents a page section.
active = true  # Activate this widget? true/false
weight = 10  # Order that this section will appear.

#title = "Beef"

# Hero image (optional). Enter filename of an image in the `static/img/` folder.
hero_media = "Beef384.png"

[design.background]
  # Apply a background color, gradient, or image.
  #   Uncomment (by removing `#`) an option to apply it.
  #   Choose a light or dark text color by setting `text_color_light`.
  #   Any HTML color name or Hex value is valid.

  # Background color.
  # color = "navy"

  # Background gradient.
  gradient_start = "#4bb4e3"
  gradient_end = "#2b94c3"

  # Background image.
  # image = ""  # Name of image in `static/img/`.
  # image_darken = 0.6  # Darken the image? Range 0-1 where 0 is transparent and 1 is opaque.

  # Text color (true=light or false=dark).
  text_color_light = true

# Call to action links (optional).
#   Display link(s) by specifying a URL and label below. Icon is optional for `[cta]`.
#   Remove a link/note by deleting a cta/note block.
[cta]
  url = "setup/BeefSetup_0_43_5.exe"
  label = "Beef（Windows）"
  icon_pack = "fas"
  icon = "download"

[cta_alt]
  url = "/docs"
  label = "查看文档"

# Note. An optional note to show underneath the links.
[cta_note]
  label = '<a href="docs/getting-start/building/">从源码构建</a><br><a href="https://github.com/beefytech/Beef_website/tree/master/Samples/SpaceGame">查看示例源码</a><br><a href="spacegame">体验 Space Game Wasm 示例</a>'
+++

Beef 是一门高性能、多范式的开源编程语言，专注于提升开发者生产力。

### 一级平台（IDE + 二进制）
Windows 64 位和 32 位

### 二级平台（从源码构建）
Linux、macOS、Wasm

### 三级平台（实验性）
Android、Nintendo Switch、PS5、Xbox Series X
