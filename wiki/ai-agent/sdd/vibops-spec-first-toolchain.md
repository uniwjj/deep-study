---
title: VibOps——从 BizDevOps 到 spec-first 的企业级 AI CodingAgent 工具链
description: 需求级 Workspace（work_id 连接需求工件/代码仓库/运行状态）+ Context Snapshot/Contract 冻结事实与行动边界 + 双阶段 Evidence 验收（Business Accepted=需求完成），spec-first AI Coding Harness 承载代码执行，把个人 AI 提效转化为组织交付力
aliases: [VibOps, spec-first, spec-frst, BizDevOps, 况雨平, VibOps平台]
tags: [ai-agent, practice, tool]
sources: [2026/08/11/企业级Agent开发工具链论坛/03-况雨平-从 BizDevOps 到 VibOps：基于 spec-frst 的企业级 Al CodingAgent 工具链实.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# VibOps——从 BizDevOps 到 VibOps

> 演讲：况雨平（leo，spec-first 团队）（DataFun · Agentic AI Summit，2026 Agent 大会「企业级Agent开发工具链论坛」）。主题：从 BizDevOps 到 VibOps——构建 AI 时代从需求到交付的完整闭环，让**个人提效**转化为**组织交付力**。
>
> 核心公式：**组织交付力 = AI 生产力 × 生产关系适配度。** "AI 让节点更快，VibOps 让端到端流动跟上。"

演讲四章：① 生产力之变（AI 让节点更快，组织交付为何没更快？）→ ② 生产关系之困（节点提速之后，研发协作关系如何适配？）→ ③ 需求闭环之实（VibOps 如何让一个需求更快、更好地完成？）→ ④ 交付进阶之路（我们已经怎么用，下一步走向哪里？）。

## 一、生产力之变：AI 让节点更快，交付为什么没有等比提升？

### 真实观察：节点确实变快了

- 内部观察 | 2026-01—2026-03 | 50+ 研发：部分编码环节产出效率约 **2-4×**
- 仅描述部分编码环节，不代表端到端需求周期或组织吞吐

### 完整交付时钟：一个需求远不止写代码

被验证提速的局部作业段只有【编码】：完整交付周期还包括**理解与约束**（规则/历史/架构/权限）、**变更与集成**（复用/接口/依赖）、**验证与运行**（Review、测试/灰度/业务接受）。

### 机制示意：同一条交付时钟，只压缩其中一段

假设完整交付周期 = 100（Amdahl 定律推算，假设 Coding 占 20%、该段提速 10x、其余环节不变，**非内部实测**）：提速前 其余环节 80 + 编码 20 = 100；提速后 其余环节 80 + 编码 2 = 82，整体缩短约 **18%**。

**节点作业时间下降，不等于端到端交付周期等比下降。**

### 节点节省下来的时间，被什么重新消耗了？

AI 更快完成：分析草稿、方案草稿、代码实现、测试生成——节点内部实际作业时间下降。但同一个需求仍有五个节点间断点：①方向确认、②系统适配、③协同集成、④验证决策、⑤业务接受，对应的断裂分别是：验收未收敛、隐性依赖未知、环境/权限不同步、业务接受未闭合——每次都要重新确认、重新理解/设计、重新对齐/联调、返工/重验（等待 | 上下文重建 | 协调 | 返工 | 重验）。

同时知识难发现、难复用：历史组件、技术方案、Review 结论、Bug 根因、RCA 都散落各处——下一次需求只能再次分析 → 再次实现 → 再次验证。

**省下的是首次产出时间，消耗掉的是让答案真正可交付的反复。**

## 二、生产关系之困：节点提速之后，研发协作关系如何适配？

### 流程在传递产物，需求却还要靠人串起来

案例：需求「退款列表增加"批量审批"」，变化：允许部分成功，失败项展示原因并支持导出。

- **正式产物轨道**（专业分工与流程）：需求说明 → 技术方案 → 代码变更 → 测试发布 → 业务接受 → 下一次需求
- **需求连续性**（5 类断点）：①规则未清（怎么算成功？）②旧组件与接口（还能复用吗？）③版本与依赖（现在同一版吗？）④环境与验收（证据齐了吗？）⑤经验与完成（下次找得到吗？）
- **人的协作轨道**（临时承接连续性）：问清规则、找历史 Owner、拉群协同、协调验证、判断完成与复用 → 返回上轨，推动流程继续前进

流程传递了专业产物，但同一个需求的连续性，仍然主要由人维持——不同的人，在不同时间，用沟通、记忆和判断维持同一个需求。

### AI 放大了产出并发，需求连续性不能再只靠人的注意力

三层逻辑链：

- **第一层**：AI 降低节点作业成本 → 更多产出 → 更多 Agent/Session → 更高并发、更频繁变化（从 1 条需求放大到 N 条并发需求：需求 A/B/C/D 各自重复"问清规则/找历史 Owner/拉群协同/协调验证/判断完成与复用"）
- **第二层**：人工协作运行时——并发需求 × 变化频繁 × 每条需求的人工协作动作 = 分布在多人之间的共享队列；产品/历史 Owner/研发/测试/发布/验收 Owner 共享的人类注意力与决策窗口出现：等待关键人、上下文切换、版本漂移、决策排队、重复验证
- **第三层**：责任迁移——进入需求级系统（同一需求身份、共享上下文与有效版本、变化传播与依赖状态、验证证明与完成状态），人保留判断与责任（业务目标与范围、专业判断、高风险例外、最终接受与责任）

**人不是瓶颈；需求连续性只能依赖人的注意力，才是组织扩展的上限。**

### 共同约束 × 个体化 AI

- 共同面对的企业约束：业务合规（规则/合规/口径）、工程历史、运行交付
- 个体化的 AI 使用：能力不同、输入不同、规范知识未连接、经验产出未沉淀 → 同一需求出现不同理解/不同方案/不同安全边界

组织级共同机制：**同一需求对象、同一上下文、同一责任、同一完成标准**（Feature Owner / 技术 Owner / 验收 Owner；业务结果、变更安全、Evidence·业务接受），产品/研发/测试/运维/架构/平台/安全/数据在外围专业守护，围绕需求闭环。

**不是让每个人独自把 AI 用得更好，而是围绕同一需求对象，统一上下文、责任和完成标准。**

## 三、需求闭环之实：VibOps 如何让一个需求更快、更好地完成？

### VibOps 能力分层与智能交付闭环

AI 提升执行供给，VibOps 同步重构上下文、协作、执行、验证与知识回流。分层（自顶向下）：

- **业务场景层**：新需求开发、老系统改造、代码 Review、提 Bug 修复、线上问题排查、知识沉淀与复用
- **VibOps 平台层**（AI 时代产研协同与智能交付平台）：需求/PRD 管理、知识中心、规范中心、交付中心、AI 能力治理、资产沉淀与复用
- **治理底层**：Rules；需求/研发/Review/交付（证据）；长期记忆；Evidence（交付证据）
- **spec-first AI Coding Harness 能力底层**：AI Coding Execution Harness（Spec/Plan/Work）、Review Harness、Context Harness、Evidence Harness、Knowledge Harness（防御自愈能力）、bug-fix；能力迭代、自我进化
- **工具编排层**：节点协作入口、本地 AI 能力输出、研发 Workspace、项目任务、构建/Sonar/部署、Pod 日志/配置、状态鉴权/告警
- **企业研发系统**：GitLab/代码仓库、CI/构建/Sonar、部署环境/Pod、日志平台/ELK、知识库/文档/规范、审批流、MR

底层主链：需求 → spec → Plan → Work → Review → Evidence → Knowledge → Al Coding。

**VibOps 定义协同范式，spec-first 负责 AI Coding Harness，妮蔻负责统一入口与执行，知识/规范/记忆/Evidence 构成可持续进化的上下文底座。**

### 创建需求，就是创建一个可运行的 Workspace

用同一个 **work_id** 连接需求工件、代码仓库与运行状态——一个需求从创建到 Archived 的稳定身份，从第一天就在同一现场连续演化。

- 身份与版本：work_id | Workspace Git/HEAD | workspace.yaml
- 权威工件：需求与决策（PRD/业务规则/变更记录/验收标准）；设计与方案（设计索引/技术方案/API Contract/Task/风险记录）；测试与经验（测试用例/Bug/Review/已验证 RCA/solutions）
- 关联元数据：Skill/知识库/MCP；运行元数据：日志/发布；State | Owner | Context/Snapshot 引用 | Contract 版本 | Evidence 引用 | Next Action
- 关联/引用：Repo/branch/commit/PR、飞书群聊/文档/设计

**两类 Git 的边界**：Workspace Git——版本化需求、方案、测试与经验工件；代码仓库 Git——保存真实源码，Commit、分支与 PR 只保留真相。

**Workspace 不是资料目录，而是同一需求的版本化共同现场和运行边界。**

### AI 不缺资料和工具，缺的是这次需求的正确装配

按需组合知识、代码、规范、模板、Skill、Agent 与 MCP。项目关联的知识库与 AI 组件（Skills/Rules/MCP），这些配置会注入到成员下发的 IDE 配置中：

- **Skills**：文档提交提醒、代码仓库检查、Code Review 助手、技术方案评审、requirement-clarification、接口设计助手、测试用例生成
- **Rules**：Git 提交规范、Python 编码规范、SQL 编写规范、vue3 强规范
- **MCP**：VibOps 知识 MCP（application_knowledge_base）
- 关联历史代码仓库作为 AI 知识来源（仅存储在服务端知识库，供 MCP 检索），平台每隔一段时间自动同步一次；代码由 codegraph 精准辅助

### 从专业节点水平接力，转向围绕功能结果的垂直闭环

- **过去 | 水平接力**：产品/研发/运维/业务各守节点产物（PRD/需求、代码/PR、测试报告、制品包、运行报告），等待/转译、口径分叉/返工
- **现在 | 垂直功能闭环**：一个需求级 Workspace——业务意图 → 方案/Contract → 实现 → Review/Test → 发布/运行验证 → Evidence → Business Accepted；同一 Work Object、共享上下文与档案工件、状态：Evidence·Owner，多角色共同对结果负责
- 接入方式：Web（治理与验收，审核 Workspace、审批 Contract、验收 Evidence）；IDE/Agent（读取与执行，读取 Context/Contract，执行开发/Review，回写工件与状态）；API/Event/MCP（事实与状态回写，同步工程事实、状态、Evidence 与流程事件）

Workspace 统一需求语义，但不复制工程事实，不成为第二真相源。**从水平接力到垂直闭环：一个功能在同一 Workspace 内完整交付。**

### 企业知识从 0 到 1 建立，并在交付中持续进化

身份锚点：**work_id / run_id / contract_version**。

- **代码知识链（0-1 建基座）**：存量代码/依赖 → VibOps 编排（Zread 可读、CodeGraph）→ 结构化/可读化、关联/影响面 → 工程知识底座（体系化：可查询）。让代码被理解，让复杂变可导航
- **交付知识链（1-N 长增量）**：需求/PRD、业务意图 → 可复用资产（Candidate/治理阀门、质量审核/可复用）→ 交付工件（Evidence/RCA/反馈、测试/运行/复盘）→ 下一次任务（沉淀/复用：赋能）。让交付沉淀为资产，让经验持续复用

VibOps 知识控制面：ID/Owner、来源/版本/血缘 | 权限 | 生命周期 | 质量 | 可信 | 证据 | AI 能力（K1-K4）。**全局组装公式 = 内容 + 归属 + 来源 + 约束 + 执行件 + 记忆 + 证据**，组装为本次 Context Bundle（工程基座/业务约束/可执行资产/组织记忆）→ Context Snapshot；未来 Evidence/RCA/复用反馈回到治理阀门，驱动知识持续进化。

**两条知识链让企业知识从可检索走向可执行、可验证、可进化。**

### 统一窗口，不是收消息，而是让变化生效

用同一个 work_id 让群聊变化进入权威工件，再触发失效与重算。五步流程：1. Signal（变化被发现：群聊→自动增量总结到 Workspace，保留 work_id/来源范围/时间/摘要版本）→ 2. Compare（反向核对：缺失/冲突/变更 Checklist、建议结论/影响工件/来源/需求 Owner）→ 3. Authority Gate（谁让结论生效：需求 Owner 逐项确认/补充/驳回/重新指定责任人）→ 4. Authority（更新权威工件并 Commit：需求/技术方案/API Contract/测试用例、Workspace Git Commit/Diff）→ 5. Propagation（让旧执行失效：影响识别、Owner 通知、Stale/必要时 Blocked、新 Snapshot/Contract、恢复执行）。群聊贯穿全生命周期：需求澄清、方案评审、研发/Review、测试/Bug、发布、归档。

**自动化边界：未确认内容不进入 Snapshot/State/Archive。** 统一窗口不是把消息收进来，而是让影响交付的变化最终改动同一版权威事实。

### Context Snapshot + Contract：冻结事实与行动边界

需求承诺点示例：**work_id=REQ-VIBOPS-017 · run_id=RUN-2026-001 · contract_version=v1**

- **Context Snapshot（事实底座）** v2026.05.26-1015（生成时间 2026-05-26 10:15）：基于哪些事实工作——业务目标（港股 IPO 需求）、项目/代码/依赖版本、引用知识/规范/历史决策/Evidence+来源；来源系统：VibOps / GitLab / K8s / 知识库。版本规则：work_id 不变，始终指向同一需求身份；新 run_id——每次执行生成新的 run_id；contract_version——Contract 变化时才发新
- **Contract（行动与完成边界）** v1：Feature Owner / 技术 Owner / 验收 Owner（业务结果、变更安全、Evidence/验收）；Owner 批准即执行起点（Contract Approved）；允许做什么/怎样算完成/何时必须停——目标/范围/非目标/验收示例、验收标准/Evidence 要求/完成定义、授予权限（边界）/允许工具（边界）、资源预算（边界）/风险处置边界/停止条件（边界）
- **阻断规则（红）**：来源冲突（事实不一致）、事实过期（Snapshot 过期）、权限不全（必需未授予）、验收定不下（标准/Owner 不满足）→ **Blocked，不补造**；Required Evidence（需产出与归档的证据清单）、停止条件（何时必须停止与阻断）

**执行前先冻结"基于哪些事实·允许做什么，怎样算完成·谁负责"——AI 可控、可追责、可复现的起点，也是质量与决策前移的落点。**

### 高手的方法，不能只留在高手手里：统一工程协议

让正常交付与 Debug 遵守同一套可验证协议：角色 Skill（怎样分析与检查）、项目 Rule（哪些约束不能破）、专业 Agent（以什么职责执行）。

- **正常交付主链**（spec-first Harness）：Spec → Plan Gate（未批准不执行）→ Work → Review（Build/Test/Review 事实）→ Evidence；需求不清 → 返回澄清
- **Failure/Bug 处理（不靠猜）**：稳定复现 → 收集 Evidence（日志/堆栈/配置/测试数据）→ 根因分析 → 最小修复 → 回归验证；无法复现 → Blocked（证据不足/权限缺失）
- **知识晋升**：Bug 描述 = Signal，附 Evidence，沉淀为 K2 已验证经验（RCA + 修复 + 独立 Review + 回归通过 → Workspace/solutions，归档治理后进入下一次 Context）

（spec-first 为团队自研的 AI Coding Harness。）

### VibOps 如何低成本接入现有研发平台？

**MCP 将已有能力工具化，妮蔻脚本负责确定性执行**：Workspace Task/Contract → Agent 判断与发起 → 妮蔻 MCP Tool → 妮蔻脚本 → 既有企业研发系统（**原始事实留在原系统**）。约束：Contract / 最小权限 / 环境 / 停止条件；锚点：work_id / run_id / step_id。流程：①能力发现（可用能力清单、Schema 检索）→ ②理解任务与目标（装配：获批动作、构造参数）→ ③选择合适能力（有效 Schema、输入/输出/校验、映射到具体脚本）→ ④构造参数 → ⑤请求执行 → ⑥接收结果 → ⑦Human Gate（具名审批后才能进入执行；最小权限＋环境边界、密钥权限/不可逆动作、重试/超时、审计记录）。已接入系统：飞书/群聊（需求/设计）、K8s/容器/部署、GitLab 代码/PR/分支、ELK/日志/监控、CI/构建/测试/制品、Nacos/配置/注册、BizDevOps/发布/变更、测试数据库/数据服务。

执行结果分类：Success / Failed / Partial → Result、Log Ref（外部日志引用）、Evidence、State、Next Action（Retry / Blocked / 人工接管；**禁止假装全链成功**）。回写 Workspace 只记录关联与状态。共同锚点：work_id + context_version + source/version/permission/scope。

**低成本要点：不替换、不迁移、不重建平台，只增加受控调用与结果回写。**

### Evidence + Owner Acceptance：什么才算完成

Agent completed ≠ 完成；Test Gate passed ≠ 完成；Deployed ≠ 完成。**Business Accepted = 需求完成**。双阶段决策主链：

- **阶段 1（发布前）**：Evidence 采集（功能实现/代码/单元/集成/安全等）→ Bundle/Gate（证据打包/门禁校验：质量/风险/覆盖等）→ Human Release Approval（人工发布审批）→ Deploy（发布到目标环境）
- **阶段 2（部署后）**：Evidence 采集（运行验证/监控/日志/性能/数据等）→ Acceptance Gate（验收门禁：业务/体验/指标等）→ Owner Acceptance（Owner 验收确认）→ Business Accepted（需求完成）

判定与责任（Record）：Bundle（本次交付完整证据集）、Gate Verdict（门禁判定结果：通过/驳回/需补证）、Owner Decision（Owner 业务接受，负责结果与价值）；驳回/补证/Rework/Rollback。

**spec-first 规定如何工作，Evidence 证明做成什么，Owner 决定是否接受。**

### 归档：让每一次交付，都成为下一次需求的起点

文档随需求归档，代码随 master 每日增量更新；两类回执共同通过 **Archive Check**，需求才进入 **Archived**（需求完成的唯一状态）。

- **文档知识线（事件驱动，按需求归档）**：各节点已产出的权威工件 → 提取总结 → 结构化分类入库 → 版本化 → 文档归档回执（work_id/来源/版本/权限/适用范围）。7 类文档知识库按 4 个知识域展示：需求与设计（需求 PRD 知识库、设计稿知识库）、研发与测试（技术方案知识库、测试用例知识库）、问与改进（Bug 超验支持集等）、发布与追溯（发布内容/版本知识库）
- **代码知识线（时间驱动，按 master 每日增量）**：每天凌晨自动检测已发布的新增 Commit（commit 水位）→ Zread/FastGPT 生成增量代码文档 → 更新全局代码知识库（最新已发布事实，仅包含已发布 master 的真实代码事实）→ 代码更新回执（repo/commit/采集水位）
- Archive Check（完成性校验）：来源可追溯、版本与权限正确、完整性与一致性、MD5 与幂等、Evidence 具备
- 归档后生成 Next Context（文档/代码），供后续 Workspace 复用（Context 1-N）

**一次需求沉淀文档判断，一次发布沉淀最新代码；两条线共同抬高下一次需求的起点。**

## 四、交付进阶之路：我们已经怎么用，下一步走向哪里？

产研测共享工作空间，已进入 App 与 Admin 真实交付（每个需求一个 Workspace，产研测共享上下文·同一需求事实·同一完成标准）：

- **APP | 单人双端研发流程**：共享需求语义 → Android + iOS → 双端验证
- **ADMIN | 后端跨栈研发流程**：API Contract → 服务 → Admin 页面 → 测试；**Admin 开发已全部由后端研发承担**
- 真实交付数据：**7 个已完成需求 | 效率 +45% | Bug -80%+**
- 知识增长飞轮：意图识别 → 精准回捞 → 当前交付 → 知识归档与治理 → 下一需求

下一步（覆盖更多需求 | 承担更多任务 | 兑现更多组织价值）：

- 01 意图识别 + 精准回捞（知识治理：正确版本/权限/相关性；治理/识别/回捞/使用反馈）
- 02 任务看板 + 多 Agent 协同（任务/角色/阶段；同一 Work Object；依赖/WIP/Owner/Gate/Evidence；人机协作、受控自治）
- 03 数据看板反哺流程优化（E2E/等待/返工/质量/知识使用/人工接管 → 知识治理/流程/Context/Rule/Skill/Test/模型路由），系统基于 Evidence 持续进化
- **token 成本治理（贯穿全链路）**：单需求/单任务 → 无效上下文 → 有效检索与生成 → 模型路由；压缩无效上下文 + 优化模型路由 → 降低单位有效交付成本

## 结论

**组织交付力 = AI 生产力 × 生产关系适配度。** AI 让节点更快，VibOps 让端到端流动跟上——系统越用越准、协同越跑越稳。

## 相关页面

- [[openspec-sdd-practice]] — OpenSpec SDD 实战（同为 spec 驱动的交付工具链）
- [[sdd-custom-workflow]] — SDD 薄编排架构
- [[harness-engineering-practice]] — Harness 工程（spec-first Harness 与其呼应）
- [[code-knowledge-base-pyramid]] — 代码知识库金字塔（代码知识链对照）
- [[requirement-semantic-translation]] — 需求语义翻译（产品语言→代码指令）
- [[agent-hook-governance]] — 框架层 Hook 护栏（确定性兜底）
- [[agent-mcp-protocol]] — MCP 协议（妮蔻 MCP Tool 接入既有系统）
- [[agent-skills-system]] — Skills 扩展机制（Skills/Rules/MCP 注入 IDE）
