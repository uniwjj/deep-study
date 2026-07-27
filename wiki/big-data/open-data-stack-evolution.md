---
title: 开放数据标准栈十年演进
description: 从 Parquet 到 Ossie，Apache 开源标准逐层吃掉企业数据平台的五层架构——文件→内存→表→目录→语义，十年的产业共识演进
aliases: [五层开放数据栈, Open Data Stack Evolution, 数据标准栈, 语义标准化]
tags: [big-data, concept, architecture]
sources: [2026/07/26/从 Arrow 到 Iceberg 到 Polaris 到 Ossie：语义标准化的最后一块拼图.html]
created: 2026-07-27
updated: 2026-07-27
---

# 开放数据标准栈十年演进

2018 年，企业大数据平台架构图上标着 Hadoop 底层、ETL 中间层、Tableau 顶层，数据格式 Avro/ORC/Parquet 混用，工具之间靠点对点 connector 硬接。

**七年后的 2026 年，这张图上的每一层已被 Apache 开源标准一块一块吃掉了。** 从下往上：[[apache-parquet|Parquet]] 吃了文件层、[[apache-arrow|Arrow]] 吃了内存交互层、[[iceberg|Iceberg]] 吃了表管理层、[[apache-polaris|Polaris]] 吃了目录层——2026 年 7 月，最后一块拼图来了：[[apache-ossie|Apache Ossie]]，吃掉了语义层。

这不是巧合，是酝酿了十年的产业共识。

## 五层架构总览

```
语义层         2026 · Ossie      你的数据什么意思？
目录 & 治理层  2025 · Polaris    数据在哪？谁能看？
表格式层       2020 · Iceberg    表怎么管？怎么改？
内存交换层     2016 · Arrow      数据怎么传？格式是啥？
文件格式层     2013 · Parquet    数据存成什么？怎么读？
─────────────────────────────────────────────
以上是应用层（BI / Agent / 业务系统）
```

| 层 | 项目 | 年份 | 成熟度 | 解决的核心问题 |
|----|------|------|--------|---------------|
| 文件格式 | [[apache-parquet\|Parquet]] | 2013 | ⭐ 最稳固 | 数据存成什么格式、怎么高效读取 |
| 内存交换 | [[apache-arrow\|Arrow]] | 2016 | ⭐ 成熟 | 跨语言/跨进程传数据，消除序列化开销 |
| 表格式 | [[iceberg\|Iceberg]] | 2020 | ⭐ 快速增长 | 表是逻辑概念不是文件别名，Schema 演进/时间旅行/增量查询内置 |
| 目录 & 治理 | [[apache-polaris\|Polaris]] | 2025 | ⭐ 刚毕业 | 表多了怎么找，权限怎么统一，跨引擎互操作 |
| 语义 | [[apache-ossie\|Ossie]] | 2026 | 🔥 刚进场 | 数据字段是什么意思，指标怎么算，Agent 如何理解业务 |

## 每一层的标准化模式

五层不是谁规划好的，是行业自己一层一层"交"出来的。每层遵循相同的模式：

| 阶段 | 发生了什么 | 示例 |
|------|-----------|------|
| ① 先用起来 | 各家各搞一套，格式互不兼容 | 每家数据库有自己的内存格式 |
| ② 痛够了 | 对接成本高到受不了，搬数据比算数据贵 | Pandas→Spark 来回序列化吃掉 70% 时间 |
| ③ 有人牵头 | 一两家大公司把内部方案开源 | Wes McKinney 搞 Arrow、Netflix 搞 Iceberg、Snowflake 搞 Polaris |
| ④ 竞品认了 | 竞争对手决定也用这个标准 | Databricks 和 Snowflake 一起推 Polaris |
| ⑤ 进 Apache | 交给基金会，谁也别想独占 | Arrow→Iceberg→Polaris→Ossie 全在 Apache |

**关键在 ④**：开放标准最难的从来不是技术，是让竞争对手坐下来一起签字。Polaris 之所以能成，不是技术多牛——是 Snowflake 和 Databricks 这对对手同时认了它。

## 下上依赖链

每层的开放标准让上一层的问题变得**清晰可见**。不是先见之明——是问题一层一层暴露，行业一层一层摊牌：

```
Parquet 统一文件格式
  → Iceberg 的抽象才能建立（否则要同时支持 ORC/Avro/CSV）
    → Polaris 的目录才有统一口径（否则不知道管的是表还是文件/物化视图还是快照）
      → Ossie 的语义模型才知道挂在哪（字段定义跟表走还是跟目录走？）
```

**每一层都是上一层的基础。** 2025 年之前连 Iceberg 的表格式都没稳、Polaris 目录协议还在讨论，没人敢在上面盖语义层。三层都稳了，行业才能抬头看下一层。

## Ossie 为什么意义特殊

前三层（Parquet/Arrow/Iceberg）和第四层（Polaris）解决的是"**机器跟机器怎么说话**"——存储、传输、管理、权限。

[[apache-ossie|Ossie]] 不一样。它解决的是"**人跟机器之间怎么对齐含义**"。同一个"收入"在财务、销售、运营各有定义，让 50 家公司签一份"什么是收入"的协议，比签一份"文件怎么存"难十倍。

2026 年能做成的两个推力：

1. **AI Agent 倒逼**：以前 BI 工具查数据，人还能纠错。Agent 直接根据查询结果做决策，没人复核，"含义"就变成了必须标准化的硬问题。
2. **生态成熟**：下层稳了，上层才可能标准化。

## 对架构师的启示

- **Parquet + Iceberg 是 2026 年的起点线**，不是附加分——文件用 Parquet、表用 Iceberg，没有争议。
- **Polaris 是围墙**——决定谁能进、能看到什么。内部已有 Snowflake Horizon / Databricks Unity Catalog 的可以先不换，但权限模型要往 Iceberg REST 协议方向靠。
- **Ossie 是门牌号**——决定每个房间放什么、叫什么。暂不建议上生产，但可以把本体模型转成 Ossie 格式，跟着表结构一起维护，相当于先写"语义源码"，等工具链成熟后编译即可使用。

## 历史定位

2015-2025 这十年，开源标准把企业数据平台从下到上吃了一遍。之前"开放"意味着"免费"——Parquet 替代了 Avro 的私有扩展；之后"开放"意味着"共识"——Polaris 的标准不是一家公司定的，是两家竞争对手坐下来签的字。

现在这条标准链只剩下最后一个问题：**数据的含义**。这是最难的，也是最重要的——因为 AI Agent 来了，它们不再能靠人工兜底来理解数据含义。

## 相关页面

- [[apache-parquet]] — 文件格式层
- [[apache-arrow]] — 内存交换层
- [[iceberg]] — 表格式层
- [[apache-polaris]] — 目录 & 治理层
- [[apache-ossie]] — 语义层
- [[lakehouse]] — 湖仓一体架构（五层标准栈所支撑的目标架构）
- [[table-format-selection]] — Iceberg vs Delta/Hudi/Paimon 选型对比
- [[agentic-data-cloud]] — Google 跨云 Lakehouse 依赖这些开放标准
- [[databricks-2026-summit]] — Databricks 2026 峰会四层架构与语义层布局
