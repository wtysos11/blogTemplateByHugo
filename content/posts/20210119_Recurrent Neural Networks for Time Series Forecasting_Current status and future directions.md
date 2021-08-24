---
title: "Recurrent Neural Networks for Time Series Forecasting: Current status and future directions"
date: 2021-01-19T19:50:04+08:00
tags: ["时间序列预测","综述文章","RNN"]
categories: ["计算机论文阅读"]
---
# Recurrent Neural Networks for Time Series Forecasting: Current status and future directions
2021 International Journal of Forecasting

文章对基于RNN的时间序列预测方法进行了比较全面地综述，而且这是发表在IJF上的文章，意味着这篇文章会更偏向于预测本身，而不是模型。
文章结构：第二部分是背景知识，包括传统univariate forecasting technique和不同的NN预测；第三部分包括RNN的实现细节和相关的数据预处理方法；第四部分解释了本文评测时所用的方法与数据集；第五部分进行了批判性的分析；第六部分进行总结；第七部分给出对未来的表述。

## 第二部分

### 2.1 Univariate Forecasting
传统的单变量方法即为时间序列基于其过去的值来完成对未来值的预测，即给定序列X={x1,x2,...,xT}，需要完成{X_{T+1},...,X_{T+H}} = F(x1,x2,...,xT) + \epislon，这里的F是一个函数，经过序列X的训练产生得到，H是预测的跨度（horizon）\

传统的时间序列预测方法在NN3、NN5和M3竞赛上都取得了最佳成绩，它们在数据量很小时表现非常好。但是由于数据量的限制，它们面对大量数据时的表现就不如机器学习算法了。

### 2.2 ANN
之前一直使用的是传统的FCNN，但是目前更多用的是RNN。RNN的cell比较常用的有Elman RNN cell, LSTM和GRUB，此外还有一些其他的架构。这些架构都同时在时间序列预测领域和自然语言处理中得到了使用。
一些架构介绍：
1. Smyl所用的简单复合LSTM，他后来将其与ES算法结合并取得了M4竞赛的冠军
2. Seq2Seq，被Sutskever引入。传统的S2S架构中Encoder与Decoder都是RNN，这方面比较出色的工作是Amazon的DeepAR。在后续的一些工作中，S2S架构不再作为直接的预测手段，而是作为一种特征提取方式被整合进时间序列预测框架中。
3. Seq2Seq的一种变体是引入注意力机制。S2S机制的一个问题是将所有的输入数据编码成向量会造成信息损失，而注意力机制通过对更重要的信息加权，可以尽量减少这种信息损失。比如对于以年为周期的月度数据，显然上一年的相同月份的权重应该会更大
4. 使用RNN的组合(ensemble RNN)，比如Smyl将这个问题分成两部分，即产生一组专门的RNN，并将其组合起来进行预测。也可以使用其他的组合方式，比如将meta-learner的输出作为RNN的输入继续进行预测，也有boosting的方法。
5. global方法，即权重全局计算（跨越不同的时间序列），但是状态保留在各自的时间序列中。

## 第三部分 Methodology
都是很简单的介绍，没什么细节
## 第四部分 测试框架
数据集：用的挺全的

在这部分中我比较关心的是时间序列预处理的方式。时间序列预处理是非常麻烦的，最优的肯定是全局预处理，但是这只对于波动不大的时间序列管用，而且对于极端情况处理的很糟糕。Smyl还是NBEATS提出了局部时间序列处理方式，即每次使用滑动窗口的最后一个值进行时间序列预处理。

### 4.2 数据预处理
#### 4.2.1 数据集切分
首先是要留足与forecast horizon等长的validation dataset。

#### 4.2.2 处理缺失值
对于缺失值可以将其替换成线性插值结果或者直接赋值为0.对于NN5的数据，使用的是中位数。比如某一天的数据缺失了，那这一天的数据就使用所有星期的同一天中对应数据的中位数来填补。

#### 4.2.3 对季节性建模
很显然的事情是，对于更长的预测范围，是必须要使用周期性数据。Claveria 2017的工作显示，对季节性进行了调整适应的数据在预测中表现地更好。

#### 4.2.5 多步输出问题
* Recursive Strategy: 每次预测一步，将前一次的预测结果作为下一次的输入
* Direct Strategy: 使用多个不同的模型，每个模型预测一个horizon中的一个时间点
* DiRec Strategy：结合上述两种方法，多模型的同时进行滚动预测
* Direct multi-step-ahead forecast：直接预测整个horizon，即MIMO
* DIRMO：将Direct和MIMO结合，每一个模型预测指定的窗口大小，并结合。
* Direct Multi-Horizon strategy，同样作为MIMO，可以避免错误累积

常规的产生监督数据的方式：滑动窗口法。

输入窗口选择方法：
1. 稍微比输出窗口大，m=1.25倍输出窗口大小
2. 令输入窗口稍微比季节性周期大，m=1.25倍季节性周期大小

#### 4.2.6 Trend Normalization
RNN所用的激活函数，包括sigmoid或者tanh都有一个区域，超出这个区域后输出都是一个常数。因此使用RNN的时候需要确认输入数据是被正则化过的。
Smyl提出了一个per window local normalization step，对使用滑动窗口法产生数据的正则化方式提出了一种比较好的局部正则化方法。