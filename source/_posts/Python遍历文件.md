---
title: Python遍历文件
date: 2019-03-09 11:43:50
tags: [python]
---


在进行数据处理时, 经常需要遍历一个文件夹, 在Python中简单的解决办法是使用`os.walk()`.

## 函数声明

函数声明如下:

```py
def walk(top, topdown=True, onerror=None, followlinks=False):
```

<!-- more -->
top: 根目录下的每一个文件夹(包括自己), 产生一个长度为3的元组`(dirpath, dirnames, filenames)`, 为文件夹路径, 文件夹名, 文件名. 需要完整的路径名需要使用`os.path.join()`

topdown: 设定为Ture表明一个目录的3-元组将比它任何子文件夹的3-元组先产生. 设定为False表明一个目录的3-元组将比它的任何子文件夹的3-元组后产生.

onerror: 可选, 是一个函数; 它调用时有一个参数, 一个OSError实例. 报告这错误后, 继续walk, 或者抛出exception终止walk.

followlinnks: 设置为True, 可以通过软链接访问目录.

## 实例

```py
import os

g = os.walk(r"d:\test")

for path, dir_list, file_list in g:
    # 输出目录下所有文件
    for file_name in file_list:
        print(os.path.join(path, file_name))
    # 输出目录下所有文件夹名
    for dir_name in dir_list:
        print(os.path.join(path, dir_name))


