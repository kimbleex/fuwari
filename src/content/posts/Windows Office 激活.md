---
title: Windows Office 无需任何成本激活方法
category: Tools
tags: [工具]
abbrlink: Windows-Office-Activate
published: 2024-11-14 00:00:00
uppublishedd: 2024-11-14 00:00:00
sticky: 1
ai:
    - 本文介绍了一种无需下载任何工具即可激活Windwos和Office的方法。
---

官方开源仓库: [Microsoft-Activation-Scripts](https://github.com/massgravel/Microsoft-Activation-Scripts)

本文受启发于: [Windows11专业版激活](https://yangqiuyi.com/blog/windows/windows11%E4%B8%93%E4%B8%9A%E7%89%88%E6%BF%80%E6%B4%BB/)

这是一个利用硬件ID激活的办法, 激活后一劳永逸, 即使重装系统也不需要再激活, 但是需要保持硬件不变。

## 使用方法

打开`Windwos PowerShell`, 输入以下命令(**注意不是`CMD`**):

```powershell
irm https://get.activated.win | iex
```

输入完成后回车，会弹出一个新界面，在新界面中选择你需要激活的内容即可。

- 选项: `[1] HWID`用于激活`Windows`
- 选项: `[2] Ohook`用于激活`Office`
