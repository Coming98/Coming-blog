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

# Tips

`var.item()` - 如果张量 var 中只有一个元素，使用该方法取出该元素

# 属性

`tensor.size` - shapes

# tensor的拼接

tensor的拼接操作经常用到、特地将其整理如下

* cat - dim=all
* stack - dim=new_dim
* dstack - dim=2
* hstack - dim=1
* vstack- dim=0

## cat

**功能**：将给定的 tensors 在给定的维度上拼接

**注意**：除了拼接的维度其余维度大小应一致

**参数**：

- tensors - 多个tensor组成的元组（序列）

- dim - 想要拼接的维度



**Test**

~~~python
x = torch.randn(2, 3, 3)
y = torch.cat((x, x, x), dim=0)
y_1 = torch.cat((x, x, x), dim=1)
y_2 = torch.cat((x, x, x), dim=2)
~~~

Output

~~~python
x.size() = torch.Size([2, 3, 3])
y.size() = torch.Size([6, 3, 3])
y_1.size() = torch.Size([2, 9, 3])
y_2.size() = torch.Size([2, 3, 9])
~~~

---

## stack

**功能**：将给定的 tensors 在新的维度进行拼接

**注意**：每一个 tensor 的大小都应一致

**参数**：tensors

* dim - 所要插入的维度，最小为 0 ，最大为输入数据的维度值



**Test**

~~~python
a = torch.zeros(3, 4)
b = torch.ones(3, 4)
c = torch.stack((a, b))
d = torch.stack((a, b), dim=0)
e = torch.stack((a, b), dim=1)
f = torch.stack((a, b), dim=2)
~~~

Output

~~~python
c.size() = torch.Size([2, 3, 4])
d.size() = torch.Size([2, 3, 4])
e.size() = torch.Size([3, 2, 4])
f.size() = torch.Size([3, 4, 2])
~~~

---

## dstack

**功能**：将给定的 tensors 沿着深度（depth）方向 (dim=2) 叠加

注意：其余维度大小一致

**参数**：tensors



**Test 1**

~~~python
a = torch.randn(2, 3, 4, 2)
b = torch.randn(2, 3, 4, 2)
c = torch.dstack((a, b))
~~~

Output 1

~~~python
c.size() = torch.Size([2, 3, 8, 2])
~~~

**Test 2 如果输入数据小于三维呢？**

~~~python
a = torch.randn(2)
b = torch.randn(2)
c = torch.dstack((a, b))
~~~

Output 2 会扩增维度

~~~python
c.size() = torch.Size([1, 2, 2])
~~~

---

## hstack

**功能**：将给定的 tensors 沿着水平（horizontal）方向 (dim=1) 叠加

**注意**：其余维度大小一致

**参数**：tensors



**Test**

~~~python
a = torch.randn(2, 3, 4)
b = torch.randn(2, 4, 4)
c = torch.hstack((a, b))
d = torch.cat((a, b), dim=1)
~~~

Output

~~~python
c.size() = torch.Size([2, 7, 4])
d.size() = torch.Size([2, 7, 4])
~~~

---

## vstack

**功能**：将给定的 tensors 沿着竖直（vertical）方向 (dim=0) 叠加

**注意**：其余维度大小一致

**参数**：tensors

**Test**

~~~python
a = torch.randn(2, 3, 4)
b = torch.randn(3, 3, 4)
c = torch.vstack((a, b))
d = torch.cat((a, b), dim=0)
~~~

Output

~~~python
c.size() = torch.Size([5, 3, 4])
d.size() = torch.Size([5, 3, 4])
~~~

# tensor的切分

数据的预处理以及数据集的构建会经常使用到tensor的切分操作，现整理如下：

* chunk
* split
* numpy中的切片(slice)操作
* unbind

---

## chunk

**功能**：输入数据与想要切分的块数 chunks ，将数据尽可能 (如果数据个数与块数能整除的话) 平均的切分为 chunks 块

**注意**：没有进行数据的拷贝

**参数**

* input - tensor
* chunks - 切分的块数
* dim - int - 切分的维度

**Test**

```Python
x = torch.randn(33, 16)
y = torch.chunk(x, 4, dim=0)
y_1 = torch.chunk(x, 2, dim=1)
```

Output

~~~python
# 尽可能平均的切分为了四块
y.size() = (torch.Size([9, 16]),
			torch.Size([9, 16]),
			torch.Size([9, 16]),
			torch.Size([6, 16]))
