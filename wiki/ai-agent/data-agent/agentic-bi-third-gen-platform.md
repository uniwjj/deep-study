---
title: 从 BI 到 Agentic BI：面向 Agent 的第三代数据平台
description: 安克创新数据负责人商渭清在 2026 Agent 大会的演讲——从静态展示到受控行动的数据平台演进：Launch 三层交付、数据专辑、DOM 可信底座（5+1 资产模型）、五能力与四阶段评测体系
aliases: [Agentic BI, 第三代数据平台, DOM, 数据专辑, Data Album, 商渭清, 安克创新]
tags: [big-data, ai-agent, architecture, practice]
sources: [2026/08/11/面向Agent的数据架构论坛/03-商渭清-从 BI到 Agentic Bl 面向 Agent 的第三代数据平台建设实践.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 从 BI 到 Agentic BI：面向 Agent 的第三代数据平台建设实践

> 演讲者：商渭清，安克创新数据负责人
> 主题：让数据应用从展示走向受控行动——从静态内容、场景语义到面向 AI 友好的新一代数据平台
> 核心：Agentic BI 绝非仅仅增加一个聊天框。它是让数据应用从展示走向理解，从理解走向受控行动，最终让每一次业务行动的结果完美回归到系统的追踪评测与业务闭环之中。

## 交付形态演进：Launch HTML → Launch BI → Launch App

| 形态 | 定位 | 能力 |
|------|------|------|
| Launch HTML | 托管体验 | 聚焦于稳定交付、安全访问与便捷分享；Markdown 适合表达与生成（Content），HTML 是真正的应用体验（Experience）——赋予内容布局、视觉层级与交互性 |
| Launch BI | 连接动态数据 | 让静态内容转为响应真实业务状态的实时图表 |
| Launch App | 承载受控读写 | 托管支持复杂交互与事务读写的全功能业务系统 |

核心演进逻辑：不是页面变得更复杂，而是页面连接了数据 → 数据具备了业务意义 → 系统开始承担行动责任。架构拷问：页面连接了数据库，等于理解业务吗？Agent 能直接查表，等于能正确使用数据吗？

## 三代演进：传统 BI → ChatBI → Agentic BI

| 代际 | 形态 | 能力 | 局限 |
|------|------|------|------|
| 传统 BI | 看数据（View） | 人工观察与解读已加工的静态仪表盘 | — |
| ChatBI | 问数据（Query） | 通过自然语言交互生成 SQL 或特定图表 | 破除幻觉：换成聊天框依然未跨越 BI 边界 |
| Agentic BI | 理解场景（Context）+ 受控行动（Action） | 围绕业务目标持续工作、选择正确知识、提出计划，并在权限与事务边界内协调执行 | — |

核心交付物转移：传统 BI 交付【数据视图】，Agentic BI 交付【受控行动与效果反馈】。

## 数据可用性的三个层次

1. **Level 1：技术上能访问（Technical Access）**——权限：当前触发用户能否查看渠道级明细？
2. **Level 2：业务上能被正确理解（Business Comprehension）**——口径：是订单金额还是确认收入？范围：按门店物理位置还是销售组织划分？
3. **Level 3：具体场景中能被正确使用（Scenario Application）**——行动：如果确认缺货，允许系统触发哪些补救动作？治理：退货订单是否已经从底层扣除？（当前 Agent 最大的痛点）

结论：**Schema 本身不包含业务语义。Agent 面临的不仅是模型能力问题，更是业务语义缺失问题。**

## 数据专辑（Data Album）：给数据智能体准备的一间业务工作室

以"销量下滑分析场景"为例，数据专辑包含：场景边界（Scenario Boundary）、场景语义补充（Semantic Context）、策略与权限引用（Policy References）、评测基线（Eval Baselines）、权威资产引用（Authoritative Asset Pointers），聚合企业核心数据表、预定义治理指标、权威知识库与文档。

边界与红线：
- 不复制事实数据：不创造第二套语义源头或权限源，真实读写必须由服务端重新鉴权
- 不是流程编排系统：专注为 Agent 提供上下文，读写动作必须由受治理的底层 API 真正执行

核心价值：让权威资产从"可被找到"，变成"可被 Agent 正确使用"。数据专辑不是给模型准备的一摞资料，而是给数据智能体准备的一间业务工作室——Agent 进来之后，不是自由发挥，而是被安全护航、完成任务。

## 数据访问五级路径（从最可信到兜底）

1. **经营分析 API**（首选命中 / Highest Priority）→ 2. **结构化指标 Metric DSL**（确定性计算）→ 3. **已晋级查询服务 Verified Services** → 4. **验证级查询样例 Verified Examples** → 5. **Text2SQL**（只读兜底，必须经过严格权限与 SQL Explain 审核）

让问数智能体从"会生成盲盒 SQL"升级为"会选择最正确、最稳妥的数据访问路径"。

## 信任升级：知识与行动双层

- **知识的信任升级（Knowledge）**：可溯源文本（doc）→ 结构化且已验证（Structured & Verified）→ 反复纠错使用（Trace Log）→ 持续自学习
- **行动的信任升级（Action）**：只读（Read-only）→ 自动执行（Auto-Action）→ 人机协同（Human-in-the-loop）→ 高风险工单转交（Ticket Handoff）

> 没有权威认证的上下文只是资料；有认证的上下文，才能成为智能体的行动依据。

## DOM（Data Object Model）：面向 AI 的企业级数据与知识底座

DOM 核心定位：面向人与 Agent（Applications 应用端 + Agents 智能体）的企业级数据与知识底座，由四类资产构成：知识资产（Knowledge Base）、数据资产（Data Assets）、预定义 MetricSpec、治理服务（Governance Services）。

为什么必须依赖 DOM：场景语义绝对不能靠 Prompt 临时编造，也不能任由每个独立 Agent 各自维护，数据口径必须建立在企业已有的权威资产之上。

边界红线：DOM 不负责创建 Agent，也不是通用工作流平台。当前阶段优先通过 API 与预定义 MetricSpec 消费，严禁临时捏造非正式口径。

## 平台三代演进（DOM 的演进方向）

| 代际 | 名称 | 关键能力 | 核心问题 |
|------|------|---------|---------|
| 第一代 | 数据供给平台 | 数仓、数据湖、ETL、计算引擎 | "数据有没有？能不能算出来？" |
| 第二代 | 人类消费平台 | 指标体系、语义层、BI、自助分析 | "人类能不能理解和使用数据？" |
| 第三代 | Agent-Ready 平台 | 数据与知识深度融合、受控调用、系统评测体系 | "Agent 能不能可靠地寻址、理解、调用和被治理？" |

关键洞察：**第三代绝非推翻前两代，而是深度叠加。** 如果底层的第一、第二代平台在数据质量、口径和权限上存在缺陷，直接引入大模型只会让错误以光速传播。（注：此为本次架构演进的观察框架，非行业强制代际标准。）

## DOM 五大能力（可寻址 · 可理解 · 可调用 · 可治理 · 可评测）

1. **可寻址（Addressable）**：拥有稳定标识。Agent 必须精确声明引用的资产，杜绝模糊记忆编造
2. **可理解（Understandable）**：具备清晰的业务定义、负责人、血缘网络与应用场景上下文
3. **可调用（Callable）**：通过标准化 API 获取结果。红线：Text2SQL 仅作兜底只读，严禁作为写入通道
4. **可治理（Governable）**：权限、质量与敏感度硬性拦截。Agent 绝不可成为绕过治理的超级特权用户
5. **可评测（Evaluable）**：路由路径、资产调用过程及结果正确性必须全程可追踪、可回归验证

核心理念：当对象、事实、知识、能力和治理能够被系统统一理解时，Agent 才有资格可靠地进入核心业务流程。

## AI 时代的应用组成：View / Control / Model

- **View（视图，Human-Friendly 体验）**：包含 Launch HTML、BI、App。提供给人类的交互与反馈层（极致体验、可视化）
- **Control（控制，Agent Layer 智能体）**：Agent（负责意图理解、任务拆解、上下文编排、计划生成）+ Deterministic System（确定性系统，负责鉴权、事务执行、系统审计）+ Governance & Eval（治理与评测干线，负责权限 Auth、审批）
- **Model（模型，AI-Friendly 数据地基）**：包含 DOM 与数据专辑，承载业务对象、状态与场景上下文

Controller 的分工红线：**Agent 仅参与意图与编排，确定性系统牢牢把握授权与执行的最终权威。**

## 目标态场景：华东销量下滑的自动化业务闭环

1. **发现异常**：华东销量连续 6 周下滑，业务人员发出追问
2. **Data Album 固定口径与可用资产**：限定当前场景的可用知识库与指标池
3. **Agent & DOM 检表深度事实**：调用库存服务发现渠道缺货，非整体需求下降
4. **Agent Planning 生成行动计划**：输出包含影响对象、证据及审批层级的详尽策略
5. **Launch BI & Eval 效果监测与回溯**：Trace 记录执行完整路径，观察窗内验证业务指标改善
6. **Launch App（Execution）**：Dry-run 预演，服务端严格重新鉴权，幂等写入受治理 API，回滚准备

从"下降 20%"的报警图表，走向可解释、可审批、可执行、可评测的自动化业务闭环。

## 评测体系：四阶段（Q1-Q4）

- **Q1 Golden Case（定基线）**：精准定义"应该发生什么"——Trigger、BIZ Goal（业务目的）、语义环境版本、Semantic Context、Prompt Struct、Tool Ver，构建回归基线
- **Q2 Trace（记过程）**：追踪"实际发生了什么"——Trace ID 串联 Agent 思考、DOM 检索、Approval Sys 与执行终端。红线：严禁记录敏感明文，不能替代底层独立审计
- **Q3 结果断言（判对错）**：验证"系统做对了吗"——Read Accuracy 校验读取场景的精确度、写入执行计划的合理性，以及操作后的状态副作用判断（State Side-effects）
- **Q4 业务效果（看价值）**：评估"业务有没有实质改善"——在规定的时间观察窗（Time Window）内，将基线比对与护栏指标（Guardrail Metrics）波动进行多维监测

Agentic BI 的发布门禁：不只是"模型能回答"，而是**越权指令能够正确拒绝，受控动作能够被预演、追踪和验证**。

## 演进路径与铁律

从当前起点到目标态：① 稳定、可信的只读问数（Read-Only，当前起点）→ ② 引入数据专辑提供精确场景语义（Semantic Context）→ ③ 长周期、复杂链路的主动行动规划（Proactive Action）→ ④ 受控边界内的单点写入与审批流集成（Controlled Write）→ ⑤ 完整业务闭环内的 Trace 与 Eval 建设（Observability）。

**演进铁律：切忌一步跳到完全自治系统。每一层跃迁必须先利用数据证明其安全性与正确性，方可开放下一层能力。**

分季度建设：Q1 Launch（前端承载——HTML 让内容跃迁为数字体验，BI 让体验连接业务动态数据，App 让体验承载受控读写与事务流转）→ Q2 Data Album（场景语义——动态语义容器，不复制事实数据，专职补充业务上下文，将数字赋予真实经营意义）→ Q3 DOM（可信底座——面向 AI 的基石，提供不可篡改的权威数据、治理指标与知识体系唯一源头）→ Q4 Agentic BI（洞察转化为行动——Agent 专职于意图理解与任务编排，底层确定性系统牢牢接管最终的鉴权、授权与执行动作）。

## 终局视界：面向 Agent 的第三代平台架构

- **DOM（5+1 资产模型）**——可信数据与知识底座
- **用数驱动建数**——Data Engineer Agent（数据工程师 Agent）
- **事件驱动行动**——Event Engine & KG（事件引擎与知识图谱）

未来的数据平台不再是单向的数据供给链，而是**人机协同执行的 Agent 能力底座**。

## 相关页面

- [[data-agent-practice-guide]] — 火山引擎数据智能体实践指南（数据智能体能力框架与成熟度模型）
- [[vivo-dataagent-practice]] — 同论坛 vivo 王同欢：数据研发治理平台 DataAgent 实践（DataOps 平台 Agent 化的另一视角）
- [[dataagent-semantic-layer]] — DataWorks DataAgent 语义层（语义层/指标口径的工程实现）
- [[agentic-data-cloud]] — Google Agentic Data Cloud（语义层成为云厂商核心基础设施）
- [[data-agent-semantic-stack]] — Data Agent 8 层语义栈（语义层与治理总线的架构思想）
- [[streaming-interactive-data-agent]] — 同论坛许鹏：流式交互与实时洞察（数据 Agent 产品化与评测度量）
