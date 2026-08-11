---
title: DataWorks Data Agent
description: 阿里云 DataWorks 双引擎驱动的全托管数据智能体——代码 Agent + 运维 Agent 共享统一数据语义与上下文，MCP 协议开放 Skill 生态，覆盖数据开发、治理、分析与引擎管控运维；2026-08 定位升级：从「人机交互」到「碳硅协同」，六层语义资产 + 安全四准则 + 云端全托管三模式
aliases: [DataWorks Data Agent, 数据智能体, DataAgent]
tags: [big-data, ai-agent, platform, tool]
sources: [2026/05/12/01. DataWorks Data Agent.pdf, 2026/05/12/DataWorks DataAgent分享录音.md, 2026/05/28/DataWorks Data Agent：新一代数据智能体，企业大数据的数字员工.pdf, 2026/06/23/DataWorks Data Agent 总体架构.svg, 2026/06/23/DataWorks DataAgent入口与交互模式.svg, 2026/06/23/DataWorks 架构运行流与内核.svg, 2026/08/11/Agent 驱动的数据智能新范式(阿里云专场)论坛/04-闫建丽-DataWorks Data Agent：从“人机交互”到“碳硅协同”的企业级大数据智能体.pdf]
created: 2026-05-12
updated: 2026-08-11
---

# DataWorks Data Agent

![产品架构图](sources/2026/05/12/DataWorks%20%20DataAgent架构图.png)

2026 年正式发布的新一代数据智能体产品。定位为企业大数据的"数字员工"，覆盖数据开发、数据治理、数据分析与引擎管控运维 Agent，提供全新自然语言交互体验。

- 议题副标题：揭开面纱——新一代数据智能体，企业大数据的数字员工
- 演讲人：**孙文强**｜阿里云智能集团高级技术专家、DataWorks 团队
- 现场金句（2026-05-28 杭州虾聊日录音）：
  - "两个 Agent、两类场景，共用一颗大脑。"
  - "真正决定它们与众不同的，不是左边能写代码、右边能调度，而是下边这层共享的数据语义层。"
  - "AI 并不是写代码非常快，而是把数据开发任务从人串联的过程，变成人提出目标、AI 推进过程、最终成果交付。"
- 现场两个故事：①周五下班前业务方临时要求"周一上线一张用户活跃度表"——传统 7-8 个环节、3-4 个系统、以天计算；DataAgent 接管目标→生成代码→调试→交付。②凌晨周期调度任务故障——传统值班同学打电话上线翻日志；DataAgent 主动把错误信息+原因+解决方案推送到钉钉/飞书/企微 IM 群，自然语言交互即可解决。

## 摄取说明

2026-05-31 按多模态资料首次摄取规则重新复核 `2026/05/28/DataWorks Data Agent：新一代数据智能体，企业大数据的数字员工.pdf`：不以派生 TXT/OCR 为唯一可信来源，而是从 PDF 正文转为页面图片逐页视觉检查。该 PDF 共 17 页，均按高密度 PPT 处理；架构图、演示截图、流程图、表格、命令示例、数值与路线图均纳入逐页索引。演示截图中的模糊小字只摄取可可靠辨认的部分。

## 逐页摄取索引

