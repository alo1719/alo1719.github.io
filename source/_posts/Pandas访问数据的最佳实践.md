---
title: Pandas访问数据的最佳实践
date: 2019-04-14 22:26:54
tags: [python, pandas]
---

[文档链接](http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html)

要切片, 切丁数据, 要关注的是`Series`类和`DataFrame`类.

## 索引的不同选择

`.loc`方法主要适用于*标签*的索引, 出错时会报`KeyError`.

`.loc`接受:

- 单个标签, 例如`'a'`
- 多个标签, 例如`['a', 'b', 'c']`
- 切片对象, 例如`'a':'f'`(`'a'`和`'f'`都包含在内)
- 一个布尔数组
- 一个函数

`.iloc`主要用于数字位置的索引, 出错时会报`IndexError`.

`.iloc`接受:

- 一个数字, 例如`5`
- 一组数字, 例如`[4, 3, 0]`
- 切片对象, 例如`1:7`(左闭右开)
- 一个布尔数组
- 一个函数

<!-- more -->

## 缺省

不填写详细值即为空切片. 例如, `p.loc['a']`等价于`p.loc['a', :, :]`

## 三种类

从一维到二维到三维的三种类分别为: `Series`, `DataFrame`, `Panel`, 索引方式分别为:`s.loc[indexer]`, `df.loc[row_indexer, column_indexer]`, `p.loc[item_indexer, major_indexer, minor_indexer]`

## 交换DataFrame两列数据

```py
df[['B', 'A']] = df[['A', 'B']]
```

以下交换方法不起作用, 因为pandas规定`.loc`和`.iloc`中列对齐在赋值操作之前:

```py
df.loc[:, ['B', 'A']] = df[['A', 'B']]
# 解决方法
df.loc[:, ['B', 'A']] = df[['A', 'B']].to_numpy()
```

## 双引号

双引号指明间隔顺序, 例如`s[::2]`表明从第一个开始每隔1个元素选择, 而`s[::-1]`表明从最后一个开始往前.

## 通过函数切片

```py
df1 = pd.DataFrame(np.random.randn(6, 4), index=list('abcdef'),
 columns=list('ABCD'))

df1.loc[lambda df: df.A > 0, :]
```

## at和iat

用处和loc和iloc类似, 只不过at和iat每次只能获取单一数据, 而loc和iloc可以获取一行数据或者一列数据. 读单一数据的话at和iat会比loc和iloc快非常多.

## 通过布尔切片

```py
s = pd.Series(range(-3, 4))
s[(s < -1) | (s > 0.5)]
s[~(s < 0)]

df2 = pd.DataFrame({'a': ['one', 'one', 'two', 'three', 'two', 'one', 'six'], 'b': ['x', 'y', 'y', 'x', 'y', 'x', 'x'], 'c': np.random.randn(7)})
# criterion 得到的是一个boolean list
criterion = df2['a'].map(lambda x: x.startswith('t'))
df2[criterion]
# 等价但是慢一些
df2[[x.startswith('t') for x in df2['a']]]
```
