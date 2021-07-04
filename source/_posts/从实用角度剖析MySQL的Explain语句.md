---
title: 从实用角度剖析 MySQL 的 Explain 语句
date: 2021-05-26 17:00:32
tags: [mysql, database]
---

MySQL 的 Explain 语句常用来分析慢 SQL 的原因，会返回以下的数据：

select_type、table、partitions、type、possible_keys、key、key_len、ref、rows、filtered、extra。

其中最重要的是 type、rows、key 和 extra 这四个字段。

## MySQL 逻辑架构

先介绍一下基础知识，MySQL 的逻辑架构：

客户端->（查询缓存，5.7.20 已 deprecated，8.0 已移除）->解析器（生成解析树）->预处理器（生成新解析树）->查询优化器（生成查询计划）->执行引擎（执行调度）->存储引擎（数据查找）->执行引擎（数据过滤、排序等）->客户端和查询缓存

## type、rows 和 key 字段

type 字段用来说明连接的类型。连接类型从快到慢可以分为以下几种：

1. const，system：最多一个匹配行，使用主键或者 unique 索引

2. eq_ref：返回一行数据，通常在联接（join）时出现，使用主键或 unique 索引

3. ref：使用 key 的**最左前缀**，且 key 不是主键或 unique 键
<!-- more -->
4. range：索引范围扫描，对索引的扫描开始于某一点，返回匹配的行

5. index：以索引的顺序进行**全表扫描**，优点是不用排序，缺点是还要全表扫描

6. all：全表扫描

其中，rows 字段是 MySQL 估算的为了找到所需记录而需要检索的行数，用来给查询优化器选择 key 作参考。

index、all 类型要走全表扫描，一般来说 rows 对应的行数会很大。所以线上环境是要避免 index 和 all 的连接类型。

key 字段说明了用到了什么索引，如果没有用到索引就为 null。

**index** 类型其实是相当有迷惑性的，名字上是**索引**，实际上根本没有利用索引进行查询，只是在全表扫描的基础上用索引进行排序，这点特别值得注意。

## extra 字段

extra 字段用来说明额外的**重要信息**。从快到慢可以分为以下几种：

1. using index：索引覆盖，查询只用到索引即可，无需读取数据块

2. using where：在存储引擎返回记录后再做过滤

3. using temporary：使用临时表，通常在使用 GROUP BY，ORDER BY 时出现

4. using filesort：用到非索引顺序的额外排序，当 ORDER BY 未用到索引时发生（有可能是内存排序，也有可能是外排）

在线上环境要注意尽量避免 using temporary 和 using filesort。

## 排查慢 SQL 最佳实践

1. 先看 key 字段，看有没有用到索引。

2. 再看 type 字段，不能是 index 或 all。

3. 最后看 extra 字段，不能是 using temporary 或 using filesort。