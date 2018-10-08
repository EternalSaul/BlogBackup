---
title: 'Notes of CS231n:Neural network 3'
date: 2018-03-10 13:41:24
toc: true
tags:
 - 学习笔记
 - 深度学习
---

  在上一篇文章中，记录了如何构造一个神经网络，比如如何初始化参数，如何选择激活函数和损失函数，应该用哪种方法来防止过拟合等等，在这篇记录中，记录的是如何训练一个神经网络，包括如何学习参数以及如何将超参数最优化。

<!--more-->

## 梯度检测

  梯度检测就是把解析梯度和数值梯度进行比较，这里只是记录一些小技巧：

### 使用中心化公式而不是有限差值来近似

即使用[f(x+h)+f(x-h)]/2h来计算数值梯度，该公式计算出的梯度会近似于O(h^2)，该结果是通过对f(x+h)，f(x-h)进行泰勒展开得到的。

### 使用相对误差来进行梯度检测

我们知道数值梯度和解析梯度的差值正负值都有可能，很明显我们需要取这个差值的绝对值，但是我们不得不考虑实际梯度的大小，即我们要考虑这个误差绝对值占实际梯度的百分比，如果梯度是1000，误差是0.1和梯度是1e-4梯度是1.0，实际上比率是一样的，如果一个梯度是1而误差达到了0.5这个样子显然我们就得考虑梯度计算是不是错了。

所以我们利用|f'[a]-f'[n]|/max(f'[a],f'[n])来计算相对误差作为我们的误差值，这是一个比率值，我们倾向于在实践中采用这种判断：

相对误差>1e-2时，梯度可能出错了

1e-2>相对误差>1e-4：这个值可能不是很好，但勉强说得过去

1e-4>相对误差：可以接受

1e-7或者更小：比较完美了

### 注意误差伴随深度积累

**有时候我们还是得视情况而定**，要知道误差是在不断积累的，随着深度的加深误差也在不断增大，那么我们可以改变上面的策略，比如一个网络深度达到数十层时1e-2就是可以接受的了。

### 注意目标函数中存在的不可导点

ReLU等函数存在着不可导点，我们知道ReLU中如果出现了x<0的情况，在该点梯度就是0，但是如果这个x值比较小，比如-0.000001我们增加了一个小的增量h，可能此时的h就不是负数了，f(x+h)在可导的点上，从而梯度检查认为这个梯度计算错了。

**使用少量数据点。**解决上面的不可导点问题的一个办法是使用更少的数据点。因为含有不可导点的损失函数(例如：因为使用了ReLU或者边缘损失等函数)的数据点越少，不可导点就越少，所以在计算有限差值近似时越过不可导点的几率就越小。还有，如果你的梯度检查对2-3个数据点都有效，那么基本上对整个批量数据进行梯度检查也是没问题的。所以使用很少量的数据点，能让梯度检查更迅速高效。

### 梯度检查中h的设置

h并不是越小越好，太小了会遇到精度和数值问题，一般可以设置为1e-4或者1e-6

### 梯度检查前先“预热”

我们的梯度检查只是在参数空间中一个特定的单独的点进行的，所以并不能很稳的确定全局梯度都正确。一般来说我们不再一开始就进行梯度检查，而是等到损失函数开始下降之后在进行梯度检查。

### 当心正则化损失

上一文记录损失函数最后的值是正则化损失和数据损失的和，有些情况梯度主要来源于正则化部分，因此会掩盖数据损失梯度不正确的事实。为了防止这种情况出现我们可以先关掉正则化对数据损失做单独检查，当然这也做会导致我们需要再对正则化损失做单独检查。

### 关闭随机失活和数据扩张

在进行梯度检查时，记得关闭网络中任何不确定的效果的操作，比如随机失活，随机数据扩展等。不然它们会在计算数值梯度的时候导致巨大误差。关闭这些操作不好的一点是无法对它们进行梯度检查（例如随机失活的反向传播实现可能有错误）我们可以在计算f(x+h)和f(x-h)时强制增加一个特定是随机种子，在计算解析梯度时也同样如此。

### 检查少量的维度

在实际中，梯度可以有上百万的参数，在这种情况下只能检查其中一些维度然后假设其他维度是正确的。注意：确认在所有不同的参数中都抽取一部分来梯度检查。在某些应用中，为了方便，人们将所有的参数放到一个巨大的参数向量中。在这种情况下，例如偏置就可能只占用整个向量中的很小一部分，所以不要随机地从向量中取维度，一定要把这种情况考虑到，确保所有参数都收到了正确的梯度。

## 跟踪各个重要数值

  跟踪并可视化我们的训练过程可以让我们尽早的发现一些问题，我们所跟踪的一般是一些比较重要的数值，我们通常每个**周期**进行一轮跟踪，这里的**周期**衡量了在训练中每个样本数据都被观察过次数的期望（一个周期意味着每个样本数据都被观察过了一次）。

1.跟踪损失值

以周期为x轴度量，损失值的曲线往往会反应我们所设置的学习率的问题，比如：

![loss_learningrate](\img\loss_learningrate.png)

上图左图中我们可以观察到学习率与损失值的关系，右图则反应了损失值的震荡程度，这与批尺寸（batch size）有关（这里的批指的是我们一般把训练集拆分成多个批次，每次取一个训练），如果批尺寸太小了比如1，那么每次修正方向以各自样本的梯度方向修正，难以达到收敛震荡幅度就大，而如果以整个训练集大小为批尺寸则震荡幅度最小，但是这并不是一个好的想法，首先内存容量增加，第二我们可能得不到最好的结果，batchsize 的正确选择是为了在内存效率和内存容量之间寻找最佳平衡，比如我们可以看下图：

![lU3sx](\img\lU3sx.png)

红色代表批尺寸为1，绿色为一个适中的尺寸，蓝色为不分批次，即全尺寸，我们可以观察到震荡程度是不同的。

2.跟踪训练集和验证集的准确率

跟踪准确率可以知道我们的模型是不是过拟合了，如图：

![QQ图片20180310170549](\img\QQ图片20180310170549.png)

如果观察到我们的模型过拟合了，就应该增大正则化强度，比如调高随机失活率p，或者将L1损失换位到L2损失，采用更丰富的数据集等等，当然如果你的模型太小也会导致过拟合，这时候我们就应该考虑更换更大的模型了。

3.跟踪权重的跟新比例

  权重中更新值的数量和全部值的数量之间的比例也是我们值得更新的。注意：是*更新的*，而不是原始梯度（比如，在普通sgd中就是梯度乘以学习率）。需要对每个参数集的更新比例进行单独的计算和跟踪。一个经验性的结论是这个比例应该在1e-3左右。如果更低，说明学习率可能太小，如果更高，说明学习率可能太高。

4.跟踪每层激活数据和梯度分布

  我们可以观察激活函数的输出，把这些数据变成柱状与表示会更明显，比如对于tanh函数，如果神经元输出总是0，或者总是-1或1，那说明可能出现了某些问题了。

5.如果是图像数据考虑将第一层特征可视化

如果需要做的是图像方面的识别，我们考虑将第一层特征显示出来，比如：

![96573094f9d7f4b3b188069726840a2e_r](\img\96573094f9d7f4b3b188069726840a2e_r.jpg)

  上图中，很明显前面的图，特征非常不平滑，很多噪点，而后面则好得多，一般如果我们训练模型得到的是第一张图这种特征你需要仔细考虑是不是要重新训练模型了。