| 页码 | 主题 | 已摄取信息 |
|------|------|------------|
| P1 | 封面 | DataWorks Data Agent；双引擎驱动的全托管数据智能体；一句目标，数据链路端到端自动完成；2026 official launch |
| P2 | 两个真实场景 | Coding 场景“周五 17:30 加用户活跃度日表”和 Ops 场景“凌晨 2:47 核心数据链路告警”，对比过去人工流程与今天 Agent 自主完成 |
| P3 | 从增强到自主 | 智能补全、智能问答、IDE Copilot、ChatBI、DataAgent 五阶段演进；Copilot 到 Autopilot；会问、会写、会做 |
| P4 | 两个 Agent 两类场景 | CLI 模式代码 Agent 自主交付；Claw 模式运维 Agent 自主执行；共享统一上下文层 |
| P5 | 产品架构 | 用户交互层、Data Agent 智能体层、大模型层、计算资源层、数据湖仓层；四类 Agent 与扩展能力 |
| P6 | DataAgent Core 技术架构 | Natural Language Access、ACP Gateway、Master Router、Code/Ops Executor、统一策略内核、数据语义层、平台服务、云底座 |
| P7 | CLI 模式代码 Agent | CLI/IDE 自主完成代码任务；秒级、多轮、跨文件、交付即测；终端示例展示规划、执行、测试、提交 review |
| P8 | Claw 模式运维 Agent | 企业 IM 群自然语言指令，秒级告警响应、字段级默认脱敏、写前确认、全场景运维自治；故障诊断示例 |
| P9 | 多端协同触达 | 一个 DataAgent 统一内核，CLI 主入口、IM 主入口、API 与 IDE 延伸触点，多端一致体验 |
| P10 | DataAgent CLI 演示 | DataWorks DataAgent powered by Qwen Code，版本、模型、示例问题、Beta 会话提示 |
| P11 | DataClaw 演示截图 | IM 内 DataWork_DataClaw，DataWorks 领域 Skill 表、工作空间巡检、系统内置 Skill 与“约 13 个 Skill，11 个专注 DataWorks” |
| P12 | 数据分析 Agent 演示封面 | DataWorks Data Agent / 数据分析 Agent 产品演示 |
| P13 | 数据开发 Agent 演示封面 | DataWorks Data Agent / 数据开发 Agent 产品演示 |
| P14 | Skill 开放生态 | 三类贡献者、一个 Skill 库、CLI/Claw 双模式调用；MCP 协议、Schema 校验、权限/配额/计费策略/灰度发布 |
| P15 | 全托管运行底座 | DataWorks 原生承载；资源组统一承载 Agent 运行负载；代码执行沙箱、Claw 分布式服务、MCP Skill Runtime |
| P16 | 路线图 | DataAgent 不是终点，是新的起点；现在正式发布、下一阶段 Skill 开放与生态共建、远景数据领域自治智能体平台 |
| P17 | 结束/转场页 | DataWorks Data Agent；接下来真实场景客户实践与扩展能力大数据引擎团队 |

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

## 2026 年正式发布

2026-05-28 虾聊日上，DataWorks Data Agent 作为**双引擎驱动的全托管数据智能体**正式发布。产品 Slogan：**"一句目标，数据链路端到端自动完成"**。

发布现场演示了三个核心场景：
- **DataAgent CLI 演示**：输入目标 → 代理自主完成代码任务的完整链路
- **DataClaw 演示**：在 IM 群中通过自然语言指令完成运维全流程
- **数据分析 Agent 演示**：自然语言驱动的交互式数据分析

> DataAgent 不是终点，是新的起点 —— 开放架构、可扩展生态、持续演进。

## 2026-08-11 Agent 大会补充：从「人机交互」到「碳硅协同」

2026-08-11 2026 Agent 大会「Agent 驱动的数据智能新范式（阿里云专场）论坛」分享《DataWorks Data Agent：从"人机交互"到"碳硅协同"的企业级大数据智能体》。分享人：**闫建丽**｜阿里云智能集团高级产品专家。PDF 共 22 页，已按文本层 + 关键页转图 OCR 视觉复核。以下为本次演讲相对 2026-05-28 发布内容的新增信息。

### 新范式：从「支撑人」到「支撑 Agent」

Agent 时代，企业数据平台的核心能力正在被重新定义：

- 过去·数据平台「支撑人」：平台服务于人的查询、开发与运维；人机交互：人点、人写、人排障；生产与消费效率受限于人力
- 现在·数据平台「支撑 Agent」：平台服务于 Agent，也服务于用 Agent 的人；智能协同：Agent 懂语义、调平台、能执行；平台面向 Agent 供数、编排、安全执行

