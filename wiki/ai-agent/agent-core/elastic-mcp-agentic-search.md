---
title: Elastic MCP 及 Agentic AI——可信、上下文感知的企业搜索
description: Elasticsearch 如何用 ES|QL 把 RAG 变成一次查询、用 MCP Server（elastic/mcp-server-elasticsearch）把搜索平台暴露给 agents、用混合搜索 + DECAY 时间衰减做 Agent 记忆层，并通过 Agent Builder 六大要素构建 agentic 工作流
aliases: [Elastic MCP, Agentic AI, Elasticsearch MCP Server, mcp-server-elasticsearch, Agent Builder, Elastic Workflows, 刘晓国]
tags: [ai-agent, tool, concept]
sources: [2026/08/11/企业级Agent开发工具链论坛/02-刘晓国-Elastic MCP 及 Agentic Al-构建可信、上下文感知能力的搜索.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# Elastic MCP 及 Agentic AI

> 演讲：刘晓国（DataFun · Agentic AI Summit，2026 Agent 大会「企业级Agent开发工具链论坛」）。主线：搜索市场演变 → Elastic MCP → AI agents——构建可信、上下文感知能力的搜索，把企业数据通过 MCP/API/A2A 暴露给 Agent 生态。

## 一、搜索市场演变

### 搜索市场变迁（三阶段）

| 阶段 | 定位 | 典型内容 |
|------|------|---------|
| **基于搜索的应用**（传统使用场景） | 支持广泛的使用场景 | 文档与业务数据搜索；终端用户 App 体验（例如抖音、滴滴、美团等）；内部应用（大型组织中有数百个 app）；数据库分流；电子商务；可扩展、灵活的查询与相关性 |
| **向量与混合搜索**（快速增长） | 提升相关性，并为 GenAI / RAG 奠定基础 | 语义搜索提升现有的 app 和使用场景；GenAI / RAG 工作流需要 Vector DB+；现有数据更有价值并推动新的使用场景；终端用户对开发者的期望不断提高；更好的相关性带来更好的结果 |
| **对话式与 Agentic AI**（早期采用） | 基于搜索构建的体验 | 聊天/对话式体验；基于用户参与和自动化的 AI 驱动工作流；在现有数据源之上启用新的应用类别；期望直接由终端用户驱动；需要卓越的相关性，尤其是在长尾查询中 |

客户旅程（成熟度递进）：LEVEL 1 基于搜索的应用（关键词搜索 → 全文搜索 → 分面导航 → 同义词 → 聚合 → 地理空间搜索 → NLP）→ LEVEL 2 向量与混合搜索（语义搜索 → 向量搜索 → 混合搜索 → 语义重排序 → RAG → 多语言）→ LEVEL 3 对话式与 Agentic AI（MCP 服务器 → 高级 RAG → Agent 记忆体 → 个性化 → Skills → 智能助手）。提升客户应用体验 = 收入增长；提升员工应用体验 = 生产力提升。

### Elasticsearch AI 搜索平台整体架构

- **企业数据**：数据存储和索引——向量/文本/对象/列存、日志/地理/时序/行存
- **ES|QL**：统一计算分析引擎
- **Agent 编排**：Tools / Skills / Agents / Workflows
- **大语言模型应用集成**：MCP / API / A2A / 自定义 Agents / 自定义 AI 应用（Custom applications）
- **统一推理接口**：接入 Anthropic、OpenAI、Alibaba Cloud、Mistral、Hugging Face、cohere 等模型

### 一个 API 支持最复杂的混合搜索

Elasticsearch 最强的混合搜索引擎，按管道列：

- **数据写入**：AI 生态系统、200+ 数据连接器、RAG Playground、API 写入数据、Eland 模型导入、第三方模型整合
- **数据类型**：Dense vectors、Sparse vectors、Numerics、Spatial、Temporal、Boolean、Terms、Objects、Aliases
- **召回**：Semantic Text、BM25f、kNN、Rank features、Scripts、Rules、Filters、Suggesters、Aggregations、Pre filtering
- **重排**：Jina Rerank、Rank fusion (RRF)、Linear retrieval、Reranking、Rescoring、Post filtering
- **其它**：Highlighting、Sorting、Pagination；优化——Learning to rank、Engagement data、Ranking evaluation、Benchmarking、Profiling
- **AI 整合**：云 AI 基础设施、自部署 AI、NLP 模型直接部署到 ES、第三方开源 AI 工具、第三方商业 AI 方案

AI 搜索平台能力：可组合的模板、插件、脚本；弹性自动扩展、高可用性、快照；部署、索引、文档、字段的权限管理（基于角色访问控制和基于属性访问控制）；无缝搜索跨索引、部署、区域。

### 提问的方式变了，搜索也应该跟着改变

在单个 ES|QL 查询中，自然语言查询（例如"What is ES|QL?"）可以分阶段组合：1. 初始结果检索 → 2. RRF 合并 + 语义重排序结果 → 3. 对前 n 个结果进行 LLM 摘要：

```
FROM search-index METADATA _score
| FORK (WHERE semantic_content:"what's esql?" | SORT _score DESC)
       (WHERE content:"what's esql?" | SORT _score DESC)
| FUSE
| RERANK "what's esql?" ON content WITH "elastic_rerank"
| LIMIT 6
| COMPLETION CONCAT("Summarize :", content) WITH "elasticllm" AS summary
```

**在 Elasticsearch 中，RAG 就像执行一个 ES|QL 查询一样简单。**

### 将企业数据引入 agents 是一个挑战

Chat Experiences（Claude 等）与 Custom agents（LangChain 等）接入企业数据面临三组问题：**相关性与可靠性**、**可扩展性与效率**、**集成与可延续性**。

Elastic 的解法：Chat Experiences / Custom agents → **MCP 与 API/A2A** → Tools / Agents → **Search AI Platform**。搜索比以往更相关：搜索（结果）、聊天（答案）、Agentic（自动化）三种体验分别由 Lexical Search 与 Vector/Semantic/Hybrid Search 支撑（Receive → Analyze → Match → Deliver → Capture）；目标：理解语义意图并匹配内容、提升相关性、**向 LLM 返回最少量的上下文数据**。

RAG（检索增强生成）- 消除幻觉：用户问题 + 混合搜索（检索私有数据）→ 来自数据的搜索结果 + 用户问题 → GAI/LLM（排除公共互联网数据）→ GenAI 回答。

## 二、Elastic MCP

### Elasticsearch + LLM 的力量

用自然语言（英语、葡萄牙语或中文）对数据提问并进行推理。示例（Natural Language/MCP）："At what time of day did I take the most steps exploring Las Vegas yesterday?" → 自动翻译为 Elasticsearch Query（DSL）或等价的 SQL。

### Elasticsearch MCP Server 当前状态

官方仓库 **elastic/mcp-server-elasticsearch**，当前内置工具：

| 工具 | 功能 |
|------|------|
| `list_indices` | 列出连接集群中的所有索引 |
| `get_mappings` | 检索指定索引的字段映射 |
| `search` | 执行完整的 Elasticsearch query DSL 搜索 |
| `esql` | 在索引上运行 ES|QL（类 SQL）查询 |
| `get_shards` | 检查一个或多个索引的 shard 分布和状态 |

### MCP Server + Elastic 的两种方法

1. **在 Kibana 中集成有一个 MCP Server**：Manage MCP、Manage agents、Copy MCP Server URL、Bulk import MCP tools，附 Documentation/Tutorials（如 "Building an MCP server with Elasticsearch for real health data"）
2. **定制**：领域特定、需要编码、完全可控（参考 https://www.elastic.co/search-labs/blog/how-to-build-mcp-server）

### MCP 社区贡献

elasticsearch-mcp、mcp-server-kibana，以及官方示例应用：elastic/example-mcp-app-observability、elastic/example-mcp-app-security、elastic/example-mcp-dashbuilder。

## 三、AI agents

### 什么是 Agent？

AI agent 是一种软件程序，它可以通过做出决策、从其环境中学习，并使用工具来实现目标，从而自主执行复杂任务（Input → LLM → Tools（Search/RAG/API/Maths/Library）→ observation → Agent 决定下一步 → output）。Credit: What's Your Definition Of An AI Agent? | Cobus Greyling on Medium。

### 上下文窗口与上下文工程

上下文窗口 = LLM 一次能够处理的最大 token 数量。三类问题：信息过少导致幻觉或错误响应（缺乏足够上下文，无法确定语义上下文生成准确响应）；信息过多导致上下文溢出（压垮 LLM 的注意力范围，降低整个上下文窗口的相关性，使模型难以判断哪些部分最重要）；分散或冲突信息让模型混淆（更大的上下文窗口增加了冲突或无关信息的机会，干扰 LLM 回答）。

**上下文工程**确保 agents 拥有正确的信息：Instructions/System Prompt、State/History（短期记忆）、Retrieved Information（RAG）、Available Tools、Long-Term Memory、User Prompt → Structured Output。"垃圾进，垃圾出"；"搜索没有消失，它只是换了位置"（Credit: Effective context engineering for AI agents | Anthropic）。对比：Prompt engineering（单轮查询：Context window 内 System prompt/User message/Assistant message）vs Context engineering for agents（向模型提供可能需要的全部上下文：Doc、Tool、Memory file、Message history、Tool result 等，经 Curation 组装）。

**混合搜索是各种方法中最优的**（Hybrid search is the best of all worlds）：Search Engines（SQL "Order by"、Filtering、Relevance scoring、Aggregations、Ranges）与 Vector Search（Nearest neighbor）经 Reciprocal Rank Fusion（RRF）融合，再做 Reranking（Semantic Re-ranking、Learn-to-Rank (LTR)）。

### 新兴的 agentic 模式与 Agent 记忆

新兴模式：Human question → plan →（Search）+ MCP servers → Tools → Agents → App/Stream，配合 Gen AI Memory（Long Term Memory / Short Term Memory / retrievers）。**使用 Elasticsearch 管理 agentic 记忆。**

为什么 Elasticsearch 适合做记忆（五点）：

1. **开箱即用的混合搜索**：BM25 词法评分 + 密集 vector search（指向 inference point 的 semantic_text 字段）
2. **通过 ES|QL 实现查询表达能力**：memory 是可通过完整查询语言访问的文档——支持按类型、日期范围、agent ID、访问范围或任意组合过滤，支持聚合、时间函数以及 FUSE 多 retrieval 融合；比大多数 vector store 提供的轻量级 metadata 过滤更强
3. **在语义评分之前进行 metadata 过滤**：按 agent 作用域隔离 memory、时间窗口过滤、基于类型过滤，都是标准查询能力，只需组合已有原语
4. **通过 DECAY 实现时间衰减**：系统对最近的 memory 赋予更高权重，使其排名高于过时内容
5. **你已经具备的运维能力**：监控、告警、索引生命周期管理、备份

Agent-memory 架构（在 Elasticsearch 上构建持久 agent 记忆层——MCP Server）：Claude Code Agent 的 SessionStart / PostToolUse / stop 钩子 → offline writes（fallback/outbox（offline queue）/bridge CLI）→ Elasticsearch 中按 agent 隔离的索引（agent-memory、agent-messages、agent-tasks、agent-sessions、agent-status、agent-entities 等，部分索引名 OCR 无法完全辨认）。

### 什么是 Agent Builder？

Elasticsearch 中的一个新 AI 层，提供构建 AI agentic 工作流的框架，使用混合搜索为 agents 提供其推理和行动所需的上下文。体验端点（Experience Endpoints）：Customer App（Claude、Cursor、LangChain）→ MCP / API / A2A → Tools / Agents → Search AI Platform，建立在开放标准与互操作性（Open Standards & Interoperability）、Evaluation、Security、Workflow Automation 之上，数据来自 documents、knowledge bases、logs。

Elastic 将知识暴露给 Agentic 生态系统（三层）：AI 原生体验（Chat Assistants, Voice Assistants, AI Reasoning；Custom Apps：Claude/salesforce/LangChain）→ 由工具与 agents 驱动（MCP / A2A → Tools / Agents → Search AI Platform）→ 由平台支持（Ingest / Process / Storage & Replication / Search / AI & ML Analysis / Visualization / Workflow Automation → Search AI Lake），构建于企业数据（Enterprise Data Sources：salesforce、GitHub、Confluence 等）。

### Agent Builder 功能与六大要素

功能：Agentic Search 流程（User Query → Interpret Input & Objective → Tools Selection & Execution → Analyse Tools Response → Return Response? → Final Answer to User）；使用原生对话式 agent 与你的数据聊天并查看完整的步骤和结果追踪；利用内置工具查找相关索引、理解数据结构，将自然语言翻译为结构化的语义、词汇或混合查询；使用 ES|QL/MCP/Workflow/Index search 构建自定义工具供自己的 agents 使用；内置聊天可视化（可编辑图表类型、可添加到仪表板）；使用 MCP 和 A2A 与外部 agents 和应用集成。

六大要素：

| 要素 | 说明 |
|------|------|
| 开箱即用的对话式 agent | Built in Chat Agent |
| 可复用的指令和工具集 | Skills/plugins |
| 定义 agents 可以做什么 | Tool Creation |
| 衡量、分析、改进 | Evaluation & Improvement Roadmap |
| Agent Creation (A2A) | 为你的用例构建自定义 agents |
| Tool Interface (MCP) | 通用通信协议——你如何集成 |

### 内置工具与创建 tools 的四种方式

内置工具（前缀 `platform.core`）：执行 ES|QL 查询（execute_esql）、从自然语言生成 ES|QL 查询（generate_esql）、根据 ID 获取完整文档内容（get_document_by_id）、检索一个或多个指定索引的映射（get_index_mapping）、根据自然语言查询列出相关索引/别名和数据流（index_explorer）、列出集群中的索引/别名和数据流（list_indices）、关键搜索工具（search），以及获取工作流执行状态（get_workflow_execution_status）、产品文档检索等。

创建 tools 的四种方式（Type 选择）：**ES|QL / Index search / Workflow / MCP**。

### Elastic Workflows - YAML 结构

工作流用 YAML 声明（示例 name: National Parks Demo——创建索引、加载样例数据、搜索并展示结果），结构：

- **name / description / tags**：用于标识和搜索的元数据
- **consts**：整个工作流中复用的固定值，通过 `{{ consts.name }}` 访问
- **inputs**：每次执行时变化的参数，由用户在运行时提供
- **triggers**：工作流如何启动——手动、定时、告警或 webhook
- **steps**：内部动作（在 Elasticsearch 和 Kibana 内执行的操作，如查询索引、运行 ES|QL、创建 case 或更新告警）；外部动作（在外部系统上执行的操作，如发送 Slack 消息或创建 Jira 工单，可用任何 connector，也可用 HTTP 步骤调用任意 API 或内部服务）；流程控制（条件、循环和并行执行）；AI（从向 LLM 提示，到将 agent 作为工作流步骤启用，解锁 agentic 工作流用例）

Agent 的组成部分（Agent Builder 界面）：Agent ID（property_search_skill）、Skills（Search for property）、Tools/Skills/Plugins 计数、System references、Custom Instructions（"You are an information extraction assistant. Extract real estate search parameters from the user query."）、Owner 等。

## 相关页面

- [[agent-mcp-protocol]] — MCP 协议基础（JSON-RPC 2.0 工具发现与调用）
- [[google-context-engineering]] — 上下文工程（Context Engineering）
- [[agent-memory-system]] — Agent 记忆系统（与 ES Agent 记忆层对照）
- [[openclaw-agentic-search-memory]] — 阿里云 OpenClaw 的 Agentic Search & Memory
- [[specialized-knowledge-search]] — 专用知识搜索（企业资产语义搜索）
- [[agent-skills-system]] — Skills 扩展机制（Agent Builder 的 Skills/plugins 对应）
- [[agent-multi-agent-collaboration]] — 多 Agent 协作（A2A 协议场景）
