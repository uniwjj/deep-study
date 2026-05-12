---
title: DataWorks Data Agent
description: 阿里云 DataWorks 双引擎驱动的全托管数据智能体——代码 Agent + 运维 Agent 共享统一数据语义与上下文，MCP 协议开放 Skill 生态，覆盖数据开发、治理、分析与引擎管控运维
aliases: [DataWorks Data Agent, 数据智能体, DataAgent]
tags: [big-data, ai-agent, platform, tool]
sources: [2026/05/12/01. DataWorks Data Agent.pdf, 2026/05/12/DataWorks DataAgent分享录音.md]
created: 2026-05-12
updated: 2026-05-12
---

# DataWorks Data Agent

![产品架构图](sources/2026/05/12/DataWorks%20%20DataAgent架构图.png)

2026 年正式发布的新一代数据智能体产品。定位为企业大数据的“数字员工”，覆盖数据开发、数据治理、数据分析与引擎管控运维 Agent，提供全新自然语言交互体验。

## 核心架构

![DataAgent Core](sources/2026/05/12/DataWorks%20%20DataAgent%20Core.png)

### 五阶段演进路径

从辅助到自主的 AI 化演进：

| 阶段 | 产品形态 | 核心能力 |
|------|---------|---------|
| 1. 代码补全 | Copilot 行级补全 | 单行代码推荐 |
| 2. 对话问答 | Conversation / Copilot | 问答式辅助，如 Google/Baidu 替代 |
| 3. IDE Copilot | IDE 内嵌对话 | 注释、翻译、代码生成，提效 30-40% |
| 4. ChatBI | 自然语言取数 | 运营人员自助取数，准确性是关键难点 |
| 5. DataAgent | 全自动驾驶 | 端到端完成需求理解→探查→编码→上线→运维 |

> "会写 vs 会做：Copilot 帮你写，Agent 帮你做。"

### 双引擎驱动

- **代码 Agent（Code Agent / CLI 模式）**：在 CLI 中自主完成代码任务，擅长复杂任务的代码编写、文件读取和多轮交互
- **运维 Agent（Ops Agent / Claw 模式）**：嵌入企业 IM 群（钉钉/企微/飞书），通过自然语言指令自主完成运维流程，专注点状/突发场景

CLI 模式和 Claw 模式的区别：

| 维度 | CLI 模式 | Claw 模式 |
|------|---------|----------|
| 交互方式 | 黑白屏/TUI/IDE | IM 聊天窗口/手机端 |
| 擅长场景 | 代码编写、复杂工程任务 | 运维告警、快速操作确认 |
| 使用人群 | 研发工程师 | 所有角色（含非技术人员） |
| 核心优势 | 不关心过程，只看结果 | 不带电脑也能处理紧急事务 |

### ACP 网关智能路由

上层通过标准 ACP 网关，根据用户语义和需求智能路由到对应 Agent 类型。例如：
- "把某张表的数据拆成三张表" → 路由到 CLI 模式
- "帮我诊断昨天某个任务为什么报错" → 路由到 Claw 模式

结果可通过钉钉群/飞书推送。

两个 Agent 共享同一份**数据语义与上下文**（数据语义层），避免上下文割裂。

### DataAgent Core 技术架构

- **统一运行时内核**：双执行器架构（代码执行器 + 运维执行器）
- **数据语义层**：统一的数据字典、术语映射和业务上下文
- **企业级全托管**：基于 DataWorks 资源组与云原生运行时，统一承载 Agent 的调度、执行与负载

### 多端协同

![DataAgent CLI演示](sources/2026/05/12/DataWorks%20%20DataAgent%20CLI演示.png)

统一智能内核，主场景与延伸触点协同，多端一致体验：CLI 模式、IM 群（Claw 模式）、Web 控制台。

## Skill 开放生态

基于 [[agent-mcp-protocol|MCP 协议]]，引擎团队、合作伙伴、客户都能扩展：

- **一次注册，处处可用**：Skill 注册后可在所有 Agent 触达端调用
- 从“工具辅助”到“智能体驱动”的范式跃迁
- 五阶段演进路径：从增强到自主

## 关键观点

> DataAgent 不是终点，是新的起点 —— 开放架构、可扩展生态、持续演进。

## LLM 与 Tools 延迟优化

Tool 在 CPU 上的处理对 agentic workload 的执行延迟有实质性影响，需联合 CPU-GPU 优化而非仅依赖 GPU 加速（引用 Turin chiplet topology 分析）。

## 相关页面

- [[superetl]] — 菜鸟基于 DataWorks Data Agent 的 SuperETL 实践
- [[maxcompute-skills]] — MaxCompute 的 MCMCP 与 Skills 体系
- [[hologres-skills]] — Hologres CLI 与 Skills
- [[flink-skills]] — Flink Agent Skills 重塑实时计算
- [[emr-skills]] — EMR Skills 与智驾数据标注
- [[dataworks-stock-case]] — 个人量化案例：用 Data Agent 搭建行情分析系统
- [[data-agent-practice-guide]] — 火山引擎数据智能体实践指南
- [[openai-data-agent]] — OpenAI 内部 Data Agent 实践