双向重构（数据侧 + 平台侧同时发生）：① 数据的生产与消费：人机交互 → 碳硅协同（人与 Agent 的智能协同）② 数据平台自身：面向人 → 面向 Agent（供数·编排·安全执行）。

Data Agent = 企业员工的「数字同事」，已在阿里巴巴集团及公共云客户生产环境规模化落地，演进路径：助理协同 → 独当一面 → 引领全局。执行链路六步：意图理解 → 方案规划 → 任务执行 → 安全校验 → 人工确认 → 任务完成——安全校验与人工确认内置在链路中，智能不等于放任。

### 产品架构（2026-08 版）

与 2026-05 版一致的层式架构（OCR 复核）：Data Agent 智能体层——ChatUI（具备 CLI 完整能力）、IM Channel、Remote Control (H5)；四类 Agent：数据开发 Agent（需求分析、NL2SQL、任务构建、工作流构建、代码 Review、运维诊断）、数据治理 Agent（快捷找表、元数据增强、AI 检查器、数据质量生成、血缘分析、治理计划生成）、数据分析 Agent（快捷问数、深度分析、报告生成、报告分享）、数据运维 Agent（异常诊断、智能运维、资源管理、性能优化、知识问答、集群配置）；Agent 协议层（ACP/A2A）；Agent 上下文与扩展（第三方 Skill、第三方 MCP、知识库/Rules、语义模型）；计算资源层（MaxCompute、EMR Spark、EMR StarRocks、AI 搜索 ES/OS、AI 计算 PAI DLC）；数据湖仓层（OpenLake (Lakehouse)、Data Warehouse）。

### 差异化优势与云端全托管

别人做「数据消费」，我们重构「数据生产方式」：全链路覆盖（集成 → 开发 → 治理 → 运维 → 分析，端到端闭环而非单点工具）；平台深度集成（元数据/血缘/模型/指标/代码原生供给）；企业级可控（数据不出域·关键操作人工确认·权限与 GUI 一致·全程可审计）；业务价值转化（生产环境量化提效，规范自动执行、口径统一）。生产环境实测：需求到上线 天/数小时 → 分钟级别。

从 IDE 插件到云端全托管：一个 Agent，多入口随时接管，7×24 自主在线——

- **ChatUI 模式**：浏览器内对话式开发，打开即用、无需本地安装，具备完整的 CLI 命令执行能力
- **IM Channel 模式**：钉钉·飞书·企微，告警即诊断即修复，DataClaw·7×24 自主运维
- **Remote Control 模式**：临时离开电脑，手机端查看任务进展，随时接管

### 懂语义：六层语义资产

AI 已经会写 SQL（NL2SQL），为什么还问不到准答案？缺的是把「业务语言」翻译成「数据语言」的语义资产。语义资产 = Agent 的数据护城河：

| 层级 | 内容 |
|------|------|
| L1 字段 | Schema · 释义 · 字段血缘 |
| L2 指标 | 口径 · 同义词 · 归属 · 版本 |
| L3 血缘 | 上下游链路 · 影响面分析 |
| L4 质量 | 健康度 · DQC · 可信度评分 |
| L5 反馈 | 使用反馈 · 采纳率 · 持续学习 |
| L6 制度 | 法规 · 业务规则 · 敏感策略 |

Agent 使用示例：提问「本季度东部区域营收环比」→ 识别「营收/环比」调 L2 指标 → 命中口径卡后补 L1 字段、校 L4 质量 → 生成 SQL 时自动带上正确的表与过滤逻辑。价值：回答更准（口径一致减幻觉）· 成本更低（指标复用）· 治理沉淀（越用越懂业务）。

### 融安全：数字员工的第一准则

智能 ≠ 放任·安全与可控不是附加特性，而是被企业信任、可长期运营的前提。四准则：二次确认（关键操作人工确认）；权限一致（Agent 权限 ≤ 用户权限）；全程审计（操作留痕·链路可追溯）；数据不出域（私有化·沙箱·脱敏）。

