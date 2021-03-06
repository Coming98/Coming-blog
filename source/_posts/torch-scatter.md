---
title: torch.scatter
mathjax: false
date: 2021-10-28 20:12:19
summary: 对 torch.scatter 函数的理解
categories: Pytorch
tags:
  - DL
  - pytorch
---

# scatter

scatter 是撒播的意思，希望能先入为主的理解一下这个函数的功能，就是往目标矩阵撒一些数值进去，往哪撒（dim？index？）与撒什么值（src）将由我们输入的参数决定

官方一些的说明：将 src 中数据根据 index 中的索引按照 dim 的方向填进 input 中

```python
tensor.scatter(dim, index, src)
```

从数学公式上来看执行流程：

```python
self[index[i][j][k]][j][k] = src[i][j][k]  # if dim == 0
self[i][index[i][j][k]][k] = src[i][j][k]  # if dim == 1
self[i][j][index[i][j][k]] = src[i][j][k]  # if dim == 2
```

1、原始的对应填充方式 `self[i][j][k] = src[i][j][k]` 将被 `index` 索引张量影响，因此才能体现出撒（scatter）的感觉

2、dim 决定影响的维度，其余维度不变

3、我们对 index 进行遍历时，dim 影响的那一个维度会由 index 内部的值决定，其余维度索引保持一致

希望以上三点能加深对下面例子的理解

> 为了保证索引对应 index 索引张量的维度应与 src 张量的维度一致，但是 shape 可以不一致

让我们以一个二维的情形为例：

![image-20211028195119040](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211028195119040.png)

从 index 出发，因为 dim = 1，所以我们能够得知目标填充索引为 (0, 3)

> 可以认为先从 (i = 0, j=0) 找到了 index 和 value 的值（3, 9），又因为 dim = 1，因此 index 的值将作用于列索引所以得到最终索引为 (i=0, j=3)

其实再次看这句话

> 我们对 index 进行遍历时，dim 影响的那一个维度会由 index 内部的值决定，其余维度索引保持一致

那我们看 value 中的 5，它的行索引是 1，因此它必然撒到 dest 中的第一行（dim=1），但是撒在第几列将由 index 决定，index 说撒在第 4 列，done！

![image-20211028195554262](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211028195554262.png)

# Example

**Example： Lable 转 one-hot**

Label 通常是一维的 `shape=(N,)`，而模型的 output 通常是 softmax 之后的二维结果 
`shape=(N, dim)`，因此将 label 变为 二维的 one-hot 形式非常常见

首先指定 dim = 1，是因为我们填充值要修改的维度索引是列向维度，行索引值将与 Label 的索引值对应，即通过控制列向维度保证哪个地方值是 1

```python
# ref: https://www.cnblogs.com/dogecheng/p/11938009.html
class_num = 10
batch_size = 4
label = torch.LongTensor(batch_size, 1).random_() % class_num
#tensor([[6],
#        [0],
#        [3],
#        [2]])
torch.zeros(batch_size, class_num).scatter_(1, label, 1)
#tensor([[0., 0., 0., 0., 0., 0., 1., 0., 0., 0.],
#        [1., 0., 0., 0., 0., 0., 0., 0., 0., 0.],
#        [0., 0., 0., 1., 0., 0., 0., 0., 0., 0.],
#        [0., 0., 1., 0., 0., 0., 0., 0., 0., 0.]])
```

label 即是我们要索引的列，我们需要完成对 label 的遍历

1、第一个值为 `6` 告知第一行第 `6` 列

1、第二个值为 `0` 告知第二行第 `0` 列

….
