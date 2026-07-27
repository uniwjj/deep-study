---
title: Apache Iceberg
description: 开放表格式——Netflix 三初衷、三层元数据架构、快照/Manifest 查询流程、分区与 Schema 演进、时间旅行、v1→v3 演进、与 Delta/Hudi/Paimon 定位对比
aliases: [Iceberg, Apache Iceberg, Iceberg 表格式, 开放表格式]
tags: [big-data, tool, concept]
sources: [2026/07/01/iceberg-wikipedia.html, 2026/07/01/iceberg-architecture-design.html, 2026/07/01/iceberg-query-principle.html, 2026/07/01/iceberg-segmentfault-analysis.html, 2026/07/01/iceberg-principle-optimization.html, 2026/07/01/iceberg-aliyun-basics.html, 2026/07/01/iceberg-snowflake-docs.html, 2026/07/01/iceberg-v3-watershed.html, 2026/07/01/iceberg-tencent-batch-stream.html, 2026/07/01/iceberg-bytedance-feature-store.html, 2026/07/01/iceberg-tencent-governance.html, 2026/07/01/iceberg-xiaomi-practice.html, 2026/07/26/从 Arrow 到 Iceberg 到 Polaris 到 Ossie：语义标准化的最后一块拼图.html]
created: 2026-06-15
updated: 2026-07-27
---

# Apache Iceberg

Apache Iceberg 是一种面向超大规模分析的**开放表格式（Table Format）**。它定义数据文件、元数据、表结构的存储规范与访问协议，把"散装文件"变成可靠的"表"——支持 ACID 事务、Schema 演进、分区演进、时间旅行与跨引擎互操作，是开放湖仓（Open Lakehouse）的基础设施层。

## 三个初衷

Iceberg 2017 年由 Netflix 的 Ryan Blue 和 Dan Weeks 创建，2018 年捐给 Apache，2020 年毕业为顶级项目。其诞生是为解决 Apache Hive 表的三个根本缺陷（来源 Wikipedia）：

1. **正确性** —— Hive 无法保证正确性，不提供稳定的原子事务。Iceberg 确保数据正确性并支持 ACID。
2. **性能** —— Hive 操作粒度粗。Iceberg 通过更细粒度的文件级操作优化写入与查询性能。
3. **易运维** —— Hive 表运维复杂、易出错。Iceberg 简化并抽象表的通用运维。

Netflix 当时很多团队为避免 Hive 格式的副作用，刻意回避使用这些服务、不敢改数据——这正是 Iceberg 要消除的痛点。

## 三层元数据架构

Iceberg 的核心设计是**分层元数据**，这是它区别于 Hive（靠目录结构隐式管理）的根本。三层从上到下：

| 层 | 内容 | 作用 |
|----|------|------|
| **Catalog 层** | 管理表的当前元数据指针（指向最新 metadata 文件） | 记录表最新状态，支持原子操作（可存 HDFS / Hive Metastore / Nessie / REST Catalog） |
| **元数据层（Metadata）** | metadata.json + 快照（Snapshot）+ Manifest List（清单列表）+ Manifest File（清单文件） | 管理表 schema、分区信息、快照历史，索引数据文件 |
| **数据层（Data）** | 实际数据文件（Parquet / ORC / Avro） | 存储数据，通过 Manifest 索引访问 |

### 查询流程：从 metadata 到 data 文件

一次读取最新快照数据的过程（来源 底层数据查询原理）：

1. 从 Catalog 获取表最新 metadata 文件（如 `00000-ec504.metadata.json`）。
2. 解析 metadata，取 `current-snapshot-id`，从 `snapshots` 数组找到对应快照。
3. 快照指向一个 **Manifest List**（如 `snap--32800.avro`），其中列出多个 **Manifest File**。
4. 每个 Manifest File 描述一批数据文件的位置、分区范围、列统计信息等。
5. 读取 Manifest 指向的 Parquet 数据文件。

