---
title: Keras中的Embedding层是如何工作的
date: 2020-09-08T20:57:04+08:00
tags:
  - 分类/知识库/计算机/问题
  - 分类/知识库/计算机/深度学习/Keras
categories:
  - 问题
---

在学习的过程中遇到了这个问题，同时也看到了[SO](https://stats.stackexchange.com/questions/270546/how-does-keras-embedding-layer-work)中有相同的问题。而keras-[github](https://github.com/keras-team/keras/issues/3110)中这个问题也挺有意思的，记录一下。

这个解释很不错，假如现在有这么两句话
* Hope to see you soon
* Nice to see you again

在神经网络中，我们将这个作为输入，一般就会将每个单词用一个正整数代替，这样，上面的两句话在输入中是这样的
```
[0, 1, 2, 3, 4]

[5, 1, 2, 3, 6]
```
在神经网络中，第一层是
```python
Embedding(7, 2, input_length=5)
```
其中，第一个参数是input_dim，上面的值是7，代表的是单词表的长度；第二个参数是output_dim，上面的值是2，代表输出后向量长度为2；第三个参数是input_length，上面的值是5，代表输入序列的长度。

一旦神经网络被训练了，Embedding层就会被赋予一个权重，计算出来的结果如下：
```
+------------+------------+
|   index    |  Embedding |
+------------+------------+
|     0      | [1.2, 3.1] |
|     1      | [0.1, 4.2] |
|     2      | [1.0, 3.1] |
|     3      | [0.3, 2.1] |
|     4      | [2.2, 1.4] |
|     5      | [0.7, 1.7] |
|     6      | [4.1, 2.0] |
+------------+------------+
```

根据这个权重，第二个输入计算出来的embedding vector就是下面这个：
```
[[0.7, 1.7], [0.1, 4.2], [1.0, 3.1], [0.3, 2.1], [4.1, 2.0]]
```

原理上，从keras的那个[issue](https://github.com/keras-team/keras/issues/3110)可以看到，在执行过程中实际上是查表，将输入的整数作为index，去检索矩阵的对应行，并将值取出。至于这个embedding matrix是怎么维护的我还没有搞明白。