---
title: Python遍历文件
date: 2019-03-09 11:43:50
tags: [python]
categories: 入门教程
---

在进行数据处理时，经常需要遍历一个文件夹，在Python中最简单的解决办法是使用 `os.walk()`。

## 函数声明

函数声明如下:

```py
def walk(top, topdown=True, onerror=None, followlinks=False):
```

top为传入的根目录。

`os.walk()` 函数会递归地为根目录下的每一个文件夹（包括根目录本身），产生一个三元组 `(dirpath, dirnames, filenames)`。

`dirpath` 为当前文件夹路径；`dirnames` 为 `dirpath` 下所有文件夹的名称（非递归，类似于资源管理器中或Finder中看到的文件夹）；`filenames` 为 `dirpath` 下所有文件的名称（同样非递归）。
<!-- more -->

需要完整的路径名可以使用 `os.path.join()`。

topdown：设定为Ture表示一个目录的三元组将比它任何子文件夹的三元组先产生。设定为False表明一个目录的三元组将比它的任何子文件夹的三元组后产生。简单来讲，设定为Ture，`dirpath` 是自顶向下产生的。

followlinnks：此项设置为True，可以访问软链接。

## 实例

```py
import os

g = os.walk(r"d:\test")

for path, dir_list, file_list in g:
    # 输出目录下所有文件
    for filename in file_list:
        print(os.path.join(path, filename))
    # 输出目录下所有文件夹
    for dirname in dir_list:
        print(os.path.join(path, dirname))
```

## 总结

Python的 `os.walk()` 确实很像人手动打开文件夹的过程：`dirpath` 为当前路径，`dirnames` 为当前路径下所有的文件夹，`filenames` 为当前路径下的所有文件。要查看更多文件，需要再打开 `dirnames` 中的子文件夹，确实很像 `walk`。和JavaScript、Java的文件遍历相比，确实有一点“优雅”的感觉😀。