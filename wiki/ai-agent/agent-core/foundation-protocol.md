---
title: Foundation Protocol：Agent 社会的基础协议与协调底座
description: FP 把 Agent/人/工具/服务/组织统一建模为可寻址实体 Entity——交流协作/安全/信任/交易四类关键行为统一归属，中心化 Host 信任背书，Mail/Contract/Pay 热插拔组件，AI-Link-Net 与 Agent Economy Bench 应用实践
aliases: [Foundation Protocol, FP, FoundationAgents, foundation-protocol, 古永丰]
tags: [ai-agent, concept, architecture]
sources: [2026/08/11/多Agent协作论坛/03-古永丰-Foundation ProtocolAl社会的基础协议与协调底座.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# Foundation Protocol：Agent 社会的基础协议与协调底座

2026 Agent 大会「多Agent协作论坛」演讲（演讲嘉宾：古永丰，Foundation Agents 开源社区成员、DeepWisdom Research Intern）。FP（Foundation Protocol）定位为 Agent 社会的基础协议与协调底座。

## 背景：从 Chatbot 到 Agent 社会

### 从 LM 演进到 Agent 元年

- 2022 年以前：LM / 基础模型发展阶段（2016.06 ResNet、2017.06 Transformer、2018.10 BERT、2019.02 GPT-2、2020.10 ViT、2021.01 CLIP）
- 2022—2024：LM 爆发，Agent 能力逐步成形（2022.01 Chain of Thought、2022.10 ReAct、2022.11 ChatGPT/GPT-3.5、2023.02 Toolformer、2023.06 Function Calling、2024.10 Computer Use、2024.11 MCP）
- 2025：Agent 元年（2025.01 DeepSeek-R1/GRPO、2025.01 Operator、2025.03 Manus、2025.04 A2A、2025.05 Codex、2025.11 OpenClaw）

核心变化：模型能力长期技术积累 → Agent 技术积累 → Agent 元年 → ？

### Agent 有了大脑和手，但还没有"社会属性"

Human（brain + hands）↔ Agent（LLM + tool calling），两者都已具备 cognition + action。Human society 由 Identity、Reputation、Economy 组成；Agent society 仍缺：**身份、声誉、经济**——组成社会成员的必要属性。

### 主流 Agent 协议全景与能力边界

| 协议 | 定位 | 强项 | 边界 |
|------|------|------|------|
| MCP | 工具调用 | 标准化工具接入 | 身份、信誉、交易、治理 |
| A2A | 任务协作 | Agent 间任务协同 | 统一实体、审计、长期信任 |
| DIDComm | 安全消息 | 去中心身份与加密通信 | 协作语义、经济活动 |
| A2UI | 人机交互 | Agent 驱动人机界面 | 跨协议身份、行为审计 |
| UCP | 商业交易 | 商业交易与结算 | 统一实体、全过程治理 |

**共同缺口：跨协议 Entity，以及统一的行为监管、证据留存与信誉语义。**

### 核心 Feature：四类关键行为统一归属于 Entity

| 行为 | 关键问题 |
|------|---------|
| 交流协作 Collaboration | 社会成员之间通过什么管道流程进行交流？ |
| 安全 Safety | 行为是否符合权限、策略与安全边界？ |
| 信任与信誉 Trust | 历史活动如何形成信誉并影响信任？ |
| 交易 Trade | 如何达成契约并结算？ |

统一 Entity 贯穿四类核心 Feature；关键行为经过 Checkpoint 监管，并以 Provenance 留存——记录持续进入信誉档案，影响后续信任、授权与安全。FP 希望实现 entity 在 agent 社会中的所有行动能行为可控、身份可信（所有关键行为回到同一身份：跨协议身份锚点，监督并见证行为、留存证据与上下文、聚合评估协议活动记录）。

## FP 核心方案

### 一句话定义

把 **Agent、人、工具、服务、组织**统一建模为可寻址实体 **Entity**，用同一套协议串起来 Entity 的身份信誉、行动监督、交易审计。

### 分层架构（Graph View）

- **Regulation & Oversight Plane**：Policy enforcement、audit & provenance、monitoring、compliance、dispute signals
- **Interaction & Organization Plane**：Identity & credential methods；Schemas（Envelope & codecs、Schema & event-type registries、Extension & pattern libraries）；Events & Streams（Ordering & backpressure、workflows）；Sessions & Orgs（Groups, roles, membership）
- **Economy Primitives**：Metering, receipts, settlement（auctions, games, …）
- **Transport & Routing Plane**：Addressing & discovery hooks、channel setup、protocol bindings、termination；Bridges to MCP / A2A / A2UI / DIDComm
- **Entity & Trust Plane**：Who（ID, keys, version）、What（Capability statement）、Trust（reputation, privacy）、Credit（Permissions, ownership）
- **Profiles & Bridges**：Transport bindings（HTTP/SSE, QUIC, …）
- 图视图：Entities（nodes）= Agents, tools, humans, institutions, orgs；Relations（edges）= Sessions, memberships, trust, delegations；Activities（events）= Invocations, messages, negotiations, transactions；Policies & Provenance = Constraints, rules, evidence over time

### 中心化 Host 做信任背书

- HOST TRUST NETWORK：Parent Host → Current Host（挂载 Entity、分配身份）→ Child Host A/B；Host 互联、网络关联
- 技术路线选择：Blockchain（去中心化共识；治理与接入更复杂）vs Centralized Host（责任主体明确、可治理、易落地）——**FP 选择 Centralized Host**
- Entity A/B/C 各自持有 Address/ID、信誉、身份
- 原则：信誉一定要有组织作为背书；组织可以是去中心化的，也可以是中心化的；可扩展的拓扑结构的网络，身份跨网络可信

### Transport & Routing——传输与路由

Message 从 Entity A 出发：检查社会关系 → 加密 + 签名 → Mail（Carbon Copy）→ HOST ROUTER → CHECKPOINT PIPELINE → Entity B HANDLE（Checkpoint 1 验证社会关系 → 2 解密验签 → 3 安全校验 → 4 自定义 Checkpoint → 邮箱 Mailbox）。

- Entity 社会关系：Owner 关系、好友关系、组织关系
- Signature Encryption（加密签名）；Carbon Copy：可监管、可追溯；CheckPoint 机制

### Regulation & Oversight——监管与监督

- **合同建立 Contract**：Party A ↔ Party B 提议/确认 → Contract 1..N（已签名）→ ARBITER / 监管核心。Contract Snapshot 状态：DRAFT / SETTLING / SETTLED / CANCELLED / DISPUTED；Canonical JSON、revision + terms_hash、SHA-256、Ed25519 签名；条款修改 → 旧审批失效，双方批准同一版本
- **信誉派生 Reputation**：ReputationEvent → Feature Vector → Reputation Profile：质量 Quality 35%、可靠性 Reliability 25%、协作 Collaboration 20%、效率 Efficiency 10%、完整性 Integrity 10%；Confidence × Recency；签名行为链 Signed Snapshot Chain（DRAFT S0 → PENDING S1 → ACTIVE S2 → … → COMPLETING/SETTLING Sn → SETTLED，hash/prev-hash 链）；Entity Reputation Overall Score 0–100（当前实现：party_b）
- 特性：防篡改、可离线验证、一个合同 = 一次可验证信誉贡献、分数可重算、签名合同链是信任根

### 交易 & 支付（四方模式 FOUR-PARTY MODEL）

付款方 Payer Entity ↔ HOST / ARBITER（状态协调、协议协调）↔ 收款方 Payee Entity。流程：Contract·SETTLING → PAY_COLLECT（支付链接/收款信息）→ 授权并支付 → 资金转移（去中心化支付 Crypto/On-chain；中心平台支付 Bank/Gateway；资金清算 Settlement Off-protocol）→ 到账感知 PAY_CONFIRM_RECEIPT → PAY_COMPLETED → Contract·SETTLED。付款账户/Wallet（Payer Account）、收款账户/Wallet（Payee Account）；Owner 审批（APPROVING）。

支付状态机 Payment State Machine：REQUESTED → APPROVED → EXECUTING → CONFIRMING → COMPLETED；REJECTED、DISPUTED。

### 核心理念

1. **热插拔组件：Mail、Contract、Pay**
2. **Agent 用 Entity 身份参与活动，Entity ≠ Agent**（原文符号辨识不清，疑为"≠"）
3. **只负责协调监督、与语义解耦**

## 应用实践

### 应用 1：AI-Link-Net

Motivation：能不能让公司内部的各个 agent 自己进行任务协商和业务沟通交流？公司内部使用的 Agent 网络拓扑结构：Ania、MiyCodex、Milo、self、Designer、Translator、Coder、Reviewer、Recruiter、Researcher（含 parent-3ebfb907、child-a、child-b 等关系节点）。

**Agent Market**（PPT 页面标题 OCR 为 "Agent Merket"）：Agent 间的二手/匹配/招聘/服务交易广场，帖子带标签（如 Secondhand：求购 HHKB Professional 机械键盘 1500、出 Dell U2723QE 4K 显示器 2800；Matchmaking：找周末户外运动搭子；Job：求职数据分析与研究、招聘全栈工程师 AI 方向 300；Service：专业中英日翻译 50、UI/UX 设计与原型 120、代码审查与质量分析 60、全栈编程服务 80；Task：深度研究与竞品分析服务 100），tag 如 keyboard hhkb、dell 4k、dating hangzhou、data-analysis remote、hiring fullstack ai、translation、design UX、code-review security、research analysis。

**AiLinkNet 效果（Live network）**：公司内部 Agent 协作网络——实体注册、跨 Host 协作、Agent Market（广播 & 撮合；分类 Task / Matchmaking / Job / Secondhand / Service）。"货币 消息在网络中流转"。运行数据：**16 实体、12 连接、8 活跃广播、2 撮合成交**。

### 应用 2：Agent Economy Benchmark

核心差异：从"单个 Agent 会经营"升级为"多个公司组成社会，衡量整体经济水平增长"。

- 单体经营能力 benchmark：CEO-Bench（一个 CEO Agent 经营公司）、Coffee-Bench（公司/市场仿真框架）；启发：长期经营任务天然暴露规划、记忆和经济判断能力
- Agent Economy Bench（多公司经济社会）：Company A/B、Market、Public Service；多个公司、应用和公共服务共享 global state；评测：社会经济水平增长——系统总剩余 / GDP-like growth（企业利润 + 库存价值 - 负债）、ELO / 排名 / 存活率、公平性：ROI 基尼系数 / 破产率约束
- 价值：把 FP 的 Entity、通信、交易与审计能力，放进一个可复现、可排名、可迭代的经济实验场

### 应用 3：Agent-Lab

AGENT LAB - chem.station v0.9：一个化学实验站的 Agent 协作模拟界面（LAB / AGENTS / EXPERIMENTS / LOGS；RUN / PAUSE；AGENTS ONLINE、EXPERIMENTS 5、TICK 85、AGENT COMMS；Agent 有 Bohr、Curie、Dirac、Euler、Fermi 等，日志如 "Curie filtration at pH 7.2"、"Fermi reflux at 1.184 g"、"Euler spectrum peak 8517m"）。

## 总结：面向 Agent 社会的下一步

不足：01 工程稳定性；02 信誉体系中的边界问题；03 协议语义层的兼容度。

从"能跑"走向"可治理、可信任、可审计"：**可迁移身份**（Agent 的身份不绑定单一平台）、**可验证履约**（履约记录可被第三方验证）、**可组合组织**（角色、成员、组织关系可拼装）、**跨平台声誉**（信用在不同平台间流通）。

> 当模型越来越强、Agent 越来越多，真正的瓶颈不只是能不能完成任务，而是这些行动者能不能在同一个社会语义里找到彼此、信任彼此、协作彼此，并在出问题时说清楚发生了什么。

## 相关页面

- [[agent-mcp-protocol]] — MCP 协议（FP 对比的既有协议之一，Transport 层可桥接）
- [[agent-multi-agent-collaboration]] — 多 Agent 协作模式
- [[ai-governance]] — AI 治理（行为监管、信誉、可审计）
- [[agent-harness-overview]] — Agent Harness 综述（协调与监督层对照）
- [[agent-architecture-patterns]] — Agent 架构模式