高危动作拦截示例：「删除 dwd_order_fact 表」→ 识别 DROP·影响下游 12 张表 → 拒绝执行，返回影响面报告；「一键发布 47 个节点到生产」→ 拆解变更清单 → GUI 人工二次审批 → 平台执行并回写审计。权限体系一套账号贯穿到底：1 企业身份中心（继承当前用户在 DataWorks 的角色）→ 2 DataWorks 权限（继承当前用户的数据表/字段权限）→ 3 Agent 会话权限（只能做用户能做的事）→ 4 审计留痕（每次调用/生成/执行全链路可查）。可运营：操作留痕可审计 → 反馈闭环·驱动 Agent 持续学习。

### 开放能力：按需安装 Skill / MCP

开放可扩展：支持自定义 Skill / MCP，以扩展 Data Agent 任务完成能力，演讲以 Elasticsearch 实例诊断为例演示：1 创建实例 → 2 配置直连 → 3 故障注入 → 4 智能诊断（演讲页面为演示截图占位）。

### 全场景智能协同与客户实践

- **数据开发 Agent**：需求 → 建表/ETL/UDF → 调试 → 发布；对话式：需求分析·代码开发·调试·调度；开发耗时 小时 → 分钟，集团内规模化验证。深挖：痛点（需求理解靠人工调研、SQL 手写、代码规范难落地、离职脚本没人看得懂、需求到发布以天计）vs 解法（需求分析 → 结构化开发设计文档；代码开发 → 建表 DDL/ETL/UDF；数据探查 → 数据预览&结果洞察；运行调试 → 自动定位一键修改；调度配置 → 对话式修改&发布）。效果：开发耗时 小时 → 分钟｜从需求到发布 30 分钟通关｜淘宝闪购已规模化验证。现场 Demo 链路：需求文档 → 任务分析 → 规划制定 → 元数据及知识库查询 → 数据探查 → 代码生成 → 工作流创建 → 发布生产
- **数据运维 Agent**：告警即诊断：自动拉日志/血缘/配置，输出结构化诊断报告，IM 端秒响应；排障 小时级 → 分钟级，经验自动沉淀
- **数据治理 Agent**：自然语言找数·对话式指定范围；质量规则自动配置·自主问题治理；被动治理 → 自主式治理，自动生成治理计划
- **数据分析 Agent（Chat BI）**：智能语义层更懂业务（元数据 + 语义增强）；快捷问数 + 深度分析双模式；洞察 > 归因 > 行动，全流程一键转任务
- **客户实践·淘宝闪购 × 菜鸟**（零售 + 物流，集团内两个规模化标杆）：淘宝闪购·零售：需求到上线 12–23h → 5–10 分钟；菜鸟·物流：16+ 平台人工 → AI 全链路。共同点：把专家经验沉淀为 Agent 能力。四要素：规范内嵌（命名/分层/质量规范内嵌 Skill）、端到端驱动（需求→探查→评估→开发→测试→发布全链路 Agent 驱动，无需人工流转）、Know-How 沉淀（业务 Know-How 深度融入 Agent，而非「工具 + AI」外挂）、即插即用复制（最佳实践沉淀为 Skills 与知识库，新业务线接入无需重新教学）

### 展望：自主数字员工

从「智能协同」迈向「自主执行」——硅基自主执行：Agent 自主发现问题 → 自主决策方案 → 自主完成交付，不再需要人提需求——Agent 主动巡检、主动优化、主动汇报。未来 12 个月三大核心方向：更多源/频道对接（像八爪鱼般触达更多 IM·数据源·文档·技能）；知识库（融合公共·企业·客户知识，沉淀更强知识体系）；Agent 连接器（双向开放·既接入第三方 MCP·Skill·Agent，也把 Data Agent 能力多样化对外输出）。

