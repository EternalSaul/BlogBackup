---
title: 'Notes of CS231n:Structure of CNN'
date: 2018-03-12 11:04:18
toc: true
tags:
 - 学习笔记
 - 深度学习
---

  卷积神经网络也由有学习能力的神经元组成，这些神经元中也有权重和偏差。整个卷积神经网络实为是一个可导的评分函数，在神经网络中我们实现的各种技巧和要点依旧适用于卷积神经网络。但是**卷积神经网络与一般神经网络不同的是，它基于一个假设，即输入的数据是图像**。基于这个假设，我们添加了一些特有的特性，从而让我们的前向传播更为高效，**同时也大大降低了网络中的参数数量**。降低参数数量是一个很好的特性，因为在一般的神经网络中，因为是全连接的，如果要处理一个大点的图像，比如一个200\*200\*3的图像，一个神经元就有12万个权重，显然这是不可接受的。

<!--more-->

## 卷积神经网络结构

### 三维排列的神经元

在一般的神经网络中，神经元的排列一般是如下图所示：

![QQ图片20180214180333](\img\QQ图片20180214180333.jpg)

我们可以观察到，如果硬要以一个形式的语言来描述的话，可以用宽度和高度来描述这个二维的神经元排列，但是卷积神经网络与此不同，它是三维排列的，如下图所示，我们以**宽度，高度，和深度**标记了这个神经元排列：

![QQ图片20180312120608](\img\QQ图片20180312120608.png)

  从上图中你没办法发现神经元之间具体是如何连接的，但是你可以先了解一点，**卷积神经网络的各层中的神经元不再采用全连接的方式**，层中的神经元将只与前一层中的一小块区域连接，当然输出层例外，输出层还是全连接的。

  上图中的卷积神经网络的神经元排列可能不能让你清楚的知道深度的意义。我们以CIFAR-10举例，你知道CIFAR-10中的数据体是32\*32\*3这个大小，后面的3是代表了RGB颜色通道，那么它的神经元排列就是深度为3，宽度和高度都是32的排列。最后的输出层是1\*1\*10的，表示了评分向量。

> 卷积神经网络是由层组成的。每一层都有一个简单的API：用一些含或者不含参数的可导的函数，将输入的3D数据变换为3D的输出数据。

### 以层构造的卷积神经网络

卷积神经网络由不同类型的层按照一定的排列顺序组成，这些层会使用一个可微分的函数将激活数据从自身层传到另一个层。比如一个简单的用于CIFAR-10图像分类的网络结构：

>[输入层-卷积层-ReLU层-汇聚层-全连接层]

1.输入层即是输入[32\*32\*3]的原始图像像素值

2.卷基层中，神经元与输入层的一个局部区域相连，每个神经元都计算自己与输入层相连的小区域与自己权重的内积。卷积层会计算所有神经元的输出。如果我们使用12个滤波器（也叫作核），得到的输出数据体的维度就是[32x32x12]。

3.ReLU层将会逐个元素地进行激活函数操作，比如使用以0为阈值的max(0,x)作为激活函数。该层对数据尺寸没有改变，还是[32x32x12]。

4.汇聚层在在空间维度（宽度和高度）上进行降采样（downsampling）操作，数据尺寸变为[16x16x12]。

5.全连接层将会计算分类评分，数据尺寸变为[1x1x10]，其中10个数字对应的就是CIFAR-10中10个类别的分类评分值。正如其名，全连接层与常规神经网络一样，其中每个神经元都与前一层中所有神经元相连接。

具体说来，卷积层和全连接层（CONV/FC）对输入执行变换操作的时候，不仅会用到激活函数，还会用到很多参数（神经元的突触权值和偏差）。而ReLU层和汇聚层则是进行一个固定不变的函数操作，它没有参数（汇聚层有额外的超参数）。卷积层和全连接层中的参数会随着梯度下降被训练，这样卷积神经网络计算出的分类评分就能和训练集中的每个图像的标签吻合了。

## 卷基层与汇聚层

### 卷基层

  从卷积神经网络的名字你就应该知道什么层是核心，卷基层产生了网络中的大部分计算量。

