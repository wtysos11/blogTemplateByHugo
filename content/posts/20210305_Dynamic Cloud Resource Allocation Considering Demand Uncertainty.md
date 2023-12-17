---
title: Dynamic Cloud Resource Allocation Considering Demand Uncertainty
date: 2021-03-05T10:46:04+08:00
tags:
  - 分类/笔记/科研/资源调度预测
categories:
  - 计算机论文阅读
---

# Dynamic Cloud Resource Allocation Considering Demand Uncertainty

2019 TCC,CCF C类

看到C类效果这样心里还是有点底，这个用来PK应该是没问题的

## 1 

本文提出了一种混合方法来为基于云的网络应用分配云资源。结合了按需分配和预付费资源的有点，实现了混合的解决方案来最小化总部署费用的同时，满足流量变化下的QoS。

贡献可以分为以下部分：
1. 部署了动态云资源分配方法，解决了在资源预分配和动态分配两阶段的资源调度问题。开发了随机优化方法来将用户需求建模为随机变量，并实现了10%的部署代价提升。

## 2 Related work

动态资源分配分为两个阶段：
1. 第一阶段，资源在不考虑用户需求的情况下被分配。
2. 第二阶段，为了保证QoS，采用on-demand的方式分配资源。

由于是离散的，因此不能使用凸优化方法，不能保证有全局最优解。

Robust Cloud Resouce Provisioning，考虑了三个不确定性：demand、price和cloud resource availability
* 在第一阶段，预付费资源完成，将特定数量的资源分配给了应用。
* 在第二阶段，判定资源是否够用，开始采购on-demand资源

总体来说该作者列的引文都是关于stochastic programming的

## 3 System model and assumptions

### 3.1 Problem Definition

为了满足不同用户的需求，云服务提供商会提供不同配置的VM，这将作为算法的输入。

算法主要将数据库应用与一般网络应用进行区分。(database instnace and computing instance)

然后进行了一系列的数学符号定义

## 4 Dynamic Cloud Resource Allocation Algorithm

本文采用的是两阶段算法，第一阶段，使用预付费的资源来满足最低QoS的需求。
第二阶段，将non-deterministic user demand建模成随机变量，来动态分配on-demand的资源。

### 4.1 DCRA Flowchart Overview

在reservation phase，算法会决定满足最细骄傲用户需求的资源，来作为分配预付费资源的依据。

在dynamic provision phase，DCRA算法会在每一个周期内(15-min)监控用户的实际需求。

除此之外，对于云服务提供商而言，如果购买了on-demand的机器，它不会在意你是使用了一小时还是几秒钟。因此，较短的动态分配周期并不能帮助减少分配的花费，但是可以帮助减少资源不足的情况，因为它可以尽快启动机器。

在动态阶段，如果监控平均用户流量$r_{avg}$超过了最低期望流量$r_{min}$，网络服务将处于资源不足的状态，这时就应该启动资源动态分配算法来满足动态的流量需求。输入当前流量，输出期望的on-demand资源数量。

### 4.2 Cloud Resource Reservation Optimization

reserved phase，目标是确保资源在最小代价下满足QoS的最小需求。

代价函数包括了，DB实例的代价、存储代价、计算实例代价和IO的代价。

满足的约束是，实例的数量与单位实例能处理流量的乘积，要大于最小的流量估计值。

这里默认web application request arrival rate服从泊松过程分布，直接用了排队论的公式来联系到达速率、响应时间与CPU占用率之间的关系。

接下来是一些合法性约束条件，确保后续不会出现非法解

### 4.3 Dynamic Cloud Provision Optimization

我觉得它这里还是说的不错，流量本身是uncertaint nature，这使得deterministic resource allocation optimization不足以应对现实的情况。

作者引经据典，指出demand uncertainty can be modeled by a normal distribution。

作者希望使用stochastic optimization approach，考虑服从正态分布的随机过程。

最优化的目标optimization variables是on-demand的计算资源和数据库的实例数，以及它们的服务速率来满足更新的请求速率。

建模的目标式流量的期望值$r_{ave_c}$与$r_{ave_{db}}$及其方差。为了保证满足95%的置信区间，选用两倍的方差。

同样采用排队论方式建模，来建立请求率与访问时间之间的关系。

## 5 Performance Analysis

### 5.1 Experimental Setup

用户请求到达率依据排队论建模（2011,Optimal Resource Allocation for Multimedia Cloud Based on Queuing Model）

reserved instance固定使用一年的合约，on-demand资源使用小时级合约

测试流量，其中之一为RUBiS流量；另外一个是SPEC。这两个都是benchmark流量。