## PDF 逐页复核补充

### P2：两个真实场景

P2 用两个真实工作场景说明 DataAgent 的诉求一直很具体：过去大量消耗人工时间的工作，今天由 Agent 自主完成。

- Scene 01 Coding：周五 17:30，业务方提出“下周一上线前，加一张用户活跃度日表”。过去流程包括对齐口径、查阅规范、溯源上游、编写 SQL、配置调度、接入监控、测试、提 PR，合计 8 个环节、跨 3 个系统，预估 2 个工作日；今天变成 CLI 中一句目标，代码 Agent 自主完成端到端交付。
- Scene 02 Ops：凌晨 2:47，客户核心数据链路告警触发，值班 oncall 接入。过去需要 oncall 唤起、登录平台、日志检索、血缘追溯、任务修复、重新调度，合计 6 个步骤，MTTR 平均 30 分钟以上；今天变成 IM 群内运维 Agent 自主诊断、自动定位、自动重跑。

这一页的核心结论是：两类工作过去消耗大量人力时间，今天由 Agent 自主完成，这就是 DataAgent。

### P3：从增强到自主的五阶段

P3 把数据智能体演进总结为同一条曲线上的五个范式：

| 阶段 | 定位 | 核心能力 |
|------|------|----------|
| 智能补全 | 工具 | 代码补全 |
| 智能问答 | 助手 | 单轮问答 |
| IDE Copilot | 副驾 | 上下文理解 |
| ChatBI | 翻译官 | 自然语言取数 |
| DataAgent | 智能体 | 自主规划与执行 |

PPT 将前 4 个阶段归入 Copilot/Augmented/辅助驾驶，特点是 AI 增强但仍由人主导决策；DataAgent 进入 Autopilot/Autonomous/自动驾驶，特点是给定目标后自主完成。页脚用“会问、会写、会做”概括能力跃迁，即从 Augmented 协作走向 Autonomous 自主。

### P4：两个 Agent、两类场景、一颗大脑

P4 将 DataAgent 拆成两个入口/模式，但强调二者共享同一份数据语义与上下文。

- CLI 模式：代码 Agent，自主交付。在 CLI 主入口与 IDE 融合中自主完成代码任务，覆盖 SQL、Python、Notebook 端到端。
- Claw 模式：运维 Agent，自主执行。在 IM、告警、任务流中自主完成运维任务，覆盖诊断、调度、治理。
- 统一上下文层：数据语义、会话记忆、安全权限。

页面底部明确说明：两者均为自主 Agent，差异仅在使用场景与入口，共享上下文，可跨场景协同。

### P5：DataWorks Data Agent 产品架构

产品架构从上到下分为五层：

- 用户交互层：ChatUI、TUI/CLI、IM Channel、Remote Control（H5）。
- Data Agent 智能体层：数据开发 Agent、数据治理 Agent、数据分析 Agent、数据运维 Agent，以及 Agent 上下文与扩展。
- 大模型层：Qwen 系列、GLM 系列、DeepSeek 系列、NL2SQL 专家模型、私有化部署模型服务。
- 计算资源层：MaxCompute、Hologres、EMR Spark、EMR StarRocks、Flink、AI 搜索（ES、OS）、AI 计算（PAI DLC）。
- 数据湖仓层：OpenLake（Lakehouse）与 Data Warehouse。

四类 Agent 的能力在架构图中展开为：

| Agent | 能力 |
|-------|------|
| 数据开发 Agent | 需求分析、NL2SQL、任务构建、工作流构建、代码 Review、运维诊断 |
| 数据治理 Agent | 快捷找表、元数据增强、AI 检查器、数据质量生成、血缘分析、治理计划生成 |
| 数据分析 Agent | 快捷问数、归因分析、深度分析、行动建议、报告生成、报告分享 |
| 数据运维 Agent | 异常诊断、智能运维、资源管理、性能优化、知识问答、集群配置 |

