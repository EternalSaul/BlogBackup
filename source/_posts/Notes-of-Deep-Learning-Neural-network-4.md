---
title: 'Notes of CS231n:Neural network 4'
date: 2018-03-11 21:30:24
toc: true
tags:
 - 学习笔记
 - 深度学习
---

  我们利用反向传播计算出解析梯度，然后利用计算出的解析梯度来进行参数更新，这里记录一些有些的最优化技巧，但不进行细节分析。

<!--more-->

## 参数更新方法

### 普通更新

  沿着负梯度方向改变参数是一个简单有效的方法，比如一个参数向量x，它的梯度为dx，那么这个简单的方法可以表示为：

```python
x+=learning_rate*dx
```

### 动量更新

  普通更新的缺点是，其更新方向完全依赖于当前的batch，使得更新非常不稳定，从物理角度得到启发，人们想出来一种在深度网络上有着更好收敛速度且更新更稳定的方法，即**动量(Momentum)更新**。

  在上面的普通更新中我们可以看到梯度的大小直接影响到了x的值的更新，**而动量更新则不同，它会由梯度去影响“速度”，然后速度再去影响x的更新**，研究这个方法的人把损失值理解为一个高地，而参数更新这个最优化过程可以看做是一个质量以初始速度0，慢慢的加速，把梯度看做是提供这种加速度的力。听起来很玄学，这也能扯上关系？实际上所做的就是改变了x的更新方式而已，把以前的learning_rate*dx增量变成了一个更加科学的“速度”，如下：

```python
v = mu * v - learning_rate * dx
x += v
```

其中这个mu是一个超参数，叫做动量，它模拟的物体运动的惯性，即更新的时候在一定程度上保留之前更新的方向，同时利用当前batch的梯度微调最终的更新方向，实际上它的意义更加贴近于阻尼系数，mu*v提供了一个阻力，这个变量会抑制速度，**一般mu设置为0.9**，也有一些研究表明mu随时间变化能略改善最优化效果，比如一个典型的设置是一开始mu=0.5，而随后在多个周期中慢慢提升到0.99，[0.5,0.9,0.95,0.99]。

### Nesterov动量更新

  对于动量更新，还有一个增强版的思路，理论上它对于凸函数的收敛有帮助，实践中似乎也确实比上面的标准动量更新表现好点。

  这个思路在于考虑到，更新时，mu\*v这个项会稍加改变参数向量，我们最好是在x+mu\*v这个近似未来的位置附近计算梯度，而不是在x处计算，这样我们的代码可以变为：

```python
x_ahead = x + mu * v
# 计算dx_ahead(在x_ahead处的梯度，而不是在x处的梯度)
v = mu * v - learning_rate * dx_ahead
x += v
```

<img src="\img\20150906103038485.png" width=600px\>

  首先，按照原来的更新方向更新一步（棕色线），然后在该位置计算梯度值（红色线），然后用这个梯度值修正最终的更新方向（绿色线）。上图中描述了两步的更新示意图，其中蓝色线是标准momentum更新路径。

### 二阶方法

  实际上除了各种随机梯度下降算法以外还有一种方法是基于牛顿法的，但是因为实现起来比较困难并不常用。

## 学习率动态设定

### 学习率退火

学习率随着时间慢慢减小对深度学习训练很有帮助，我们并不想看着参数向量无规律的跳动，但是如果学习率过高这很可能发生，并且意味着我们很难达到最好的位置。一般我们有以下三种退火方式：

**随步数衰减**：每隔几个周期就根据某些因素降低学习 率，比如每5个周期减半，或者每20个周期减少到之前的0.1。训练的同时观察验证集的错误率，每当验证集错误率停止下降，就将原来的学习率乘以一个常数比例来降低它。

**指数衰减**：a=a[0]e^(-kt),a0和k是超参数，t是迭代次数或者周期

**1/t衰减**：a=a[0]/(1+kt),a0和k是超参数，t是迭代次数或者周期

实践中一般都采用第一种方法。

### 逐参数适应学习率方法

学习率调参是很耗费计算资源的过程，所以很多工作投入到发明能够适应性地对学习率调参的方法，甚至是逐个参数适应学习率调参。这里将记录一些常用的适应算法。

#### Adagrad

 ```python
# 假设有梯度和参数向量x
cache += dx**2
x += - learning_rate * dx / (np.sqrt(cache) + eps)
 ```

Adagrad的一个缺点是，在深度学习中单调递减的学习率被证明通常过于激进且过早停止学习。这里的**eps是一个很小的数**，通常为1e-4到1e-8之间，**引入它的目的是为了防止出现除以0这种不合法情况出现**。

