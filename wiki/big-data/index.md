---
title: 大数据技术
description: 大数据技术领域索引——数据湖、湖仓一体、流批一体、表格式、流计算引擎、实时数仓
aliases: [big-data, 大数据]
tags: [big-data, meta, summary]
sources: [2026/05/10/lint-stub.md, 2026/07/26/从 Arrow 到 Iceberg 到 Polaris 到 Ossie：语义标准化的最后一块拼图.html]
created: 2026-05-10
updated: 2026-07-27
---

# 大数据技术

大数据技术栈——存储格式、计算引擎、架构范式。聚焦纯技术本身，Data Agent 与 AI 数据平台相关内容见 [[ai-agent/data-agent/index|Data Agent 与 AI 数据平台]]。

## 架构范式

- [[open-data-stack-evolution]] — 五层开放数据标准栈十年演进：Parquet→Arrow→Iceberg→Polaris→Ossie
- [[ltap-architecture]] — Databricks LTAP 架构：存储层统一 OLTP/OLAP 的"第三条路"
- [[lakehouse]] — 湖仓一体：数仓→数据湖→湖仓一体演进、存储层统一、批流一体（计算层统一）、CIDR 理论源头、大厂落地
- [[realtime-data-warehouse]] — 实时数仓：离线→Lambda→Kappa 演进、ODS/DWD/DWM/DWS/ADS 分层、Lambda vs Kappa 选型、实时数仓分层实践、实时计算平台化

## 流计算引擎

- [[apache-flink]] — Apache Flink 流处理引擎：有界/无界流、有状态计算、运行时架构与四层图
- [[flink-checkpoint]] — Flink Checkpoint 容错：Chandy-Lamport 分布式快照 → 异步 Barrier → 对齐/非对齐
- [[flink-state-backend]] — Flink 状态后端：Keyed/Operator State、Memory/Fs/RocksDB 选型
- [[flink-cdc]] — Flink CDC 变更数据捕获：增量快照(DBLog+FLIP-27)、与 Debezium 协同、3.0 pipeline 演进

## 表格式与数据湖

- [[apache-parquet]] — Apache Parquet 列式文件格式（五层栈最底层基石）
- [[apache-arrow]] — Apache Arrow 跨语言列式内存格式（五层栈内存交换层）
- [[iceberg]] — Apache Iceberg 开放表格式：三层元数据架构、快照/Manifest、分区与 Schema 演进、v1→v3、四格式定位对比
- [[paimon]] — Apache Paimon 流式湖仓：Streaming First、LSM Tree、三大核心能力(主键表/Changelog Producer/Merge Engine)、批流一体表抽象

## 元数据治理

- [[apache-polaris]] — Apache Polaris 开放目录：Iceberg REST Catalog、Snowflake+Databricks 联手推进
- [[gravitino]] — Apache Gravitino 统一元数据湖：Metalake→Catalog→Schema→Table 四层模型、统一 REST API、与 Hive Metastore 关系、RBAC 授权下推、AI 模型元数据

## 语义层

- [[apache-ossie]] — Apache Ossie 语义层：YAML 字段映射、指标口径、同义词标注，让 AI Agent 和 BI 工具理解数据含义

## 选型对比

- [[table-format-selection]] — 湖格式选型：Iceberg vs Paimon vs Hudi vs Delta 横向机制对比、腾讯真实选型决策、按场景建议、国内格局

## 平台与产品

- [[maxcompute-data-ai]] — MaxCompute Data+AI 演进
- [[odps-multimodal-agentic-upgrade]] — 下一代 ODPS：全模态引擎和 Agentic 全面升级（2026-08 Agent 大会）

## Agent 时代数据基础设施（2026-08 Agent 大会）

- [[ai-native-data-platform-vision]] — AI 时代的数据基础设施展望（腾讯云专场）
- [[tencent-ai-dlc]] — 腾讯云智能数据湖计算 AI DLC 发布（腾讯云专场）
- [[tencent-ai-dlc-engines]] — AI DLC 三大核心引擎解读（腾讯云专场）
- [[bosch-ray-hybrid-computing]] — 博世 Ray 混合计算实践（腾讯云专场）
- [[bosch-lance-ray-data-production]] — Bosch 自动驾驶数据生产：Lance 与 Ray（腾讯云专场）
- [[selectdb-agent-native-infra]] — SelectDB Agent Native 数据基础设施：让数据库成为 Agent 的第一等公民（2026-08 论坛）
- [[agentic-olap-architecture]] — 微信面向 Agentic 的 OLAP 架构探索：可观测改造与记忆底座（2026-08 论坛）

## 相关页面

- [[ai-agent/data-agent/index]] — Data Agent 与 AI 数据平台（原 big-data/ 下 Data Agent 内容已迁此）
- [[homepage]]
