---
title: Apache Ossie
description: Apache Ossie 语义层——用 YAML 将数据字段映射到业务概念，定义指标口径与同义词，让 AI Agent 和 BI 工具理解"数据是什么意思"
aliases: [Ossie, Apache Ossie, 语义层, Semantic Layer, 数据语义层]
tags: [big-data, tool, concept]
sources: [2026/07/26/从 Arrow 到 Iceberg 到 Polaris 到 Ossie：语义标准化的最后一块拼图.html]
created: 2026-07-27
updated: 2026-07-27
---

# Apache Ossie

Apache Ossie 是 2026 年 7 月进入 Apache 的开源**语义层（Semantic Layer）**项目，也是 [[open-data-stack-evolution|开放数据标准栈]] 五层架构的最新一块——继 [[apache-parquet|Parquet]]（文件）、[[apache-arrow|Arrow]]（内存）、[[iceberg|Iceberg]]（表）、[[apache-polaris|Polaris]]（目录）之后，Ossie 吃掉的是**语义层**。

## 解决的问题

Polaris 管好了"数据在哪"，但没管"数据是什么意思"。

假设 BI 工具或 AI Agent 读到一张 Iceberg 表 `sales.orders`，字段 `o_sts_cd = 3`。Polaris 能告诉你：这张表在哪个 bucket、最新快照号、谁有读权限。但它不能告诉你：`o_sts_cd=3` 就是"已完成"，"已完成"的定义是"已发货 + 已签收 + 已入账"三个条件同时成立。

Ossie 补的就是这个窟窿——给企业里的所有数据表加一份**"翻译说明书"**，用 YAML 把字段映射到业务概念，定义指标怎么算，标注同义词。

## 与 Polaris 的关系

| 层 | 项目 | 回答的问题 |
|----|------|-----------|
| 目录层 | Polaris | "订单表在 S3 的 bucket-xyz，最新快照是 snapshot-456" |
| 语义层 | Ossie | "`o_sts_cd=3` 叫'已完成'，定义是发货+签收+入账，等价词：OrderComplete、订单完成、状态3" |

两个东西工作在同一张表的不同维度上——Polaris 负责**定位**，Ossie 负责**理解**。

## 为什么 2026 年才出现

语义层不是新概念，但之前一直做不成标准。两个因素在 2026 年形成合力：

### AI Agent 倒逼

以前 BI 工具查数据，出错人还能纠——分析师看"客单价 3000"觉得不对劲会去查 SQL。但 AI Agent 不同：Agent 直接根据查询结果**做决策**，如果口径错了没有人检查。当没有人"复核"的时候，"含义"就变成必须标准化的硬问题，不能再靠人工兜底。

### 生态成熟

2025 年之前，Iceberg 的表格式标准还没稳、Polaris 的目录协议还在讨论。下层不稳，没人敢在上面盖语义层。现在 Parquet、Iceberg、Polaris 三层都稳了，行业可以抬头看上一层。

## 五层之间依赖关系

从下往上，每层的开放标准让上一层的问题变得**清晰可见**。不是谁先见之明——是问题一层一层暴露，行业一层一层摊牌。

- 没有 Parquet 统一文件格式 → Iceberg 抽象无法建立（需同时支持 ORC、Avro、CSV，复杂度爆炸）
- 没有 Iceberg 统一表格式 → Polaris 目录无统一口径（管表还是文件？物化视图还是快照？）
- 没有 Polaris 统一目录 → Ossie 语义模型不知道该挂在哪（字段定义跟表走还是跟目录走？）

## 行业现状

Ossie 目前已有 50+ 家公司签署支持，但**尚未进入大规模生产环境**。长期能不能维持共识，取决于第一个重大分歧出现时社区如何处理——这也是开放标准能否成立的第四个关键步骤（"竞品认了"）。

## 架构师建议

- 暂时不需要把 Ossie 推到生产环境——工具链和生态仍在早期。
- 可以先**把本体模型转成 Ossie 格式**，放到仓库里跟着表结构一起维护。相当于先写好"语义源码"，等生态成熟后编译即可使用。
- 当前管理数据含义的手段（dbt docs 注释、Excel 记口径）都是过渡方案，语义层标准化是大势所趋。

## 相关页面

- [[open-data-stack-evolution]] — 五层开放数据标准栈的十年演进全景
- [[apache-polaris]] — 目录 & 治理层，Ossie 的下层依赖
- [[iceberg]] — 表格式层，Ossie 的底层基础
- [[agentic-data-cloud]] — Google 跨云 Lakehouse 中语义层的角色
- [[databricks-2026-summit]] — Databricks 2026 峰会对语义层的布局
