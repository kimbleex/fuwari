---
title: Python手写肘部法则确定Kmeans聚类算法中的K值
category: Programming
tags: [Python, 机器学习, 算法]
abbrlink: K-Means-Elbow
published: 2021-10-15 00:00:00
uppublishedd: 2021-10-15 00:00:00
ai:
  - 本文介绍了Python手写肘部法则确定Kmeans算法中的K值的方法。
---

上次手写了`Kmeans`算法，但是关于分类数目`K`的取值，只是固定了两类，这次就手写一下`Kmeans`算法中`K`值的确定方法——肘部法则。

介绍一个词`WCSS`: `WCSS`是衡量聚类结果紧密程度的指标，表示每个样本点与其所属簇中心的距离平方和，简言之，就是样本类里面每个点到中心点的距离的平方，然后全部相加。

## 肘部法的详细步骤

- 确定K值范围
选择一个合理的`K`值范围，一般从1开始，逐步增加，直到达到一个预设的最大`K`值(例如，`K=10`)。

- 计算不同`K`值下的`WCSS`
对于每一个K值，执行以下步骤：

- 运行`K`均值算法：将数据集划分为K个簇。
计算`WCSS`：计算簇内误差平方和，即所有样本点到其所属簇中心的距离平方和。

- 绘制肘部图
在图中，横轴表示簇的数量`K`，纵轴表示对应的`WCSS`值。绘制`K`值与`WCSS`的关系曲线。

- 识别“肘部”位置
观察曲线中`WCSS`下降速度显著减缓的位置，即曲线出现“肘部”的点。该`K`值被认为是最佳的聚类数。

## Python代码实现

沿用之前手写的`Kmeans`算法，这次我们只需要在`Kmeans`算法的基础上，增加计算`WCSS`和绘图流程即可即可。并且增加了一些数据。

```python
import random
# 一些数据点
data = [[1, 2], [1, 4], [1, 0], [4, 2], [4, 4], [4, 0], [1,6], [5,6], [9,9], [2,7], [3,3], [6,4], [2,5], [3,5], [4,5],[5,5]]
# K的取值范围
K = range(1, 10)
WCSS = [] # 存放各个K对应的WCSS
# 遍历K值
for i in K:
    index_list = []
    for j in range(i):
        while True: # 随机选择中心点，但不能重复
            index_ = random.randint(0, len(data)-1)
            if index_ in index_list:
                continue
            else:
                index_list.append(index_)
                break
    centerPoint = [data[x] for x in index_list] # 确定各个k值的中心点
    print("center: ", centerPoint)
    # 确定每个点在K值下的分类
    cate_data = {}
    for z in range(len(data)):
        distance = []
        for y in centerPoint:
            distance.append(((y[0]-data[z][0])**2 + (y[1]-data[z][1])**2) ** 0.5)
        cate_data[f"{data[z]}"] = distance.index(min(distance))     
    print(cate_data)
    # 计算每个K值对应的WCSS
    wcss_list = []
    for p in range(i):
        points = [eval(key) for key in cate_data.keys() if cate_data[key] == p]
        wcss_list.append(sum([((points[n][0]-centerPoint[p][0])**2 + (points[n][1]-centerPoint[p][1])**2) ** 0.5 for n in range(len(points))]))
    WCSS.append(sum(wcss_list))
print(WCSS)
```

## 运行结果

