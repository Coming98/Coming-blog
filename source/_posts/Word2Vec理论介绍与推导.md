---
title: Word2Vec理论介绍与推导
mathjax: true
date: 2021-07-12 22:24:58
summary: 对优秀论文解读，向大佬致敬！
categories: NLP
tags: 
  - DL
  - python
  - Article
---

本文主要根据论文《word2vec Parameter Learning Explained》进行介绍，部分内容添加了自己的理解与思考，还望大家批评指正。

# 概念

## 反向传播基础

**Back Propagation Basics**

### 单一神经元

![image-20210711175018396](https://gitee.com/Butterflier/pictures/raw/master/image-20210711175018396.png)

如图为人工神经元（articial neuron unit） $\{x_1,...,x_k\}$ 为输入向量，$\{w_1,...,w_k\}$ 为对应的权重，$f$ 为激活（非线性）函数，$y$ 是输出结果

1. 输入向量与权重向量进行内积操作，其结果我们赋值给$u$：

$$
\sum_{i=1}^{K}x_iw_i = \mathbf{w}^T\mathbf{x}:= u
$$

2. $u$ 输入到神经元中，进行激活：

$$
y = f(u)
$$

3. 求导更新权重，其中 $t$ 为真实标签，$y$ 表示预测的标签值，$y$ 对 $\mathbf{w}$ 求偏导为 $\mathbf{x}$

$$
\mathbf{w}^{new} = \mathbf{w}^{old}-\eta \cdot (y-t) \cdot \mathbf{x}
$$

这里我们没有考虑到激活函数和损失函数，只简单的让误分类样本作反向传播。如果激活函数连续可导或有具体的损失函数，还是需要考虑进来的。

例如我们引入激活函数 Sigmoid 激活函数：$\sigma (u) = \dfrac{1}{1+e^{-u}}$ ，其函数图像为：

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210711181646466.png" alt="image-20210711181646466" style="zoom:50%;" />

单从函数图像可以看出：其处处连续可导，值域 $y \in [0,1]$ 

在引入简单的平方差损失函数：$E = \frac{1}{2}(t-y)^2$

> 为什么有个 $\frac{1}{2}$ 呢？是为了后续反向传播求导方便而添加的，对结果显然没有较大影响

3. 求导更新权重，其中 $t$ 为真实标签，$y$ 表示预测的标签值，$y$ 对 $\mathbf{w}$ 求偏导为 $\mathbf{x}$

$$
\dfrac{\partial{E}}{\partial{w_i}} = \dfrac{\partial{E}}{\partial{y}} \cdot \dfrac{\partial{y}}{\partial{u}} \cdot \dfrac{\partial{u}}{\partial{w_i}} = (y-t) \cdot y(1-y) \cdot x_i
$$

> $$
> \dfrac{\partial{y}}{\partial{u}}= [(1+e^{-u})^{-1}]' = -(1+e^{-u})^{-2} \times (-e^{-u}) \\= \dfrac{1}{1+e^{-u}} \times \dfrac{(1 + e^{-u}) - 1}{1+e^{-u}} = y \cdot (1-y)
> $$

也就得到了权重的更新策略：
$$
\mathbf{w}^{new} = \mathbf{w}^{old} - \eta \cdot (y-t) \cdot y(1-y) \cdot \mathbf{x}
$$

### 多层神经网络

单一神经元下的反向传播求导大致明白啦，那接下来推广到多神经元多层网络的情况：

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210711190544339.png" alt="image-20210711190544339" style="zoom: 80%;" />

我们的输入为 ${x_k} = \{x_1,\dots,x_K\}$ ，隐藏层神经元为：${h_i} = \{h_1,\dots,h_N\}$，输出层为 ${y_j}=\{y_1,\dots y_M\}$

> 需要注意的是，现在的操作都还没有矩阵化，一次仅处理一个输入，得到一个 $M$ 维的输出，以此完成一次 $M$ 分类任务

与单一神经元类似

1. 对输入层进行加权求和并激活得到隐藏层的结果 $h_i$

$$
h_i = \sigma(u_i) = \sigma(\sum_{k=1}^{K} w_{ki}x_k) = \sigma(\mathbf{W}\mathbf{x})
$$

2. 再次前向传播，得到输出层的结果 $y_j$

$$
y_j = \sigma(u_j^{'}) = \sigma\left(\sum_{i=1}^{N} w_{ij}h_i\right) = \sigma(\mathbf{W'}\mathbf{h})
$$

3. 使用平方差损失函数：

$$
E(\mathbf{x},\mathbf{t},\mathbf{W},\mathbf{W'})= \frac{1}{2}\sum_{j=1}^{M}(y_j - t_j) ^2
$$

4. 反向传播，输出层到隐藏层：

   4.1 首先是损失函数对输出层的反向传播求导：

   > 注意这里对 $y_j$ 求偏导，相当于 $j$ 已知，因此可以消除求和符号

$$
\dfrac{\partial{E}}{\partial{y_j}} = y_j - t_j
$$

	4.2 继续向下，反向传播到激活函数层（和上述单一神经元一致嗷），结果记为$EI^{'}_{j}$：
$$
\dfrac{\partial{E}}{\partial{u_j^{'}}} = \dfrac{\partial{E}}{\partial{y_j}} \cdot \dfrac{\partial{y_j}}{\partial{u_j^{'}}} = (y_j - t_j) \cdot y_j(1-y_j) := EI^{'}_j
$$
	4.3 继续向下，到达了隐藏层和输出层之间的权值矩阵（和上述单一神经元一致嗷）：
$$
\dfrac{\partial{E}}{\partial{w_{ij}^{'}}} = \dfrac{\partial{E}}{\partial{u_j^{'}}} \cdot \dfrac{\partial{u_j^{'}}}{\partial{w_{ij}^{'}}} = EI^{'}_j \cdot h_i
$$

5. 到这里即可完成对 $W^{'}$ 的更新

$$
{w'}_{ij}^{(new)} = {w'}_{ij}^{(old)} - \eta \cdot EI^{'}_j \cdot h_i
$$

6. 继续向下，我们以 $h_i$ 为连接点，尝试反向传播到输入层

   6.1 首先完成对 $h_i$ 的反向传播

   > 注意这里对 $h_i$ 求偏导，相当于 $i$ 已知，但是 $j$ 未知，因此其偏导数来源于所有链接的输出层

   $$
   \dfrac{\partial{E}}{\partial{h_{i}}} = \sum_{j=1}^{M}\dfrac{\partial{E}}{\partial{u_j^{'}}} \cdot \dfrac{\partial{u_j^{'}}}{\partial{h_{i}}} = \sum_{j=1}^{M}EI^{'}_j \cdot w_{ij}^{'}
   $$

   6.2 打通了 $h_i$ ，然后走到输入层与隐层之间的激活函数，结果记为$EI^{'}_{i}$：
   $$
   \dfrac{\partial{E}}{\partial{u_{i}}} = \dfrac{\partial{E}}{\partial{h_i}} \cdot \dfrac{\partial{h_i}}{\partial{u_{i}}} = \sum_{j=1}^{M}EI^{'}_j \cdot h_i(1-h_i) := EI_i
   $$
   6.3 然后就触碰到了输入层与隐层之间的权值矩阵 $w_{ki}$ ：
   $$
   \dfrac{\partial{E}}{\partial{w_{ki}}} = \dfrac{\partial{E}}{\partial{u_i}} \cdot \dfrac{\partial{u_i}}{\partial{w_{ki}}} = EI_i \cdot x_k
   $$

7. 至此即可完成输入层与隐层之间权值矩阵的更新：

$$
{w'}_{ki}^{(new)} = {w'}_{ki}^{(old)} - \eta \cdot EI^{'}_i \cdot x_k
$$



# 背景

word2vec 模型被越来越多的科研工作者解除和使用，并且效果十分不错：能够捕获到语义信息

# 挑战

模型虽然优秀，但是截至目前却缺少对“词嵌入模型”参数学习过程的全面解释，从而使非神经网络专家无法理解这种模型的工作机制。

> I notice that there lacks a material that comprehensively explains the parameter learning process of word embedding models in details, thus preventing researchers that are non-experts in neural networks from understanding the working mechanism of such models.

# 方案

## Continuous Bag-of-Word Model

### One-word Context

依旧是由简入繁，首先模型的输入也就是目标单词的上下文仅一个单词

> We assume that there is only one word considered per context, which means the model will predict one target word given one context word, which is like a bigram model.

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210711195544139.png" alt="image-20210711195544139" style="zoom:67%;" />

模型图如上所示，其中：$\mathbf{x} = \{x_1,\dots,x_V\}$ 为输入的词向量（one-hot），嵌入维度为 $V$ (词典的长度)；$h=\{h_1,\dots, h_n\}$ 为隐层神经元

1. 首先完成输入到隐层的前向传播：词向量与权值矩阵相乘，因为是 one-hot编码，不妨令词向量 $k$ 位为 $1$ 其余位为 $0$ 那么结果就是权值矩阵的第 $k$ 行向量（ $\mathbf{W}^T_{(k,·)}$ 我理解的是表示 $W$ 矩阵的第 $k$ 行行向量）并将结果记作 $v_{W_I}^T$

$$
\mathbf{h} = \mathbf{W}^T\mathbf{x}  = \mathbf{W}^T_{(k,·)} := v_{w_I}^T
$$

2. 然后完成隐层到输出层的前向传播

   1. 先与权值矩阵 $W’$ 进行矩阵乘法：一个维度是 $(N, 1)$ 一个维度是 $(N, V)$，我们记第 $j$ 个输出为 $u_j$ (是一个标量哦，激活后对应 $y_j$，表示第 $j$ 个词是目标词的概率大小）
      $$
      u_j = {v'}_{w_j}^T\mathbf{h}
      $$
      其中 ${v'}_{w_j}$ 是 $W’$ 的第 $j$ 列

   2. 然后我们对第 $j$ 个输出进行激活 (Softmax 函数)：（这里面的 $w$ 表示 word ）
      $$
      p(w_j|w_I) = y_j = \dfrac{exp(u_j)}{\sum_{j'=1}^{V}exp(u_{j'})}
      $$

3. 将中间变量 $u_j, u_j’$ 作一下替换得到详细

4. 形式：

$$
p(w_j|w_I) = \dfrac{exp({v'}_{w_j}^T v_{w_I})}{\sum_{j'=1}^{V}exp({v'}_{w_{j'}}^T v_{w_I}^T)}
$$

4. 选择最优化目标，得到损失函数：

   令 $w_O$ 表示目标单词，其索引记为 $j^*$ ，因此我们要最大化如下这一概率值：
   $$
   \mathop{max} p(w_O|w_I) = \mathop{max}y_{j^*}
   $$
   为了简化运算，我们引入的对数函数消除了除法，在这里默认以 $e$ 为底，方便求导
   $$
   = \mathop{max} \mathop{log} y_{j*} = u_{j*} - \mathop{log} \sum_{j'=1}^{V}exp(u_{j'})
   $$
   最后，最大化转为最小化得到损失函数：
   $$
   E = -\mathop{log}p(w_O|w_I) = \mathop{log} \sum_{j'=1}^{V}exp(u_{j'}) - u_{j*}
   $$

5. 执行反向传播，输出层到激活函数，将其结果记为 $e_j$
   $$
   \dfrac{\partial{E}}{\partial{u_j}} = y_j - t_j := e_j
   $$
   这里 $t_j = 1$ 当且仅当 $j = j*$ 否则 $t_j = 0$

   > 这里对 $u_j$ 求偏导，对于 $j’ \neq j$ 的 $u_{j’}$ 都看做常数
   > $$
   > \dfrac{\partial{(\mathop{log} \sum_{j'=1}^{V}exp(u_{j'}))}}{\partial{u_j}} = \dfrac{1}{\sum_{j'=1}^{V}exp(u_{j'})} \times (C + exp(u_j))' \\ =  \dfrac{exp(u_j)}{\sum_{j'=1}^{V}exp(u_{j'})} = y_j
   > $$

6. 执行反向传播，激活函数到权值矩阵 $W'$
   $$
   \dfrac{\partial{E}}{\partial{{v'}_{w_j}}} = \dfrac{\partial{E}}{\partial{u_j}} \cdot \dfrac{\partial{u_j}}{\partial{{v'}_{w_j}}}= e_j \cdot \mathbf{h}
   $$

7. 至此我们就可以完成对权值矩阵 $W'$ 的更新了
   $$
   {v'}_{w_j}^{(new)} = {v'}_{w_j}^{(old)} - \eta \cdot e_j \cdot \mathbf{h} \ \ (\mathop{for}\ j\ \mathop{in}\ \{1, 2, \dots, V\})
   $$

   > 从这里我们也能看出，第 $j$ 列的权值跟新主要来源于其输入向量 $h$ ,而其系数 $y_j - t_j$ 则判断其更新方向（$y_j \in [0, 1]$）
   >
   > 1. 当 $t_j = 1$ 时，如果 $y_j \rightarrow 1$ 那么效果很好，对该权值的更新也就很小
   > 2. 当 $t_j = 1$ 时，如果 $y_j$ 不趋近于1，表示 $y_j$ 有点偏小，则 $u_j$ 偏小，而 $u_j = {v'}_{w_j}^T\mathbf{h}$ 因此要适度增加 ${v'}_{w_j}$
   > 3. 当 $t_j = 0$ 时，$y_j - t_j$ 必定为正，只不多将视程度减小 ${v'}_{w_j}$

8. 执行反向传播，激活函数到隐层 $\mathbf{h}$ , 结果记为 $EH$ 
   $$
   \dfrac{\partial{E}}{\partial{\mathbf{h}}} = \sum_{j=1}^{V}\dfrac{\partial{E}}{\partial{u_j}} \cdot \dfrac{\partial{u_j}}{\partial{\mathbf{h}}}= \sum_{j=1}^{V}e_j \cdot {v'}_{w_j} := EH
   $$

9. 隐层到权值矩阵 $\mathbf{W}$
   $$
   \dfrac{\partial{E}}{\partial{v_{w_I}^T}} = \dfrac{\partial{E}}{\partial{\mathbf{h}}} \cdot \dfrac{\partial{\mathbf{h}}}{\partial{v_{w_I}^T}}= \sum_{j=1}^{V}e_j \cdot {v'}_{w_j} := EH
   $$

10. 至此我们就可以完成对权值矩阵 $\mathbf{W}$ 的更新了：
    $$
    {v}_{w_I}^{(new)} = {v}_{w_I}^{(old)} - \eta EH^T = {v}_{w_I}^{(old)} - \eta(\sum_{j=1}^{V}e_j \cdot {v'}_{w_j})^T
    $$

    > 虽然有个求和符号，这时第 $I$ 列的权值的更新依旧取决于 $y_{j*}$, 
    >
    > 1. 当 $y_{j*} \rightarrow 1$ 那么其余 $y_j \rightarrow 0$, 从而 $y_j - t_j \rightarrow 0$ 那么更新幅度很低
    > 2. 当 $y_{j*}$ 不趋近于1时，其会产生一个 （$y_{j*} - t_{j*}$ ）小于 0 的更新幅度，而对于其余 $y_j$ 必然产生较大的大于 0 的更新幅度，根据这两个幅度以及隐层与输出层间权值矩阵的信息，动态调节权值信息

### Multi-word context

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210711232406495.png" alt="image-20210711232406495" style="zoom: 67%;" />

上图即为 CBOW 的模型图，可以看出主要变化就是多个词向量的输入

1. 前向传播，输入层到隐层，将输入的多个词向量统一加权求和求平均后得到隐层结果：

$$
\mathbf{h} = \frac{1}{C} \mathbf{W}(\mathbf{x_1+x_2+\dots x_C}) \\ = \frac{1}{C} (\mathbf{v_{w_1}+v_{w_2}+\dots v_{w_C}})
$$

		其中 $C$ 表示与目标词相关的上下文单词个数

2. 然后完成隐层到激活层的传播：$u_j = {v'}_{w_j}^T\mathbf{h}$
3. 然后进行激活：$p(w_O|w_{I,1},\dots,w_{I,C}) = y_j = \dfrac{exp(u_j)}{\sum_{j'=1}^{V}exp(u_{j'})}$
4. 展开，看下最终的前向传播结果：

$$
$p(w_O|w_{I,1},\dots,w_{I,C}) = \dfrac{exp({v'}_{w_j}^T \frac{1}{C} (\mathbf{v_{w_1}+v_{w_2}+\dots v_{w_C}}))}{\sum_{j'=1}^{V}exp({v'}_{w_{j'}}^T \frac{1}{C} (\mathbf{v_{w_1}+v_{w_2}+\dots v_{w_C}})^T)}
$$

5. 得到我们的损失函数：

$$
E = \mathop{log} \sum_{j'=1}^{V}exp(u_{j'}) - u_{j*} = \mathop{log} \sum_{j'=1}^{V}exp({v'}_{w_{j'}}^T\mathbf{h}) - {v'}_{w_{O}}^T\mathbf{h}
$$

6. 执行反向传播，输出层到激活函数，将其结果记为 $e_j$
   $$
   \dfrac{\partial{E}}{\partial{u_j}} = y_j - t_j := e_j
   $$

7. 执行反向传播，激活函数到权值矩阵 $W'$

$$
\dfrac{\partial{E}}{\partial{{v'}_{w_j}}} = \dfrac{\partial{E}}{\partial{u_j}} \cdot \dfrac{\partial{u_j}}{\partial{{v'}_{w_j}}}= e_j \cdot \mathbf{h}
$$

8. 至此我们就可以完成对权值矩阵 $W'$ 的更新了

$$
{v'}_{w_j}^{(new)} = {v'}_{w_j}^{(old)} - \eta \cdot e_j \cdot \mathbf{h} \ \ (\mathop{for}\ j\ \mathop{in}\ \{1, 2, \dots, V\})
$$

9. 执行反向传播，激活函数到隐层 $\mathbf{h}$ , 结果记为 $EH$ 

$$
\dfrac{\partial{E}}{\partial{\mathbf{h}}} = \sum_{j=1}^{V}\dfrac{\partial{E}}{\partial{u_j}} \cdot \dfrac{\partial{u_j}}{\partial{\mathbf{h}}}= \sum_{j=1}^{V}e_j \cdot {v'}_{w_j} := EH
$$

10. 隐层到权值矩阵 $\mathbf{W}$

    注意这里对权值矩阵 $W$ 针对单词 $w_I$ 所作用的列求偏导，那么以它列相当于常数，因此 $\dfrac{\partial{\mathbf{h}}}{\partial{v_{w_I}^T}} = \frac{1}{C}$

$$
\dfrac{\partial{E}}{\partial{v_{w_I}^T}} = \dfrac{\partial{E}}{\partial{\mathbf{h}}} \cdot \dfrac{\partial{\mathbf{h}}}{\partial{v_{w_I}^T}}= EH \cdot \dfrac{1}{C}
$$

11. 至此我们就可以完成对权值矩阵 $\mathbf{W}$ 的更新了：

$$
{v}_{w_{I,c}}^{(new)} = {v}_{w_{I,c}}^{(old)} - \eta \cdot \frac{1}{C} EH^T = {v}_{w_{I,c}}^{(old)} - \eta \cdot \frac{1}{C} (\sum_{j=1}^{V}e_j \cdot {v'}_{w_j})^T
$$

可以看出来多个单词的输入对最终词向量的影响不仅仅在于 $\frac{1}{C}$ 而且还有 ${v'}_{w_j}$ , 即权值矩阵 $W’$, 其反向传播中也将以众多输入向量的隐藏表达为指导，配以 $e_j = y_j - t_j$ 这一 “程度稀疏” 来更新权重，进而影响到权值矩阵 $W$。这一现象也符合我对 word2vec 模型的假象逻辑。推理至此应该是成功的。

## Skip-Gram Model

![image-20210712094615755](https://gitee.com/Butterflier/pictures/raw/master/image-20210712094615755.png)

Skip-Gram 模型图如上图所示，可以看出它和 CBOW 正好相反，以中间词（一个词）作为输入，输出确实中间词的上下文单词（多个词）。

> It is the opposite of the CBOW model. The target word is now at the input layer, and the context words are on the output layer.

需要注意的是，模型图中隐层与输出层之间的权值矩阵 $W’$ 仅有一个，因此我们有着相同的输入 $\mathbf{x}$ 进行加权求和后有着相同的隐层表达 $\mathbf{h}$，再加权求和激活或就有着 $C$ 个相同的输出 $\mathbf{y}$ ,为什么要这么设计呢？

我认为的是，我们最终的输出不是尽可能去完整的匹配到具体的一个词向量，而是去学习到输入词 $w_I$ 的一个上下文语境的表达 $\mathbf{y}$（一个词对应一个语境表达），而让这个 $\mathbf{y}$ 尽可能的和上下文单词表达差距最小，这样反复迭代，即可达到词向量间语义的抓取。

那这样就和Continuous Bag-of-Word Model【One-word Context】的前后向传播一致了

1. 输入到隐层的前向传播：

$$
\mathbf{h} = \mathbf{W}^T\mathbf{x}  = \mathbf{W}^T_{(k,·)} := v_{w_I}^T
$$

2. 然后完成隐层到输出层的前向传播

   1. 第 $c$ 个输出层表达：
      $$
      u_{c,j} = u_j = {v'}_{w_j}^T\mathbf{h}
      $$

   2. 对第 $c$ 个输出进行激活 (Softmax 函数)：
      $$
      p(w_{c,j}| w_{I}) = y_{c,j} = \dfrac{exp(u_{j})}{\sum_{j'=1}^{V}exp(u_{j'})}
      $$
      

      这里变量有些过多了：输出层一共有 $C$ 个，我们现在考虑第 $c$  个输出层，一共有 $V$ 个单词，我们现在考虑第 $c$  个输出层的表达是第 $j$ 个单词的概率

3. 接下来直接进入到损失函数环节：

$$
E = -\mathop{log} p(w_{O,1},w_{O,2},\dots,w_{O,C}|w_I) = -log \prod_{c=1}^{C} \dfrac{exp(u_{c,j_c^*})}{\sum_{j'=1}^{V}exp(u_{j'})} \\ = C \cdot \mathop{log}\sum_{j'=1}^{V}exp(u_{j'}) - \sum_{c=1}^{C} u_{c,j_c^*}
$$

4. 执行反向传播，输出层到激活函数，将其结果记为 $e_{c,j}$
   $$
   \dfrac{\partial{E}}{\partial{u_{c,j}}} = y_{c,j} - t_{c,j} := e_{c,j}
   $$

5. 合并各个输出层 $c$ 的反向传播结果：
   $$
   EI_j = \sum_{c=1}^{C}e_{c,j} = \dfrac{\partial{E}}{\partial{u_j}}
   $$

6. 执行反向传播，激活函数到权值矩阵 $W'$
   $$
   \dfrac{\partial{E}}{\partial{{v'}_{w_j}}} = \dfrac{\partial{E}}{\partial{u_j}} \cdot \dfrac{\partial{u_j}}{\partial{{v'}_{w_j}}}= EI_j \cdot \mathbf{h}
   $$

7. 至此我们就可以完成对权值矩阵 $W'$ 的更新了

$$
{v'}_{w_j}^{(new)} = {v'}_{w_j}^{(old)} - \eta \cdot EI_j \cdot \mathbf{h} \ \ (\mathop{for}\ j\ \mathop{in}\ \{1, 2, \dots, V\})
$$

8. 执行反向传播，激活函数到隐层 $\mathbf{h}$ , 结果记为 $EH$ 

$$
\dfrac{\partial{E}}{\partial{\mathbf{h}}} = \sum_{j=1}^{V}\dfrac{\partial{E}}{\partial{u_j}} \cdot \dfrac{\partial{u_j}}{\partial{\mathbf{h}}}= \sum_{j=1}^{V}EI_j \cdot {v'}_{w_j} := EH
$$

9. 隐层到权值矩阵 $\mathbf{W}$

$$
\dfrac{\partial{E}}{\partial{v_{w_I}}} = \dfrac{\partial{E}}{\partial{\mathbf{h}}} \cdot \dfrac{\partial{\mathbf{h}}}{\partial{v_{w_I}}}= EH
$$

10. 至此我们就可以完成对权值矩阵 $\mathbf{W}$ 的更新了：

$$
{v}_{w_I}^{(new)} = {v}_{w_I}^{(old)} - \eta EH^T = {v}_{w_I}^{(old)} - \eta(\sum_{j=1}^{V}EI_j \cdot {v'}_{w_j})^T
$$

> 可以看出 “程度系数“ $EI_j$ 将考虑到输入词的上下文表达 $\mathbf{y}$ 与真实上下文单词 $j_{c}^*$ 的相差程度，来利用 $v’_{w_j}$ 更新权值矩阵

# 优化计算效率

Optimizing Computational Efficiency

在前面的学习过程中，就可以看出我们模型对第 $j$ 个单词的预测概率需要去计算其余所有单词的概率和 $\dfrac{exp(u_{j})}{\sum_{j'=1}^{V}exp(u_{j'})}$，虽然都通过矩阵运算一次性计算出来了，但是字典 $V$ 通常是 million 级别的应用，因此还是会大大影响了计算效率

## Hierarchical Softmax

这是解决方案之一，看名字能够知道是要在 softmax 上做文章，最终实现 softmax 的高效计算

![image-20210712113315199](https://gitee.com/Butterflier/pictures/raw/master/image-20210712113315199.png)

数据结构为二叉树，叶子节点为词典中的词，一共有 $V$ 个词，因此二叉树内部节点个数为 $V-1$

> 证明：
>
> 设节点总数为 $n$，叶子节点个数为：$n_0$，度为 1 的节点个数为：$n_1$，度为 2 的节点个数为 $n_2$，边的个数为 $b$，度为 1 的节点生出一条边，度为 2 的节点生出两条边
> $$
> n = n_0 + n_1 + n_2 \\
> b = n - 1 = n_1 + 2n_2
> $$
> 等量代换得到：$n_0 = n_2 + 1$

对于每一个叶子节点，都存在到达根节点的唯一一条（不重复）路径，这一条路径就将用来估计叶节点所代表单词表达的概率。图中的 $n(w_2, j)$ 表示到达 $w_2$ 叶节点途径的第 $j$ 个节点。每一个中间节点都存在着一个输出变量：$\mathbf{v'}_{n(w, j)}$

计算单词 $w$ 为本模型输出单词 $w_O$ 的概率为：$L(w)$ 表示路径长度，$ch(n)$ 表示节点的左孩子，$[\![x]\!]$ 表示如果 $x$ 为真，其值为 1，否则为 -1

1. 从根节点 $n$ 出发，我们要根据单词的隐层表达，计算下一步的方向（左 OR 右）：

$$
p(n, left) = \sigma({v'}_n^T \cdot \mathbf{h}) \\
p(n, right) = 1 - p(n, left) = \sigma(-{v'}_n^T \cdot \mathbf{h})
$$

	在这里如果是 CBOW 模型，$\mathbf{h} = \frac{1}{C} \sum_{c=1}^{C} v_{w_c}$ ; 如果是 skip-gram 模型，$\mathbf{h} = \mathbf{v}_{w_I}$, $\sigma$ 表示 sigmoid 激活函数，进行二分类选择

2. 因此我们用乘法连接路径，定义公式处理下符号问题即可得到：

$$
p(w=w_O) = \prod_{j=1}^{L(w)-1} \sigma([\![n(w,j+1) = ch(n(w,j))]\!] \cdot {v'}_{n(w,j)}^T\cdot\mathbf{h})
$$

接下来我们以 One-word Context 为例演算下反向传播，为了简化表达并且向 One-word Context 中定义的变量考虑，采用了以下缩写：
$$
[\![\cdot]\!] = [\![n(w,j+1) := ch(n(w,j))]\!] \\
v_j' := {v'}_{n(w,j)}
$$

1. 得到我们的损失函数：

$$
E = -\mathop{log} p(w=w_O | w_I) = -\sum_{j=1}^{L(w)-1} \mathop{log} \sigma ([\![\cdot]\!] \cdot {v'}_{j}^T\cdot\mathbf{h})
$$

2. 然后对隐层与输出层之间的权值矩阵进行更新：

   $E$ 中的求和不可怕，非 $j$ 的都看作常数：先来对 $\mathbf{v}'_j\mathbf{h}$ 求偏导，然后再对 $\mathbf{v}'_j$ 求偏导

$$
\dfrac{\partial{E}}{\partial{\mathbf{v}'_j\mathbf{h}}} = - [\mathop{log} \sigma([\![·]\!]\mathbf{v}'_j\mathbf{h})]' = - \frac{1}{A} \cdot A' = - \frac{1}{A} \cdot A(1-A) \cdot ([\![·]\!]\mathbf{v}'_j\mathbf{h})' \\= (A-1) \cdot [\![·]\!] = \left(\sigma([\![·]\!]\mathbf{v}'_j\mathbf{h}) - 1\right) [\![·]\!]
$$

	化简 $[\![·]\!]$
	
	当 $[\![·]\!] = 1$ 时 $\dfrac{\partial{E}}{\partial{\mathbf{v}'_j\mathbf{h}}} = \sigma(\mathbf{v}'_j\mathbf{h}) - 1$ ；	
	
	当 $[\![·]\!] = -1$ 时 $\dfrac{\partial{E}}{\partial{\mathbf{v}'_j\mathbf{h}}} = 1 - \sigma(-\mathbf{v}'_j\mathbf{h}) = \sigma(\mathbf{v}'_j\mathbf{h})$；
	
	这时候就和 $ y_j - t_j$ 碰上了，我们定义此时的 $t_j = 1$ 当且仅当 $[\![·]\!] = 1$ 时，否则$t_j = 0$
	
	因此对 $\mathbf{v}'_j\mathbf{h}$ 求偏导的结果如下：
$$
\dfrac{\partial{E}}{\partial{\mathbf{v}'_j\mathbf{h}}} = \sigma(\mathbf{v}'_j\mathbf{h}) - t_j
$$

	然后对 $\mathbf{v}'_j$ 求偏导，这个就比较简单啦：
$$
\dfrac{\partial{E}}{\partial{\mathbf{v}'_j}} = \dfrac{\partial{E}}{\partial{\mathbf{v}'_j\mathbf{h}}} \cdot \dfrac{\partial{\mathbf{v}'_j\mathbf{h}}}{\partial{\mathbf{v}'_j}} = \left(\sigma(\mathbf{v}'_j\mathbf{h}) - t_j\right) \cdot \mathbf{h}
$$



3. 更新隐层与输出层之间的权值矩阵：

$$
{v'}_{j}^{(new)} = {v'}_{j}^{(old)} - \eta \cdot \left(\sigma(\mathbf{v}'_j\mathbf{h}) - t_j\right) \cdot \mathbf{h}
$$

对比一下优化前，可以看出优化前我们要计算 $V$ 次，而采用了 Hierarchical softmax 我们只需要计算 $log_2(V)$ 次，实际上是 $1, 2, \dots, L(w) -1 $ 次（下方为优化前更新公式）
$$
{v'}_{w_j}^{(new)} = {v'}_{w_j}^{(old)} - \eta \cdot e_j \cdot \mathbf{h} \ \ (\mathop{for}\ j\ \mathop{in}\ \{1, 2, \dots, V\})
$$

> 当 $[\![·]\!] = 1$ 时 $t_j = 1$ 表示应该向左走，而 $\sigma(\mathbf{v}'_j\mathbf{h}) = p(n, left)$ 表示向左走的概率，如果 $ p \rightarrow 1$ 那么对权重没有太大更新，反之，则应根据 $\mathbf{h}, \eta$ 增大权重，以增加 $\sigma(\mathbf{v}'_j\mathbf{h})$ 的值
>
> 当 $[\![·]\!] = -1$ 时 $t_j = 0$ 表示应该向右走，而 $\sigma(\mathbf{v}'_j\mathbf{h}) = p(n, left)$ 表示向左走的概率，如果 $ p \rightarrow 0$ 那么符合正确情况，对权重没有太大更新，反之，则应根据 $\mathbf{h}, \eta$ 减小权重，以减小 $\sigma(\mathbf{v}'_j\mathbf{h})$ 的值

4. 继续向后传递，计算对 $\mathbf{h}$ 的偏导数：

$$
\dfrac{\partial{E}}{\partial{\mathbf{h}}} = \sum_{j-1}^{L(w)-1}\dfrac{\partial{E}}{\partial{\mathbf{v}'_j\mathbf{h}}} \cdot \dfrac{\partial{\mathbf{v}'_j\mathbf{h}}}{\partial{\mathbf{h}}} = \sum_{j-1}^{L(w)-1}\left(\sigma(\mathbf{v}'_j\mathbf{h}) - t_j\right) \cdot \mathbf{v}'_j := EH
$$

	对比一下优化前的，可以看出来大体的一致性：
$$
\dfrac{\partial{E}}{\partial{\mathbf{h}}} = \sum_{j=1}^{V}\dfrac{\partial{E}}{\partial{u_j}} \cdot \dfrac{\partial{u_j}}{\partial{\mathbf{h}}}= \sum_{j=1}^{V}e_j \cdot {v'}_{w_j} := EH
$$

5. 后续的应当一致了，有时间补充完整（等下次回顾的时候吧….)

## Negative Sampling

为了解决因词典数目导致的算力问题，我们只需要将目标词典做一个采样，当成本次更新的词典即可，那么问题来了，我们如何采样呢？除了正样本需要更新被采样到之外，其余的就能随便采吗？并不是。Word2vec 使用了 `unigram distribution raised to the 3/4 powre` 的采样模型，具体在这里也没介绍。

我觉得负采样这里需要钻研的是如何采样，采样完成后，后续的更新就与前面基本一致了~
$$
E = - \mathop{log} \sigma(\mathbf{v'}_{w_O}^T \mathbf{h}) - \sum_{w_j\in W_{neg}} \mathop{log} \sigma(-\mathbf{v'}_{w_j}^T \mathbf{h})
$$
 可以看出后续权值矩阵的更新将只会涉及到 $w_j$ 前向传播所经的权值向量。

# 实验

无

# 不足

无