#### RMSprop

```
cache =  decay_rate * cache + (1 - decay_rate) * dx**2
x += - learning_rate * dx / (np.sqrt(cache) + eps)
```

RMSprop（Root Mean Square Propagation）方法是Adagrad的改进版本，没有公开发表，但是却非常高效，它来自Geoff Hinton的Coursera课程的[第六课的第29页PPT](http://link.zhihu.com/?target=http%3A//www.cs.toronto.edu/%257Etijmen/csc321/slides/lecture_slides_lec6.pdf)。上面的代码里decay_rate是一个超参数，常用值是[0.9,0.99,0.999]。你可以观察这个方法改变了cache的计算部分，而第二个式子却是相同的。RMSProp任然是基于梯度的大小来对每个权重的学习率进行修改，但是学习率不会再单调递减，这是和Adagrad的主要区别，也就是说该方法更不那么激进。

#### Adam

```python
m = beta1*m + (1-beta1)*dx
v = beta2*v + (1-beta2)*(dx**2)
x += - learning_rate * m / (np.sqrt(v) + eps)
```

Adam（Adaptive Moment Estimation）是推荐使用的方法，一般而言跑起来比RMSProp要好一点。但是也可以试试SGD+Nesterov动量。Adam方法看起来真的和RMSProp很像，除了使用的是平滑版的梯度**m**，而不是用的原始梯度向量**dx**。论文中推荐的参数值**eps=1e-8, beta1=0.9, beta2=0.999**。完整的Adam更新算法也包含了一个偏置*（bias）矫正*机制，因为**m,v**两个矩阵初始为0，在没有完全热身之前存在偏差，需要采取一些补偿措施。

## 超参数调优

  你知道一个模型中往往存在着很多超参数，比如随机失活的概率p，以及我们上面提到的种种适应学习率算法中的超参数，我们上面已经给出了一些好的实践值，但是有可能你想去找到更好的值，或者你研究新算法需要有新的超参数。

### 采用仆程序

我们可以采用仆程序持续的设置参数然后进行最优化，并且它对各个周期后验证集的准确率进行监控，并将这个记录点的模型数据（记录点中有各种各样的训练统计数据，比如随着时间的损失值变化等）记录到相关文件中。

### 只使用单个验证集

为了让代码更简单，可以只采用一个合理尺寸的验证集，而不是采用几个数据集来交叉验证。

### 使用随机搜索

比起“网格式”搜索（即相同的间距），随机搜索往往更容易实现也更加好。

### 对于边界上的最优值要小心

这种情况一般发生在你在一个不好的范围内搜索超参数（比如学习率）的时候。比如我们使用**learning_rate = 10 \** uniform(-6,1)**来进行搜索。假设我们得到一个比较好的值-6，一定要确认你的值不是出于这个范围的边界上，不然你可能错过更好的其他搜索范围,就像这里可能-6是边界值而最好的结果在-8和-6之间。

### 从粗到细地分阶段搜索

在实践中，先进行初略范围（比如10 ** [-6, 1]）搜索，然后根据好的结果出现的地方，缩小范围进行搜索。进行粗搜索的时候，让模型训练一个周期就可以了，因为很多超参数的设定会让模型没法学习，或者突然就爆出很大的损失值。第二个阶段就是对一个更小的范围进行搜索，这时可以让模型运行5个周期，而最后一个阶段就在最终的范围内进行仔细搜索，运行很多次周期。

## 模型集成

实践中有一个方法总是能提升神经网络几个百分点的准确率，即在训练的时候训练几个独立模型，然后在测试的时候平均它们的预测结果。集成的模型数量增加，算法的结果也单调提升（但提升效果越来越少）。还有模型之间的差异度越大，提升效果可能越好。进行集成有以下几种方法：

1.相同的模型，不同的初始化。

2.使用不同的超参数的模型。这些模型都是分别训练的。

3.如果训练太耗时，可以采用一次训练中不同的记录点（仆程序记录）的模型来进行集成。

4.在训练的时候跑参数的平均值。和上面一点相关的，还有一个也能得到1-2个百分点的提升的小代价方法，这个方法就是在训练过程中，如果损失值相较于前一次权重出现指数下降时，就在内存中对网络的权重进行一个备份。这样你就对前几次循环中的网络状态进行了平均。你会发现这个“平滑”过的版本的权重总是能得到更少的误差。直观的理解就是目标函数是一个碗状的，你的网络在这个周围跳跃，所以对它们平均一下，就更可能跳到中心去。

