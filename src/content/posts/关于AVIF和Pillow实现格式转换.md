---
title: 关于新图片格式AVIF和Pillow格式转换方法
category: Programming
tags: [图像处理, Python, 前端]
abbrlink: AVIF-Pillow
published: 2024-11-13 00:00:00
uppublishedd: 2024-11-13 00:00:00
ai: 
  - 这篇文章介绍了AVIF图像格式的特点和优势，包括高效压缩、HDR支持和透明度等。它与JPEG、PNG、WebP和HEIC相比，提供了更小的文件大小和优质的图像质量。文章还展示了如何使用Python的Pillow库进行AVIF格式的转换，并说明了AVIF在主流浏览器和操作系统中的兼容性。
---

`AVIF (AV1 Image File Format)`是一种现代图像格式, 基于`AV1`视频编码技术。它由`Alliance for Open Media`开发, 旨在提供高效的图像压缩, 同时保持优良的图像质量。

## 背景介绍

`AVIF`源于`AV1`视频编码格式, 这是一种开源、免版税的视频编码格式, 旨在替代`VP9`和`H.264`。`AVIF`利用`AV1`的压缩技术来处理静态图像和图像序列, 提供高效的图像存储和传输。

## AVIF格式的特点和优势

- 高压缩效率: `AVIF`具有出色的压缩效率, 能够在较小的文件大小下提供优质的图像质量。这使得它在带宽和存储有限的情况下非常有用。
- 支持`HDR`: `AVIF`支持高动态范围`HDR`图像, 能够呈现更丰富的色彩和对比度。
- 色彩深度: 支持8位、10位和12位色深, 能够更好地处理复杂的色彩信息。
- 透明度: 支持透明度通道, 使其适用于需要透明背景的图像。
- 动画支持: `AVIF`支持动画, 与`GIF`和`APNG`相比, 提供了更高效的动画压缩。

## 与其他格式的比较

- `JPEG`: `AVIF`在相同图像质量下通常比`JPEG`文件小得多, 并且支持更高的色彩深度和透明度。
- `PNG`: 虽然`PNG`支持无损压缩和透明度, 但`AVIF`在有损压缩情况下提供了更小的文件大小。
- `WebP`: `AVIF`通常比`WebP`提供更高的压缩效率, 尤其是在处理高分辨率图像时。
- `HEIC`: `AVIF`与`HEIC`相似, 但`AVIF`是免版税的, 而`HEIC`可能涉及专利许可问题。

## 兼容性（截至2024年）

:::tip[浏览器支持]

- `Google Chrome`: 支持。
- `Mozilla Firefox`: 支持。
- `Microsoft Edge`: 支持。
- `Safari`: 从`Safari 16`开始支持, 但支持可能不如其他浏览器全面。

:::
:::note[操作系统支持]

- `Windows`: `Windows 10`及以上已通过系统更新支持。
- `macOS`: 从`macOS Ventura`开始支持。
- `Android`: 较新的`Android`版本支持。
- `iOS`: 从`iOS 16`开始支持。

:::

总结
`AVIF`作为一种现代图像格式, 提供了卓越的压缩效率和图像质量, 逐渐被主流浏览器和操作系统采纳。随着技术的进步和普及, `AVIF`有望成为广泛使用的图像格式之一。

## Pillow库实现AVIF格式转换

`Pillow`是一个强大的Python图像处理库, 支持多种图像格式, 包括`AVIF`。通过`Pillow`, 可以轻松实现`AVIF`与其他图像格式的相互转换。

### 安装Pillow库

在开始之前, 需要确保已安装`Pillow`库以及`pillow-avif-plugin`库。可以使用以下命令安装:

```bash
pip install Pillow pillow-avif-plugin
```

### AVIF格式转换为其他格式

以下是一个示例, 将`AVIF`格式转换为其他格式:

```python
import os
from PIL import Image    # Pillow                    9.0.0
import pillow_avif       # pillow-avif-plugin        1.2.2
#以上只是其中一个可用版本，并非必须

AVIFfilename = 'test.avif'
AVIFimg = Image.open(AVIFfilename)
# 将AVIF格式转换为JPEG格式，当然其他格式都可以
AVIFimg.save(AVIFfilename.replace("avif",'jpg'),'JPEG')
```

当然, 也可以将其他格式转为`AVIF`格式:

```python
import os
from PIL import Image    # Pillow                    9.0.0
import pillow_avif       # pillow-avif-plugin        1.2.2
#以上只是其中一个可用版本，并非必须

img_path_ = "./test.png"
img = Image.open(path_)
img.save(path_.replace("jpg",'avif'),'AVIF')
```

转换完成后，我们可以在根目录看到转换完成后的文件，文件名后缀为`.avif`。
![图片转换完成](https://images.kimbleex.top/BlogIMG/AVIF_Pillow/converted.avif)

我们可以在文件管理器中比较转换前后的图片大小。
![图片大小比较](https://images.kimbleex.top/BlogIMG/AVIF_Pillow/size_compare.avif)
可以看到，转换后的图片大小明显小于转换前的图片大小。由此可见，`AVIF`格式具有更高的压缩效率。

最后让我们从视觉上感受两张图片的差距。
![图片对比](https://images.kimbleex.top/BlogIMG/AVIF_Pillow/vis_compare.avif)
可以看到，转换前后的图片几乎没有什么差别，说明`AVIF`格式在保持图像质量的同时，也具有很高的压缩效率。
