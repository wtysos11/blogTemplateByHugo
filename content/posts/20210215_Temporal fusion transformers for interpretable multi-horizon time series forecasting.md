---
title: Temporal fusion transformers for interpretable multi-horizon time series forecasting
date: 2021-02-15T11:00:04+08:00
tags:
  - 计算机/科研/时间序列预测
categories:
  - 计算机论文阅读
---

# Temporal fusion transformers for interpretable multi-horizon time series forecasting

## 摘要

文章关注的是multi-horizon forecasting，这方面包含了很多的输入数据，包括static covariate，known future input，以及其他只在过去被观察到的外源时间序列（即没有它们如何与目标值交互的信息）。

作者指出现有的这些模型都是黑盒模型，并没有展示出它们如何使用当前场景的所有输入。在本文中，我们介绍了Temporal Fusion Transformer，一个新的基于attention的架构，包含高性能的多领域预测与时域上可解释的功能。

为了学习不同尺度上时域的关系，TFT使用了循环层来进行本地的处理与可解释的自注意力层，用作长期依赖。

TFT使用特殊的组件来选择相关的特征，以及使用一系列的门机制来抑制不必要的组件，允许在大范围内都具有较强的性能。

通过一系列真实世界的数据库，我们证明了其卓越的性能。

## 1 Introduction

与one-step-ahead prediction相比，multi-horizon forecast显得更为常用。（不太清楚static covariate到底是什么，听起来有点像是多变量时间序列预测）

传统的multi-horizon forecasting应用可以访问大量的数据源，包括未来已经知道的数据（比如假期的时间）、其他外源的时间序列数据（比如游客的流量）以及静态的元数据（商店的位置），但是我们不知道它们是如何与最终的目标数据进行交互的。

DNN使用注意力机制与循环神经网络来加强对过去相关的时间步的选择，包括Transformer-based model。

现有的这些模型大多都是黑盒模型，其中的参数之间包含了复杂的非线性关系。这使得实现模型难以解释它们的预测结果，使得用户难以去相信模型的输出并让模型建设者去进行修改与调试。

可惜的是，通用的可解释性方法并不适用于时间序列。在它们过去的工作中（LIME和SHAP）并咩有考虑输入特征的时间顺序问题。

核心的贡献包括：
1. static covariate encoder，对一些静态的数据进行编码并送入到神经网络中。
2. 贯穿全局的门机制与基于样本的变量选择，来最小化不相关输入的贡献。
3. Seq2Seq的架构来进行本地处理已知输入
4. temporal self-attention decoder层，来学习长期关系


## 2 Related work

multi-horizon预测可以分为两种：
1. Iterated approaches，使用One-step-ahead prediction model，并重复调用多次。
2. 另一方面，直接方法被训练来直接预测多个预定义的horizon，它们的架构一般是基于Seq2Seq模型。

## 3 Multi-horizon forecasting

时间序列预测的定义

## 4 Model Architecture

主要部件
1. Gating mechanism，用来跳过架构中没有使用的组件，提供一种自适应的深度和网络复杂度来应对不同的环境
2. Variable selection network，用来选择一个时间步内相关的时间序列
3. Static covariate encoder，用来将静态特征整合进网络中。
4. temporal processing，用来学习长期和短期的特征，从观察值和已知值中得到随时间变化的输入。一个Seq2Seq层用来进行本地处理，同时长期依赖关系被可解释的attention block捕捉。
5. Prediction interval，使用quantile forecast进行预测。但是这个可以不用做进一个模型里面，传统的时间序列预测方法中就有类似的做法。

我的感觉是，最大的区别在于attention层的位置。原版中attention层是在每一个Block中的，总共有6个Block来组成一个区域。而现在TFT只在中间使用了一个self-attention层，更接近于原本的Seq2Seq模型。

### 4.1 Gating mechanisms

外部输入与目标的关系一般是不知道的，这使得知晓哪些变量是相关变得非常困难。更重要的是，决定是否采用非线性处理同样非常困难，有些时候使用简单的模型可能效果反而更好，比如数据集很小或者充满噪声。

作者实现了Gated Residual Network，输入为已知输入a和可选context vector c。与传统的Transformer相比，

核心层是Gated Residual Network，$GRN(a,c) = LayerNorm(a+GLU( ...  )))$，与原版的区别有两个，一个是使用了GLU来抑制不需要的输入变量，一个是用了ELU(Exponential Linear Unit activation function)，ELU的一个性质是当输入远远大于0的时候会作为identity function，当其远远小于0的时候则是一个常数。

GLU则是使用了sigmoid来进行激活函数，并做Hadamard乘积来完成这个压制功能。GLU可以控制GRN中原始输入a的贡献。GLU的输出接近于0的时候这就是一个跳层，从而抑制非线性关系的贡献。dropout在Layer Norm之间进行。

### 4.2 Variable Selection Network

虽然知道有很多的输入变量，但是它们之间的相关性与对输出的贡献仍然是未知的。

变量选择网络除了让TFT知道哪一个变量是最重要的之外，同样使得它能够移除不必要的、可能对性能产生负面效果的noisy input。

使用entity embedding来作为分类变量的特征表示方法，对连续变量用线性转换（使得向量长度符合网络要求）。

### 4.3 Static Covariate Encoder

与其他的时间序列预测架构不同，TFT将静态的元数据中有用的信息也整合进了预测过程中。

这一部分产生context vector，四个context vector会作为输入导入到不同的网络过程中。

### 4.4 Interpretable mutli-head attention

说实话没太看明白，不过这里主要的改进不是对注意力机制本身，而是对其可解释性。具体来说是对A函数进行修改。

### 4.5 Temporal Fusion Decoder

这里都没怎么看明白，比较细节

#### 4.5.1 Locality Enhancement with Sequence-to-Sequence Layer

作者在这里支出，时间序列中的点的重要性并不是由它自己决定的，而是由它周围的值决定的，比如离群点、变点或者循环模式等。通过利用本地内容，构建利用特征信息的特征，就可以提升最终的预测性能。

比如[12]采用了单个CNN来进行locality enhancement，使用同一个过滤器来提取本地的特征。但是对于observed input存在的情况下，这种方法可能并不成立，因为过去和未来的输入并不总是一致的。

为此，作者实现了Seq2Seq架构的模型来处理这种特征维数的变化，将前k个给encoder，将后d个给decoder，然后产生一组时序特征，输入到decoder中。

### 4.6 Quantile outputs

我不太清楚为什么置信区间输出是放在神经网络里面的，从理论上置信区间输出应该是根据过去预测值与过去真实值的残差进行的，通过对残差的方差进行估计，拟合正态分布可以得到未来的预测值的置信区间。

文中输出的是10th,50th和90th，产生的方法是对原预测值进行线性转换。但是监督数据是怎么来的，挺奇怪的。