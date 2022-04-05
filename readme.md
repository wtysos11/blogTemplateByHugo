# 一些思考

对于博客而言，最重要的是内容。对于内容而言，最麻烦的是组织。

在我十几年前开始写我的第一篇博客的时候，我就遇到了这个问题：我该基于什么原则来划分我的内容？早年的时候是QQ空间，它只支持按照某一个选定的类别进行划分。我在使用后不久就发现，有些内容注定是跨分类的，它们没有办法简单地被某一个分类约束。到初中以后，我开始在csdn上撰写我的内容，这时候我又遇到了一个问题：CSDN上的内容是按标签进行区分的，这使得整个博客看起来很散乱。

Hugo同时具备标签（tags）和目录（categories），就目前来说，我希望目录更多是以专题的形式呈现，就像是集合中的划分。而标签则类似于提供给读者的快速搜索，对于某种内容一定会存在对应的标签，从而帮助他们迅速找到内容。

## 目录

目前的目录（categories）：

### 内容集中

这部分的文章只是内容上的集中，并没有结构上的安排

#### 科研
 * 计算机论文阅读
 * 科研实验

#### 计算机基础
 * 计算机基础：与课程相关的内容，以及与基础相关的内容
 * 计算机课程学习笔记：对一些在线课程学习的记录
 * 面试相关
 * 开源代码学习笔记（同时也是代码学习笔记，不只是开源）：对某些开源库或者开源代码进行学习的笔记。可能不够写成专门的专题，但需要归结一下。（in a word, learn something)
 #### 其他
 * 操作实践
     * 操作实践更注重可行性而不是重复性。
 * 事件记录：遇到了某件事情，对其进行记录。可能无法重现，或者文章中的内容不足以重现，谁知道呢（一般是对某个bug的修复总结等）
 * codebase：有部分可能需要重复的操作，需要进行留存
     * 与操作实践相比，则部分操作可能经常要用到，更注重重复性。
 * 问题
 * 资源整理：网络上的资源整理
 * 快速复习：写作的目的是为了将来某天能用上，所以进行了大量的删减和优化来帮助之后的自己快速复习掌握。
     * 要求：必须配上对应的最小可重现示例，最好包括对应环境的配置信息。
 * 英语学习
 * 无关随笔

### 专题类型

 * 算法（专题，与面试相关部分重合）
 * 博客
#### 编程相关
 * Go基础学习：在使用Go的过程中遇到了很多基础上的问题，算是入门之后对某些东西掌握不太充分后留下来的问题。
 * Java基础学习
 * Git学习：Git是非常重要的工具，但是一直以来我对其的了解都不够充分 
## 标签

tags

### 状态类
* 待完善：文章还需要继续改进，完成后该标签将被取消

### 应用类
* 考研
* 学习方法
* 学习笔记：在学习某件事情后记录下来的东西，一般涉及到外部链接等
    * 网络文章笔记：对某个专栏或者某篇文章进行专门学习后留下的笔记
    * 网络视频笔记
    * 计算机书籍阅读笔记
* 实验报告：可以用来指导复现的文章
* 问题解答：从某个问题出发进行的实验
* 问题延申：从SO或者其他博客上的问题解答进行的进一步延申与探索

### 学术类
* 论文总结
* 综述文章

#### 科研方向
* 微服务调度
* 云资源调度
* 时间序列预测
* RNN
* Transformer
### 技术类型
* 后端
* 深度学习
* 机器学习
* 强化学习

### 计算机基础
* 软件测试
* 数据结构与算法
* 操作系统
    * linux内核
    * linux网络
* 计算机网络
* 计算机组成
* 分布式系统
* 软件工程
    * 开发模式
### 编程语言类
* Go
* Java
* Python

### 算法
* 算法题解：OJ等的题解，要求必须包含解题思路+代码，详细的还要画图
* 算法学习：对部分算法的学习，可以包含例题
* 算法思考：可以不给出具体代码，只包含思考过程

#### 具体算法与数据结构
* 算法_字符串：字符串相关的题目
##### 数据结构
* 红黑树
* B树
* 图
##### 经典算法
* 排序算法
* KMP
* 并查集
##### 具体思想
* 动态规划
* 搜索
* 分治法、贪心法
### 工具与框架
#### IDE等工具
* vscode
* git
* linux命令
#### 云相关
* vagrant
* kubernetes
* docker
* istio
#### 其他工具
* hugo

#### 深度学习
* tensorflow
* keras
* pytorch

### 其他类

* 工具记录
* 测评
* 安卓软件
* 心理学


## 自动工具

目前配置了travis，自动上传algolia的index.json，并将hugo产生的页面上传到gitpage仓库下。