---
title: FZ Agent OS——供应链智能体从 Demo 走向生产的操作系统
description: 高磊（来源文件标注顺丰供应链）在 2026 Agent 大会「工业场景Agent实践论坛」的演讲——FZ Agent OS 供应链 Agent 操作系统：供应链 Agent 落地 5 大断点、生产级八要素（Semantics/Policy/Durable State/Secure Access/Sandbox/Trace/Version/Eval）、四大组件（Ontology Engine / Agent Studio / Evolver / Skill Evolver）、No Endpoint No Credential 安全网关与证据驱动 Skill 自进化闭环
aliases: [FZ Agent OS, FZ AgentOS, 顺丰供应链Agent, 供应链AI全域智能操作系统, 高磊, FZ Ontology Engine, FZ Agent Studio, FZ Evolver, FZ Skill Evolver]
tags: [ai-agent, practice]
sources: [2026/08/11/工业场景Agent实践论坛/01-高磊-顺丰供应链Agent落地实践供应链AI从工具走向全域智能操作系统.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# FZ Agent OS——供应链智能体从 Demo 走向生产的操作系统

2026 Agent 大会「工业场景Agent实践论坛」演讲，演讲者高磊（来源文件名标注为「顺丰供应链Agent落地实践」）。幻灯片内容为 **FZ Agent OS**——让供应链智能体从 Demo 走向生产，定位「业务语义 × 企业级运行 × 算法自动化 × 技能进化」四合一。

## 核心命题：Demo 与 Production 之间的鸿沟

一次成功的 Demo，不能证明一个 Agent 可以上生产。

| Demo | Production |
|------|-----------|
| Prompt + Model + Tools | Semantics（业务语义与数据契约）|
| | Policy（策略与合规约束）|
| | Durable State（持久状态与断点续跑）|
| | Secure Access（安全接入与凭证受控）|
| | Sandbox（沙箱隔离与资源守护）|
| | Trace（全链路追踪与可观测）|
| | Version（版本管理与回滚治理）|
| | Eval（评测与持续验证）|

难点从「能不能回答」转向「能不能长期、稳定、安全地工作」。

## 供应链场景中 Agent 落地的 5 大断点

Agent 面对的是多个互不相认的局部真相。业务概念域：需求计划、产销协同、订单管理、生产排程、仓储管理、履约交付；系统孤岛：计划系统、ERP、OMS、MES、WMS、TMS。

| 断点 | 表现 |
|------|------|
| 重复建 | 不同业务概念各自建设，Agent 烟囱 |
| 语义乱 | 同名不同义、同义不同名、状态不贯通 |
| 算不对 | 预测、优化与仿真不能靠语言模型猜 |
| 学不会 | 结果与原因没有进入下一次执行 |
| 不敢做 | 真实副作用缺少统一边界 |

## FZ Agent OS 五类能力，逐一消除五个断点

| 断点 | 对应能力 | 组件 | 作用 |
|------|---------|------|------|
| 语义乱 | Understand | FZ Ontology Engine | 统一业务语义，业务对齐、沟通一致 |
| 不敢做 | Act | FZ Agent Studio | 企业级运行治理，安全执行、合规可控 |
| 重复建 | Act | FZ Agent Studio + Skill Hub | 工程化复用，能力复用、高效交付 |
| 算不对 | Solve | FZ Evolver | 算法模型工厂，准确求解、交付价值 |
| 学不会 | Evolve | FZ Skill Evolver | 证据驱动进化，持续迭代、越用越强 |

全程治理贯穿（Govern）：Identity、Policy、Trace、Eval、Version。

## 全景架构：角色化 Agent 共享统一能力底座

角色化 Agent：需求计划、产销协同、订单履约、生产排程、库存、运输（六大角色 Agent）。

- **FZ Agent OS 底座**：Identity · Policy · Sandbox · HITL · Trace · Eval · Version
- **能力交付**：FZ Agent Studio（Agent 配置、发布、运行）、FZ Evolver（算法模型、求解器开发与优化）
- **共享引擎**：FZ Ontology Engine（统一企业业务语义）、FZ Skill Evolver（证据驱动的持续进化）
- **注册与分发**：Model & Solver Registry、Skill / 调用接口
- **连接层**：Data Gateway、API Gateway 对接企业系统与数据源（ERP、OMS、MES、WMS、TMS、数据中台/数据湖/数仓、IoT/外部数据）

## 01 Understand：FZ Ontology Engine

先让 Agent 理解业务，再让它查询、决策与行动。

### Ontology Engine：把分散信息升级为企业统一语义契约

Schema（数据长什么样）、RAG（材料说过什么）、API（接口如何调用）只提供碎片信息；企业语义中心定义业务如何被理解、决策与执行。

「可用库存」同名不同定义的例子：库存-预留（仅订单）、可分配库存（含部分在途）、物理库存-冻结库存、未来可用（含预计入库）——字段不同、口径不同、状态不同；各 Agent 自己解释 = 重建新烟囱。

FZ Ontology Engine 建立共同契约的四要素：

- 对象与关系（有什么、如何关联）
- 规则与计算（怎么算、有何约束）
- 动作与权限（能做什么、何时可做）
- 指标与口径（如何衡量、统一标准）

一次建模，沿版本统一演进：同一对象、同一口径、同一动作边界；可版本、可审阅、可复用；新增 Agent、工作流、求解器均可复用。

**边界**：Ontology 不取代数据库或业务系统；真实取数经 Data Gateway，真实动作经 API Gateway 与业务 Runtime 受控执行。

### AI Native 增量建模：语义边用边长

图谱概览：**122 个节点 / 508 条边**，实体类型 4/4。

- 建模链路：PRD / DDL / API / 访谈 → 搜索已有本体 → 四类对象 → YAML 校验 → 人工审核 → Git 版本
- 消费链路：语义搜索 → Top-K → 详情 → 邻居扩展 → Agent Context（按需进入，降低噪音、提升回答质量）

### 四类对象：理解-决策-执行-度量闭环

| 对象 | 作用 | 关键属性 |
|------|------|---------|
| Entity | 业务概念 / 数据理解 | 属性、关系、约束、状态机 |
| Function | 决策 | 业务逻辑、输入输出、对象关系、Skill 引用 |
| Action | 执行 | 业务步骤、前置条件、审批权限、状态变更 |
| Indicator | 指标 / 度量 | 业务定义、计算/数据源、维度/粒度、阈值 |

闭环：理解业务 → 做出决策 → 安全执行 → 衡量结果 → 驱动下一轮决策。

## 02 Act & Govern：FZ Agent Studio

把 Agent 与 Skill 变成可发布、可治理的企业能力，让 Agent 在统一语义与治理边界内完成业务任务。

### 三个平面分工：治理、执行与资源访问各守边界

- **Agent Control Plane**：Identity · Tenant/Project/RBAC · Lifecycle——责任：每个 Run 接收用户、Agent、项目和逻辑资源上下文
- **Core / Execution Plane**：Session · Memory · Skill · Sandbox · Tool · Model——责任：仅使用逻辑资源执行任务，并执行治理与审计
- **Secure Resource Access Plane**：Data Gateway · API Gateway · Credential · Policy · Audit——责任：在信任边界内解析真实端点与凭据（Endpoint / Credential 不进入 Agent）

External Resources：Ontology、Data、Model/Solver、Business Systems。

### No Endpoint - No Credential to Agent

Agent 只持有逻辑资源，真实地址和凭证留在安全网关：

- **Secure Data Gateway**：多源连接、User&Agent 权限、表/列/行级控制、不安全 SQL 拦截
- **Secure API Gateway**：统一注册鉴权审计、参数前置校验、限流/配额控制
- **Sandbox + HITL + Audit**：隔离高副作用动作、全链路留痕；HITL 证据——高风险动作可暂停并人工审批；工具执行需审批（exec_command_with_approval）、可拒绝、可 Reset Session
- 控制与审计层：User&Agent Policy、Audit（审计日志）、Trace / Artifact

### Skill Hub：统一沉淀企业能力，同一治理底座上实现「千人千面」

SKILL FIRST——Skill 是 FZ Agent OS 的第一公民，所有业务能力均通过 Skill 扩展。

高价值对话 → Skill Draft → 脱敏/验证/审核 → Skill Hub。分级：

- 系统级：Ontology 绑定
- 组织级：审核后共享
- 个人级：本人私有，可申请发布

Trace / Score / Feedback 回流 Skill Evolver；Ontology、Policy、Version、Eval、Audit 统一治理；统一企业底座 + 角色能力 + 个人 Skill = 千人千面。

### Agent Studio：从写代码到做配置

把 Agent 发布为生产 Endpoint：Configure（Identity · Prompt · Model · Skill · Tool · Ontology · Resource）→ Lifecycle（Draft → Staging → Production Endpoint）→ Publish（发布为 REST API）；Govern：Version · Eval · Rollback。界面示例显示 Agent 可配置身份、系统提示词、模型（如 DeepSeek V3 Flash）与部署形态（Isolated Container，2 vCPU / 4GB）。

### Debug Lens：失败不再是黑盒

一条可定位、可介入、可恢复的工作流：

- Workspace：输入、输出、文件与目录快照
- Conversation Flow：内容、推理、工具调用与暂停
- Inspector / Logs：参数、结果、耗时、错误、模型、记忆与 Hook

统一事件语言：Run · Tool · Data · API · Policy · Approval · Model · Hook。

## 03 Solve：FZ Evolver

把业务问题交付为可运行、可验证的求解器资产。

### 传统人工建模 vs FZ Evolver

传统人工建模：建模依赖专家、算法运维成本高、难以适应业务变化。

FZ Evolver：AI 辅助理解与设计，专家聚焦目标、约束和关键确认；Workflow、Skill、测试、Gate、Artifact 与发布链路标准化；沿同一工作流重跑、评测、优化并发布新版本。**大模型负责理解、设计与迭代；OR 优化器/求解器负责确定性计算**（looped_execution 内部：Plan → Implement → Gate）。

### 四类领域基座，共用一条智能求解交付链

| 领域基座 | 输入 | 优化 | 输出 |
|---------|------|------|------|
| 运输路径规划 | 订单、车辆、时窗 | 多约束路径优化 | 可执行路线、地图与指标 |
| 仓网规划 | 选址与服务范围 | 选址/分配优化 | 选址方案、分配关系与评估报告 |
| 库存优化 | 服务与成本平衡 | 策略计算与仿真 | 策略参数、仿真对比与补货建议 |
| 需求预测 | 未来需求 | 多模型训练、融合与评测 | 预测结果、评测报告与版本模型 |

共用交付链：设计 → 实现 → 优化 → 发布。

### 三层优化：业务、模型与研发能力持续变好

- L1：算法模型/求解器优化业务问题 → 业务 KPI 改善 → 业务结果
- L2（FZ Evolver）：优化算法模型/求解器（VM → VM+1），Gate、Trace/Artifact、Benchmark、反向证据
- L3（FZ Skill Evolver）：优化 FZ Evolver 的 Skill（Skill vN → vN+1）

## 04 Evolve：FZ Skill Evolver

让每一次运行证据，成为下一版能力的起点。

### 双证据流驱动的共同进化内核

Skill Evolver 是 Agent Studio 与 FZ Evolver 的共同进化内核：

- **运行证据流**（运行行为与反馈证据）：Run、Tool、HITL、Success、Feedback、Trace
- **求解证据流**（算法与求解质量证据）：Workflow、Task、Trace、Artifact、Benchmark
- **共同进化内核**：候选登记 → 安全扫描 → 人工评审 → 发布登记 → 只读存储；Attribution 归因评估、Anti-overfit 防过拟合、失效记忆库
- **受治理发布管线**（Skill Hub / Registry）：审批发布、可回滚、可审计；只读装配为 Skill / 调用接口；批准版本才能回流使用（职责清晰边界）

### Trusted Evolution Loop：Skill 自进化的完整流程

从冻结基线到版本晋级：Baseline（逐 case 评分）→ Analyze（Trace 与证据）→ Synthesize（聚类共性问题）→ Mutate（复制 Best 候选，预算与反特化）→ Review → Gate（模型提出改进，框架在同一实验条件下证明改进）→ Update Best 或进入 Failure Memory → Learn（更新知识与谱系）。沿单调谱系持续演进。

**两条铁律**：没有行为证据，不允许修改 Skill；没有确定性 Gate，不允许晋级版本。

- 证据归因：Activation + Event + Source-line 三者同时成立，才归因为 Skill
- 修改 Gate：Mutation Policy、Anti-overfit、Read-only Review
- Performance Gate：Holdout、Repeat；candidate > incumbent + ε 才晋级（Quality / Safety / Cost / Latency 多维度对比）
- Pass → Publish → Trusted Cache → Rerun；未晋级 → Failure Memory

## 总结：让智能体真正进入供应链生产系统

覆盖供应链六大环节：需求计划、生产排程、产销协同、仓储管理、订单管理、履约交付。

1. **语义先于行动**：先统一业务契约，再让 Agent 查询、计算和操作
2. **工程承载责任**：Identity · Gateway · Sandbox · HITL · Trace · Artifact
3. **证据驱动进化**：模型提出变化，框架和人决定什么能够生效

FZ Agent OS 集成核心：FZ Ontology Engine | FZ Agent Studio | FZ Evolver | FZ Skill Evolver。

演进路径：能力自进化 → 系统自优化 → 供应链自适应（Capability Evolution → System Optimization → Adaptive Supply Chain）。

金句：「The model is probabilistic. The operating system must be accountable.」——从数据到语义，从语义到决策，从决策到执行，从执行到进化。

## 相关页面

- [[agent-harness-overview]] — Agent Harness 综述（生产级运行六承重层与本页八要素呼应）
- [[high-certainty-agent-practices]] — 高确定性商业场景 Agent 落地实践（同为 2026 Agent 大会，确定性执行思路相近）
- [[dynamic-workflow-agent-paradigm]] — 能力契约驱动的企业级 Agent 执行范式
- [[agent-self-evolution]] — 自进化智能体研究与实践（Skill Evolver 相关）
- [[ontology]] — 本体论（Ontology Engine 的语义基础）
- [[ontology-driven-knowledge-engineering]] — 本体驱动知识工程（同大会）
- [[cosmo-claw-ontology-agent]] — 同论坛：COSMO-Claw 本体语义罗盘
- [[digital-employee]] — 数字员工
- [[ai-governance]] — AI 治理
