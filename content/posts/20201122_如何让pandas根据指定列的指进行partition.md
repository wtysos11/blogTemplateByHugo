---
title: "如何让pandas根据指定列的指进行partition"
date: 2020-11-22T19:20:04+08:00
tags: ["问题解答","Python"]
categories: ["问题"]
---

## 问题描述

我拿到了一个维基百科的列表，其数据如下：

datehour | title | views
-- | -- | --
2015-10-17 13:00:00 UTC | Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License | 2
2015-06-01 14:00:00 UTC | Dulce_Mar铆a | 10
2015-06-01 21:00:00 UTC | Dulce_Mar铆a | 25
2015-06-01 06:00:00 UTC | Dulce_Mar铆a | 18
2015-08-30 12:00:00 UTC | Portal:Current_events | 116

UTF-8的问题暂且不谈，现在需要将其作为csv文件读入内存中，并且按照title分成不同的datehour->views表，并按照datehour排序。将2015~2020的数据按照同样的操作进行处理，并将它们拼接成一张大表，最后将每一个title对应的表导出到csv，title写入到index.txt中。

##解决方案
### 朴素想法
最朴素的想法就是遍历一遍原表的所有行，构建一个字典，字典的每个key是title，value是两个list。不断将原有数据放入其中，然后到时候直接遍历keys，根据两个list构建pd，排序后导出。

### 更python的做法
朴素想法应该是够用的，但是不美观，不够pythonic，看着很别扭。于是我搜索了`How to partition DataFrame by column value in pandas?`

#### boolean index
[stackoverflow](https://stackoverflow.com/questions/33742588/pandas-split-dataframe-by-column-value)里有人提问如何将离散数据进行二分类，把小于和大于某个值的数据分到两个DataFrame中。直接用`df1 = df[df["Sales"]>=s]`这样的语句就可以完成。
但是这在我们的场景上并不太适用。当然，可以提前遍历一遍把title做成集合再循环遍历，不过这也不是很pythonic。

#### groupby
同样是上面那个问题，有人提到可以使用groupby方法。[groupby](https://www.yiibai.com/pandas/python_pandas_groupby.html)听着就很满足我的需求，它让我想起了SQL里面的同名功能。
* `df.groupby('ColumnName').groups`可以显示所有的列中的元素。
* `df.groupby('ColumnName')`可以进行遍历，结果是一个(name,subDF)的二元组，name为分组的元素名称，subDF为分组后的DataFrame
* 对`df.groupby('ColumnName')`产生的对象执行`get_group(keyvalue)`可以选择一个组

此外还有聚合、转换、过滤等操作，不赘述。
