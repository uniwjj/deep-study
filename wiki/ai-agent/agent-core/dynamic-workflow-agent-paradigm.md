---
title: 能力契约驱动的企业级 Agent 执行范式（Dynamic Workflow）
description: 企业级任务不是"生成答案"而是"推进状态"——Skill Set Package 设计时交付能力契约、Stage Contract 约束委派与完成判断、Execution Ledger 保证可恢复可审计、Supervisor 沿轨迹纠偏，构成 Agent 主导的 Dynamic Workflow 执行架构
aliases: [DynamicWorkflow, Dynamic Workflow, 能力契约, 能力契约驱动, 魏粲实]
tags: [ai-agent, concept, practice]
sources: [2026/08/11/企业级Agent开发工具链论坛/01-魏粲实-能力契约驱动的企业级 Agent执行范式：DynamicWorkflow.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 能力契约驱动的企业级 Agent 执行范式（Dynamic Workflow）

> 演讲：魏粲实（DataFun · Agentic AI Summit，2026 Agent 大会「企业级Agent开发工具链论坛」）。核心主张：**设计时交付能力契约，运行时生成执行轨迹。** 持续运行的主体从 Agent Session 转向 Business Capability，形成 Agent 主导的 Dynamic Workflow。

演讲脉络五章：① 问题重构 → ② 常见范式 → ③ 行业信号 → ④ 执行架构 → ⑤ 能力驱动的企业级 Agent。副标题：从任务状态，到能力空间，再到执行轨迹。

## 一、问题重构：企业级任务到底在执行什么？

**企业级任务不是"生成答案"，而是"推进状态"。** 只要任务会改变系统状态，workflow 语义就会回来。

同样叫 Agent，企业级任务多了三类责任：**证据、验证、状态提交**。

- CONSUMER AGENT：Question → Answer（目标说明）
- ENTERPRISE AGENT：Goal → Evidence → Action → Verify → State Commit（目标说明、证据被收集、动作有作用、验证结果、状态提交）

企业级任务不只是生成答案，而是推进一个可审计的业务状态向前走。只要有状态，就会有边界、恢复和责任。

### 案例：一次投诉升级，原本要完成什么？

一次投诉升级 = 从异常的定位，到沉淀策略：

| 层次 | 内容 |
|------|------|
| **业务目标** | 定位异常原因 / 形成处理建议 / 沉淀优化策略 / 更新业务状态 |
| **稳定 SOP** | 工单进来，证据支撑归因；必要时人审，结果进入归档；归档后，再反哺标签、规则和策略 |
| **系统要求** | 证据可追踪 / 责任可解释 / 风险可判断 / 策略可复用 / 结果可归档 |

一次处理，反哺回系统，形成下一次判断的依据。SOP 能稳定主干，但是在真实场景中，很难每次都会沿着这条主干顺利的向前走。

### 真实执行不会只沿 SOP 往前走

同一条业务闭环上，**下一步会被证据改写**：

- 输入证据包：ticket（投诉对象与状态）、SLA（服务承诺边界）、alert timeline（异常发生顺序）、chatlogs（上下文证据）
- 稳定 SOP：工单 → 证据 → 归因 → 建议 → 人审 → 归档
- 状态账本：每次路径变化，都要回答两个问题——"为什么改？"（evidence_ref / decision_reason）、"改完能不能提交？"（risk_level / commit_policy，提交前验证）
- 典型偏差：分类证据不一致、SLA 缺失、告警不匹配、高风险建议 → 改归因 / 补证据 / 暂停提交 / 进入人审

同一条 SOP 在执行过程中可能出现回退、跳转，甚至循环。SOP 定义的是稳定的业务主干，真实执行路径由当时的证据和状态决定。

### 企业级执行范式要同时回答两个问题

1. **路径怎么调整？**——路径能不能根据现场情况调整：证据缺失时能不能先补证；结论冲突时能不能回退重判；目标满足时能不能提前结束
2. **结果怎么落地？**——调整以后产生的结果，系统能不能接得住：证据、权限、风险能不能解释；验证失败时能不能恢复；结果通过后能不能提交状态

两条失败路径：路径可以调整但系统接不住 → 最后就是一个很自由、却没法负责的 Agent；系统边界很稳但路径完全不能改 → 又会回到一张越来越复杂的流程图。**Agent 设计的差异，就在于这两个问题回答到什么程度。**

## 二、常见范式：动态判断被放在哪里？

判断留在设计阶段 vs 判断进入执行过程。三种形态的差别，不在于谁更动态，而在于**控制权放在哪里**。

### Workflow / Chain：动态行为运行在预定义的控制协议中

Workflow / Chain 并不等于固定串行流程，四种动态模式：

| 模式 | 说明 |
|------|------|
| 01 Prompt Chaining | 顺序编排——一个任务拆成连续的处理步骤 |
| 02 Routing | 条件路由——根据用户意图选择不同的处理分支 |
| 03 Parallelization | 并行与汇合——任务并行展开再汇合 |
| 04 Evaluator-Optimizer | 评价与迭代——多轮评价和修正 |

LLM 可以进入任务节点，也可以承担分类、评价和局部任务拆分。

- **工程价值**：控制流、状态传递、失败处理显式可见；可测试·可监控·可运营
- **设计边界**：运行时可以动态，编排协议不能临时改写；跨阶段关系、状态语义与完成条件仍由 Workflow 定义

Workflow 不缺少动态行为；它把动态行为组织在预先定义的控制协议里。

### Agent + Skill：Agent 负责执行，Skill 提供专业方法

- 执行期 loop：Observe → Decide → Act → Check，持续观察、判断、行动和检查
- Skill Library：instructions / scripts / references / checks
- 执行边界：顺序不被强制；步骤可能跳过；**关键检查需外部保证**

Skill 让通用 Agent 成为领域专家，但不是一套执行协议。写进 Skill 的 workflow，首先是一套给模型理解和遵循的方法论，并不是一套由执行引擎强制调度的控制流。

### Agent in Workflow：Agent 成为 Workflow 中的执行节点

Workflow 组织业务链路（Step 1 → Agent Node → Step 3 → End），按当前状态继续推进。Agent 完成局部任务（Plan → Tool → CheckResult，可调用 Skill、工具和上下文），结果回到 Workflow（节点结果＋当前状态），继续推进后面的业务链路。

Workflow 组织整条链路，Agent 处理其中需要开放判断的任务。**Agent 获得的是节点内的自主权，Workflow 保留的是链路级控制权。**

### 三种形态的控制权分配

| 形态 | 控制权 | 控制范围 |
|------|--------|---------|
| Workflow / Chain | Workflow 控制整条业务链路 | 整条链路 |
| Agent + Skill | Agent 控制一次任务执行 | 当前任务（按需加载 Skill） |
| Agent in Workflow | Workflow 控制链路，Agent 控制节点内部 | 链路＋节点内部 |

动态判断不是有或没有，而是被放在了不同的执行层级。把动态判断权交给 Agent 的同时，必须明确业务控制权由谁持有。

### 路径可以动态，业务约束必须保持稳定

Agent 的自由度不能带着判断依据和完成标准一起漂移。

- **稳定业务约束**：业务目标、证据要求、完成条件、状态提交规则
- **动态执行路径**：当前状态 → Agent 根据当前业务状态选择下一步（开放判断、路径可以变化）；继续·补证·转人工

工程问题：怎么把动态执行和稳定的业务约束拆开？

## 三、行业信号：执行为什么开始走出 Agent？

执行过程正在从对话里外置出来。

### Claude Code Dynamic Workflows：计划进入外部编排

典型场景：代码库级审计·迁移·重构——一个 Agent 会话无法协调。workflow.js orchestration script 由 Runtime 执行：plan（持有循环）、branches（持有分支）、state（保存中间结果）、dispatch（分派与校验）；subagents 负责局部判断；loop 持有循环、branch、collect 汇总中间结果、reviewer 交叉校验输出。

以前，计划、分支和中间结果都留在对话上下文。现在：**计划从对话上下文，迁移到可执行、可重跑的外部编排。** Agent 负责局部判断，script 负责整体编排。

### Loop Engineering：人不再接下一句，而是设计下一轮

- 过去：人自己就是 loop——human 说下一句 / 补充指令 → prompt → agent → result，看结果 → 补 prompt → 再跑一轮
- 现在：设计一个能持续驱动 Agent 的系统循环——source → 任务入口 → trigger（下一轮条件）→ maker（生成结果）→ State（读写每一轮）→ verifier（检查）→ stop（完成/升级）

人的工作就从"接下一句"，变成设计这个循环本身。**人不再逐轮 prompt Agent，而是设计一个持续驱动 Agent 的系统循环。**

### 外部结构承接了控制，但具体路径没有提前锁死

执行编排从对话里拿出来，只是第一步。外部结构持续保存当前状态，也负责驱动结束后的结果检查；不需要在开始前全部写死，而在这一轮结果出来以后再决定。

Task → 执行一轮（Action A）→ Verification（检查本轮结果）→ Current State（任务状态＋校验结果）；下一步在运行时决定：继续 / 回退 / 换一种做法 / 停止（实线：已经发生）。

Dynamic 的含义：**控制结构保持稳定，具体路径由每一轮的任务状态和校验结果逐步形成。** 下一步由当前任务状态和校验结果决定。

## 四、执行架构：能力怎样交付，Agent 怎样执行，系统怎么承接

当路径在运行中展开，结构由系统承接，交付的重点也要发生变化：**设计时交付能力契约，运行时生成执行轨迹。**

### OpenSpec Signal：从 Phase Workflow 到 Action-Oriented Workflow

OpenSpec：面向 AI 编码的轻量级规范层，用可追踪、可修订的 artifacts 组织一次软件变更。

- 传统工作流（phase-locked）：计划、实现、完成被当成阶段门（PLANNING → IMPLEMENT → DONE）；发现偏差时 Can't go back
- Actions, Not Phases：不是不要过程，而是不要把工作锁死在一个个阶段门里；artifacts are the live plan；implement / update as you learn

核心哲学：不是回到某个阶段，而是随时更新当前 artifacts。**动态不是取消结构，而是让 action 可以在执行中修正前置判断。**

### Artifact Graph：承接变化的可验证上下文

Agreement Layer → Artifact Graph → Source of Truth 三层：

| 层 | 内容 |
|----|------|
| **Agreement Layer** | 人和 AI 先对齐同一个 change：Intent（为什么改）、Scope（边界在哪里）、Approach（大方向怎么走） |
| **Artifact Graph** | 将变化做成一个可操作的对象：schema.yaml、proposal（意图与范围）、delta specs（行为变化）、design（技术判断）、tasks（执行拆解） |
| **State Update** | 执行、校验、回写同一组 artifacts：apply（读 artifacts 执行）、verify（完整性/正确性/一致性）、archive（delta 写回 source of truth） |

执行过程中无论在哪一步发现问题，都可以回到对应的 artifact 进行修正。稳定性来自每一次的回溯都有对象可改、有依据可查、有结果可验。放到企业级 Agent 中，与 artifact graph 对应的，也就是我们的业务状态、证据、调用记录和提交结果。

### Skill Set Package：设计时交付的能力边界

设计时交付物：**围绕同一个业务目标，把四类内容放在同一个版本里交付**：

| 文件 | 内容 |
|------|------|
| **CAPABILITY.md** | 业务目标与完成语义（业务目标、完成证据） |
| **capability.yaml** | Observation · 可选 Stage |
| **skills/\*** | 局部执行方法（改变多个状态的动作） |
| **Domain Action Refs** | 领域动作引用 |

- **版本化**：固定版本、不可变 Digest
- **按运行职责拆分**：Supervisor 可见部分（目标 + Observation + 可选 Stage、Stage Contract 模板：动作＋证据＋完成条件）与 Runtime 持有部分（业务动作绑定；当前业务状态＋选定 Stage → 具体 Stage Contract）
- PACKAGE 封装：四类内容作为一个版本整体交付；RUNTIME 加载（解析、校验）

**设计时交付的不是一条执行路径，而是一套可以在运行时生成契约的能力边界。**

### Agent Run Architecture：动作决策与阶段决策分离

ONE RUN · TWO DECISIONS · ONE STAGE CONTRACT：

- **RUN CONTROL**：Observation（业务事实引用、业务状态）→ Supervisor（判断距离目标还缺什么）→ Stage Contract（阶段目标、允许动作/限制要素、完成条件）→ Completion Evaluation（继续下一个 Stage 还是返工；达成完成的证据）
- **LOCAL EXECUTION**：Stage Agent 持有动作决策（进行局部的动作、按 Contract 的限制使用）→ 02 ACTION PROPOSAL（选择动作、输入与处理、预期产生什么结果）→ Runtime（按 Contract 的限制执行）→ 03 COMPLETION EVIDENCE（执行结果·输出可选 Stage；阶段决策回到 Supervisor）
- **BUSINESS FACTS**：Fact vN → Domain Action → Fact vN+1（当前业务事实 / 基于证据核实后、改变业务的事实）

**Stage Agent 决定局部动作；Supervisor 根据事实与证据决定下一阶段。**

### Stage Contract：约束委派和完成判断

同一份 Contract，在 Stage 开始和结束时各使用一次（ONE CONTRACT · TWO CONTROL POINTS）。Stage Contract 在本次 Stage 内保持不变，包含三部分：

- **进入条件**：阶段目标·事实引用
- **动作边界**：允许的工具·业务动作
- **完成判断**：证据完整·完成条件·退出条件

委派时：Contract 保持不变，Agent 根据现场情况调整局部路径；限定阶段目标和事实版本、提供可见上下文、限定动作空间。完成时：对照最新业务事实，按证据和完成条件判断（取证/完工）。Stage Agent 局部执行：观察 → 判断 → 行动 → 检查；每一轮执行都会继续累积结果、证据引用和业务事实。

### Execution Ledger：让执行状态可恢复、可审计

执行路径可以变化，系统必须知道现在走到哪里，以及为什么。Execution Ledger 按顺序追加执行状态与证据引用：

- STAGE 开始（Contract 版本＋阶段目标＋动作边界）
- ACTION 执行（动作意图·输入引用＋实际结果）
- EVIDENCE 累积（证据变化·业务事实引用·执行异常）
- SUPERVISOR 评价（继续/出偏/返工/结束）

这些记录直接参与后续运行：**中断恢复**（从最近一次确定的执行状态恢复）、**路径重建**（重建动作、证据与判断之间的因果关系）、**评价失败后的问题诊断**（Supervisor 回溯轨迹，定位从哪一步开始偏离）。

注意边界：Ledger 记录执行状态和引用，**权威业务事实仍由 Domain System 持有**。Ledger 可以恢复执行进度；已经发生的业务结果，需要通过业务补偿处理。

### Supervisor Evaluation：沿执行轨迹定位并纠正偏差

评价失败只是结果，真正的纠偏要沿着执行轨迹定位偏离点（EVALUATION-DRIVEN CORRECTION）。

Stage Agent 执行（观察·判断·行动·检查）→ Completion Evaluation（对照 Stage Contract 评价结果）→ 评价通过：进入下一 Stage/完成；评价不通过：**定位偏离点**（回顾 Execution Ledger：复盘实际动作、证据和结果）→ **传递纠偏意图**（从哪一步开始偏离目标；必须回退到其他 Stage）→ 补证·调整策略·重试/停止。

自进化从在线纠偏开始：执行轨迹成为下一轮 Supervisor 意图的依据。

### Architecture Fit：不是所有业务都值得动态化

选择执行形态，先问两个问题：**路径何时形成？运行时决策权交给谁？**（BUSINESS FIT MAP，两轴：路径设计时预设 ↔ 运行时形成；系统控制 ↔ Agent 自主）

| 形态 | 定位 |
|------|------|
| **Direct Automation** | 最靠近系统控制，规则负责触发，系统侧完成 |
| **Workflow + Agent** | 某个环节需要开放判断，Agent 负责局部任务，Workflow 推进链路 |
| **Capability-Driven** | 阶段由 Supervisor 选择，完成由 Supervisor + Contract 协同评估 |
| **Free Agent** | 路径和完成判断主要由 Agent 自己负责 |

这四种形态不是能力强弱排序，而是系统控制和 Agent 自主之间不同的分工。

## 五、能力驱动的企业级 Agent：一项业务如何跨越多次执行保持连续

持续运行的主体，从 Agent Session 转向 **Business Capability**（TRANSIENT SESSION · PERSISTENT CAPABILITY）。

- **Agent Session**：只服务于第一次局部判断——context/memory/tools 服务于局部判断，按需启动，可以结束·可以替换
- **Business Capability**：持续保存业务运行所需的稳定信息——业务目标（这项业务要完成什么）、运行约束、完成条件（什么业务条件代表完成）

权威业务事实仍由 Domain System 持有。Session 可以结束；下一次执行仍从同一 Business State 继续。Agent、系统、人工和外部事件围绕 Business State 发力，形成 Dynamic Workflow。**持续保存的是目标、约束、完成条件和当前状态。**

### 最终架构：Capability 驱动的 Dynamic Workflow

| 要素 | 含义 |
|------|------|
| **CAPABILITY** | 持续存在的业务运行空间 |
| **BUSINESS STATE** | 业务进展与能力激活依据 |
| **AGENT HARNESS** | 开放判断的执行能力 |
| **DYNAMIC WORKFLOW** | Business State 的实际迁移轨迹 |

## 相关页面

- [[loop-engineering]] — 执行过程从对话中外置的人侧解读（Loop Engineering）
- [[agent-architecture-patterns]] — Agent 架构设计模式
- [[agent-design-paradigms]] — Agent 设计范式对比
- [[openspec-sdd-practice]] — OpenSpec 的 Action-Oriented Workflow 与 Artifact Graph 落地
- [[harness-engineering-practice]] — Harness 工程：Agent-aware 工程系统转型
- [[agent-hook-governance]] — 框架层 Hook 护栏（与 Stage Contract/Supervisor 同属系统承接控制）
- [[agent-mcp-protocol]] — MCP 工具协议（工具调用边界）
- [[agent-skills-system]] — Skills 扩展机制（与 Skill Set Package 对照）