y_1.size() = (torch.Size([33, 8]),
			  torch.Size([33, 8]))
~~~

---

## split

**功能**：

* 可以输入整型数据 size 表示将数据尽可能 按照大小 size 进行分割，自行计算分割的块数
* 可以输入序列数据 list 将数据分割为 len(list) 块，每块的大小对应于list中的值

**注意**：没有对输入数据进行拷贝

**参数**：

* input - tensor
* split_size_or_sections - int or (ints) - 详见功能介绍
* dim - int - 切分的维度

**Test 1 输入整型数据**

```python
a = torch.arange(10).view(2, 5)
# 在维度 1 上进行切分，使得每一块大小尽可能为 2
b = torch.split(a, 2, dim=1)
```

Output 1

```python
a = tensor([[0, 1, 2, 3, 4],
            [5, 6, 7, 8, 9]])
b = (tensor([[0, 1],
            [5, 6]]), 
     tensor([[2, 3],
            [7, 8]]), 
     tensor([[4],
             [9]])) # 最后剩余 - 无法按照size大小进行分配
```

**Test 2 输入为整型序列**

```python
a = torch.arange(10).view(2, 5)
# 在维度 0 上进行切分，目标切分为 2 块，第一块维度大小为 1，第二块维度大小为 1
b = torch.split(a, [1, 1], dim=0)
```

Output

```pyhon
a = tensor([[0, 1, 2, 3, 4],
            [5, 6, 7, 8, 9]])
b = (tensor([[0, 1, 2, 3, 4]]), 
     tensor([[5, 6, 7, 8, 9]]))
```

---

## slice

**功能**：torch支持numpy中对数据的切片操作

**Test**

~~~Python
x = torch.randn(3, 2)
~~~

Output

~~~python
x = tensor([[-0.2893, -1.1715],
            [-0.2477, -2.7052],
            [-0.2827, -0.2669]])
x[:,1] = tensor([-1.1715, -2.7052, -0.2669])
x[0,:] = tensor([-0.2893, -1.1715])
x[1:,-1]tensor([-2.7052, -0.2669])
~~~

---

## unbind

**功能**：删除tensor的一个维度，返回各个子块组成的 tuple

**参数**：

* input - tensor
* dim - int - 想要删除的维度

**Test**

~~~python
a = torch.randint(-5, 5, (2, 3, 4))
b = torch.unbind(a, 1)
~~~

Output

~~~Python
# a
tensor([[[ 2, -5, -5,  4],
         [ 4, -4,  4, -5],
         [-4, -4,  1, -4]],

        [[ 1, -2,  2, -4],
         [-2, -3, -2,  1],
         [ 1, -2,  2, -2]]])
# b
tensor([[ 2, -5, -5,  4],
        [ 1, -2,  2, -4]])
tensor([[ 4, -4,  4, -5],
        [-2, -3, -2,  1]])
tensor([[-4, -4,  1, -4],
        [ 1, -2,  2, -2]])
~~~

---

# tensor的索引

根据索引获取数据中特定位置的值

* gather
* index_select
* masked_select
* narrow
* nonzero
* where
* take

---

## gather

**功能**：沿指定的维度收集值

**注意**：除了指定的维度、其余维度都遍历一遍

**参数**：

* input - tensor
* dim - int - 收集的维度
* index - LongTensor - 要收集的元素的索引，维度数和 input 一致
* sparse_grad - bool - 如果 input 为 sparse tensor 那么设定为 true

$$
out[i][j][k] = input[\ index[i][j][k]\ ][j][k]\ with\ dim=0 \\
out[i][j][k] = input[i][\ index[i][j][k]\ ][k]\ with\ dim=1 \\
out[i][j][k] = input[i][j][\ index[i][j][k]\ ]\ with\ dim=2 \\
$$

**Test 1 ** 

~~~python
a = torch.arange(6).view(2, 3)
b = torch.gather(a, 1, torch.tensor([
    [0, 1, 2],
    [1, 2 ,0],
])) # 确定维度后线性索引即可
~~~

Output

~~~python
a = tensor([[0, 1, 2],
            [3, 4, 5]])
b = tensor([[0, 1, 2],
            [4, 5, 3]])
~~~

## index_select

**功能**：在指定的维度上根据索引取值

