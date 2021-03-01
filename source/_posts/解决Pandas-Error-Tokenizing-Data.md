---
title: 解决Pandas Error Tokenizing Data
date: 2019-03-09 11:41:39
tags: [python，pandas]
categories: 疑难杂症
---

在使用Pandas读取表格时，如果表格内容不规整（例如Schema信息包含8列，但是某一行包含了9列信息），Pandas会报 `Pandas Error Tokenizing Data` 的错，程序会立即停止执行。报这一错误可以分为两种场景：

## 表格Schema信息解析异常

第一种场景是表格的Schema信息，也即第一行不规整，导致Pandas无法正确解析。

据官方文档，如果 `sep=None`（不传sep参数则默认为None），pandas会**自动推断**分隔符，但是在某些场景下自动推断并不好用。

```py
df = pandas.read_csv(fileName, sep='delimiter', header=None)
```
<!-- more -->

此时手动设置 `sep` 参数为需要的分隔符，可以避免pandas进行错误的自动推断。如果表格没有Schema信息的话就指定 `header=None`。

## 表格内容有误

另一种场景是表格的某一行不规整，和Schema信息对不上，例如少了一列或者多了一列。

```py
data = pandas.read_csv('file.csv'，error_bad_lines=False)
```

这时可以通过设置 `error_bad_lines` 解决。这时pandas会跳过有问题的行，不再直接打断程序。