#### 滤波器

  首先你肯定想知道卷积层是怎么样计算的，先明确的一点是：**卷基层的参数是由一些可学习的滤波器（即一组固定权重的集合，又被称作卷积核）集合构成的。**这些滤波器的宽度和高度都比较小，但是深度与输入数据一致。比如在上面CIFAR例子中的第一层卷积层与输入层相连，它的滤波器大小可能是5\*5\*3，其中5*5是宽高，而3则与数据体的RGB通道深度相同。在前向传播的时候，让每个滤波器都在输入数据的宽度和高度上滑动（更精确地说是卷积），然后计算整个滤波器和输入数据任一处的内积。当**滤波器沿着输入数据的宽度和高度滑过**后，会生成一个2维的激活图（activation map），激活图给出了在每个空间位置处滤波器的反应。直观地来说，网络会让滤波器学习到当它看到某些类型的视觉特征时就激活。**在每个卷积层上，我们会有一整个集合的滤波器（比如12个），每个都会生成一个不同的二维激活图。将这些激活映射在深度方向上层叠起来就生成了输出数据。**

#### 感受野

  图像数据是高维的，如果让每个神经元与之进行全连接，那么将会产生大量的参数以至于计算量不能忍受。于是我们让每个神经元只与输入数据的一个局部区域进行连接，这个局部区域的尺寸（高宽）是一个超参数，我们把它叫做神经元的**感受野**。但是可以确定的是，在数据深度上这个连接大小与输入数据一致。比如：

*例1*：假设输入数据体尺寸为[32x32x3]（比如CIFAR-10的RGB图像），如果感受野（或滤波器尺寸）是5x5，那么卷积层中的每个神经元会有输入数据体中[5x5x3]区域的权重，共5x5x3=75个权重（还要加一个偏差参数）。注意这个连接在深度维度上的大小必须为3，和输入数据体的深度一致。

*例2*：假设输入数据体的尺寸是[16x16x20]，感受野尺寸是3x3，那么卷积层中每个神经元和输入数据体就有3x3x20=180个连接。再次提示：在空间上连接是局部的（3x3），但是在深度上是和输入数据体一致的（20）。

### 输出数据体

  现在我们知道了卷积层中神经元与输入数据体中的连接方式，那么卷积层的输出数据体是怎么样的呢？有三个超参数控制着输出数据体的尺寸：**即深度，步长和零填充。**

  输出数据体的深度是一个超参数，它与滤波器数量一致，每个滤波器在输入数据中寻找一些不同的东西。举例来说，如果第一个卷积层的输入是原始图像，那么在深度维度上的不同神经元将可能被不同方向的边界，或者是颜色斑点激活。我们将这些沿着深度方向排列、感受野相同的神经元集合称为**深度列（depth column）**，也有人使用纤维（fibre）来称呼它们。

  另外，滑动滤波器的时候，需要指定步长，这个步长大小同零填充的大小一起决定了输出数据体的高宽。步长和零填充的大小都是超参数。填充有一个良好性质，即可以控制输出数据体的空间尺寸（最常用的是用来保持输入数据体在空间上的尺寸，这样输入和输出的宽高都相等）。

  输出数据体在空间（高度和宽度）上的尺寸可以通过输入数据体尺寸（W），卷积层中神经元的感受野尺寸（F），步长（S）和零填充的数量（P）的函数来计算。（**\*译者注**：这里假设输入数组的空间形状是正方形，即高度和宽度相等*）**输出数据体的空间尺寸为(W-F +2P)/S+1**（注意如果这个公式得出的值不是一个整数，说明你的步长和零填充的设置是无效的）。比如输入是7x7，滤波器是3x3，步长为1，填充为0，那么就能得到一个5x5的输出。

