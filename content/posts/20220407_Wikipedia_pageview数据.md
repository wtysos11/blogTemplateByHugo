---
title: Wikipedia pageview数据获取(bigquery)
date: 2022-04-07T16:41:07+08:00
tags:
  - 分类/学习/学习笔记/笔记/科研/数据集
  - 分类/学习/学习笔记/笔记/科研/时间序列预测
categories:
  - 操作实践
summary: 维基百科pageview数据集原始数据获取，可以从bigquery上获得2015年以来的小时级访问数据
---


## pageview数据介绍

[维基百科pageview数据](https://wikitech.wikimedia.org/wiki/Analytics/Data_Lake/Traffic/Pageviews)是Wikimedia技术团队所维护的访问量数据集。该数据集自2015年五月启用，其具体的pageview[定义](https://meta.wikimedia.org/wiki/Research:Page_view)为对某个网页内容的请求，会对爬虫和人类的访问量进行区分，粒度为小时级别，如下图：

![维基百科数据](/assets/20220407_Wikipedia_pageview数据/data.png)

## bigquery介绍

维基百科数据可以通过其API获取。但是API只能拿到每个页面天级别的数据或者全部页面小时级的数据，如果需要获取每个页面小时级的数据，则需要通过其原始数据文件进行分析。但是这部分文件的数量实在是太多了，因此使用bigquery是一个不错的选择。

### bigquery请求

可以使用SQL命令对其进行请求。由于数据在bigquery中使用分区表的形式存放，因此每次请求一年的数据。
以下代码以2015年的数据请求为例：

WARNING：Bigquery并不是免费的，每次请求可能需要消耗十几个GB的额度，请注意！

1. 获取全部数据

```sql
SELECT wiki,datehour,SUM(views) as totalViews FROM `bigquery-public-data.wikipedia.pageviews_2015` WHERE datehour BETWEEN "2015-01-01" AND "2016-01-01" 
GROUP BY datehour,wiki;
```

2. 获取各个语言版本维基的首页数据。这个是因为大部分维基百科的页面数量都非常小

```sql
SELECT * FROM `bigquery-public-data.wikipedia.pageviews_2020`
 WHERE datehour BETWEEN "2020-01-01" AND "2021-01-01" AND ( (wiki='en' AND (title='Main_Page' OR title='Special:Search')) 
      OR (wiki='en.m' AND (title='Main_Page' OR title='Special:Search'))
      OR (wiki='zh' AND (title='Wikipedia:首页' OR title='Special:搜索'))
      OR (wiki='de' AND (title='Wikipedia:Hauptseite' OR title='Spezial:Suche'))
      OR (wiki='fr' AND (title='Wikipédia:Accueil_principal' OR title='Spécial:Recherche'))
      OR (wiki='ja' AND (title='メインページ' OR title='特別:検索'))
      OR (wiki='ru' AND (title='Заглавная_страница' OR title='Служебная:Поиск'))
      OR (wiki='ar' AND (title='الصفحة_الرئيسية' OR title='خاص:بحث'))
      OR (wiki='ca' AND (title='Portada' OR title='Especial:Cerca'))
      OR (wiki='it' AND (title='Pagina_principale' OR title='Speciale:Ricerca'))
      );
```

3. top100en_Leakage：英文维基百科2015年访问量最大的前100个页面数据，但是写错了，最后变成了访问量大于100的页面。

```sql
SELECT title FROM (
SELECT title,AVG(views) AS perviews FROM `bigquery-public-data.wikipedia.pageviews_2015` WHERE datehour BETWEEN "2015-07-12" AND "2015-07-13" AND wiki='en' 
GROUP BY title)
WHERE perviews>100;
```

4. top100en：英文维基百科2015年访问量最大的前100个页面数据。

```sql
SELECT wiki.datehour,wiki.title,wiki.views FROM (
  SELECT innerViewer.title,innerViewer.perviews FROM(
    SELECT title,AVG(views) AS perviews,COUNT(*) as viewCount FROM `bigquery-public-data.wikipedia.pageviews_2015` WHERE datehour BETWEEN "2015-01-01" AND "2016-01-01" AND wiki='en' 
    GROUP BY title ORDER BY perviews) innerViewer
   WHERE innerViewer.perviews>500 AND viewCount > 3600 LIMIT 100) doc, `bigquery-public-data.wikipedia.pageviews_2019` as wiki
 WHERE wiki.datehour BETWEEN "2019-01-01" AND "2020-01-01" AND wiki.title = doc.title AND wiki.wiki='en';
```

由于各种原因，总耗费折算为人民币超过一千元。当然，并没有超过谷歌给新用户的免费额度，所以实际上应该是没有花费。为了方便之后获取，我将其上传到百度云盘上了。

防止爬虫，链接使用了base64进行加密：aHR0cHM6Ly9wYW4uYmFpZHUuY29tL3MvMWJRbll2OFUyZTZKTi1NV3c0MjJDOWc=，提取码为p3o5。

## 进一步处理

写了个python程序进行进一步的处理，以获取每个页面的pageview访问数据。
目标为得到对应页面五年来的pageview数据并保存为csv文件。该csv文件至少有两列，一列为日期，一列为小时级别的访问量。

数据使用top100en数据为基础，放在E盘的wikidata中。

```python
from datetime import datetime
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
import os
os.chdir(r"E:\wikidata")
# 2015的版本作为基底
dirname = 'top100en'
filename = '2015.csv'
baseData = pd.read_csv(dirname+'\\'+filename,encoding='utf-8')

grouped_result = baseData.groupby('title')
# convert result to dictionary
baseDict = {}
for name,group in grouped_result:
    baseDict[name] = group
# 开始遍历后面的所有年份
for year in range(2016,2021):
    keys = list(baseDict.keys())
    filename = str(year)+'.csv'
    yearData = pd.read_csv(dirname+'\\'+filename,encoding='utf-8')
    grouped_result = yearData.groupby('title')

    # 遍历所有的keys，尝试将pandas DataFrame数据进行拼接
    errorList = []
    for key in keys:
        try:
            newDataFrame = grouped_result.get_group(key)
            #将获取到的新值与旧有数据进行拼接
            baseDict[key] = pd.concat([baseDict[key],newDataFrame])
        except KeyError:
            #如果该值没有找到，则会报这个错误。此时记录下来，循环结束后将其从baseData中删除
            errorList.append(key)
    
    print("error_list of year {} is {}".format(year,errorList))
    print('Delete them')
    for errorItem in errorList:
        del baseDict[errorItem]


# 获取数据
data = baseDict["Donald_Trump"] # ! 此处修改需要获取的页面名称
data.sort_values("datehour",inplace=True)
outputData = data["views"].to_numpy()
print("数据长度",len(data))
# 进行改写
newDF = data[["datehour","views"]]
newDF.columns = ["datetime","view"]
newDF["datetime"] = newDF["datetime"].apply(lambda x: datetime.strptime(x,"%Y-%m-%d %H:%M:%S %Z").strftime("%Y%m%d%H"))
newDF.to_csv("result.csv",index=False) # 导出
```