---
title: Python使用Schedule模块实现定时任务
category: Programming
tags: [Python, 定时调度]
published: 2023-12-05 00:00:00
uppublishedd: 2023-12-05 00:00:00
abbrlink: Python-Schedule
ai:
    - 本文介绍了Python的Schedule模块，它是一个用于在Python中实现简单任务调度的库。Schedule模块使用一种类似自然语言的方式来安排任务，可以每分钟、每小时、每天、每周等时间间隔执行任务。下面是关于Schedule模块的介绍和代码示例。
---

有时候写爬虫的时候会想要每隔固定的时间执行一次，所以就想到了设计一个定时任务，于是找到了Python的Schedule模块，记录一下关于Schedule模块的一些知识点。

`schedule` 模块是一个用于在 Python 中实现简单任务调度的库。它允许你以简洁的语法安排任务。

## 安装

首先，安装 `schedule` 模块：

```bash
pip install schedule
```

## 基本用法及案例

### 一个整体的示例

```python
import schedule
import time

def job():
    print("I'm working...")

# 每三秒钟执行一次上面的任务job
schedule.every(3).seconds.do(job)

while True:
    schedule.run_pending()
    time.sleep(1)
```

其中`schedule.run_pending()` 来运行所有准备好的任务。通常将其放在一个循环中，以持续检查和执行任务。

### 其他的一些定时任务写法

`schedule` 模块使用一种类似自然语言的方式来安排任务。以下是一些常用的方法:

- 每隔 3 sec/min/hour/day/week执行一次

```python
schedule.every(3).seconds.do(job)
schedule.every(3).minutes.do(job)
schedule.every(3).hours.do(job)
schedule.every(3).days.do(job)
schedule.every(3).weeks.do(job)
```

- 每分钟的第23秒执行一次任务

```python
schedule.every().minute.at(":23").do(job)
```

- 每小时的第40分钟执行一次任务

```python
schedule.every().hour.at(":40").do(job)
```

- 每隔5个小时，在第五个小时的20分钟30秒执行一次任务

```python
# 如果 02:00 执行, 那么将会在 06:20:30 执行
schedule.every(5).hours.at("20:30").do(job)
```

- 在每天的特定时间执行任务`HH:MM:SS`

```python
schedule.every().day.at("10:30:42").do(job)
```

### 还可以使用装饰器来修饰

使用`@repect`装饰器装饰任务函数后，也可以做到同样的效果

```python
from schedule import every, repeat, run_pending
import time

@repeat(every(10).minutes)
def job():
    print("I am a scheduled job")

while True:
    run_pending()
    time.sleep(1)
```

### 带有参数的任务

如果任务中需要传入参数，可以在`do()`中传入参数。

```python
import schedule

def greet(name):
    print('Hello', name)
schedule.every(2).seconds.do(greet, name='Alice')
schedule.every(4).seconds.do(greet, name='Bob')
```

装饰器版本传参代码:

```python
from schedule import every, repeat

@repeat(every().second, "World")
@repeat(every().day, "Mars")
def hello(planet):
    print("Hello", planet)
```

### 标记任务并按标签获取

使用`tag()`可以给任务上标签。
使用`get_jobs()`方法获取任务所有任务，如果传入标签名则按标签获取任务。

```python
import schedule

def job1():
    print("Job 1")

def job2():
    print("Job 2")

# 上标签,可以无限标签
schedule.every().day.do(job1).tag("daily", "haha")
schedule.every().hour.do(job2).tag("hourly")

# 获取所有带有 "daily" 标签的任务
daily_jobs = schedule.get_jobs("daily")
```

### 取消任务

使用`cancel()`方法可以取消任务。
使用`clear()`方法清除所有任务，如果传入标签名即可按标签取消

```python
import schedule

def job1():
    print("I'm working...")

def job2():
    print("I'm working too...")

job1 = schedule.every(3).seconds.do(job1).tag("ok")
job2 = schedule.every(5).seconds.do(job2).tag("no")

# 取消任务
schedule.cancel_job1(job) # 取消job1
schedule.clear("ok") # 按标签取消
schedule.clear() # 清除所有任务
```

### 任务终止判定

使用`until()`来设置任务的终止时间。

```python
# 2030年1月1日18:33后不再执行
schedule.every(1).hours.until("2030-01-01 18:33").do(job)
```

## 注意事项

`Schedule`模块的定时任务并不是真正的定时任务，而是通过循环来模拟定时任务。因此，如果任务执行时间过长，可能会导致任务堆积，影响后续任务的执行。因此，在编写任务时，需要注意任务的执行时间，避免任务堆积。

简单来说，就是`Schedule`模块不计算任务的具体执行时长，如果一个任务需要6min执行完成，但是你的调度方法时每5min执行一次，那么程序会产生等待，直到任务执行完成，才会继续执行下一个任务。
