---
title: Apache Arrow
description: Apache Arrow 跨语言跨进程列式内存格式——统一数据在内存中的形态，消除序列化开销，成为整个数据生态的底层基础设施
aliases: [Arrow, Apache Arrow, Arrow 内存格式, 列式内存格式]
tags: [big-data, tool, concept]
sources: [2026/07/26/从 Arrow 到 Iceberg 到 Polaris 到 Ossie：语义标准化的最后一块拼图.html]
created: 2026-07-27
updated: 2026-07-27
---

# Apache Arrow

Apache Arrow 是一种**跨语言、跨进程的列式内存格式**，2016 年由 Wes McKinney（Pandas 作者）主导创建。在 [[open-data-stack-evolution|五层开放数据标准栈]]中占据**内存交换层**。

## 解决的问题

2016 年前后，数据工程领域最头疼的不是算得慢——是**传数据**。Spark 读 Pandas DataFrame 需要序列化成 JSON 再反序列化，几百 GB 的数据 70% 时间花在来回翻译格式上。

Arrow 的核心设计：数据在内存里是**标准布局**，Python 能读、Java 能读、C++ 能读，同一个指针传过去就行，不需要序列化。

一句话概括：**"我跟你说数据是什么样的"，而不是"我给你发一个 CSV 你自己猜"。**

## 衍生产品

Arrow 后来推出了多个子项目，但内核没变——先统一数据在内存里的形态，再谈怎么用：

| 子项目 | 作用 |
|--------|------|
| **Arrow Flight** | 数据怎么传（高性能数据传输协议） |
| **ADBC** | 数据库驱动怎么接（统一数据库连接接口） |
| **DataFusion** | 查询引擎（基于 Arrow 的 SQL 查询引擎） |

## 生态地位

Arrow 现在是整个数据生态的基础设施：

- **分析引擎**：Polars、DuckDB、InfluxDB 3.0 底层都使用 Arrow
- **文件与内存桥接**：Parquet 文件读出来是 Arrow，传进 GPU 是 Arrow
- **分布式系统**：shuffle 过程也使用 Arrow

这层的标准已经稳固——"没人再问'我们该用哪个内存格式'"。

## 在五层栈中的位置

Arrow 是 [[apache-parquet|Parquet]]（文件格式层）的上层：Parquet 定义了数据怎么存，Arrow 定义了数据在内存中怎么表示。下层稳定后，Arrow 让上层 [[iceberg|Iceberg]]（表格式层）的标准化成为可能。

## 相关页面

- [[open-data-stack-evolution]] — 五层开放数据标准栈全景
- [[apache-parquet]] — 下层依赖，文件格式层
- [[iceberg]] — 上层，表格式层
