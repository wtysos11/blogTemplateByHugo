---
title: Response Time Characterization of Microservice-Based Systems
date: 2020-04-23T17:21:04+08:00
tags:
  - 计算机/微服务调度
categories:
  - 计算机论文阅读
modified: 2024-01-21
---

[原文地址](https://github.com/wtysos11/blogWiki/issues/23)

看到的挺好的一篇文章
November 2018
DOI: 10.1109/NCA.2018.8548062
Conference: 2018 IEEE 17th International Symposium on Network Computing and Applications (NCA)
ConferenceIEEE International Symposium on Network Computing and Applications

以排队论分析响应时间的一篇论文。
说实话，我一没看懂排队论的过程，二没看懂他的实验。
师兄的说法是排队论依据的假设太多了，比如要求请求以泊松分布到来之类的，现实世界根本做不到，以此建模误差太大了，就到此为止吧。

# Response Time Characterization of Microservice-Based Systems
涉及到的词语与翻译：
* bottleneck，翻译为瓶颈、瓶颈层、瓶颈点
* multi-server，我认为这里指的是多层服务，和multi-tier应该是同义的。
## 摘要
背景：微服务架构较之传统的单体应用有着很多的优势，但是它也阻碍了系统的可视化(hinders system observability)。特别地，对于服务性能的监控和分析变得更加有挑战，特别是对于那些重要的生产系统，必须要迭代增长、连续操作且不能够进行线上的基准测试(benchmark)。这些系统一般非常的巨大且昂贵，因此成为了完全调度的较差选择。
为这样的服务和系统来创建一个模型来进行特征分析可以很好地缓解上述的问题。性能，特别是响应时间，是本工作关注的重点，我们注重于瓶颈点检测(bottleneck detection)和资源最优化调度（optimal rsource scheduling）。我们采用了一个方法来对生产应用建模，使用请求数据的排队系统。除此之外，我们提供了对响应时间进行分析和资源最优化调度的分析工具。我们的结果显示一个有着单个队列和数个同构(homogeneous)的服务器的简单的排队系统有着一个较小的参数空间，可以在生产中被估算出来。这个结果的模型可以被用来预测响应时间的分布和必要的实例数来维护期望的服务级别，在给定的流量下。

## 1 introduction
对于生产环境来说，黑盒监控是非常轻量且高效的，然而这样的监控是无法去分辨和预测服务质量。除了已经收集到的指标、警示和配置项目之外，分析任务主要依赖于管理员。为此，我们设计了一个建模组件，比如同构多层服务队列，这使得可以从统计学上去分析一些性能指标，比如延迟或流量。而且由于队列可以组成网络，该方法可以用于微服务架构中的建模。为了更加准确地对独立的服务进行建模，我们需要对它们的性能进行更细致地分析。
在得到了额外的记录信息之后，我们就能够去为微服务系统来创建和更新一个动态的依赖模型。这允许我们去提取服务端点之间的依存关系，且更重要的是，可以对每一个微服务的表现进行建模。目标是简单但重要的：取得使每一个服务表现最佳的动作。这会导向不同的分支，比如明白什么时候应该进行调度、减少基础设施的消耗，以及保证SLA。额外地，可以在瓶颈出现前去解决它(the possibility of pinpointing bottlenecks in the system without stressing it)，这也是一个主要的优点。
在本文中，我们开发并部署了一个基于微服务架构的系统，对它的记录结果进行排序，来对multi-server queues进行建模(M/M/c)。该方法可以预测响应时间的分布范围以及服务的最佳动作区域，同时决定需要多少个实例数来维护指定流量下服务的理想服务质量。更重要的是，建立一个模型使得方法可以建立一个最大容量的定义，这是全系统性能最优化和瓶颈检测的第一步。我们的结果证明了尽管十分简单，我们的模型能够准确地预测微服务的动作，更精确地来说，预测响应时间分布，同时不需要更加复杂的模型或参数。从结果上来说，这样的模型对于线上的分析是足够的。
剩下的部分如此组成：S2描述了我们用来对微服务建模的基于队列的模型；S3描述了实验设置；S4展示了我们的实验结果；S5评价了实验结果；S6列出了相关工作；S7对该论文进行了总结。

## 2 Performance Modeling
为了分析系统的性能并对其进行预测，我们需要一个合适的模型。对瓶颈点(bottleneck)的检测需要服务的容量上限（处理能力上限）的定义，作为一种在执行上对两个服务进行比较的方法，然后取预测服务在资源变动后响应时间的变化。在基于微服务的系统中，资源调度一般是实例级别的(instance level)，单位是虚拟机或容器。如果上述要求得到满足，就可以通过分析决定哪一个微服务不够有效（分配过多资源）或超出容量（超出处理能力上限）。
我们对于模型的要求是其能表示响应时间、流量和并行性，我们计划在在线建模中使用它。排队论系统(queueing system)是我们的理想方案，能够描述我们想要的方面和指标，且能够组成排队网络（queueing networks）。可以证明，如果有大量顾客独立地对一个服务发送请求，这个到达过程是马尔科夫的Markovian(论文：Enhancing Elasticity of SaaS Applications using Queuing Theory)。因此我们可以使用M/M/c队列对服务进行建模。
我们的参数空间比较小且符合直觉的，有lambda,miu和c三个参数组成。其中，lambda和miu代表着呈指数分布的请求到来时间和服务时间，c是同构的服务器的数量，代表着服务的并行性。(lambda and miu represent the rate parameters for the exponentially distributed inter-arrival and service times, and c the number of homogeneous servers, representing service parallelism.)需要注意的是，c并不是必须要表示实例的数量，在某些情况下，服务内部也会存在着并行性。
![表1-Notoin](https://user-images.githubusercontent.com/21279827/80081261-4ac83200-8585-11ea-8a79-5424bb89bbfb.png)
表1表示了本文所用的符号表，本文所构建的模型有两个直接的应用场景：1. 评估应用能处理的最大流量(estimate the maximum throughput capacity of an instance, bottleneck detection) 2.决定实例数量，来对于给定的流量到达速度达到一个理想的服务质量/响应时间分布。

给定M/M/c模型的参数miu'和c'，我们需要问三个问题：
1. 这个服务能处理的最大流量是多少？
2. 对于给定的流量lambda，整个系统的时间分布是什么(what is the distribution of total system time T)
3. 对于给定的流量lambda，使得请求的加权响应时间低于系统时间阈值所需的必要的实例数量是多少？
对于第一个问题，对给定的M/M/c队列中获取最大容量C_miu是有一套比较简单的估算方式。另外两个问题可以使用累计分布函数CDF P(T\<t)在给定lambda、miu和c的时候决定，其中T是一个随机变量，代表整体系统时间。因为CDF于上述式子相关，我们有时候会使用这样的表示：CDF(lambda,mu,c,t) = P(T\<t)。由Adan的论文(I. Adan and J. Resing, “Queueing theory,” 2015)，T可以由W和S计算，这两个都是随机变量，W表示等待时间、S表示服务时间。见式(1)
![式1](https://user-images.githubusercontent.com/21279827/80081249-4734ab00-8585-11ea-87fa-9b8fab15ac82.png)
在这里有一个自然的应用，是去计算一个期望的响应时间分位点(expected percentage of requests)，在total system time比given threshold r稍微大一点的时候，方式是通过1-P(T<r)预测负载情况。
对于最后一个问题，我们希望能够决定最小的整体实例数I来确保给定的整体系统时间的分布，其中超出给定阈值的概率为p，即1-P(T<r)<=p

### 参数估计parameter estimation
为了对给定的微服务建模，我们需要去估计(estimate)两个参数，服务的速率miu'和它的并行性c'，从给定的一个请求的到达样本与离开时间中进行估计。
对于请求的到达速率lambda'可以通过moment or maximum-likelihood methods估计出来。对于服务速率miu'的估计，我们需要去确定请求的样本l被采样的时候队列要是空的这使得对于所有样本l中的请求，它们在队列中消耗的时间都是0。寻找一个充足的间隔可能开始会显得比较吓人，但一旦我们能估算出c'的值，我们就可以在间隔中寻找一个连续的样本集合S，在样本集合中there are at most c' request in the system.
另外有一个启发式的方法，可以与c'独立，即运行一个滑动平均窗口，窗口的值为一个统计学重要的宽度，然后找到一个连续的子空间，其平均数最小。
...很多没有看懂。
只要我们能够有一个方法来估算服务的速率miu'，以及样本的请求时间，我们就可以计算出c'来最大化观测到的请求时间分布与预测的CDF之间的MSE。

## 3 Experiment setup
我们会使用一个模拟仿真和一个真实的微服务来验证模型。
模拟仿真被用来验证参数确认方法，在该情况下最佳结果是已知的。第二个实验，在真实环境下进行，验证参数设置在真实环境中的效果。