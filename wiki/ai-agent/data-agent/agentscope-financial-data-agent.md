---
title: 基于 AgentScope 的金融 DataAgent 平台
description: 瓴岳科技基于 AgentScope Java 的金融数据平台 Agentic 进化之路——六层分层架构+横切护栏、DataPilot/DataAgent/DataBench 三服务协同、喂得准/改得对/敢执行三支柱的生产级落地实践
aliases: [AgentScope金融数据平台, 瓴岳科技DataAgent, Agentic进化之路, AgentScope Java DataAgent, 王博数据平台]
tags: [big-data, ai-agent, practice]
sources: [2026/08/11/从Data Agent到Data Engineer Agent论坛/03-王博-Agent驱动旅游B2B：全球旅游资源分发的数据架构智能化升级之.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 基于 AgentScope 的金融 DataAgent 平台

> 演讲者：王博 · 瓴岳科技
> 场合：DataFun「AGENTIC AI 超级智能体系统架构峰会」· 2026-08
> 注：本 PDF 文件名标注为「Agent驱动旅游B2B：全球旅游资源分发的数据架构智能化升级之」，但 PDF 正文标题页实际为《基于 AgentScope 的金融数据平台：Agentic 进化之路》，本页以正文实际内容为准。

## 01 数据领域的 Agentic 演进

**2026：Agentic 规模化落地，垂直数据 Agent 涌现**：从 Copilot 补全，迈向 Agent 自主完成任务；标杆已商业化（阿里云 DataWorks Data Agent、Databricks Agent Bricks）；能力质变：推理 → 调工具 → 执行 → 纠偏 → 记忆的闭环；但能力只是入场券，「生产级」才是真正的分水岭。

> "The core agent loop is ~1% of the work; the other 99% is deployment, security, evaluation, monitoring, context." — Databricks · DAIS 2026

**生产级门槛：金融数仓的复杂度与三维要求**。现实复杂度：全球化部署、协作链路长、表多且口径不容错。三维要求：

- **喂得准**（知识可信）：元数据规范化 · 口径统一 · 脏数据清洗
- **改得对**（效果可控）：系统可观测 · 效果可评测 · 回归可追踪
- **敢执行**（边界可守）：高危拒答 · 权限前置 · 全链路审计 · 不越权

而这三维，恰恰是通用 Agent 框架给不了的。

## 02 方案选型与 DataAgent 整体架构

### 框架选型：为什么落在 AgentScope Java

- 年初横评主流框架：Spring AI / LangChain / LangGraph / Google ADK / AgentScope
- 定位差异：Spring AI 强在「接入抽象」，AgentScope 强在「Agent 运行时」
- 要的正是运行时：原生 ReAct、Hook 门控、PlanNotebook 开箱即用
- 决定性主因：Java 技术栈一致 · 部署/CICD/监控无缝 · 与官方团队共建
- 实践验证：稳定支撑平台化 Agentic 落地

### DataAgent 六层分层架构 + 横切护栏

六层严格单向依赖，禁止环依赖；业务 Agent 三源上下文：Tools · Skills · RAG；横切护栏两阶段站岗：请求前 + 行动前。图例：蓝＝框架内核 · 橙＝业务 Agent（含 Tools/Skills）· 绿＝安全/质量护栏 · 箭头＝严格单向依赖。

| 层 | 职责 | 能力 |
|----|------|------|
| 接入层 | 对外入口，请求接收与身份边界 | REST · SSE 流式 · 身份鉴权 |
| 编排层 | AgentScope 收口 / Agent 运行时编排中枢 | 意图识别 · Memory · Checkpoint · Plan · 流式编排 |
| 上下文工程 | 为 Agent 装配「喂得准」的知识 | RAG 检索 · 知识装配 · Token 预算 |
| 业务 Agent 层 | 三源上下文喂养 · 增量加 Agent/工具/技能，不改内核（「增量扩展」） | 业务 Agent（任务开发 / 数据地图）· Tools 工具（校验/执行/权限/探查）· Skills 技能（编码规范 · 交互卡 · Plan） |
| 集成层 | 对接外部模型与平台能力，执行与元数据经此 | LLM · Embedding · Rerank · 平台 OpenAPI |
| 存储层 | 向量、缓存、结构化存储底座 | PostgreSQL · pgvector · Redis |

横切护栏（绿色竖带，贯穿接入—编排—业务 Agent）：**请求前**——安全护栏 · 交互原语（权限 / 澄清 / 上下文缺失）；**行动前**——质量门 · SQL 类型白名单（权限校验 · 用户确认卡）。拦截/改写/终止——危险操作绝不放行，边界由业务自持。

### 全链路 Harness：Agent 编织进数据加工链路

不是外挂聊天窗，而是介入建模→开发→校验→发布→质量→运维。左翼上下文工厂「喂得准」，右翼评测飞轮「改得对」，中心内核在受控边界里「敢执行」。全场主线三角：喂得准 × 改得对 × 敢执行。

### 三服务协同：职责边界与调用闭环

- **DataPilot 做基座**：平台经 OpenAPI 提供知识与执行
- **DataAgent 长其上**：Agent 只管编排/意图/Plan/上下文
- **DataBench 在旁护航**：Trace 上报 → 评测回归 → 反向迭代

## 03 落地挑战与解决思路

### 从 Demo 到生产：三大落地挑战

1. **ReAct 可靠性**：元数据幻觉、推诿跳过工具、高危误操作
2. **工程集成**：流式中断丢消息、跨线程上下文、Plan 跨请求恢复
3. **效果量化**：改好还是改坏，缺一把可信的标尺

### 三支柱 ↔ 三挑战

- **敢执行**（能力）：意图 · 生成执行 · 门控 · Plan
- **喂得准**（上下文）：上下文工程 + 上下文工厂
- **改得对**（评测）：评测飞轮 + LLM 评委

### 意图识别：先便宜后贵、逐层短路

- 短路优先：前端带意图 / 指定业务域，直接跳过分类
- 串行三层：L1 规则（毫秒拦截）→ L2 LLM 分类 → L3 安全护栏
- 现状规模：2 个业务域、20 个一级意图
- 扩能力只在路由表加处理器，内核不动

### 代码生成 + 执行闭环：自动落盘与版本回退

- 生成 → 多层校验 → 执行 → 报错回流纠偏，形成闭环
- 自动落盘：只覆盖开发版草稿，生产版本零污染
- Checkpoint：会话内快照，随时回退历史版本
- 交互卡（HITL）：权限 / 澄清 / 高危变更，人工确认才继续

### 工具门控：把边界站在「行动前」

- 执行前先过质量门 / 安全栅栏，而非事后补救
- SQL 类型白名单：放行 SELECT/INSERT/只读，拒绝 DDL/DELETE/UPDATE
- 四出口分流：直接拒绝 · 改写自愈 · 交互卡 · 安全放行
- 「能不能执行」沉淀成规则，不靠模型自觉

### Plan 模式：人机协同 + 三轨持久化

- 如何进入：由运行模式决定，与复杂度无关、非模型自主
- 两阶段状态机：规划 → 硬停等确认 → 自动执行
- 三轨持久化：对话记忆 · 会话快照 · 计划独立存储
- AgentScope 管运行时，Plan 真值业务自己存

### 上下文工程：两层装配 + 动态 Token 预算

- 两层装配职责切开：编排前装代码/摘要/引用；schema 与知识留到 ReAct 按预算装
- 好处：避免编排层与运行层两套 Token 配置漂移
- 表名精确预取 + 精简模式，语义知识按需调工具拉
- 预算降级：先裁 RAG 知识 → SQL 换占位 → 最后整体截断
- 对话历史独立治理：滑动窗最近 x 轮 + 超阈值摘要压缩

### 上下文工厂：从源头规范化到多领域知识库

- 源头规范化：数据探查 → LLM 元数据推荐 → 人工复核（AI + 人 双保险）
- 按业务域沉淀：表 / 字段 / 指标 / 枚举组织成数仓知识库
- 保新鲜：增量同步 Agent RAG，Schema 变更触发更新
- 分工：数仓定义知识、Agent 消费知识——真值在业务手里

### 数据飞轮：线上问题自进化为评测样本

- 线上调用 → 一键转用例 → 回归重放 → 三类打分 → 驱动迭代
- 闭环底座：全链路追踪打业务标签，线上/评测流量隔离
- 只认结构化卡片里「被采纳的 SQL」作标准答案
- 虚线「优化上线」回到线上，形成自进化闭环

### 评测引擎 + LLM 评委

- 四层执行：评测方案 → 回归批次 → 单用例 → 单次执行
- 快照锁定中间两层：改了评测集，历史结果仍可复现
- 自动打分：语法校验 + 结果集行级对比
- LLM 评委：把「意图理解 / SQL 质量」等主观维度量化
- 踩坑：失败用例也要「早退」打分，否则永远卡在待打分

## 04 未来演进与总结

### 团队共建：平台 × 数仓，缺一不可

- 痛点：平台研发离一线业务远——懂框架，不懂业务口径
- 贴近真实场景：真实任务 / 痛点 / 口径由数仓输出
- 评测与数据共建：标准答案、评测口径、历史样本靠数仓把关
- SOP 沉淀为技能模板：Agent 按公司规矩产出
- 数仓管真值、标准与知识，平台管编排、工程与护栏

### 总结与挑战：架构立住了，但仍在路上

- 已立住：分层架构 + 全链路 Harness + 喂得准/改得对/敢执行 三支柱
- 待补齐：Prompt 调优管理 · 工具治理 · 长程任务 · 数据持续迭代
- 横向展望：一个底座多领域生长（数据地图已上线，运维/质量规划中）
- 纵向深化 + 横向扩展，才是长期可用的关键
- AgentScope 管运行时，业务管真值与边界

## 与本知识库论述的关联

- 六层架构的"横切护栏两阶段站岗"与 [[agent-hook-governance]] 的 Hook 门控治理同构；SQL 类型白名单、四出口分流是"把规则沉淀为代码而非模型自觉"的实践
- 上下文工厂（数据探查 → LLM 推荐 → 人工复核）与 [[tencent-data-agent-practice]] 的 AI 增强元数据（生产 SQL → AI 反推 → 用户确认）是同一范式的两个实现
- 评测飞轮（线上问题转用例 + 快照复现 + LLM 评委）呼应 [[data-agent-practice-guide]] 的"置信度保障 + 人在环路"
- AgentScope 管运行时、业务管真值，与 [[dataworks-data-agent]]、[[databricks-agent-control-plane]] 的平台化控制面思路一致

## 相关页面

- [[tencent-data-agent-practice]] — 腾讯 TEG Data Agent 设计实战（同届峰会）
- [[agent-hook-governance]] — Agent Hook 治理（门控思想）
- [[data-agent-practice-guide]] — 火山引擎数据智能体实践指南
- [[data-agent-semantic-stack]] — Data Agent 8 层语义栈
- [[dataworks-data-agent]] — DataWorks Data Agent 产品架构
- [[databricks-agent-control-plane]] — Databricks Agent Control Plane
- [[agent-design-paradigms]] — Agent 设计范式（ReAct/Plan-and-Execute）
- [[dataworks-2026-0528-xialiaori]] — 阿里 DataWorks 2026-05-28 虾聊日分享汇总
