---
title: KDB+效能玩票测试
date: 2024-04-11T14:12:43+08:00
toc: true
tags: 
categories:
- 数据库
- KDB+
---

## 动因

之前使用Man-Group的Arctic系统（老Arctic，基于MongoDB的那个）存储各种交易标的的数据，但发现比较慢。测试下传说中的KDB+看能否替换。目前并不期望能使用KDB+强项的内部计算（由于`q`语言学习成本过高），只是简单作为时序数据库使用，支持读取写入增删改查即可。

## 架构

使用一台Linux主机作为数据库服务器，与客户机在同一个局域网（千兆以太网）下。具体配置过程略。

## 简单测试

使用数据是浦发银行（600000.SSE）的1分钟数据，长度大约10年，一共599280行x9列。

1. 写入
    KDB+: 
    ```python
    t = con('{`:trade/ set x}', df1)
    ``` 
    CPU times: total: 844 ms
    Wall time: 574 ms

    Arctic:
    ```python
    barlib.write("test_1m", df1)
    ```
    CPU times: total: 1.48 s
    Wall time: 1.62 s

2. 整体读取
    KDB+:
    ```python
    %%time
    a = con('get `:trade')
    df_get = a.pd().set_index("date")
    ```
    CPU times: total: 297 ms
    Wall time: 338 ms

    Arctic:
    ```python
    barlib.read("test_1m")
    ```
    CPU times: total: 2.08 s
    Wall time: 3.24 s

3. 部分读取
    KDB+
    ```python
    a_p = con("select date, open, close from `:trade where date within 2023.01.01T10:00 2024.03.01T14:00")
    df_get_p = a_p.pd().set_index("date")
    ```
    CPU times: total: 15.6 ms
    Wall time: 18 ms

    Arctic
    ```python
    barlib.read("test_1m", chunk_range=DateRange("2023.01.01 10:00", "2024.03.01T14:00"), columns=["open", "close"])
    ```
    CPU times: total: 188 ms
    Wall time: 332 ms