Agent 上下文与扩展包括第三方 Skill、第三方 MCP、知识库/Rules、语义模型，并通过 Agent 协议层（ACP/A2A）连接。

### P6：DataAgent Core 技术架构

DataAgent Core 的技术架构关键词是“统一运行时内核、双执行器架构、数据语义层、企业级全托管”。入口层是 Natural Language Access，覆盖 CLI、IDE、IM、OpenAPI；请求进入 ACP Gateway 后由 Master Router 路由。

执行层分为两类：

- Code Executor：CodeAgent/代码执行器，服务 CLI 模式。能力包括 K8s 沙箱、自主多步规划、跨引擎 SQL、ETL 工程理解、多文件读写；典型任务是数据开发、ETL 改造、复杂代码变更。
- Ops Executor：Claw/运维执行器，服务 Claw 模式。能力包括多步任务自治、人在回路确认、跨域协同、调度/治理/监控；典型任务是故障诊断、任务调度、补数、治理巡检。

中间的 Runtime Kernel 被称为统一策略内核，负责路由、权限、计划、审计，强调“不是聊天框，而是受控执行的内核”。底部数据语义层由 Data Semantic Grounding、Semantic Objects、Operational Context、Governance Rules 组成，分别对应数据世界转成执行器可用语义、表/字段/指标/口径、血缘/任务依赖/SLA、权限/质量/影响分析。平台服务包括 MCP Hub、Session/RAG、Observability；云底座包括通义模型、ACK Pro K8s、MaxCompute/Hologres/Flink/EMR、KMS/RBAC/OTel。

### P7：CLI 模式代码 Agent

P7 明确 CLI 模式是代码 Agent，在 CLI 中自主完成代码任务，流程是输入目标、获取交付。它的主入口是 CLI/终端，融合 IDE 内可调起，从需求到 PR 端到端。

开发者日常表达示例包括：

- “写个用户活跃度日表，源表 `user_event`，加完跑测试” → 端到端交付。
- “把 MySQL 的 `ods_user_info_d` 同步到 MaxCompute” → 数据集成。
- “这 SQL 跑太慢了，优化下 + 加注释” → 优化重构。
- “报错 invalid identifier，给我修一下” → 纠错调试。
- “所有跟‘用户’相关的表，跨引擎找一下” → 跨引擎找表。
- “JOIN 改写成窗口函数，顺便加测试用例” → 改写 + 测试。

能力标签是秒级规划响应、多轮自主迭代、跨文件读写与改写、交付即测自动验证。终端截图中可见 `dataagent "在 dwd 层加一张用户活跃度日表，源表 user_event，加完跑测试"`，随后执行 Planning、读取 ETL 工程结构、识别源表、找到 dwd 层规范、创建 SQL、创建调度依赖、生成单元测试和数据质量规则、跑通测试，最后显示 Done、已完成提交、等待 review。

### P8：Claw 模式运维 Agent

P8 明确 Claw 模式是运维 Agent，在企业 IM 群中通过自然语言指令自主完成运维全流程。它的主入口是 IM 群，支持 7x24 可达；执行特征是规划、执行、二次确认；敏感数据默认脱敏，写操作前确认。

IM 群自然语言交互示例包括：

- “@DataAgent 诊断任务实例 12345678 失败的原因” → 故障诊断。
- “对工作流 wf-001 补数据，2026-01-01 到 01-31” → 工作流补数。
- “查看基线 base_test 的 SLA 达成情况” → 监控告警。
- “表 `ods_user_info` 的 DQC 规则补一下，缺空值校验” → 数据质量。
- “分析当前未恢复的告警 + 给修复建议” → 告警自治。
- “诊断项目空间健康度，列出风险任务” → 治理资产。

能力标签是秒级告警响应、字段级默认脱敏、写前确认人在回路、全场景运维自治。右侧示例展示在数据告警群中请求诊断任务实例失败原因，Agent 自动读取实例日志、追溯上游血缘、检查上游表数据延迟、关联下游任务影响范围，诊断结果指出上游 `user_event` 表数据延迟 18 分钟导致本任务空数据失败，并建议重跑当前任务，且已有下游 12 个任务等待确认。