**注意**：进行了拷贝

**参数**：

* input - tensor
* dim - int
* index - LongTensor - 一维 tensor 指定索引

**Test**

~~~python
a = torch.arange(6).view(2, 3)
b = torch.index_select(a, 1, torch.tensor([0, 2]))
~~~

Output

~~~python
a = tensor([[0, 1, 2],
            [3, 4, 5]])
b = tensor([[0, 2],
            [3, 5]])
~~~

## masked_select

**功能**：根据 booltensor 进行索引，将索引到的值以一维 tensor 的形式返回

**注意**：boolTensor 支持广播

**参数**：

* input - tensor
* mask - boolTensor

**Test 1**

~~~python
a = torch.arange(12).view(3, 4)
# ge - great_equal
mask = a.ge(6) 
b = torch.masked_select(a, mask)
~~~

Output

~~~python
a = tensor([[False, False, False, False],
            [False, False,  True,  True],
            [ True,  True,  True,  True]])
tensor([ 6,  7,  8,  9, 10, 11])
~~~

**Test 2 广播测试**

~~~python
a = torch.arange(12).view(3, 4)
b = torch.masked_select(a, torch.tensor([[True], [False], [False]])) # 可以广播
~~~

Output 2

~~~Python
a = tensor([[ 0,  1,  2,  3],
            [ 4,  5,  6,  7],
            [ 8,  9, 10, 11]])
b = tensor([0, 1, 2, 3])
~~~

## narrow

**功能**：类似于 numpy中的切片操作，针对某一维度进行索引，（连续）取值

**参数**：

* input - tensor

* dim - int - 所要操作的维度
* start - int - 开始的索引
* length - 保留长度

**简单理解：**

~~~python
torch.narrow(input, dim=1, start, length) = input[:,start:start+length,:,:...]
~~~

**Test**

~~~python
x = torch.arange(12).view(2, 2, 3)
y = torch.narrow(x, 2, 1, 2)
z = x[:,:,1:3]
~~~

Output

~~~python
x = tensor([[[ 0,  1,  2],
             [ 3,  4,  5]],
 
            [[ 6,  7,  8],
             [ 9, 10, 11]]])
y = tensor([[[ 1,  2],
             [ 4,  5]],

            [[ 7,  8],
             [10, 11]]])
z = tensor([[[ 1,  2],
             [ 4,  5]],

            [[ 7,  8],
             [10, 11]]])
~~~

## nonzero

**功能**：找出给定 tensor 中非零元素的索引

**注意**：as_tuple 参数与输出样式有关

* True - 返回一个二维 tensor，每一行表示一个非零值的索引
* False - 返回 n 个大小为 z 的一维 tensor - n 表示输入数据的维度 - z表示非零元素的个数

**参数**

* input - tensor
* as_tuple - default (False)

**Test 1**

~~~python
a = torch.randint(0, 2, (3, 3))
i_2D = torch.nonzero(a)
i_1D = torch.nonzero(a, as_tuple=True)
~~~

Output 1

~~~python
a = tensor([[0, 0, 0],
            [0, 0, 1],
            [0, 0, 1]])
i_2D = tensor([[1, 2],
               [2, 2]])
i_1D = (tensor([1, 2]), tensor([2, 2]))
~~~

**扩展**：

结合 torch 中类似 numpy 的 bool索引，我们可以构造条件索引的形式

* 通过条件语句生成 bool tensor
* True为1，False为0，即nonzero会找到所有Ture值的索引（条件筛选出来的元素）

**Test 2 利用条件语句实现逆 nonzero**

~~~Python
a = torch.randint(0, 2, (3, 3))
bool_tensor = (a == 0)
index_nonzero = torch.nonzero(bool_type)
~~~

Output 2

~~~python
# a
tensor([[1, 0, 1],
        [1, 1, 1],
        [1, 0, 1]])
# bool_tensor
tensor([[False,  True, False],
        [False, False, False],
        [False,  True, False]])
# index_nonzero
tensor([[0, 1],
        [2, 1]])
~~~

**Test 3 如何合并多个条件呢**

利用集合操作即可
* 根据原子条件分别生成多个 booltensor

* 根据需求采用交或并等的方式合并多个 booltensor

~~~Python
a = torch.randint(0, 8, (2, 5)) # 1 < a < 4: a = 2 or 3
bool_tensor_1 = a > 1
bool_tensor_2 = a < 4
bool_tensor = bool_tensor_1 & bool_tensor_2 # 交集
index = torch.nonzero(bool_tensor)
~~~

