---
title: Python字符串前缀
date: 2019-03-09 11:44:39
tags: [python]
---

## u

`u"sample-string"`表明Unicode字符串, 代表对字符串进行Unicode解码. 建议包含中文的字符串都表明编码.

## r

`r"sample-string"`表明raw string不对blackslash反斜线进行转义. 例如, `\n`即为两个字符不再代表换行.

## b

Python3中, 默认的str的Unicode编码, Python2中, 默认的str使用Bytes编码. `b"sample-string"`表明使用Bytes编码, 用到的地方很少.