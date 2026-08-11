---
title: MIP 可治理数据智能体平台
description: 镜舟科技 MIP（MirrorShip Intelligence Platform）——从实时数据底座到可治理数据智能体的 AI 原生数据工程：L1 MirrorData / L2 MirrorMind / L3 MirrorPilot 三层联动 + Control Plane 闭环治理，NL2SL2SQL 受控语义链路
aliases: [MIP, MirrorShip Intelligence Platform, 镜舟科技MIP, MirrorData MirrorMind MirrorPilot, 实时数据底座到可治理Agent]
tags: [big-data, ai-agent, architecture]
sources: [2026/08/11/从Data Agent到Data Engineer Agent论坛/02-方营-从实时数据底座到可治理 Agent：MIP 的 AI 原生数据工程.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# MIP 可治理数据智能体平台

> 演讲者：方营 · 镜舟科技
> 场合：DataFun「AGENTIC AI 超级智能体系统架构峰会」· 2026-08
> MIP = MirrorShip Intelligence Platform

## 核心判断：为什么容易停在 demo

**Agent 时代的数据系统，要同时回答：能不能查、怎么查、为什么这样查、查完能不能信。**

| Demo 形态 | 生产形态 |
|-----------|---------|
| Chatbot | 实时数据 + 企业上下文 |
| + 裸数据库 schema | 语义口径 + 权限 |
| + 一次性 Text-to-SQL | 执行 + Trace + Evaluation |

从"能生成 SQL"升级为"能解释、可复盘、可治理、敢进生产"。

**Text-to-SQL 的生产风险**（单次 SQL 生成失败时，问题往往不在模型单点，而在缺少受控语义和执行治理）：

1. 指标口径：同名指标、时间窗口、过滤条件容易被误用
2. 权限边界：用户、字段、客户维度、敏感数据需要继承控制
3. 查询成本：高成本 SQL、资源争抢、慢查询不能只靠 Prompt
4. 异常处理：空结果、口径冲突、数据延迟需要可解释
5. 上下文缺失：字段注释、业务文档、复盘知识无法协同
6. 多轮状态：追问、修正、审批、人工确认需要任务状态

**目标：数据任务链路**——企业真正需要的是一条可执行、可解释、可审计的数据分析链路：问题（业务意图、约束条件）→ 语义（指标/维度、口径/权限）→ 执行（SQL Safety、StarRocks 查询）→ 解释（结果归因、证据引用）→ 治理（Trace、Eval / Policy）。每一步都有输入、输出、责任边界和可回放证据。

## MIP 三层联动架构

**系统定位**：不是"在数据库上加一个 Chatbot"，不是"把 Agent Runtime 外挂到数据平台"，而是把 MirrorData / MirrorMind / MirrorPilot 收敛到一条 Agent-Native 主链路。三层同体：L1 MirrorData 支撑 L2/L3，真实任务反馈驱动底座与认知自进化。

### L1：MirrorData AI Native 数据底座

L2/L3 不是凭空建设上下文层，它们依赖 L1 MirrorData（MirrorBase + MirrorLake）同时具备实时访问、检索、语义和治理能力：

- 能力面：高性能 OLAP 查询、向量检索、全文检索、开放湖仓
- 治理面：Semantic View、AI 函数、Agent 负载、权限继承

> 没有 MirrorBase + MirrorLake 的查询、检索、语义和负载治理，L2 只能堆 RAG，L3 只能赌模型。

### L2：MirrorMind 上下文治理层

MirrorMind 把 L1 能力组织成"问题相关、权限正确、来源清楚、时效可见"的上下文对象：

- Knowledge（输入）：文档 / Wiki、数据库元数据、BI 指标、业务记忆
- Memory（处理）：解析 / chunk、Embedding、混合召回、语义建模、权限继承
- Evidence（输出）：上下文对象、证据包、Skill 元数据、可审计引用

> MirrorMind 不只是 RAG，而是 Knowledge / Memory / Skill / Evidence 一体的企业上下文治理层。

### L3：MirrorPilot 可追溯 Agent Runtime

MirrorPilot 把上下文和语义变成可执行任务，人与业务 Agent 皆可用，而不是只返回一次自然语言回答：

1. 理解问题：识别业务对象、指标、约束
2. 语义计划：选择 Semantic View 与上下文
3. SV SQL 生成：生成受控查询计划
4. 执行解释：查询、归因、多轮追问

- 支持任务状态管理：执行中、待确认、审批、降级、完成
- 支持 Human Review：高风险结论进入人工确认链路
- 支持结果可追溯：最终回答可回到上下文、语义定义和 SQL

### Control Plane：闭环治理

控制平面把 MirrorData、MirrorMind、MirrorPilot 串成可观测、可评估、可策略控制的闭环：

- Trace：问题、召回、语义、SQL、结果、策略动作全链路记录
- Evaluation：SQL 正确率、召回质量、回答质量持续评估
- Policy：高成本查询、越权访问、审批阈值与降级策略

## Semantic View 到执行链路

**Semantic View：准确性桥梁**——L3 不应直接基于裸表结构猜 SQL，而应基于受控语义生成查询计划：

- 裸表结构（表名/字段名、隐含业务口径、容易猜错 join 与过滤）→ Semantic View（指标、维度、口径、过滤条件、时间窗口、业务语义可复用）→ 语义查询计划（少猜表字段、降低指标误用、结果更可解释）
- Text-to-SQL 的目标从"猜 schema"变成"基于受控语义规划查询"：**NL2SL2SQL**

**一次问数的执行链路**（示例：上个月华东区哪些客户增长最快？）：问题理解 → Context API → Semantic View → SQL Safety → StarRocks 查询 → 结果解释 → Trace/Eval。Semantic View 负责指标与维度口径，避免裸表猜测；SQL Safety 负责成本、权限和风险判断；Trace 记录每个中间结果，便于复盘和优化。

**反馈如何驱动自进化**：运行信号（慢查询、召回失败、语义漂移、语义缺口、高成本 SQL、资源争抢）→ Control Plane（Trace + Evaluation 定位责任边界、形成改进任务）→ L1/L2 迭代（索引 / MV / 缓存、分区与资源隔离、向量与全文检索、语义漂移检测、Semantic View 建模）→ 回流到 MirrorMind 的知识与记忆。

> 越用越强的前提：反馈能进入底座和认知中枢，而不是停在 Prompt 调参。

## 场景闭环与落地清单

### 场景一：自然语言问数

「上个月华东区哪些客户增长最快？」路径：问题理解 → Semantic View → SQL Safety → StarRocks 查询 → 结果解释。重点：L3 不直接裸查表；Semantic View 提供指标和维度口径，执行反馈沉淀为高频查询优化与物化视图建议。场景价值：人人可问数，且每个答案都能解释和追溯。

### 场景二：混合分析问答

「这些异常客户最近在文档和复盘里有没有共性原因？」路径：结构化查询 + 向量 / 全文混合检索 → 证据封装 → 结果归因。重点：数值结果和知识证据必须在同一条 Trace 里互相支撑，避免数据库查询与 RAG 解释各说各话。场景价值：数值结果与知识证据进入同一条 Trace。

### 场景三：高风险审批

「生成影响经营决策的异常归因报告，并访问敏感客户维度。」路径：风险识别 → Policy 判断 → Approval 状态 → 人工确认或降级 → 可审计输出。重点：高风险任务不能只靠 Prompt 约束，必须由 L1 权限/成本、L2 上下文权限、L3 状态和 Control Plane 策略共同控制。场景价值：高风险分析安全可控地进入生产。

## 工程痛点与设计原则（五条）

1. **L1 不能只停留在 OLAP**：需要 MirrorBase + MirrorLake，覆盖查询、检索、元数据、权限、成本和资源治理
2. **语义层决定准确性上限**：指标、维度、口径、时间窗口必须稳定表达
3. **结构化与非结构化要协同**：SQL 结果、知识证据和业务解释在同一 Trace 里互相支撑
4. **反馈必须回流到底座与认知中枢**：慢查询、召回失败、语义缺口、语义漂移要变成 L1/L2 迭代任务
5. **Trace / Eval 必须前置**：这两个能力要先做，错误可能来自查询、召回、语义、权限、模型或策略判断

## 与本知识库论述的关联

- NL2SL2SQL（自然语言 → Semantic View 语义查询计划 → SQL）与 [[data-agent-semantic-stack]] 的 SemQL / Text2Semantic2SQL 模式同构，是语义层范式的厂商落地
- "Trace / Eval 必须前置"与 [[data-agent-practice-guide]] 的"工程可靠性优先、置信度保障"一致
- MirrorMind（Knowledge/Memory/Skill/Evidence 一体上下文治理）对应 [[data-agent-semantic-stack]] 的锚定层与会话层的企业上下文实现
- 三层架构 + Control Plane 治理闭环可对照 [[dataworks-data-agent]]、[[databricks-genie]] 的平台化 Data Agent 路线

## 相关页面

- [[data-agent-semantic-stack]] — Data Agent 8 层语义栈（SemQL 范式）
- [[data-agent-practice-guide]] — 火山引擎数据智能体实践指南
- [[dataworks-data-agent]] — DataWorks Data Agent 产品架构
- [[databricks-genie]] — Databricks Genie
- [[data-agent-vs-ontology-agent]] — Data Agent 与 Ontology Agent 推理范式对比
- [[agentic-data-cloud]] — Google Agentic Data Cloud
- [[tencent-data-agent-practice]] — 腾讯 TEG Data Agent 设计实战（同届峰会）
- [[agent-design-paradigms]] — Agent 设计范式
- [[agent-hook-governance]] — Agent Hook 治理