Output 3

~~~Python
# a
tensor([[1, 4, 2, 6, 0],
        [5, 3, 1, 2, 6]])
# bool tensor 1
tensor([[False,  True,  True,  True, False],
        [ True,  True, False,  True,  True]])
# bool tensor 2
tensor([[ True, False,  True, False,  True],
        [False,  True,  True,  True, False]])
# bool tensor
tensor([[False, False,  True, False, False],
        [False,  True, False,  True, False]])
# index
tensor([[0, 2],
        [1, 1],
        [1, 3]])
~~~

## where

**功能 1**：根据 booltensor 返回目标值的索引

**简单理解**

~~~python
torch.where(condition) == torch.nonzero(condition, as_tuple=True)
~~~

**参数**：condition - booltensor

**Test 1**

~~~python
x = torch.randn(3, 2)
y = torch.where(x > 0)
~~~

Output

~~~python
x = tensor([[ 1.2484, -0.4649],
            [ 1.7428, -0.3610],
            [ 0.3865, -1.1223]])
y = (tensor([0, 1, 2]), tensor([0, 0, 0]))
~~~



**功能 2**：根据 booltensor 定位到目标值，选择性进行 值替换

**注意**：支持广播

**简单理解**
$$
out_i = 
\begin{cases}
x_i \ &if\ condition_i \\
y_i \ &otherwise
\end{cases}
$$
**参数**：

* condition - booltensor
* x - tensor
* y - tensor

**Test 2 大于0不变 小于等于0改变**

~~~python
x = torch.randn(3, 2)
y = torch.ones(3, 2)
z = torch.where(x > 0, x, y)
z_1 = torch.where(x > 0, x, torch.tensor([[1.], 
                                          [2.], 
                                          [3.]])) # 支持广播
~~~

Output

~~~python
x = tensor([[-1.9040, -0.3891],
            [ 1.0645, -1.0039],
            [-0.1270, -0.2566]])
z = tensor([[1.0000, 1.0000],
            [1.0645, 1.0000],
            [1.0000, 1.0000]])
# z_1
tensor([[1.0000, 1.0000],
        [1.0645, 2.0000],
        [3.0000, 3.0000]])
~~~

## take

**功能**：根据一维索引进行取值，返回一维tensor （内部将输入的tensor铺平后进行索引）

**功能**：根据一维索引进行取值，返回一维tensor （内部将输入的tensor铺平后进行索引）

**注意**：进行了数据拷贝

**参数**

* input
* indices - 一维索引

**Test**

~~~python
x = torch.arange(6).view(2, 3)
indices = torch.randint(6, (3, ))
y = torch.take(x, indices)
~~~

Output

~~~python
x = tensor([[0, 1, 2],
            [3, 4, 5]])
indices = tensor([0, 5, 2])
y = tensor([0, 5, 2])
~~~

# tensor 的填充

ref：https://pytorch.org/docs/stable/tensors.html#torch.Tensor.scatter_

## scatter

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