```Terminal
center:  [[6, 4]]
{'[1, 2]': 0, '[1, 4]': 0, '[1, 0]': 0, '[4, 2]': 0, '[4, 4]': 0, '[4, 0]': 0, '[1, 6]': 0, '[5, 6]': 0, '[9, 9]': 0, '[2, 7]': 0, '[3, 3]': 0, '[6, 4]': 0, '[2, 5]': 0, '[3, 5]': 0, '[4, 5]': 0, '[5, 5]': 0}

center:  [[2, 7], [4, 5]]
{'[1, 2]': 1, '[1, 4]': 0, '[1, 0]': 1, '[4, 2]': 1, '[4, 4]': 1, '[4, 0]': 1, '[1, 6]': 0, '[5, 6]': 1, '[9, 9]': 1, '[2, 7]': 0, '[3, 3]': 1, '[6, 4]': 1, '[2, 5]': 0, '[3, 5]': 1, '[4, 5]': 1, '[5, 5]': 1}

center:  [[3, 5], [5, 5], [3, 3]]
{'[1, 2]': 2, '[1, 4]': 0, '[1, 0]': 2, '[4, 2]': 2, '[4, 4]': 0, '[4, 0]': 2, '[1, 6]': 0, '[5, 6]': 1, '[9, 9]': 1, '[2, 7]': 0, '[3, 3]': 2, '[6, 4]': 1, '[2, 5]': 0, '[3, 5]': 0, '[4, 5]': 0, '[5, 5]': 1}

center:  [[4, 2], [1, 0], [1, 4], [3, 5]]
{'[1, 2]': 1, '[1, 4]': 2, '[1, 0]': 1, '[4, 2]': 0, '[4, 4]': 3, '[4, 0]': 0, '[1, 6]': 2, '[5, 6]': 3, '[9, 9]': 3, '[2, 7]': 3, '[3, 3]': 0, '[6, 4]': 0, '[2, 5]': 3, '[3, 5]': 3, '[4, 5]': 3, '[5, 5]': 3}

center:  [[5, 6], [4, 0], [1, 0], [3, 3], [6, 4]]
{'[1, 2]': 2, '[1, 4]': 3, '[1, 0]': 2, '[4, 2]': 3, '[4, 4]': 3, '[4, 0]': 1, '[1, 6]': 3, '[5, 6]': 0, '[9, 9]': 0, '[2, 7]': 0, '[3, 3]': 3, '[6, 4]': 4, '[2, 5]': 3, '[3, 5]': 3, '[4, 5]': 0, '[5, 5]': 0}

center:  [[4, 5], [5, 6], [5, 5], [6, 4], [2, 5], [1, 0]]
{'[1, 2]': 5, '[1, 4]': 4, '[1, 0]': 5, '[4, 2]': 3, '[4, 4]': 0, '[4, 0]': 5, '[1, 6]': 4, '[5, 6]': 1, '[9, 9]': 1, '[2, 7]': 4, '[3, 3]': 0, '[6, 4]': 3, '[2, 5]': 4, '[3, 5]': 0, '[4, 5]': 0, '[5, 5]': 2}

center:  [[1, 4], [3, 3], [4, 4], [6, 4], [2, 7], [1, 2], [3, 5]]
{'[1, 2]': 5, '[1, 4]': 0, '[1, 0]': 5, '[4, 2]': 1, '[4, 4]': 2, '[4, 0]': 1, '[1, 6]': 4, '[5, 6]': 2, '[9, 9]': 3, '[2, 7]': 4, '[3, 3]': 1, '[6, 4]': 3, '[2, 5]': 6, '[3, 5]': 6, '[4, 5]': 2, '[5, 5]': 2}

center:  [[4, 0], [4, 5], [6, 4], [4, 4], [9, 9], [3, 3], [1, 4], [1, 2]]
{'[1, 2]': 7, '[1, 4]': 6, '[1, 0]': 7, '[4, 2]': 5, '[4, 4]': 3, '[4, 0]': 0, '[1, 6]': 6, '[5, 6]': 1, '[9, 9]': 4, '[2, 7]': 1, '[3, 3]': 5, '[6, 4]': 2, '[2, 5]': 6, '[3, 5]': 1, '[4, 5]': 1, '[5, 5]': 1}

center:  [[3, 3], [9, 9], [5, 5], [3, 5], [1, 4], [4, 5], [4, 0], [1, 0], [6, 4]]
{'[1, 2]': 4, '[1, 4]': 4, '[1, 0]': 7, '[4, 2]': 0, '[4, 4]': 5, '[4, 0]': 6, '[1, 6]': 4, '[5, 6]': 2, '[9, 9]': 1, '[2, 7]': 3, '[3, 3]': 0, '[6, 4]': 8, '[2, 5]': 3, '[3, 5]': 3, '[4, 5]': 5, '[5, 5]': 2}

[58.638979289620025, 39.93955755931159, 28.611595782243192, 27.34009275541994, 25.482605577751237, 21.89292222699217, 19.471938219632754, 13.071067811865476, 10.650281539872886]
```

可以看到每个数据点都有了自己的分类，并且最后的`WCSS`也有对应所有`K`的值。

下面我们可以通过绘图来观察`WCSS`的变化情况，从而确定最佳的`K`值。

```python
import matplotlib.pyplot as plt


plt.figure(figsize=(8, 5))
plt.plot(K, WCSS, 'bo-', markersize=8)
plt.xlabel('K')
plt.ylabel('WCSS')
plt.title('Elbow Method => K')
plt.xticks(K)
plt.grid(True)
plt.show()
```

运行上述代码，可以得到如下的图像:
![Kmeans肘部法则确定K值](https://images.kimbleex.top/BlogIMG/Kmeans-Elbow/WCSS.avif)

可以看出`K`在4或者5这个点附近有一个明显的拐点，因此我们可以认为最佳的`K`值为4或者5。

## 完整代码

```python
import random
import matplotlib.pyplot as plt

data = [
    [1, 2], [1, 4], [1, 0], [4, 2], [4, 4], [4, 0], [1,6], [5,6], [9,9], [2,7], [3,3], [6,4], [2,5], [3,5], [4,5],[5,5]
]

K = range(1, 10)
WCSS = []

for i in K:
    index_list = []
    for j in range(i):
        while True:
            index_ = random.randint(0, len(data)-1)
            if index_ in index_list:
                continue
            else:
                index_list.append(index_)
                break
    centerPoint = [data[x] for x in index_list]
    print("center: ", centerPoint)

    cate_data = {}
    for z in range(len(data)):
        distance = []
        for y in centerPoint:
            distance.append(((y[0]-data[z][0])**2 + (y[1]-data[z][1])**2) ** 0.5)
        cate_data[f"{data[z]}"] = distance.index(min(distance))     
    print(cate_data)
    
    wcss_list = []
    for p in range(i):
        points = [eval(key) for key in cate_data.keys() if cate_data[key] == p]
        wcss_list.append(sum([((points[n][0]-centerPoint[p][0])**2 + (points[n][1]-centerPoint[p][1])**2) ** 0.5 for n in range(len(points))]))
    
    WCSS.append(sum(wcss_list))
    # WCSS.append()
    
print(WCSS)

plt.figure(figsize=(8, 5))
plt.plot(K, WCSS, 'bo-', markersize=8)
plt.xlabel('K')
plt.ylabel('WCSS')
plt.title('Elbow Method => K')
plt.xticks(K)
plt.grid(True)
plt.show()
```
