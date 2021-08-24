---
title: "Business-Driven Long-Term Capacity Planning for SaaS Applications"
date: 2021-03-01T16:57:04+08:00
tags: ["云资源调度"]
categories: ["计算机论文阅读"]
---

未完成

本次阅读的关注重点是，如何根据已经给定的流量需求来分析所需要的满足SLA的实例数量？
是否可以用强化学习来进行？ 

# Business-Driven Long-Term Capacity Planning for SaaS Applications

2015 TCC ，没记错应该是C类

## 摘要

本文关注的是capacity planning，从定义上，该技术的目标是估计提供计算资源所需的资源数量，从而实现高的QoS级别，为公司带来更高的经济效益。

在现实中，可以有这样的场景，SaaS出于经济效益的考虑去购买IaaS服务商的实例。这样，SaaS可以减少在操作上的花费与复杂度，但是需要对自身的长期资源使用情况进行一定程度的估算。

本文采用了模拟实验，使用同步的电子商务数据流。分析显示，使用启发式方法来优化能够每年提升9.65%的利润。

重点在于启发式搜索的方式

## 3 Utility model

Utility是微观经济学上的概念，用来描述客户的偏好。一般而言，更大的值代表了更高的偏好性。因此，客户的行为也会受到Utility的影响，即他们会倾向选择最喜欢的输出。

Utility function将outcome映射到utility value上。

本文提出的Utility model将SaaS的利润（作为提供一个应用的结果）映射到utility value上。这样，一个capacity planning的agent就可以使用本模型来制作capacity plan来最大化utility value。此时，就可以达到SaaS provider的最大利润。

### 3.1 Revenue model

utility model认为SaaS provider可以提供一个或多个计划给他们的顾客，每一个顾客根据自身的需要选择一个计划并与SaaS provider签订合同。

revenue model包括：
1. SaaS consumer周期性地收取费用（每月或每年）
2. 每一个application有着使用限制，由provider提供
3. 合同包括赔偿条款，即SLA违约的情况

SaaS将提供应用A给一个SaaS顾客的集合$U={u_1,...,u_{|U|}}$。同时，SaaS provider会构建一个计划的集合$P={p_1,...,p_{|P|}}$，每一个计划$p_j$会满足一类顾客的需求，因此期望上$|P|<|U|$，每一个顾客会选择一个计划来使用应用A。

在签订计划后，顾客$u_k$可以在时间$[n_k^b,n_k^e]$区间内使用应用A，比如如果$p_j$是半年计划，这两个时间点的差值就是六个月。简单期间，SaaS提供的所有计划都以一个月作为最小单位。并且，新的顾客只能在每一个周期$n$到达之后才能加入，n随着时间推移单调递增。

顾客$u_k$签订合同之后，SaaS provider就必须配置并部署应用A来服务。之后，顾客$u_k$需要支付配置费用configuration fee$I_j^b$，该费用由计划$p_j$决定。

----

后续太罗嗦了就省了。本文的核心模型是utility model，就是一个收益模型，利润=总收入-总支出，基本没用。

核心算法有两个

一个是utilization model，这个utilization指的是reserved instance的利用率，比如说一个reserved instance买一年，需要有效使用50%才能比单纯买on-demand便宜，这个50%就是utilization。

然后这个算法本身的输入是包含了一个trace，就是说实例是预先分配好的，后续的工作只是在不同的云服务提供商中通过启发式方法比较出最优的那一家。就……挺不相关的。