---
title: python日志模块logging学习与快速复习笔记
date: 2020-05-05T09:08:04+08:00
tags:
  - 计算机/python
  - 计算机/日志
categories:
  - 快速复习
modified: 2024-01-21
---

因为内容比较简单，就不做学习总结了，全部内容归类为快速复习。
参考：
* [csdn-python logging](https://blog.csdn.net/z_johnny/article/details/50812878)，有例子，可以快速入门
* [cnblog - python日志处理模块](https://www.cnblogs.com/yyds/p/6901864.html)，很详细，比较规范
## 基础知识

### 日志级别
CRITICAL > ERROR > WARNING > INFO > DEBUG > NOTSET，如果将日志级别设置为INFO，则INFO以下的日志将不会输出。默认设置级别为WARNING。

### 常用函数
```python
logging.basicConfig() # 使用一系列key-value值规定日志的基本配置信息，level规定输出的log级别，format定义输出log的格式，datafmt为输出的时间格式，filename为log文件名，filemode为打开log文件的模式。
logging.debug/info/warning/error/critical(str) #输出错误信息
logging.log(logging.DEBUG/..., str) #另外一种输出错误信息的方式


#可以使用如下代码将log信息同时输出到console上
console = logging.StreamHandler()                  # 定义console handler
console.setLevel(logging.INFO)                     # 定义该handler级别
formatter = logging.Formatter('%(asctime)s  %(filename)s : %(levelname)s  %(message)s')  #定义该handler格式
console.setFormatter(formatter)
# Create an instance
logging.getLogger().addHandler(console) 

```

## 例子
### 例1：控制台与文件输出
设置输出格式，同时输出到控制台与文件
```python
#coding:utf-8
 
# =======================================================================
# FuncName: console_out.py
# Desc: output log to console and file
# Date: 2016-02-19 17:32
# Author: johnny
# =======================================================================
 
import logging
 
def console_out(logFilename):
    ''' Output log to file and console '''
    # Define a Handler and set a format which output to file
    logging.basicConfig(
                    level    = logging.DEBUG,              # 定义输出到文件的log级别，                                                            
                    format   = '%(asctime)s  %(filename)s : %(levelname)s  %(message)s',    # 定义输出log的格式
                    datefmt  = '%Y-%m-%d %A %H:%M:%S',                                     # 时间
                    filename = logFilename,                # log文件名
                    filemode = 'w')                        # 写入模式“w”或“a”
    # Define a Handler and set a format which output to console
    console = logging.StreamHandler()                  # 定义console handler
    console.setLevel(logging.INFO)                     # 定义该handler级别
    formatter = logging.Formatter('%(asctime)s  %(filename)s : %(levelname)s  %(message)s')  #定义该handler格式
    console.setFormatter(formatter)
    # Create an instance
    logging.getLogger().addHandler(console)           # 实例化添加handler
 
    # Print information              # 输出日志级别
    logging.debug('logger debug message')     
    logging.info('logger info message')
    logging.warning('logger warning message')
    logging.error('logger error message')
    logging.critical('logger critical message')
 
if __name__ == "__main__":
    console_out('logging.log')
```

控制台
```python
2016-03-06 14:38:49,714  console_out.py : INFO  logger info message
2016-03-06 14:38:49,714  console_out.py : WARNING  logger warning message
2016-03-06 14:38:49,714  console_out.py : ERROR  logger error message
2016-03-06 14:38:49,714  console_out.py : CRITICAL  logger critical message
```

文件logging.log
```python
2016-03-06 Sunday 14:38:49  console_out.py : DEBUG  logger debug message
2016-03-06 Sunday 14:38:49  console_out.py : INFO  logger info message
2016-03-06 Sunday 14:38:49  console_out.py : WARNING  logger warning message
2016-03-06 Sunday 14:38:49  console_out.py : ERROR  logger error message
2016-03-06 Sunday 14:38:49  console_out.py : CRITICAL  logger critical message
```