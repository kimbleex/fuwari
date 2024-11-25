---
title: Corn调度任务表达式写法笔记
category: Programming
tags: [Linux, 定时调度]
abbrlink: Corn
published: 2024-11-20 00:00:00
uppublishedd: 2024-11-20 00:00:00
ai: 
  - 本文介绍了Corn调度任务表达式写法，包括秒、分、时、日、月、周、年等字段的含义和用法，以及一些示例和注意事项。
---

起因是写`Github`主页的时候，突发奇想打算设置`Action`来定时推送仓库，于是就涉及到了Corn调度任务表达式写法，记录一下。

## `Corn`规范

Corn表达式由6个字段组成，每个字段之间用空格分隔，格式如下：

```bash
*  *  *  *  *  *
|  |  |  |  |  |___________ 星期， 数值为0-6，周日为0，也可以使用星期缩写，比如mon等
|  |  |  |  |______________ 月份， 数值为1-12，也可以使用月份缩写，比如jan等
|  |  |  |_________________ 日期， 数值为1-31
|  |  |____________________ 小时， 数值为0-23
|  |_______________________ 分钟， 数值为0-59
|__________________________ 秒钟， 数值为0-59，可选字段
```

## 其他符号支持

除了上面的基础字段写法，`Corn`还支持一些符号

- 星号(\*) 表示匹配任意值，即全部值 。例如，* 在分钟字段中表示每分钟都执行。
- 逗号(,) 用于分隔多个值。例如，1,3,5 在小时字段中表示 1 点、3 点和 5 点执行。
- 斜线(/) 用于指定间隔值。例如，*/5 在分钟字段中表示每 5 分钟执行一次。
- 连字符(-) 用于指定范围。例如，10-20 在日期字段中表示从 10 号到 20 号。
- 问号(?) 仅用于日期和星期几字段，表示不指定具体值。通常用于避免冲突。

## 一些例子

:::note[每分钟执行一次]
0 \* \* \* \* \*
:::

:::tip[每天上午9点执行一次]
0 0 9 \* \* \*
:::

:::important[每周一上午10点执行一次]
0 0 10 \* \* mon
:::

:::warning[每个月10-20号的每天8点执行一次]
0 0 8 10-20 \* \*
:::

:::caution[每两个小时执行一次]
0 0 \*/2 \* \* \*
:::

## 一些注意

上面说到秒钟单位是可以省略的，所以在写表达式时，可以直接省略秒的那一位。
