---
title: Apache Parquet
description: Apache Parquet 列式文件格式——2013 年成为标准，是五层开放数据栈最底层、最稳固的基石
aliases: [Parquet, Apache Parquet, Parquet 文件格式, 列式存储格式]
tags: [big-data, tool, concept]
sources: [2026/07/26/从 Arrow 到 Iceberg 到 Polaris 到 Ossie：语义标准化的最后一块拼图.html]
created: 2026-07-27
updated: 2026-07-27
---

# Apache Parquet

Apache Parquet 是 Hadoop 生态中诞生的**列式文件存储格式**，2013 年由 Twitter 和 Cloudera 联合创建。在 [[open-data-stack-evolution|五层开放数据标准栈]]中占据最底层**文件格式层**，也是五层中最稳固的一层。

## 解决的问题

在大数据平台早期的混乱阶段，Avro、ORC、Parquet 等格式混用，不同工具之间靠点对点 connector 硬接。Parquet 最终胜出，成为列式文件的事实标准。

## 在五层栈中的位置

Parquet 是整条标准链的地基：

- 向上：[[apache-arrow|Arrow]] 定义了文件读进内存后的标准表示
- 向上：[[iceberg|Iceberg]] 在 Parquet 文件之上建立表的抽象
- 向上：[[apache-polaris|Polaris]] 在此基础上提供统一目录
- 向上：[[apache-ossie|Ossie]] 在最顶层附加业务语义

没有 Parquet 统一文件格式，Iceberg 的抽象就建立不了——因为必须同时支持 ORC、Avro、CSV，复杂度爆炸。

## 相关页面

- [[open-data-stack-evolution]] — 五层开放数据标准栈全景
- [[apache-arrow]] — 上层，内存格式层
- [[iceberg]] — 上层，表格式层
