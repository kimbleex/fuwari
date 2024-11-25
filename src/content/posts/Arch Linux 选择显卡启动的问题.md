---
title: Arch Linux 选择显卡启动的问题
category: School
tags: [Arch Linux]
abbrlink: Arch-GPU
published: 2023-07-03 00:00:00
uppublishedd: 2023-07-03 00:00:00
ai: 
  - 本文反映了一个Arch Linux的关于显卡驱动的问题。
---

**本机配置：AMD核显 + NVIDIA独显**
成功安装了optimus-manager管理显卡驱动，它可以很好接触显卡驱动的切换.于是昨晚因为屏幕刷新率问题尝试切换startup mode.

于是尝试切换启动项显卡驱动模式：

- 尝试nvidia独显直连启动：成功启动.使用感受上感觉到性能下降，图形动画效果变差.
- 尝试nvidia+amd混合启动：成功启动.使用感受极佳，推荐.
- 尝试amd核显启动：启动失败.无法进入kde.黑屏(显卡未通电的那种黑屏).可以进入tty.

![黑屏](https://images.kimbleex.top/BlogIMG/Arch_GPU/black.avif)  

于是尝试在tty中解决：

- 尝试卸载了optimus管理器，reboot，无效.
- 尝试optimus-manager - -switch nvidia，报错，无有效解决方法.
- 尝试进入bios，禁用核显启动，无法进入tty，风扇狂转，黑屏.

<iframe width="100%" height="468" src="https://images.kimbleex.top/BlogIMG/Arch_GPU/gameover.mp4" title="YouTube video player" frameborder="0" allowfullscreen></iframe>

**至此，Arch宣告滚挂…….GG！**

os驱动问题由于机器各有所不同很难解决，考虑到短学期需要使用电脑，所以打算重装.这次会尝试在grub里安装iso，这样就无需插u盘抢救系统.并且尝试使用btrfs文件系统代替传统的ext4文件系统.
