---
title: "A two-phase cloud resource provisioning algorithm for cost optimization"
date: 2021-03-01T19:49:04+08:00
tags: ["云资源调度"]
categories: ["计算机论文阅读"]
---

# A two-phase cloud resource provisioning algorithm for cost optimization

## 1 Introduction

主要的三种付费方式：
* On-demand pricing model
* Reserved pricing model
* spot pricing model

现有的模型大多都只考虑了on-demand的收费方式，有一些考虑了on-demand和reserved的收费方式。但是它们的目标都只是使用reserved收费的资源来满足最低服务需求，并用on-demand的资源来满足剩余需求。

本文的目标是找到最佳的实例数，即resource provisioning problem。为了降低资源的租用成本，本文使用了on-demand和reserved instance两种方式来进行一个两阶段的资源分配，并决定最佳的reserved instance的实例数来最小化花费。为此，需要基于预测的流量信息来满足SLA的要求。

主要贡献：
1. 使用on-demand和reserved instance的两阶段资源部署算法来减少资源租用花费
2. 在第一阶段，将资源配置问题建模为two-stage stochastic programming problem，并用sample average approximation的方式和dual decomposition method的方式来求解
3. 在第二阶段，使用ARIMA-Kalman model来预测流量，并决定on-demand的实例数量。
4. 使用现实世界的流量和Amazon EC2购买模型来验证结果。

## 2 related work

主要提到了云资源分配问题可以被建模为stochastic programming problem，然后使用branch and bound and cutting plane method求解，或者使用启发式方法，比如genetic algorithm, particle swarm optimization, hybrid algorithm等。一般使用PSO来探索解空间，使用GA来更新结果。

## 3 Problem Formulation

本节主要内容为模型的假设，包括VM的配置与价格模型。基于这些假设，我们可以对云资源分配问题进行建模。

### 3.1 云环境

云资源提供商提供多种不同的VM给用户，$V={V_1,...,V_M}$代表了不同种类的虚拟机，M为虚拟机的种类。每一个VM的种类有着自己的资源配置和处理容量。令$C_i$表示$V_i$类虚拟机在不违反QoS的情况下所能处理的最大并发用户数或者最大服务请求速率。

我们采用小时计费的方式，并考虑两种不同的模型：on-demand实例和instance实例。令$p_i^o$表示类型$V_i$的on-demand费用，令$p_i^R$与$p_i^r$分别代表reserved instance的总支付费用与每小时费用。令$T$代表reserved instance的使用时间。这样，虚拟机类型$V_i$的实际成本为$p_i^R/T+p_i^r$，即平均每小时的总费用加上每小时的实际费用。一般来说，reserved instance的每小时费用都要低于on-demand的付费。

### 3.2 Cloud Resource Provisioning Problem

在一个reservation period中考虑这个cloud resource provision problem，令t=1,2,...,T为该reservation period中的hour index，令$d_t$为时间t时的workload，$R=(n_1^r,n_2^r,...,n_M^r)$表示reservation decision，其中$n_i^r$为$V_i$类虚拟机的数量。这样，就可以计算出reserved实例的processing capacity $\sum_{i=1}^{M}n_i^r C_i$，以及总费用。

对于每一个时间点t，如果reserved instance能够满足需求，显然也没有再购买on-demand资源的必要

因此，on demand的使用费用可以写为$U(R,d_t) = min \sum_{i-1}^M n_{ti}^o p)i^o$，其中$n_{ti}^o$就是类型$V_i$在时间t所使用的on-demand的实例数

此时，resource reservation problem可以建模为最小化reserved instance费用与on-demand instance的费用。显然这个问题是依赖于reservation阶段的流量的，这个在现在来说是不知道的，但我们可以基于历史数据来预测流量的分布情况$p_D(d)$。

## 4 Resource Reservation

### 4.1 Sample Average Approximation(SAA)

如果scenarios的数量比较大的话，直接求解前面的最优化式子是非常困难的。sample average approximation就是用来减少scenarios数量的方法。考虑到workload是一个一维的随机变量，就可以使用uniform discretization grid来产生一个scenarios的集合${\tilde{d_1},\tilde{d_2},...,\tilde{d_N}}$，其中$N$是采样大小。用采样结果来拟合复杂的概率结果，化简two-stage stochastic problem。

### 4.2 Dual Decomposition-Based Branch and Bound (DDBnB)

scenario decomposition是引入copy $R_j$作为每个scenario中第一阶段决策动作$R$，然后重新建模原问题。