### 参数共享

  在2012年的ImageNet挑战中，Krizhevsky采用了这么一个设定，输入图像的尺寸是[227x227x3]。在第一个卷积层，神经元使用的感受野尺寸**F=11**，步长**S=4**，不使用零填充**P=0**。因为(227-11)/4+1=55，卷积层的深度**K=96**，则卷积层的输出数据体尺寸为[55x55x96]。

  我们可以看到上面举出的这个第一个卷积层就有55\*55\*96=290,400个神经元，每个有11\*11\*3个参数和一个偏差，你可以乘一下就发现，第一层卷积层参数就上亿了，显然我们需要一些策略来避免这种情况发生，这个策略就是**参数共享**。

  一个合理的假设：如果一个特征在计算某个空间位置(x,y)的时候有用，那么它在计算另一个不同位置(x2,y2)的时候也有用。基于这个假设，可以显著地减少参数数量。换言之，就是将深度维度上一个单独的2维切片看做**深度切片（depth slice）**，比如一个数据体尺寸为[55x55x96]的就有96个深度切片，每个尺寸为[55x55]。在这样的参数共享下，例子中的第一个卷积层就只有96个不同的权重集了，一个权重集对应一个深度切片，共有96x11x11x3=34,848个不同的权重，或34,944个参数（+96个偏差）。）。在每个深度切片中的55x55个权重使用的都是同样的参数。**在反向传播的时候，都要计算每个神经元对它的权重的梯度，但是需要把同一个深度切片上的所有神经元对权重的梯度累加，这样就得到了对共享权重的梯度。**这样，每个切片只更新一个权重集。

  注意，如果在一个深度切片中的所有权重都使用同一个权重向量，那么卷积层的前向传播在每个深度切片中可以看做是在计算神经元权重和输入数据体的**卷积**（这就是“卷积层”名字由来）。这也是为什么总是将这些权重集合称为**滤波器（filter）**（或**卷积核（kernel）**），因为它们和输入进行了卷积。下图演示了一个卷积过程，该图中几个超参数分别是K=2（卷积层深度），F=3（感受野尺寸），S=2（步长），P=1（零填充）：

![20160707204048899](\img\20160707204048899.gif)

我们可以看到，在该图中对一个深度切片（蓝色部分每一个矩阵），都用相同的权重进行卷积的过程。

### 汇聚层

  一般我们都会在连续的卷积层之间周期性的插入一个汇聚层，目的是为了降低数据体的空间尺寸，这样的话就能够减少网络中参数的尺寸，从而降低计算成本，另外还有控制过拟合的好处。

  汇聚层也是对输入数据体的每一个深度切片独立进行MAX操作，以改变其空间尺寸（高宽）。最常见的形式是汇聚层使用尺寸2x2的滤波器，以步长为2来对每个深度切片进行降采样，将其中75%的激活信息都丢掉。每个MAX操作是从4个数字中取最大值（也就是在深度切片中某个2x2的区域）。深度保持不变。

汇聚层也有计算公式：

  输入数据体高H1，宽W1，深度为D1

  MAX操作空间大小为F,步长为S

  那么输出数据体:

  宽度：W2=(W1-F)/S+1

  高度：H2=(H1-F)/S+1

  深度不变：D2=D1

一般来说MAX汇聚层只有两种形式：F=3，S=2还有前面说的最常用的F=2，S=2。

![641c8846abcb02d35938660cf96cef1b_r](\img\641c8846abcb02d35938660cf96cef1b_r.jpg)

#### 汇聚层反向传播

**反向传播：**回顾一下反向传播的内容，其中max(x,y)函数的反向传播可以简单理解为将梯度只沿最大的数回传。因此，在向前传播经过汇聚层的时候，通常会把池中最大元素的索引记录下来（有时这个也叫作**道岔（switches）**），这样在反向传播的时候梯度的路由就很高效。

有些人更倾向于在卷积层中使用更大的步长来降低数据体的尺寸，而不是在模型中使用汇聚层，另外有发现认为，在训练一个良好的生成模型时，弃用汇聚层也是很重要的。

## 全连接层到卷积层的转换

全连接层和卷积层之间唯一的不同就是卷积层中神经元只与输入数据中的一个局部区域连接，并且在卷积列中的神经元共享参数。但是两类层中神经元都是计算点积，它们的函数形式实际上是相同的。