### P9：一个 DataAgent，多端协同触达

P9 的核心是“统一智能内核，主场景与延伸触点协同，多端一致体验”。中心是 DataAgent Unified Core，四个触点分别是：

- CLI 端：主入口，终端/Data Studio，CLI 模式主入口。
- IM 端：主入口，钉钉/飞书/企微，Claw 模式主入口。
- API：延伸触点，面向产品集成与二次开发。
- IDE 端：延伸触点，Data Studio 编辑器内嵌。

页脚总结为“一个 DataAgent，在你需要的地方都在”。

### P10-P13：产品演示页

P10 是 DataAgent CLI 演示截图。截图中可见 DataWorks DataAgent 由 Qwen Code 驱动，版本为 `v0.14.8-dataworks.1`，模型显示为 `glm-5.1`，当前在 DataWorks official Skills 环境中运行。示例问题是“员工信息表 `my_project.ods_emp_info_d` 中，工号 EMP001 的部门数据为空，请帮我排查原因并提供修复建议。”页面提示这是 Beta 版本，删除个人开发环境实例后聊天历史会丢失。

P11 是 DataWork_DataClaw 在 IM 窗口中的演示截图。可可靠辨认的 DataWorks 领域 Skill 包括：`dataworks-project-diagnostic`（项目空间诊断，健康评估、风险诊断报告）、`dataworks-project`（项目空间查询、工作空间列表/详情、成员/角色）、`dataworks-quality`（数据质量管理，质量监控、告警规则、自动配置）、`dataworks-resource-group`（资源组运维，资源组查询、水位监控、负载查询）、`dataworks-scheduler-task`（调度任务，任务查询/修改/冻结/解冻、实例重跑/终止、依赖拓扑）、`dataworks-scheduler-workflow`（调度工作流，工作流查询/修改、补数据、冒烟测试）、`dataworks-task-diagnostic`（任务诊断，实例并行诊断、日志分析）。工作空间巡检 Skill 为 `workspace-inspection`，用于工作空间定时巡检、生成运行日报（任务状态、基线、告警、异常实例）。系统内置还有 `skill-creator`、`healthcheck`、`weather` 等；页面总结约 13 个 Skill，其中 11 个专注 DataWorks 领域。

P12 是数据分析 Agent 产品演示封面，P13 是数据开发 Agent 产品演示封面。两页本身主要作为演示转场页，没有额外架构信息。

### P14：Skill 开放生态

P14 的主题是 Skill 开放生态：一次注册，处处可用。它基于 MCP 协议，引擎团队、合作伙伴、客户都能扩展。

三类贡献者是：

- 引擎团队：把引擎深度运维与诊断能力封装成 Agent 可调用的 Skill，示例包括 MaxCompute、Hologres、Flink、EMR。
- 合作伙伴：行业 ISV、数据治理、BI、监控厂商接入专业能力，示例标签包括 ISV、BI/监控、数据治理。
- 客户自建：将企业内部规范、业务流程、私域工具沉淀为专属 Agent 能力，示例标签包括业务流程、内部规范、私域工具。

中心是一个 Skill 库（Skill Registry），包括 MCP 协议、Schema 校验、权限、配额、计费策略、灰度发布，强调一次注册处处可用。右侧两种调用模式是 CLI 模式与 Claw 模式：CLI 模式中代码 Agent 自动发现并调用 Skill，服务数据开发、ETL 改造与复杂代码变更，典型能力是沙箱执行和 PR 交付；Claw 模式中运维 Agent 在 IM 群中调用同一份 Skill，覆盖任务调度、补数与治理巡检，典型能力是多步自治和二次确认。底部流程是：01 描述协议、02 注册校验、03 策略绑定、04 自动发现、05 双模式调用。

