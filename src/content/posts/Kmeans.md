---
title: Python手写Kmeans聚类算法
category: Programming
tags: [Python, 机器学习, 算法]
abbrlink: K-Means
published: 2021-10-11 00:00:00
uppublishedd: 2021-10-14 00:00:00
ai:
  - 本文介绍了Kmeans算法，展示了纯Python手写Kmeans算法的代码。
---

`Kmeans`算法是选取数据的中心点，将数据按照距离中心点的距离进行分类，从而将数据分成K类。是一个聚类算法。

下面展示脱离机器学习库，Python手写`Kmeans`算法的代码。

## `Kmeans`

```python
import random
# 假设有一些数据
data = [[1, 2], [1, 4], [1, 0], [4, 2], [4, 4], [4, 0]]
# 设置分类数量
K = 2
# 随机选取中心点
x, y  = random.randint(0, len(data)-1), random.randint(0, len(data)-1)
centerPoint = [data[x], data[y]]
print("center: ", centerPoint)
# 结果保存为字典格式
cate_data = {}
# 计算欧氏距离并将所有的点分类
for i in range(len(data)):
    distance1 = ((data[i][0] - centerPoint[0][0])**2 + (data[i][1] - centerPoint[0][1])**2)**0.5
    distance2 = ((data[i][0] - centerPoint[1][0])**2 + (data[i][1] - centerPoint[1][1])**2)**0.5
    cate_data[f"{data[i]}"] = {"label" : 0 if distance1 <= distance2 else 1}
print("cate_data:", cate_data)
```

## 运行结果

```Terminal
center:  [[1, 0], [4, 2]]
cate_data:{
            '[1, 2]': {'label': 0}, 
            '[1, 4]': {'label': 1}, 
            '[1, 0]': {'label': 0}, 
            '[4, 2]': {'label': 1}, 
            '[4, 4]': {'label': 1}, 
            '[4, 0]': {'label': 1},
        }
```

可以看到所有的点都根据距离中心点的距离进行了分类。
