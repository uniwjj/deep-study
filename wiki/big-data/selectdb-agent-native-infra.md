---
title: SelectDB Agent Native 数据基础设施
description: SelectDB（马如悦）2026 Agent 大会演讲——面向智能体时代的实时分析数据库：极速实时、极致性价比（Serverless）、多模统一，Litefuse Agent 可观测与评估平台、Agentic Analytics 语义层 + MCP Server
aliases: [SelectDB, Agent Native 数据基础设施, Litefuse, Agentic Analytics, Analytics MCP Server, 马如悦, 飞轮科技]
tags: [big-data, ai-agent, tool, architecture]
sources: [2026/08/11/面向Agent的数据架构论坛/02-马如悦-Agent Native 数据基础设施：让数据库成为 Agent 的第一等.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# SelectDB Agent Native 数据基础设施

> 演讲者：马如悦，SelectDB，2026.7 DataFun「超级智能体系统架构峰会」·面向Agent的数据架构论坛
> 主张：**The Future of Analytics Is Real-time and Agentic**——当 AI 不再是稀缺品，只有各自的数据不会撞车。对数据的实时分析能力，就是下一个核心生产力。
> 三大卖点：极速实时、极致性价比、多模统一

## 面向智能体的分析（Agentic Analytics）

| 类别 | 场景 | 要求 |
|------|------|------|
| 面向客户的分析（Customer-facing Analytics） | 用户行为分析、实时推荐、风控决策 | 低延迟、高并发 |
| 数仓分析（Warehouse/Lakehouse Analytics） | 仪表盘、报表、即时分析 | 开放格式、共享联邦 |
| 可观测分析（Observability Analytics） | 故障定位、性能监控、安全审计（日志、指标、链路统一分析） | 高吞吐、极致性价比 |

## 极速实时：亚秒级响应不是优化，是底线

Agent 的分析不是"一条 SQL"，而是一个多步推理循环：一次提问触发 5~15 条 SQL，复杂对话可触发 50+ 查询，延迟放大 ×10~50。SelectDB 提供比同类系统快 5-10 倍的极速分析性能。基准测试（相对结论）：

| 基准 | 结论 |
|------|------|
| ClickBench（43 queries，1 亿行 Web 分析数据） | SelectDB 位列第一梯队，和 ClickHouse 速度相媲美 |
| SSB SF1000（13 queries，星型模型宽表 Join 与聚合） | SelectDB 比 ClickHouse 快 7.1x |
| TPC-H SF1000（22 queries，经典决策支持 Ad-Hoc 基准） | SelectDB 比 ClickHouse 快 5.2x |
| TPC-DS SF1000（99 queries，最复杂决策支持基准） | SelectDB 比 ClickHouse 快 11.0x |

### 极速实时成功案例

- **快手**：200+ 集群、百 PB 数据；日均 20 亿+ 次查询，从 ClickHouse 到 Doris 的统一 OLAP 底座；查询性能提升数倍，多表 JOIN 和湖仓查询能力大幅超越 ClickHouse；迁移平台"星移"自动化：DDL 转换、任务复制、数据校验全流程分钟级完成
- **某全球 Top-5 Web3 业务**（总部新加坡，8000 万+ 用户）：日均 100 亿交易量，P95 查询延迟 30ms；峰值 5,000 QPS，P95 <500ms，交易排名场景 P95 仅 30ms；1-3 秒数据就绪（Flink CDC 实时同步，数据产生即查，支撑风控与反洗钱）；400TB+ 交易数据，跨 AZ 高可用，零中断，30 分钟 Spark 任务缩至 3-5 分钟

## 极致性价比：Cloud Native & Serverless

传统架构的困境：负载有规律（每天/每周固定刷新，可提前预留资源）；而 Agent 负载完全不可预测（一次追问触发数十条查询，峰值可能是平时的 10~100 倍），峰值预留、大部分时间闲置、成本严重浪费。Agent 时代，数据量和分析量都会大 10-100 倍，Agentic 分析成本的高性价比变得更为关键。

Cloud Native & Serverless 的解决之道：存算分离（计算和存储独立伸缩，互不制约）、秒级弹性（高峰自动扩容，低谷自动缩回）、按量计费（只为实际查询付费，Ad-Hoc 零浪费）、免运维（团队聚焦 Agent 场景）。阿里云 SelectDB Serverless 已于 2026 年 3 月正式商业化，提供秒级弹性伸缩，大幅降低计算费用。

### Serverless / 存算分离成功案例

- **收钱吧**（连锁餐饮数字化，SelectDB Serverless）：门店年增 50%，算力成本反降 32%+。痛点：为应对峰值按最高配置，平峰闲置超 60%，多负载争抢资源、核心报表频繁卡顿。方案：秒级弹性伸缩 + 独立计算资源组实现负载隔离。效果：算力成本降 32%+，运维成本降 83%，查询性能提升 80%+
- **MiniMax**（GenAI 可观测性，SelectDB 存算分离）：PB 级日志，扩容从天级降至分钟级。痛点：自建 Doris 扩缩容需数据迁移，PB 级耗时数小时甚至天级；多业务共享实例，资源争抢频繁宕机。效果：扩缩容分钟级完成，计算资源降 40%，热存储降 50%，P95 查询 <3s

## 多模统一：打破内外边界，融合多模数据

- **内外表统一**，一条 SQL 跨源访问：Iceberg/Hive/Paimon 湖上数据、MySQL/PostgreSQL 关系库、S3/HDFS 对象存储
- **多模数据统一**，一条 SQL 混合查询：JSON 半结构化、Text 全文、Vector 向量

### 多模统一成功案例

- **洋钱罐**（瓴岳科技，全球金融科技，湖仓一体·跨源统一）：Hive 透明加速，P95 降低 90%+，日均 500 万次查询。无需数据迁移：SelectDB Hive Catalog 直连数据湖，98% Hive SQL 兼容，业务无感切换；智能路由覆盖 95% 查询，P95 响应从 300 秒降至 20 秒，缓存命中率 90%+；计算资源 1000+ Core，数据导出从 300 秒缩至 20 秒，DQC 从 43 分钟缩至 25 秒
- **HubSpot**（全球 CRM 领导者，多模统一·搜索分析融合）：统一 ES + Snowflake 为单一平台（VeloDB），消除跨系统 JOIN 瓶颈；Variant 列存原生处理 10000+ 子列的动态 JSON，自动类型推导 + 专项倒排索引；向量检索 + 全文检索 + Iceberg 湖上查询一条 SQL，支撑智能 CRM 与 Agent 实时推理

## 三种产品形态

| 产品 | 定位 |
|------|------|
| SelectDB Cloud | 全托管的极速分析数据库，多云原生，SaaS 和 BYOC 两种模式；国内上线阿里云、华为云、腾讯云和亚马逊云科技，国外上线 AWS、GCP 和 Azure |
| SelectDB Enterprise | 自管理私有化软件，部署在物理机、虚拟机或 K8s 上；长周期稳定内核，企业级安全合规 |
| 阿里云 SelectDB | 阿里云与飞轮科技联合推出的云原生分析数据库，深度集成阿里云生态 |

## 面向智能体时代的两大解决方案

1. **Agent Observability and Evaluation Stack**——Agent 可观测与评估（Litefuse，基于 SelectDB）
2. **Agentic Analytics Stack**——Analytics MCP Server（内置语义层）

## Litefuse：基于 SelectDB 的端到端 Agent 可观测与评估平台

### Agent 观测与评估的意义

确定性系统（Legacy/Cloud-native）：同样输入 → 同样输出，唯一确定的输出；AI 时代概率性系统（AI-Native, dynamic & stochastic）：同样输入 → 不同输出，一个输出的分布。Agent 不确定性挑战：大模型幻觉、路径规划错误、工具调用失败、记忆腐化。

两大能力：**Observability**（SDK/OTel Trace 采集、100+ AI 生态集成、Trace 检索与可视化、性能成本等指标分析）+ **Evaluation**（离线测试数据集评估、在线 Trace 回流评估、人工标注评估、LLM/程序自动评估）。

Litefuse 基于 langfuse 和 selectdb 构建（Langfuse 功能丰富：100+ AI 生态集成 OpenAI/LangChain/Dify、AI Native 数据模型 LLM·Tool·Token、Prompt 管理 + Evaluation），把 **EDD（Evaluation-Driven Development，评估驱动开发）** 产品化——Test-Driven Development（failing test → Write → Refactor）→ Evaluation-Driven Development（Observe → Evaluate → Improve）。

Langfuse 规模化痛点 vs SelectDB 存储优势：

| 痛点 | SelectDB 解法 |
|------|--------------|
| 存储成本高（TB 级，占 Agent 成本大头） | 存储成本 ↓88%（VARIANT 列存高压缩比，上万动态字段；存算分离，数据只写一次、存对象存储；倒排索引；MiniMax、阶跃、字节等 PB 级生产验证） |
| 架构复杂（6 组件，部署运维重） | 架构 6 进程 → 1 |
| 文本检索慢（LIKE 全量扫描） | 检索快 5-10x |

### Variant 类型：最高效存储分析 Agent 产生的数据

Agent 产生的数据挑战：Free Schema、稀疏多列（有的场景多达成千上万列）、大字段（有的字段甚至到 MB~百 MB 级别）。SelectDB 方案：VARIANT 数据类型可以存储任何合法的 JSON，自动从 JSON 中抽取字段并推断其类型，抽取的字段存储为 VARIANT 列的子列，以列存方式高效压缩存储和分析。VARIANT 的方案使得半结构化数据处理效率逼近结构化数据，同时又满足了灵活性的目的。

### 阶跃星辰：基于 SelectDB 的 Agent Trace 观测性方案

Agent Trace 对数据底座的新挑战：记录的是 Agent 决策过程（Prompt、Reasoning、Tool Call、Sub-agent 共同构成一次完整的 Agent Invoke），而不是传统服务日志；Agent 链路天然高并发、高基数、半结构化；观测平台必须同时承载实时写入、灵活检索与持续聚合；EDD 驱动，AgentOps 流程要求数据回流与标注。

能力要求（为什么 SelectDB 适合 Agent Trace）：

- **灵活事实建模**：Unique Key 支持状态补齐，明细与终态同库分析
- **宽事实链建模**：prompt/tool/半结构化存储与跨层级关联
- **灵活检索钻取**：既要 trace_id 秒级点查，也要检索 message、metadata、tool_args；VARIANT 承载动态 metadata，倒排索引加速 trace_id/文本，支持 JSON path 过滤——点查、检索、聚合同引擎
- **多维指标聚合**：从延迟/错误率扩展到 retrieval/usage 等成功率、质量、Token、成本，需要分钟级多维 Rollup
- **混合负载治理**：实时写入、明细钻取、BI 看板和长期留存并存，要求资源隔离与弹性扩展；Workload Group 隔离负载，FE + BE 线性扩展
- **可复用 Rollup**：异步 MV 沉淀多维指标，分区增量刷新，降低重算，透明改写服务 BI/API

应对运行轨迹的不确定性，提供观测手段：推理不确定性、会话复盘（多轮任务按 session 回放，定位上下文漂移、记忆错位与失败分支）、成本治理（Token、模型版本、Prompt 模板与调用来源进入统一归因路径）、元数据检索（用户、环境、模型实例、业务标签作为高基数维度直接参与过滤与倒排）、评测闭环（线上 Trace 回流 Eval/Dataset/回归集，失败样本一键沉淀）、Infra 关联（推理调度、KV cache、沙箱状态、灰度分桶与业务链路联动）。

### MiniMax：基于 SelectDB 的日志和 Agent 观测场景实践

基于 Loki 的早期日志系统 → 基于 Doris 的全新日志系统：iLogtail（采集端）→ Kafka（消息队列）→ ingester（写入端）→ Doris（日志存储），Routine Load 写入。数据规模数 PB 级，写入性能数十 GB/s，查询秒级响应。

| 维度 | 提升 | 手段 |
|------|------|------|
| 查询性能 | 2 倍 | SQL 查询语言支持 Join，相比 DSL 简单、表达能力强；高效的执行引擎和优化器，高效的索引与文本分析 |
| 存储成本 | 降低 80% | 向量化指令提升数据解析、索引构建性能；简化去掉正排等索引结构降低索引构建开销，减少倒排数据量 30% |
| 写入吞吐 | 5 倍 | 列式存储和 ZSTD 高效压缩，提供 5-10 倍压缩比；存算分离，冷热分层，大幅降低冷数据成本 |
| 稳定性 | 大幅提升 | 资源隔离和大查询限制；高效内存管理，避免 Java GC 影响 |

## Agentic Analytics：让 Agent 理解和分析企业数据

Agent 的盲区：不知道"活跃客户"=30 天有交易还是 7 天打开过；看不懂 `tbl_usr_trx_v5` 的业务含义；"流失率"按账户、用户还是设备计算？无法自行发现跨表、跨源的数据关联关系。

SelectDB 的解法：

- **语义层 Semantic Layer**：语义指标治理，Agent 查的是治理过的业务指标，而非裸表；跨源统一口径，"月收入""活跃客户"在所有数据源上一致
- **Analytics MCP Server（内置语义层）**：MCP 标准接入，Claude / Codex / Cursor 一次接入，所有 Agent 通用——数据发现、Schema、语义检索、SQL 执行

## 相关页面

- [[agentic-data-cloud]] — Google Agentic Data Cloud（云厂商 Agent 数据基础设施，同为"数据平台成为 Agent 基础设施"演进）
- [[data-agent-semantic-stack]] — Data Agent 8 层语义栈（语义层与指标治理的架构思想）
- [[realtime-data-warehouse]] — 实时数仓（SelectDB 类 OLAP 的架构背景）
- [[agentic-olap-architecture]] — 同论坛微信冯吕：面向 Agentic 的 OLAP 架构探索（OLAP 面向 Agent 的观测/记忆/访问改造）
- [[ltap-architecture]] — Databricks LTAP（数据库/数据平台面向 AI Agent 的存储层统一）
- [[maxcompute-data-ai]] — MaxCompute Data+AI 演进（大数据平台 AI 化）
