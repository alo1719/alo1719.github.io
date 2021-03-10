---
title: Pandas访问数据的最佳实践
date: 2019-04-14 22:26:54
tags: [python, pandas]
---

[官方英文文档链接](http://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html)

## 不同数据维度对应的类

从一维、二维到三维数据对应的类分别为：`Series`、`DataFrame` 和 `Panel`，索引方式分别为：`s.loc[indexer]`、`df.loc[row_indexer, column_indexer]` 和 `p.loc[item_indexer, major_indexer, minor_indexer]`。

用常见的 Excel 来举例的话，`Series` 就是一行数据（一维）、`DataFrame` 就是一整张表（二维），而 `Panel` 则是整个Excel文件（三维，可能包含多张表）。

### 缺省切片

实际上，`DataFrame` 切片也不一定要传两个参数，`Panel` 切片也不一定要传三个参数，省略参数默认为空切片。例如，`p.loc['a']` 等价于 `p.loc['a', :, :]`，效果是选择了一张数据表。

### 双引号

双引号指明间隔顺序，例如 `s[::2]` 表明从第一个元素开始每隔一个元素选择，而 `s[::-1]` 表明从最后一个开始往前选择。
<!-- more -->

## 索引的不同选择

`.loc` 方法主要适用于*标签* 的索引，出错时会报 `KeyError`。

`.loc` 接受：

- 单个标签，例如 `'a'`
- 多个标签，例如 `['a', 'b', 'c']`
- 切片对象，例如 `'a':'f'`（`'a'` 和 `'f'` 都包含在内）
- 一个布尔数组
- 一个函数

`.iloc` 主要用于数字位置的索引，出错时会报 `IndexError`。

`.iloc` 接受：

- 一个数字, 例如 `5`
- 一组数字, 例如 `[4, 3, 0]`
- 切片对象, 例如 `1:7`（左闭右开）
- 一个布尔数组
- 一个函数

### 通过布尔切片

```py
# 示例一
s = pd.Series(range(-3, 4))
s[(s < -1) | (s > 0.5)]
s[~(s < 0)]

# 示例二
df2 = pd.DataFrame({'a': ['one', 'one', 'two', 'three', 'two', 'one', 'six'], 'b': ['x', 'y', 'y', 'x', 'y', 'x', 'x'], 'c': np.random.randn(7)})
# criterion 是一个 boolean list
criterion = df2['a'].map(lambda x: x.startswith('t'))
df2[criterion]
# 等价的写法
df2[[x.startswith('t') for x in df2['a']]]
```

### 通过函数切片

```py
# 生成DataFrame
df1 = pd.DataFrame(np.random.randn(6, 4), index=list('abcdef'), columns=list('ABCD'))
# 通过函数切片
df1.loc[lambda df: df.A > 0, :]
```

## 交换DataFrame两列数据

正确的方法为：

```py
df[['B', 'A']] = df[['A', 'B']]
```

以下交换方法不起作用，因为pandas规定 `.loc` 和 `.iloc` 列对齐操作在赋值操作之前：

```py
# 没有效果
df.loc[:, ['B', 'A']] = df[['A', 'B']]
# 解决方法
df.loc[:, ['B', 'A']] = df[['A', 'B']].to_numpy()
```

## at 和 iat

用处与 `loc` 和 `iloc` 类似，只不过 `at` 和 `iat` 只能获取单一Cell的数据，而 `loc` 和 `iloc` 可以获取一行、一列或者更复杂的数据。读单一Cell内数据的话， `at` 和 `iat` 会比`loc` 和 `iloc` 快很多。