我们可以把卷积层转化为一个全连接层，这个全连接层的权重矩阵是巨大的，除了某些特定的块，其余部分都是零（局部连接）。而大部分块中，元素都是相等的（参数共享）。

那么，全连接层也能够被转化为卷积层，因为卷积层连接局部，但是如果这个局部大小大到了和全局一样，那就成了全连接层了。**以卷积层代替全连接层可以让卷积网络在一张更大的输入图片上滑动，得到每个区域的输出（这样就突破了输入尺寸的限制）。下面这段话全部看完可以理解它的意思：**

> 举个例子，如果我们想让224x224尺寸的浮窗，以步长为32在384x384的图片上滑动，把每个经停的位置都带入卷积网络，最后得到6x6个位置的类别得分。上述的把全连接层转换成卷积层的做法会更简便。如果224x224的输入图片经过卷积层和汇聚层之后得到了[7x7x512]的数组，那么，384x384的大图片直接经过同样的卷积层和汇聚层之后会得到[12x12x512]的数组（因为途径5个汇聚层，尺寸变为384/2/2/2/2/2 = 12）。然后再经过上面由3个全连接层转化得到的3个卷积层，最终得到[6x6x1000]的输出（因为(12 - 7)/1 + 1 = 6）。这个结果正是浮窗在原图经停的6x6个位置的得分！
>
> 面对384x384的图像，让（含全连接层）的初始卷积神经网络以32像素的步长独立对图像中的224x224块进行**多次评价**，其效果和使用把全连接层变换为卷积层后的卷积神经网络进行**一次**前向传播是一样的。
>
> 自然，相较于使用被转化前的原始卷积神经网络对所有36个位置进行迭代计算，使用转化后的卷积神经网络进行一次前向传播计算要高效得多，因为36次计算都在共享计算资源。这一技巧在实践中经常使用，一次来获得更好的结果。比如，通常将一张图像尺寸变得更大，**然后使用变换后的卷积神经网络来对空间上很多不同位置进行评价得到分类评分，然后在求这些分值的平均值。**

## 一些对于层的建议

### 层的排列

一个最常见的神经网络结构如下：

**INPUT -> [[CONV -> RELU]*N -> POOL?]*M -> [FC -> RELU]*K -> FC**

其中POOL后面的？代表汇聚层是可选的，如果没有其他说明POOL层都是MAX汇聚。

一般而言，几个小滤波器卷积层的组合比一个大滤波器卷积层好。比如你一层一层地重叠了3个3x3的卷积层（层与层之间有非线性激活函数）。在这个排列下，第一个卷积层中的每个神经元都对输入数据体有一个3x3的视野。第二个卷积层上的神经元对第一个卷积层有一个3x3的视野，也就是对输入数据体有5x5的视野。同样，在第三个卷积层上的神经元对第二个卷积层有3x3的视野，也就是对输入数据体有7x7的视野。假设不采用这3个3x3的卷积层，二是使用一个单独的有7x7的感受野的卷积层，那么所有神经元的感受野也是7x7，但是就有一些缺点。

1.多层小的卷积层能更好的提取出深层次的特征

2.在这里，3个3\*3的卷积层的参数数目是3\*3\*3*通道数N，而7\*7的单个卷积层则是7\*7\*通道数N，这个数目要大于前者。

### 层的尺寸

输入层图像大小应该能被2整除多次，比如32，或者是诸如96或者224，384，512这样的数。

卷积层应该使用小的滤波器，比如3\*3或者5\*5，大的滤波器如果采用了也一般是接上原始输入层的第一个卷积层，并且我们一般会用适当的零填充来保持数据在空间维度上的尺寸，步长一般是1。在实际应用中，更小的步长效果更好。**上文也已经提过，步长为1可以让空间维度的降采样全部由汇聚层负责，卷积层只负责对输入数据体的深度进行变换。**如果卷积层值进行卷积而不进行零填充，那么数据体的尺寸就会略微减小，那么图像边缘的信息就会过快地损失掉。

对汇聚层的选择一般都是用比较小的感受野，比如2\*2，一旦感受野太大可能会使得数据信息丢失。



