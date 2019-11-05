---
title: matplotlib 漏斗图
date: 2019-11-05 09:27:09
tags:
- matplotlib
- 数据分析
- plotly
---
漏斗（funnel）图通常用于显示流程数据。漏斗图用梯形面积表示某个环节业务量与上一个环节之间的差异。漏斗图从上到下，有逻辑上的顺序关系，表现了随着业务流程的推进业务目标完成的情况。例如，它用于观察每个阶段销售过程中的收入或损失，并显示逐渐减少的值。每个阶段都以占所有值的百分比表示。

<!-- more -->

# matplitlib 画漏斗图
- 预备知识：barh, TextArea, AnnotationBbox


## barh

matplotlib.pyplot.barh(y, width, height=0.8, left=None, *, align='center', **kwargs)

Make a horizontal bar plot.

 - y: 标量或类数组：条形的y坐标
 - width: 标量，条的宽度
 - height: 标量，条的高度
 - left: 条形图左侧的x坐标
 - align: {'center', 'edge'} 
 - tick_label : 字符串或类似数据，条的刻度标签


## TextAres
matplotlib.offsetbox.TextArea(s, textprops=None, multilinebaseline=None, minimumdescent=True)

TextArea包含单个文本实例。文本以基线+左对齐方式放置在（0,0）处。TextArea实例的宽度和高度是其子文本的宽度和高度。

- s : 字符串，显示的文本
- textprops: 字典，显示的文本的格式
- multilinebaseline: bool, 如果为True，则对多行文本的基线进行调整，使其与单线文本对齐（近似）。
- minimumdescent: bool, 如果为如果为True，则该框的最小降幅为“p”。

## AnnotationBbox
matplotlib.offsetbox.AnnotationBbox(offsetbox, xy, xybox=None, xycoords='data', boxcoords=None, frameon=True, pad=0.4, annotation_clip=None, box_alignment=(0.5, 0.5), bboxprops=None, arrowprops=None, fontsize=None, **kwargs)

类似类的注释，但是使用的是offsetbox 而不是文本

- offsetbox : offsetbox 实例
- xycoords：与注释相同，但可以是两个元组
- boxcoords：类似于作为注释的textcoords，但可以是 解释为x和y坐标的两个字符串的元组。

- box_alignment：两个浮点数组成的元组，用于offsetbox偏移框的水平对齐w.r.t.框坐标。左下角为（0.0），右上角为（1.1）。

## 完整代码
```python
# @Time    : 2019/11/4 20:09
# @Author  : dli
# @Software: PyCharm
import numpy as np

import matplotlib.pyplot as plt
from matplotlib.offsetbox import (TextArea, AnnotationBbox)


plt.rcParams['font.sans-serif'] = ['Arial Unicode MS']

width = 0.5
x1 = np.array([1000, 500, 300, 200, 150])
val_max = np.max(x1)
# noinspection PyTypeChecker
x2 = np.array((x1.max() - x1) // 2)  # 占位
x3 = []
for i, j in zip(x1, x2):
    x3.append(i + j)
x3 = np.array(x3)

val_cnt = len(x1)
y = np.arange(val_cnt, 0, -1)
print(x1, x2, x3, y)

labels = ['浏览商品', '放入购物车', '生成订单', '支付订单', '完成交易']

# figure
fig = plt.figure(figsize=(8, 5))
ax = fig.add_subplot(111)

# plot
ax.barh(y, x3, width, tick_label=labels, color=["r", "b", "g", "deeppink", "lime"], alpha=0.85)
ax.barh(y, x2, width, color='w', alpha=1)  # 辅助图

# setting
transform = []
for i in range(0, len(x1)):
    if i < len(x1) - 1:
        transform.append('%.2f%%' % ((x1[i + 1] / x1[i]) * 100))

print(transform)

for i in range(1, val_cnt):
    offsetbox = TextArea(transform[i - 1], minimumdescent=False)
    ab = AnnotationBbox(offsetbox, (val_max / 2, i),
                        xybox=(0, 30),
                        boxcoords="offset points",
                        arrowprops=dict(arrowstyle="->"))
    ax.add_artist(ab)

# for i in range(1, val_cnt):
#     ax.text(val_max / 2 - 2, (y[i - 1] + y[i]) / 2, transform[i - 1])

plt.xlim(xmax=val_max)
plt.show()

```

## 结果显示

![](/images/funnel.png)

# [plotly 漏斗图](https://plot.ly/python/funnel-charts/)

```python
from plotly import graph_objects as go

fig = go.Figure(go.Funnel(
    y = ["Website visit", "Downloads", "Potential customers", "Requested price", "Finalized"],
    x = [39, 27.4, 20.6, 11, 2],
    textposition = "inside",
    textinfo = "value+percent initial",
    opacity = 0.65, marker = {"color": ["deepskyblue", "lightsalmon", "tan", "teal", "silver"],
    "line": {"width": [4, 2, 2, 3, 1, 1], "color": ["wheat", "wheat", "blue", "wheat", "wheat"]}},
    connector = {"line": {"color": "royalblue", "dash": "dot", "width": 3}})
    )

fig.show()
```

参数解释：

- `textpositon`: ( "inside" | "outside" | "auto" | "none" )中的一个。默认值为none
- `textinfo`: 确定图形上显示的文本信息，Any combination of "label", "text",
 "percent initial", "percent previous", "percent total", "value" joined with a "+" OR "none".
- `opacity` : 不透明性
- `marker` ： 字典