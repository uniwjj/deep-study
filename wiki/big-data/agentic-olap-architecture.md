---
title: 面向 Agentic 的 OLAP 架构探索
description: 微信技术架构部冯吕在 2026 Agent 大会的演讲——OLAP 遇见 Agentic：Langfuse+ClickHouse 的 Agent 可观测系统改造、OLAP 向量检索与 DiskANN 记忆底座、Agent 访问保护
aliases: [Agentic OLAP, OLAP Agent 化, Agent 可观测性, Agent 记忆底座, DiskANN, Langfuse, 冯吕, 微信技术架构部]
tags: [big-data, ai-agent, architecture, practice]
sources: [2026/08/11/面向Agent的数据架构论坛/04-冯吕-面向Agentic的OLAP架构探索.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 面向 Agentic 的 OLAP 架构探索

> 演讲者：冯吕，微信技术架构部
> 论坛：2026 Agent 大会「面向Agent的数据架构论坛」
> 四大主题：01 OLAP 遇见 Agentic · 02 Agent 可观测性探索 · 03 Agent 记忆底座探索 · 04 Agent 访问保护

## 01 OLAP 遇见 Agentic：背景

Agentic 时代的到来：Agent 广泛进入日常生活工作中；软件交互范式变革——人操作工具 → Agent 自主决策与执行。Agent 核心能力：感知 → 记忆 → 推理 → 行动（感知层：多模态输入解析；短期/长期记忆；LLM 推理引擎；行动层：工具调用与执行；安全层：权限与审计）。**OLAP 基础设施转型：服务人 → 服务 Agent。**

微信常见 Agent 业务场景：视频号小助手（AI 搜索接入）、选礼物助手、AI 志愿助手（如高考 590 分对应 2026 年广东物理类全省第 40553 名，超本科线 165 分、超特控线 51 分，处于中上 211 院校和优质省属重点高校可报区间）、平台客服、领域助手。

三大挑战：Agent 可观测性（传统可观测工具在 Agent 场景失效）、记忆存储/检索（Agent 工作需要高效地记忆存储检索和召回）、Agent 访问保护（OLAP 访问主体从"人"变成"Agent"）。

## 02 Agent 可观测性探索

### Agent 可观测 vs 传统可观测

观测对象变化：系统状态 → 决策过程。传统可观测为确定性系统设计：路径固定、指标固定、成功标准固定（固定拓扑，调用路径部署时确定；系统指标 QPS/延迟/错误率；HTTP 200 即视为正常）。

Agent 场景四个结构性盲区：

| 盲区 | 问题 |
|------|------|
| 01 动态路径 | 无法提前埋点，运行时生成点失效；Agent 不是一条固定链路，而是一段动态决策 |
| 02 成本不透明 | Token 随上下文滚雪球，难归因 |
| 03 静默偏差 | 技术成功但语义错误 |
| 04 多轮交互 | Trace 与上下文数据爆炸 |

Agent 可观测：不止是"监控"，而是"理解"——**Agent 走了哪条路？做的对不对？钱花在哪里？数据是否撑得住？**

### Langfuse + OLAP：构建大模型 Agent 可观测系统

数据产生侧：Agent/应用（LLM 调用、Tool Call、Memory、多轮会话）经 Langfuse SDK 上报（Trace/Span/Generation、Prompt/Dataset、Token/Cost/Score）。可观测结果功能层：

- **Trace 追踪**（User/Session/Trace、Generation/Span/Tool Call）→ 完整还原动态执行链路（走了哪条路？）
- **Evaluation 评估**（正确性、完整性、幻觉；LLM-as-a-Judge，LLM 评估 + 人工校准）→（做得对不对？）
- **Dashboard/Cost**（延迟、Token、成本；Token/Cost 逐层归因；质量趋势与问题定位）→（钱花在哪里？）

Trace 数据落盘 **ClickHouse**——OLAP 数据底座：高吞吐写入、多维分析。

### 为什么选择 ClickHouse

| 挑战 | 解决 |
|------|------|
| 挑战 1：大规模 LLM 观测场景下，Trace 的输入输出包含海量自然语言大文本或半结构化 JSON 数据，且通常需要长期留存 | 极致存储压缩：列存 + 稀疏索引 + 多级压缩/稀疏编码 + 冷热分层，数据存储成本降低至传统方案 1/10 |
| 挑战 2：观测数据包含大量随业务变化的扩展属性（metadata、tool calls 等），字段不固定，结构差异性大 | 原生半结构化类型 JSON/MAP 等支持：兼顾扩展属性的灵活接入和高性能多维查询 |
| 挑战 3：可观测平台承载的查询具有强烈 OLAP 属性（时间窗剪裁、多维度联合过滤、高吞吐指标聚合等），低延迟要求 | 极致查询性能：极致 IO 剪裁 + 先进执行引擎设计 + 稀疏索引/主键索引剪裁 |

### 社区版 Langfuse 弊端

1. **仅支持 ClickHouse 单分片存储**：存储、写入吞吐、大数据量下的查询性能都受限于单机性能；微信业务场景 Agent 调用频率高、产生数据量大，单机瓶颈突显（社区关于 ClickHouse multi-shard 支持的讨论 #12158、#5021 迟迟没有进展）
2. **接入性能差**：原有接入链路基于 S3 历史数据回放 + RMW 来进行合并更新——读放大（ListFiles + 全量下载 + merge）、写入产生大量 ClickHouse 点查；Redis 队列积压（高并发写入时 ingestion queue 堆积）；每次 ingest 请求都先上传 JSON 到 S3
3. **大数据量下查询性能退化，多模态点查性能差**：Traces 表主键粒度天级别，id 查询需要扫描一整天；对 input/output 的 ILIKE 匹配需要全表扫描；双实体模型 traces + observations + scores 多表 Join；推理等多模态日志数据（视频号截帧数据）存储在数仓中，列存点查性能差

### 改造实践

- **分布式架构改造**：消除单节点存储和写入物理瓶颈，写入和查询分析在物理上得到分散与解耦；配合多副本的高可用和低成本冷热存储下沉，为大规模 Agent 调用可观测性构建了稳固、弹性且可线性扩展的高性能存储底座（单分片 ClickHouse → 多分片 ClickHouse 集群，Langfuse 多副本 R1/R2/R3）
- **高吞吐接入链路**（面向大规模 Agent 接入，V4 兼容 OTEL）：Agent 封装 API → 官方 OTEL exporter / 自定义 JSON → Sinker 流式消费 → Pulsar → ClickHouse；langfuse V4、meta manager、Langfuse Web、BI 平台共享（SQL 分析、报表）
- **查询性能提升（迈向 V4）**：Trace-centric → observations-centric；从 mutable traces & observations 表演进为单一 immutable observations 表；内核增加行级索引和后过滤能力，提升点查性能
- **SDK 规范化上报**：Trace 数据除用于观测外，还用于 SFT 训练、模型蒸馏训练；问题：Langfuse SDK 版本多、接口种类多，不同用户上报格式不统一，不规范的上报导致后续难以利用；解决：规范化上报标准和上报 SDK、上报文档（Skill，供 AI 写出符合规范的上报代码）、OpenAI 数据格式

## 03 Agent 记忆底座探索

### 为什么 Agent 需要记忆

把历史信息变成当前决策可用的上下文：用户请求 → 记忆检索（Recall）→ LLM 推理（Reason）→ 工具/响应（Action），读、写回。价值：连续性（跨轮次保持上下文）、个性化（记住用户偏好）、高效率（历史结果复用）、可成长（沉淀反馈与经验，Agent 越用越好）。短期记忆（当前会话/Working context）+ 长期记忆（历史事件/用户偏好/知识/技能）。没有记忆，每次请求都是第一次；有了记忆，Agent 才能持续服务。

### 生产级 Agent 常见记忆组织形式

- **短期记忆**（Working memory）：当前任务状态、最近工具调用结果；Context Window / Session State / Checkpoint
- **Skill / 程序性记忆**：规则、步骤、工作流、工具用法（搜索、数据库、文件、外部服务）
- **mem0 memory manager / 长期记忆管理**：提取、筛选、召回、更新、合并、遗忘；用户事实、偏好、历史经历等
- **Memory Storage**：向量数据库、KV/Redis、远程文件等

算法同学关注记忆如何提取、筛选、更新等；我们关注**如何提供更好的记忆存储服务**。

### 为什么记忆需要远程存储

大规模运行的 Agent 实例通常使用沙箱作为执行环境；Agent 实例可以销毁、重启、扩容、迁移，但记忆不能跟着实例一起消失；多 Agent 实例经常需要读取同一批文档、规则、代码或 RAG 语料。因此需要远程存储保证 Agent 记忆的持久化（Harness 执行环境中 Agent Loop 通过 FileSystem/MCPs/Tools 访问本地磁盘或 NFS mount）。

### 从 OLAP 数据库到 Agent 记忆底座

**OLAP 向量检索：不止于 OLAP，BI 2.5+ ANN 混合检索 + 聚合分析 + 综合排序；一站式 SQL 分析能力：语义检索 + 结构化检索，单条 SQL 完成复杂过滤、检索、统计、排序。** 对比常规 ANN 向量检索（Vector Index → Vector Search → Cosine/L2 Distance，按向量距离排序返回 [Doc, Score]），OLAP 向量检索将 Vector Index + FTS Index + RDB SQL 结合为 Hybrid Search，支持 FILTER、HAVING、ORDER BY、GROUP BY 等关系数据库能力。

**亿级数据的磁盘向量索引 DiskANN 支持**：通过 PQ 量化对 vector 进行压缩，压缩后的 vector 放在内存中，graph 和全精度向量放在 SSD 上。效果：内存占用相比 HNSW 降低 90% 以上，QPS 降低 40%。（数据量 400 万/1 亿/10 亿档位，内存占用从 HNSW 的 GB 级降至数百 MB～数十 GB 级。）

### 大规模 Agent 服务场景支持

多 Agent 在线服务场景对 QPS、延迟、稳定性提出更高要求。基于微信后台 Svrkit 框架的高 QPS 路由架构：Pivot 按 Agent 哈希路由（集群/指标限流、水平扩容、模板注册）；只读/写入节点分离、副本间读写分离（R1/R2/R3）；向量索引、全文索引、混合检索、列式存储；Executor Pipeline / Vectorization Execution（KaLm ScaleDB，SQL 关系数据库，Shard1..n）。

### 记忆服务：基于 OLAP 数据库 + 面向文件系统

- **Memory API/SDK**：提供通用 Memory 访问 API/SDK，远程存储使用 OLAP 向量数据库，Agent 通过 memory API 访问远程存储
- **面向文件系统的记忆服务**：文件系统是 Agent 沙箱最自然、最统一的上下文访问接口，Agent 无需学习一套新的 Memory API，简单易用
- **架构**：Harness 的 Agent Loop 中 memory 组件扩展整合 OLAP 向量数据库能力（QueryPivot 提供 OLAP 查询、KV 查询、向量检索、全文检索、混合检索；Columnar Storage、Vector Index、FTS Index），为 Agent memory 提供底座能力支持，提供通用记忆存储 API
- **面向文件系统的 Agent 记忆存储**：以 OLAP 向量数据库为远程存储，把文件形式组织的"上下文和记忆"以文件形式挂载在沙箱执行环境中（Sandbox：Local Disk + [mount] AgentFS + NFS/DFS）。Agent 无需学习 SQL 语法、无需对接复杂数据库 API，以 LLM 最熟悉的命令来访问数据库。应用：Skill 挂载、文件型知识库、版本快照与回溯、memory 元信息挂载
- **多层文件视图**：通过 OverlayFS 支持多层文件视图，通过指定不同的 (User, Group, Agent) 可以获取不同的文件目录树（上层覆盖下层）——User 层文件视图、Group 层文件视图、Agent 层文件视图合并文件目录树；OLAP 数据库 + 本地数据库缓存（SQLite）

## 04 Agent 访问保护

从"人"访问到"Agent"访问：

| 维度 | 过去："人"访问数据库 | 现在："Agent"访问数据库 |
|------|---------------------|------------------------|
| 特征 | "有限次、有意图、可追责"的慢操作 | "无限次、按规则、难归因"的自动化操作，"微小错误"可能"瞬间放大" |

风险：权限滥用（Agent 访问常被过度授权）、SQL 注入与恶意载荷、审计与追责困难（数据泄露更加隐蔽）。

如何保护——**把"Agent"当"人"看**：
1. **身份注册**：每个需要访问数据库的 Agent，在上游进行"身份"注册，获取数据库访问权限
2. **权限管控与审计**：数据库访问需要携带对应的"身份"信息和动态票据进行权限认证；审计日志上报
3. **限流**：默认限流，根据身份信息进行限流
4. **异常访问快速封禁**

## 相关页面

- [[selectdb-agent-native-infra]] — 同论坛 SelectDB 马如悦：Agent Native 数据基础设施（Agent 可观测存储与 OLAP 的另一实践）
- [[realtime-data-warehouse]] — 实时数仓（ClickHouse 类 OLAP 的架构背景）
- [[data-agent-semantic-stack]] — Data Agent 8 层语义栈（Agent 记忆与权限治理的架构思想）
- [[agentic-data-cloud]] — Google Agentic Data Cloud（数据平台为 Agent 提供上下文的演进）
- [[streaming-interactive-data-agent]] — 同论坛许鹏：流式交互与实时洞察（数据 Agent 产品化）