### P15：全托管运行底座

P15 强调全托管运行底座由 DataWorks 原生承载，基于 DataWorks 资源组与云原生运行时，统一承载 Agent 的调度、执行与负载。上层 DataAgent workloads 包括 CLI/IDE、IM、OpenAPI、调度任务、MCP Skill，并统一进入 DataWorks 托管运行平面。

核心是 DataWorks Serverless Resource Group：资源组统一承载 Agent 运行负载，将调度、集成、OpenAPI、Agent 执行收敛到同一套云原生资源池，按工作空间绑定、按任务弹性伸缩。资源组能力包括统一 CU 池、弹性计划、Workspace 绑定。

底部三块运行组件是：

- 代码执行沙箱：Data Studio DevBox / ECI 弹性容器实例，支撑自主编码、文件读写与 PR 交付。
- 运维分布式服务：承载 IM 入口、任务调度、数据质量（DQC）自动化与基线 SLA 监控。
- 工具与 Skill 执行：MCP Hub 统一工具网关，具备临时凭证、权限校验、调用审计内建能力。

全托管的价值被总结为：免运维（资源组托管）、可弹性（按负载伸缩）、可联网（企业私网可达）、可隔离（租户与任务边界清晰）。页脚公式是：全托管 = DataWorks 资源组 + 云原生运行时，承接 Agent 工程化复杂度。

### P16-P17：路线图与结尾

P16 明确“DataAgent 不是终点，是新的起点”，关键词是开放架构、可扩展生态、持续演进。路线图分三阶段：

- Phase 01 现在：正式发布，双引擎、多端、全托管。
- Phase 02 下一阶段：更多 Skill 开放，更多端覆盖，生态共建。
- Phase 03 远景：数据领域的自治智能体平台。

页脚总结为“开放、可扩展、生态”，产品价值由真实场景与客户实践共同验证。P17 回到 DataWorks Data Agent 封面，并提示接下来进入真实场景、客户实践和大数据引擎团队扩展能力。

## 相关页面

- [[superetl]] — 菜鸟基于 DataWorks Data Agent 的 SuperETL 实践
- [[maxcompute-skills]] — MaxCompute 的 MCMCP 与 Skills 体系
- [[hologres-skills]] — Hologres CLI 与 Skills
- [[flink-skills]] — Flink Agent Skills 重塑实时计算
- [[emr-skills]] — EMR Skills 与智驾数据标注
- [[dataworks-stock-case]] — 个人量化案例：用 Data Agent 搭建行情分析系统
- [[dataworks-2026-0528-xialiaori]] — 2026-05-28 虾聊日全部分享内容汇总
- [[data-agent-practice-guide]] — 火山引擎数据智能体实践指南
- [[openai-data-agent]] — OpenAI 内部 Data Agent 实践
- [[dataagent-semantic-layer]] — 语义层内容分析
- [[code-agent-vs-data-agent]] — Code Agent / Data Agent 分工的更普适论述
- [[databricks-genie]] — Databricks 企业级 Data Agent
- [[infinisynapse]] — 另一种 Data Agent 基础设施方案
- [[cloud-village-data-agent-platform]] — 云村 DAS Agent 平台（另一种 Agent 平台架构实践）
- [[cainiao-data-warehouse-architecture]] — 菜鸟数仓未来架构（Data Mesh + CDM）
- [[odps-multimodal-agentic-upgrade]] — 下一代 ODPS：全模态引擎和 Agentic 升级（2026-08 同场论坛）
- [[hologres-agentic-platform]] — Hologres：一站式 Agentic 多模态检索分析平台（2026-08 同场论坛）
- [[zhupu-agentic-analytics-api]] — 舟谱数据 Agentic 数据分析与 API 生产实践（2026-08 同场论坛）
- [[xifei-data-agent-multicountry]] — 信飞科技 Data Agent 多国展业实践（2026-08 同场论坛）
