---
title: "bottleneck detection"
date: 2020-04-22T17:36:04+08:00
tags: ["微服务调度","论文总结"]
categories: ["计算机论文阅读"]
---

[原文地址](https://github.com/wtysos11/blogWiki/issues/22)
> 主要还是针对微服务调度中bottleneck的定义入手

## 瓶颈点定义
从师兄的说法上，瓶颈点指的是响应时间上的瓶颈点，即在该实例下响应时间是异常的，可以通过增加瓶颈点的实例来减少整体的响应时间。
在多层应用(multi-tier applications)中，其中的bottleneck指的是资源层面的，即在某一层增加资源能够最大化的优化服务的性能。

### 多层服务中的bottleneck
#### Agile Dynamic Provisioning of Multi-Tier Internet Applications
##### Chapter 1
师兄推荐的论文。使用了排队论方法来明确每一层应该分配多少资源。
定义：每一层的处理能力是固定的 x req/s，如果说中间有一层处理能力最低，显然整个服务的处理速度都将受到影响。
需要注意的是，对瓶颈层的增加并不代表对整个服务的服务质量都会增加，可能需要在消除所有的瓶颈层之后整个服务的服务性能才能得到提升。

  因为现在已经有很多的单层调度机制被使用了，要给很简单的想法就是给每一层都配置一个单层调度器。因此，我们的第一步就是在每一层的流量超过容纳上限的时候增加实例数量，这个可以通过监控队列长度、层间响应时间、或者丢包率来实现，这个方法称为independent per-tier provisioning.
* 想法1：多层模型，逐个找到bottleneck并增加。问题在于增加的速度可能很慢，最多可能需要遍历所有层才能完成实例扩缩，这对于变化较快的网络流量显然是不行的。
* 想法2：直接将多层模型作为一个黑箱进行考虑。问题在于需要增加多少个实例，以及在哪里增加。因为多层模型是一个黑箱，显然调度器无法获取内部的信息，即不知道究竟是哪一层出了问题。对于单层服务模型，可以建立一个应用模型来决定在响应时间范围内对于给定的流量需要多少个服务，从而进行对应的扩缩。但是将这个模型用于多层模型中并不一定可行，因为每一层的服务与应用性质都是不同的。以简单的电子商务应用为例，这意味着将HTTP服务器、Java服务器和数据库系统一起建模，这将是一个困难的工作。第三，不是所有的层都可以增加实例，比如数据库系统就很难实时增加实例。
  综上，不能将多层应用作为黑箱进行调度。

特色：
* predictive and reactive provisioning:使用预测式调度来进行小时或天级别、使用响应式调度来进行更细致的调度。
* 使用了基于排队论的分析模型来同时对多层网络应用进行分析
* 快速的服务器切换，允许虚拟机来处理更快发生的网络波动。
* 能够处理基于session的流量。

这个调度方法可以被归类为自适应adaptive或半自动的semi-autonomous，它们是能够快速适应环境同时只需要人类有限的接触。

##### Chapter2 System Overview
2.1 Multi-tier Internet Applications
多层网络架构：整个网络架构由顺序连接的多个应用层组成，如同管道一样，中间的一层会收到前面预处理的数据，并将自己处理好的数据传输给下一层。
本文定义的SLA，使用平均响应时间(average response time)或一个合适的响应时间分位点（a suitable high percentile of the response time distribution）

##### Chapter3 Provisioning Algorithm Overview
调度算法的目标：allocate sufficient capacity to the tiers of an application so that its SLA can be met even in the presence of the peak workload.（关键在于上界）
主要问题：how much and when 调度多少，什么时候调度
How much to provision:使用GG1排队论系统取分析。
When to provision：依据系统的流量。网络流量存在长期的变化，比如天或季节级别的影响，而短期的变化往往难以预测。我们的

How much to provision: modeling multi-tier applications
应用有K层，代表为T1,T2,...,Tk。让期望的end to end response time 为R，这个是SLA的规定值。可以预见，这个SLA将由每一层的响应时间组成，即R = d1+d2+...+dk。假设到来的session的访问率为lambda，因为调度是基于最坏(最小的)情况进行的，假定lambda是访问率的分布中的一个较高的分位点，是对peak session的一个估计值。
给定一个peak session rate和per-tier response time，目标是决定需要分配多少服务器，使得每个层的响应时间达到di的平均响应时间。
使用排队论模型，每个服务器都有一个队列，第一步是决定每一层能够处理的请求率和容量。给定服务器的容量，下一步是去计算需要多少台服务器才能够满足最大峰值的需要。使用G/G/1排队系统来进行建模，在G/G/1系统中，request到达服务器的间隔时间是服从一个已知的一个分布，每一个reuqest给服务器带来的工作量是相同的，队列允许长度为无限。一个G/G/1系统可以使用请求的平均响应时间和流量来代表inter-arrival和service time distribution。


#### Bottleneck Detection and Solution Recommendation for Cloud-Based Multi-Tier Application
对多层应用来说，检测瓶颈点（bottleneck points）是很重要且有挑战性的问题。因此，需要有一个机制来监视应用的表现变化，同时联系系统资源的使用情况来进行系统层面的分析。
然而，自动标记和联系bottlenecked resources并不是容易完成的事情。我们需要注意到的是，多个潜在的bottleneck可以同时存在，相互影响

##### Bottleneck Detection Using Knee Point Detection
The bottleneck pattern of the application throughput can be described as a knee point of throughput curve, while the workload to the application increases. 瓶颈点指随着流量的增加，处理能力将达到一个上限，在这个上限之后单位时间内系统处理的请求数不会增加，这样会导致请求排队、响应时间增加。
因此在knee point之后，throughput不能再增加，因为资源已经到达bottleneck了。同时，一层中相同的bottleneck会最终扩散到其他层中，因为相同的bottleneck模式会触发其他层的bottleneck模式（可以见前文的例子）。
首先在观察之后，很重要的一点是需要捕捉到knee point，即明确一层所能处理的请求的上界。然后，我们需要确定造成bottleneck的原因，在系统资源的情况下，通过分析不同层和资源的bottleneck pattern之间的时间关系来找出原因。

2.1 Individual Knee Point Detection
应用的流量在到达knee point前可以持续增加，然而因为系统的bottleneck出现，它就不能够继续增加了。
使用数学方法，通过分析曲线来计算出拐点的位置。

2.2 Performance Profiling for Identifying the System bottlenecks
在本方法中，使用资源使用率的变化曲线和流量的变化曲线来识别bottleneck。
原理：在流量到达knee point前资源使用率的变化速率可以去代表每一种资源在面对流量变化时的使用情况与性能表现的变化，从而识别出bottleneck的位置。


### 其他想法

想法二：陈鹏飞老师学生所做的Microscaler: Automatic Scaling for Microservices with an Online Learning Approach这篇文章中提到了一个现象，即对于固定访问的流量，其分布是近似固定的。在服务异常的时候，这种响应时间分布是否会发生改变？是否能以此作为异常？

## 总结

瓶颈点指的是什么：bottleneck，更多的是从资源层面上定义的。以第二篇文章为例，这里的bottleneck指的是资源到达瓶颈。其理论是同一层使用的处理能力是有上界的(knee point)，在资源使用率到达knee point之后就称这个资源到达了bottleneck，此时无论如何增加流量，都不能够使得服务处理更多的请求。（如果没有熔断器，反而可能会影响到服务的正常使用）
如何识别瓶颈点：
* multi-tier应用可以使用排队论建模