![image-20211028195119040](https://gitee.com/Butterflier/pictures/raw/master/image-20211028195119040.png)

从 index 出发，因为 dim = 1，所以我们能够得知目标填充索引为 (0, 3)

> 可以认为先从 (i = 0, j=0) 找到了 index 和 value 的值（3, 9），又因为 dim = 1，因此 index 的值将作用于列索引所以得到最终索引为 (i=0, j=3)

其实再次看这句话

> 我们对 index 进行遍历时，dim 影响的那一个维度会由 index 内部的值决定，其余维度索引保持一致

那我们看 value 中的 5，它的行索引是 1，因此它必然撒到 dest 中的第一行（dim=1），但是撒在第几列将由 index 决定，index 说撒在第 4 列，done！

![image-20211028195554262](https://gitee.com/Butterflier/pictures/raw/master/image-20211028195554262.png)

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

..

# tensor的变换

* movedim
* reshape
* view
* squeeze
* unsqueeze
* t
* transpose

## movedim

**功能**：进行（多个）维度的移动

**注意**：其余维度保持原有不变，并按照原顺序依次落位

**参数**

* input
* source - int or tuple - 维度移动前的索引
* destination - int or tuple - 维度移动后的索引

**Test 1 其余维度保持原有不变，并按照原顺序依次落位**

~~~Python
a = torch.randn(2, 3, 4, 5)
b = torch.movedim(a, 0, 3)
c = torch.movedim(a, 0, 2)
~~~

Output

~~~python
a = torch.Size([2, 3, 4, 5])
b = torch.Size([3, 4, 5, 2])
c = torch.Size([3, 4, 2, 5])
~~~

**Test 2 输入为tuple**

~~~python
a = torch.randn(2, 3, 4, 5)
b = torch.movedim(a, (1, 2), (1, 0))
~~~

Output

~~~python
a = torch.Size([2, 3, 4, 5])
b = torch.Size([4, 3, 2, 5])
~~~

## reshape

**功能**：改变输入 tensor 的维度分配

**注意**：应默认该函数不会对源数据进行拷贝

**参数**

* input
* shape - int of tuple - 其余维度确定的情况下，可以使用 -1 用于推算某一维度的大小

**Test**

~~~python
a = torch.randn(32, 16)
b = torch.reshape(a, (4, 2, 2, -1 ,2))
~~~

Output

~~~python
a = torch.Size([32, 16])
b = torch.Size([4, 2, 2, 16, 2])
~~~

## view



## squeeze

**功能**：默认移除 tensor 中维度大小为 1 的维度 - 给定 dim 后只考虑给定的 dim 是否移除

**注意**：输入 dim 必须为整型，不能为整型序列

**参数**：

* input - tensor
* dim - default(None) - int

**Test**

~~~python
a = torch.randn(2, 1, 2, 1, 1, 3)
b = torch.squeeze(a)
c = torch.squeeze(a, dim=1)
d = torch.squeeze(a, dim=2)
~~~

Output

~~~python
torch.Size([2, 1, 2, 1, 1, 3])
torch.Size([2, 2, 3])
torch.Size([2, 2, 1, 1, 3])
torch.Size([2, 1, 2, 1, 1, 3])
~~~

## unsqueeze

**功能**：与 squeeze 方法想反，增加一个大小为1的维度

**参数**

* input - tensor
* dim - int

**Test**

~~~python
x = torch.tensor([1, 2, 3, 4])
y = torch.unsqueeze(x, dim=0)
z = torch.unsqueeze(x, dim=1)
~~~

Output

~~~python
x = torch.Size([4])
y = torch.Size([1, 4])
z = torch.Size([4, 1])
~~~

## t

**功能**：对二维张量进行转置处理

**注意**：对于一维 tensor 返回的是它本身

**参数**：input - tensro

**Test**

~~~python
a = torch.randn(2)
a_t = torch.t(a)
b = torch.randn(2, 3)
b_t = b.t()
~~~

Output

~~~python
  a = torch.Size([2])
a_t = torch.Size([2])
  b = torch.Size([2, 3])
b_t = torch.Size([3, 2])
~~~

## transpose

**功能**：将给定的两个维度进行交换

**参数**：

* input - tensor
* dim0 - int
* dim1 - int

**Test**

~~~Python
a = torch.randn(2, 3, 4, 5)
b = torch.transpose(a, 1, 2)
~~~

Output

~~~python
a = torch.Size([2, 3, 4, 5])
b = torch.Size([2, 5, 4, 3])
~~~

# 与Numpy转换

Tips：Torch Tensor 与 Numpy array 共享底层的内存空间

Tips：所有在 CPU 上的 tensor （except CharTensor）都可与 Numpy array 互转换

`to numpy`：`tensor.numpy()` 

`to tensor` ：`torch.from_numpy(array)`

# 与设备交互

~~~python
if torch.cuda.is_available():
    device = torch.device("cuda")

gpu_tensor = torch.ones_like(x, device=device)
gpu_tensor = cpu_tensor.to(device)
cpu_tensor = gpu_tensor.to("cpu", torch.double)
~~~

# 自动求导

定义变量时设定：`torch.tensor([1, 2, 3], requires_grad=True)`

后续更改设定：`tensor.requires_grad_(True)`

终止某Tensor在计算图中的追踪回溯：`tensor.detach()`

终止整个流程的反向传播：`with torch.no_grad():`

## 相关属性

`.grad_fn` - 代表引用了那个 Function 创建了该 Tensor，自定义则值为 None

`.grad` - 在该 Tensor 上的所有梯度将被累加进该属性



# END