Manifest List 中每条记录含 `deleted/added/existing data files count`，可判断文件状态；Manifest File 中每条有 `status`（1=新增需读，2=已删除跳过）。这种分层让查询能**只读必要的文件**——靠 Manifest 的分区范围和列统计做文件裁剪（file pruning），无需扫描全表。

### 时间旅行

Iceberg 维护完整快照历史，支持三种时间旅行查询：
- **最新快照** —— 默认读取 `current-snapshot-id`。
- **指定快照** —— `snapshot-id` 参数（Spark: `.option("snapshot-id", ...)`）。
- **时间戳快照** —— `as-of-timestamp` 参数，Iceberg 据 metadata 的 `snapshot-log` 找到该时间点对应的快照。

支持数据版本回滚、审计合规。

## 分区演进与 Schema 演进

**分区演进（Partition Evolution）** —— Iceberg 最强大的功能之一：分区策略可随数据变化动态调整，**无需重写数据**。例如 `booking_table` 从按月分区（`MONTH(date)`）平滑过渡到按天分区（`DAY(date)`），系统自动识别新旧分区，查询兼容，零停机迁移。Hive 改分区要重写全部数据，这是 Iceberg 的关键优势。

**Schema 演进** —— 支持安全的列操作：添加列、重命名列、删除列、类型提升、重排列顺序，向前/向后兼容，无需重写表。嵌套结构同样支持演化。

**原地元数据迁移** —— 把现有表迁到 Iceberg 格式时，数据文件物理位置不变，只更新元数据，迁移快、业务不中断。

## MVCC 与快照隔离

Iceberg 通过 **MVCC（多版本并发控制）** 保证并发写入一致性：每次写入产生新快照，元数据原子更新（Catalog 层的 compare-and-swap）。读操作看到的是某个快照的一致性视图，写入不阻塞读取，写入间靠快照隔离互不干扰。这是 ACID 事务的基础。

## v1 → v2 → v3 演进

| 版本 | 关键能力 |
|------|---------|
| **v1** | 基础表格式：快照、分区演进、Schema 演进、时间旅行 |
| **v2** | **行级删除**：pos-delete（按位置删）+ eq-delete（按等值删），支持 upsert/delete |
| **v3**（2025 上半年规范批准） | 五大特性：删除向量（行级更新快 10 倍）、行级血缘（CDC 成表的原生能力）、VARIANT 类型（原生半结构化数据）、默认列值（Schema 变更从小时级变秒级）、纳秒时间戳与地理空间类型 |

v3 的意义不止特性，更在于它让 Iceberg 真正成为**行业公共标准**——一年内 Snowflake、Databricks、Google BigQuery、Dremio、AWS 集体跟进落地（来源 v3 分水岭）。

## 与 Delta / Hudi / Paimon 的定位对比

四大湖仓表格式各有"娘家"（来源 v3 分水岭）：

| 格式 | 起源 | 侧重 | 特点 |
|------|------|------|------|
| **Hudi** | Uber 2016 | Hadoop 生态 | 深度绑定 Hive/HDFS，对老 Hadoop 栈兼容最好，upsert 场景强；概念多、学习曲线陡 |
| **Delta Lake** | Databricks | Spark 生态 | 与 Spark 无缝衔接，Databricks 平台调优极佳；长期是 Databricks "自家孩子" |
| **Paimon** | Flink 社区 2022（阿里主导） | 流批一体 | CDC 入湖丝滑，实时湖仓主场；国内采用度高（字节/阿里/小米/shopee） |
| **Iceberg** | Netflix 2018 | 中立开放 | **不绑定任何引擎或云厂商**，立志做行业公共标准 |

Hudi 偏 Hadoop、Delta 偏 Spark、Paimon 偏 Flink——都有娘家。**Iceberg 没有娘家，这正是它最大的特点**：在碎片化的行业里，"标准"是最有长期价值的叙事。围绕四格式形成三种理念：湖仓一体（Databricks）、流式湖仓（Paimon 主场）、开放湖仓（Iceberg 阵营，强调不被厂商锁定）。

> Paimon 详见 [[paimon]]。选型对比详见 [[table-format-selection]]。

## 国内落地

