---
title: A two-phase cloud resource provisioning algorithm for cost optimization
date: 2021-03-04T11:03:04+08:00
tags:
  - 分类/笔记/科研/资源调度预测
categories:
  - 计算机论文阅读
---

# Profit Maximization for Cloud Brokers in Cloud Computing

CCF A类

IEEE Transactions on Parallel and Distributed Systems，2019

## 摘要

为了降低云用户的耗费，引入cloud broker（下称中间商）。中间商从云服务提供商处以reserved instance的形式租用VM，并把它们以比on-demand更低的价格与相同的付费模式租给用户。

本文关注于如何设置中间商的价格模型，为它的VM定价使得其利润最大化，在其能为用户节省成本的前提下。将最优化的多服务器配置问题和VM的定价问题建模为利润最优化问题，并使用启发式的方法来求解。近-最优解可以被用来指导配置和虚拟机的定价。

## 3 The Models

模型主要使用的是多服务器排队论模型、收入模型和花费模型

reserved instance和on demand instance的价格使用$\beta_{re}$和$\beta_{od}$来指代，单位为dollar per unit time，一般unit time指的是小时。

### 3.2 Multiserver queue system

本文所研究的中间商broker只从单一的云服务提供商处租用资源，并把它们提供给用户。因此，中间商提供的VM是同构的(homogeneous)。资源在CPU、内存、带宽等方面是一致的。本文假设用户使用了M/M/n/n的排队系统，来对其流量等进行建模。

在M/M/n/n排队系统中，VM的到达流量被认为是一个速率$\lambda$的泊松流，到达时间独立同分布且呈指数分布。考虑到中间商使用价格来吸引用户，因此实际速率$lambda$会受到两个因素的影响，即实际用户需求$\lambda_{max}$与资源价格。

通过租用VM并搭建私有云的方式，中间商可以向用户提供on-demand的产品。现假设中间商所拥有的虚拟机数量为n，则多队列系统的队列长度不超过n，此时可以根据排队论公式计算出平均服务时间与资源占用率。

得到等式1，描述$\pi_k$的式子，这个变量为在排队论系统中有k个服务请求的概率。

显然，如果请求得不到满足，用户就会流失。因此流失概率等于系统中有n个请求的概率，即$P_L = \pi_n$

### 3.3 Cost Modeling

即购买服务器的钱，显然是$C=n\beta_{re}$

### 3.4 Revenue modeling

#### 3.4.1 Analysis on the Revenue-affecting Factors

有两个主要影响盈利的因素，一个是用户的需求，即请求到达率$\lambda$。在价格不变的情况下，请求到达率越高，收入也就越高。因此提升用户需求是获取更高收入的主要手段。

但是，用户的需求是随着VM的销售价格而变化的，因此第二个影响因素就是VM的销售价格。

价格从两个方面影响收入。首先，价格直接影响收入，在给定需求下，高价格导向高收入。其次，低价格是该盈利模式的核心竞争力，提升价格将导致客户流失。

考虑到收入与成本，销售VM的价格只能在$\beta_{od}$与$\beta{re}$区间内。

考虑关系，价格越低，用户流量$\lambda_{max}$越高，可以构建出price-demand function。采用线性建模是因为这样最常见。

## 4 

没看明白