---
title: 解决Pandas Error Tokenizing Data
date: 2019-03-09 11:41:39
tags: [python, pandas]
---

在使用Pandas读取表格时, 如果表格内容不规整, 例如Schema信息中包含8列但是某一行包含了9列信息, Pandas会报`Pandas Error Tokenizing Data`的错, 且程序会立即停止执行.
<!-- more -->

## 第一种情况

一种情况是表格的第一行不规整导致Pandas无法正确读取

```py
df = pandas.read_csv(fileName, sep='delimiter', header=None)
```

此时设置显性分隔符`sep`为需要的分隔符, 如果没有Schema信息的话就指定`header=None`. 据官方文档, pandas会自动推断分隔符, 如果`sep=None`(也即不设置显性sep), 但是在某些时候自动推断并不好用.

## 第二种情况

另一种情况是表格的某一行不规整, 这是可以通过

```py
data = pandas.read_csv('file.csv', error_bad_lines=False)
```

解决. 注意这样pandas会跳过有问题的行.