- **腾讯云** DLC/EMR 以 Iceberg 为核心；TC-Iceberg 两表拆分方案（change store + base store）支持 CDC 流式消费。
- **字节跳动** 基于 Iceberg 做 EB 级 ML 特征存储（行存转 Parquet + 二次开发解决特征回填）。
- **小米** 4000+ 张 Iceberg 表、8PB 数据，CDC 入湖 + Schema 变更同步。
- **华为云** MRS/DLI、**阿里云** MaxCompute/EMR（Iceberg + Paimon 两条腿）、**网易数帆**（Amoro 项目，贡献自适应合并）均有大规模实践。

## 在跨云 Lakehouse 中的角色

Google Cloud 通过 Iceberg REST Catalog 实现跨云 Lakehouse：跨 AWS/Azure 的开放 catalog，双向联邦读取 Databricks Unity Catalog、Snowflake Polaris、AWS Glue Data Catalog，最小化出站成本。Iceberg 的中立性使其成为跨云/跨引擎互操作的事实标准。详见 [[agentic-data-cloud]]。

## Catalog 类型

Iceberg 的 Catalog 层（管理元数据指针）支持多种实现（来源 Iceberg Definitive Guide 第 5 章）：

| Catalog | 特点 |
|---------|------|
| **Hive Catalog** | 最常用，映射到 Hive Metastore，兼容已有 Hive 生态 |
| **AWS Glue Catalog** | 托管在 AWS Glue，与 AWS 服务集成 |
| **Nessie Catalog** | 提供 Git 式版本控制（分支/合并/标签），支持数据工程化 |
| **REST Catalog** | 标准化 REST 接口，解耦客户端与 catalog 实现；Gravitino 即提供 REST Catalog（见 [[gravitino]]） |
| **JDBC Catalog** | 用关系数据库存元数据指针，轻量 |

跨云/跨引擎互操作推荐 REST Catalog（中立标准）；需要数据版本管理选 Nessie；已有 Hive 生态选 Hive Catalog。

## 小文件问题与 Compaction

流式写入会产生大量小文件，导致**小文件问题（small files problem）**——影响查询规划速度与对象存储请求效率。解决方法是**定期 Compaction**：把多个小文件合并成大文件。Iceberg 提供 `Rewrite Datafiles`（合并数据文件）和 `Rewrite Manifests`（重写清单文件）两种操作，支持多种 Compaction 策略（如 bin-pack、sort）并可自动化调度。配合排序（data layout optimization）可进一步提升查询性能（文件裁剪更有效）。

## 写入的原子提交

Iceberg 写入通过**原子切换 catalog 指针**实现事务：写入产生新数据文件 + 新 manifest + 新 metadata.json，最后一步是原子更新 catalog 中"当前 metadata 指针"（compare-and-swap）。若提交前有并发写入，靠乐观并发控制 + 重试（`commit.retry.num-retries`）处理冲突。这保证读操作永远看到某个一致快照，写入要么全部生效要么失败。

## 待补充

- 与 Flink/Spark/Trino 集成的具体配置（各引擎参数差异）

## 相关页面

- [[open-data-stack-evolution]] — Iceberg 在五层开放数据标准栈中的位置（表格式层）
- [[apache-polaris]] — 目录层，Iceberg 的上层标准（基于 Iceberg REST Catalog）
- [[apache-ossie]] — 语义层，Iceberg 的再上层（为 Iceberg 表附加业务语义）
- [[apache-arrow]] — 内存格式层，Iceberg 的下层依赖
- [[apache-parquet]] — 文件格式层，Iceberg 的数据文件基础
- [[agentic-data-cloud]] — Google 跨云 Lakehouse 战略依赖 Iceberg 开放标准
- [[apache-flink]] — Flink 常作为 Iceberg 的写入引擎（Flink CDC → Iceberg）
- [[flink-cdc]] — CDC 数据常写入 Iceberg 湖
- [[ltap-architecture]] — LTAP 架构中 Iceberg 作为开放存储层
- [[paimon]] — Apache Paimon（流批一体表格式，与 Iceberg 对比）*待建*
