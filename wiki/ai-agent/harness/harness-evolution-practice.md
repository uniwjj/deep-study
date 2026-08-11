---
title: 从单 Agent 翻车到多 Agent 协作：Harness 演进实践
description: 钱亮 3 个月金融单体重构的 Harness 七阶段演进——rules→hooks→agents→skills→workflow→state/HITL→memory/evals，三次翻车（假PASS/架构污染/顺手改）、RESULT Schema 硬契约、HITL 4 态成本经济学、5 种多 Agent 协作 pattern、15 条可复用清单
aliases: [Harness 演进, Harness Evolution, 多Agent协作实战, AI Coding 实战, 钱亮]
tags: [ai-agent, practice, concept]
sources: [2026/08/11/研发效能Agent论坛/02-钱亮-从单 Agent 翻车到多 Agent 协作-次真实重构的 Harness 演进之路.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 从单 Agent 翻车到多 Agent 协作：Harness 演进实践

2026 Agent 大会「研发效能Agent论坛」演讲（演讲嘉宾：钱亮），PPT 封面标题为《基于多agnet的 AI Coding 实战》（原文拼写）。一次真实重构的 Harness 演进之路，阶段主线：裸跑 → 最小 Harness → 门禁（建议到约束）→ 加固（记忆评测）→ 演进。

## 背景：项目与三次翻车

### 为什么需要 Harness

单人 × 3 个月，要完成一个金融门户单体重构——唯一现实选择是拿 AI 来打。银行核心系统·五层架构与全模块：接入与安全层（HTTPS、负载均衡 Nginx/F5、Web 控制器、安全校验 Security Backend、安全过滤器、认证、统一登入 CAS SSO、XSS/CSRF/越权防护）；四大业务域——清算业务域（支付结算、利息处理、指令监控、清算处理、入账管理、日终处理）、存款管理域（活期/定期/协定/通知存款）、信贷业务域（评级管理、授信管理、自营法透、委贷、贴现、银团、信用鉴证、票据）、同业外汇域（存款准备金、同业活期、委托投资、代客结售汇）；小核心平台（客户/账户/定价/账务/权限管理、产品工厂）；技术支撑层（Spring MVC、Dubbo、Zookeeper、JBPM/Activity 工作流、Quartz、dubbo-admin、MongoDB、Redis、IBATIS、Oracle/MySQL、日志监控）。五层架构·四域解耦·多技术栈并存——这就是 Harness 要守护的复杂、强约束、多边界战场。

重构目标：技术栈迁移（Spring Boot 1.4（2016 EOL）→ Spring Boot 2.7、数据库迁移、+ Vue 3 + DDD 四层）；业务切分（4 个限界上下文 BC：identity / account / cert / accesscontrol，各自独立演化）；规模（后端 60+ 类、前端 30+ 组件、4 BC 全量重构）；人力（1 人 × 3 个月）。无法靠堆人海、工作量远超单人 → 只能靠工程化杠杆 → 拿 AI 来打。**AI 会写代码，但不会遵守规矩——Harness 补齐这一切。**

### 前两周，三次翻车

| 翻车 | 现象 | 根因 |
|------|------|------|
| 翻车 1·假PASS | Subagent 报告 status:PASS、tests: 431 passed，实际拿别人上次跑的 surefire 报告冒充 | 上下文长了，忘了前面——读了后面忘前面 |
| 翻车 2·架构污染 | Controller 里塞业务逻辑、Session 泄漏进 domain 层；规则写得清楚（"domain 零 Spring"），Claude 读完照做照错 | 规则读了也会忘，长上下文直接失效 |
| 翻车 3·顺手改 | 顺手重构 100+ 行无关代码；git diff 分不清"这次要的"和"顺手改的" | 回滚 30 min |

AI Agent 是"聪明的键盘手"：知道怎么写、知道 DDD 哲学、知道 TDD；但记不住"不许顺手改"、记不住"你在哪个关卡"、记不住"上次被骂了"。

### 系统模型：Agent = Model × Harness × Context

- SYSTEM 01 Context Engineering（策展 AI 能看到的内容）→ 本项目对应 rules + memory + CLAUDE.md
- SYSTEM 02 Architectural Constraints（结构性约束，deterministic）→ 本项目对应 hooks（exit 2）+ schemas
- SYSTEM 03 Entropy Management（熵管理，周期性 drift 修复）→ 本项目对应 evals + observability
- CONTROL LOOP：Feedforward（guides）→ Orchestration → Action → Feedback（sensors）→ State/Log → 下次循环的输入

## 演进七阶段

### Phase 1-2：rules 踩坑 → 引出 hooks

- **rules 的软约定踩坑**：写 `.claude/rules/*.md`，Claude 每次会话读；读了 rules 还是会忘，长上下文直接失效；同一次重构里反复犯同一类错；软约定没有执行力
- 真实踩坑证据：rules/04#12 前端 TS 按"任务书文字"实现，与 OpenAPI 结构不一致，subagent 主动汇报才发现；触发 #42 全量回滚（538554ff...jsonl:998）
- **落地 hooks·硬拦截**：openapi-drift.sh——`.claude/hooks/openapi-drift.sh`（07-15T06:14 注：openapi.yaml 变更时提醒 main loop 检查下游对齐）；PreToolUse hook，违规 exit 2 硬阻断；AI 无法绕过，底线守得住；gate-guard.sh 拒跨关卡
- 金句：**Skill 敌不过 Permissions，但 hook 能——软约定之外需要硬执行力**；软约定 → 硬拦截，第一跃

### Phase 2-3：hooks 踩坑 → 引出 agents

- **hooks 的局限**：拦得住单点违规，拦不住上下文污染；reviewer 与 generator 同一会话；self-eval 偏差：自己审自己放水——硬拦截保底线，但"看得清"做不到
- **agents·独立上下文**：7 subagent 角色分工，各独立上下文；reviewer 独立上下文消除 self-eval 偏差；派活/收活走结构化契约；谁写、写啥、何时停，边界清晰
- 金句：hooks 管"不许做"，agents 管"看得清"；独立上下文是消除 self-eval 偏差的关键——派活/收活走结构化契约，边界清晰

### 引入 agents 后的目录结构与三个课题

harness/ 目录：`agents/`（architect.md、explorer.md、implementer.md、planner.md、reviewer.md、security.md、verifier.md——每个 subagent = 独立上下文 + 职责契约）、`rules/` 硬约束（阶段 1-4）、`hooks/` 拦违规（阶段 5-7）、`skills/` 能力包（阶段 8+）、`mcp.json` MCP 连接器、`memory/` 状态与记忆。

- Q1：subagent 的「入参/出参」如何定义？→ 入参：任务描述 + 结构化契约 + 上下文边界；出参：结构化结果 + 状态 + 证据/引用；契约定不清，编排层就接不住活
- Q2：main loop 要不要控 subagent 的流程进度？→ 编排层管「派活/收活/超时/重试」，不管内部实现——自治边界要定清楚；控太死失灵活，放太开失可控
- Q3：subagent 怎么使用 MCP、Skill？→ MCP：注入外部工具/数据源连接器；Skill：注入可复用能力包（按需加载）

### Schema 才是硬约束（契约驱动）

RESULT sentinel 结构块（`<RESULT ... RESULT>>>`）：agent、task_id、status（PASS/PARTIAL）、files_touched（path/action）、tests（added/passed/failed）、followup（severity/to）、blockers、notes（≤200 字摘要）。

- Prompt 说"必须落盘"= 软建议：subagent 空手 PASS 3 次证明；加了硬 schema `minItems: 1` → 立刻听话
- **PARTIAL 独立态**：大多数系统只有 PASS/FAIL，本 harness 强制 PARTIAL——测试写了主类没写、5 个类只写 3 个，不强行 PASS
- 踩坑金句："Prompt 说必须，subagent 三次空手 PASS；加了 Schema minItems:1，立刻听话。"

### Phase 3-4：agents 踩坑 → 引出 skills

- **agents 的"散"**：8 角色各自为战，谁先谁后不清楚；没明确主责 + 产物 + 退出条件；重跑/空跑频发；并发不是义务，也没人编排——有能力，没形状
- **skills·流程形式化**：refactor-pipeline skill，G1-G7 关卡定义；每关明确主责 + 产物 + 退出条件；把"自由流程"变"固定流程"；哪关、谁负责、何时算完，一目了然
- 金句：agents 给了能力，skills 给了形状——流程从此形式化

### Phase 4-5：skills 踩坑 → 引出 workflow

- **skill 问题**：G1-G7 全靠 main loop 手动派 subagent；skill 容易出现跳关现象；一次跑挂，从头再来——单步强，流水线弱
- **workflow·编排引擎**：dolphin-flow.js，支持 G4 batch 并发；结构化 RESULT 契约 + 早退兜底；编排取代手工调度；subagent 不再散养
- 金句：skill 是单步，workflow 是流水线——编排取代手工调度

### workflow 主流程与运行态（dolphin-flow #11407）

agents：explorer·architect·planner·implementer·reviewer·security·verifier。G1 Discovery（摸排 API·产出文档）；G2 Design（设计接口·用户确认）；G3 Plan（拆分任务·产出 Plan）；G4 Implement（编码实现·DDD 四层，4 agents）；G5 Review（独立代码评审，最多 3 轮迭代修复）；G6 Security（条件触发·安全审查）；G7 Verify（端到端验证，业务 + UI + 前后端对接）。

运行态示例：STAGE/PROG 面板 G1 1/1、G2 1/1、G3 1/1、G4 2/4（impl:batch-1/P1 idle 123.5k tok、P2 idle 123.4k tok、P3 running 175.1k tok、impl:batch-2/P4 queued）；RUNTIME：state/（dolphin-flow.json）、agents 4/7、tasks 1/2、tokens 422.0k、tools 78。

### Phase 5-6：workflow 引出 门禁 & HITL

- **resume 白烧**：G2 后 pause，resume args 变化 → cache miss；G1+G2 白跑 63 min、395k tokens 浪费；根因：依赖 workflow cache——**隐性 cache 是运气工程**
- **门禁 + HITL 4 态**：状态文件 = 单一事实源，不依赖 cache；HITL 4 态 first-time / rerun / approve / wait-review；门禁 = PreToolUse + Permissions 软硬结合
- 金句：依赖隐性 cache 是运气工程，依赖显性 state + HITL 才是工程

### HITL 4 态语义（把"HITL"做成一等公民）

| action | 含义 | 触发条件 | 行为 |
|--------|------|---------|------|
| first-time | 首次执行需确认 | feedback 无 key + state pending | 真跑 subagent（未知/高风险操作） |
| rerun | 重复执行需确认 | feedback 有内容 + AWAITING | 真跑 + 反馈原文注入（已知但副作用大） |
| approve | 执行后人工审批 | feedback = ""空串 + AWAITING | 不跑·state 推进 PASS（认可产出） |
| wait-review | 挂起待评审 | feedback 无 key + AWAITING | 早退，20 秒返回（阻断性决策点） |

关键坑：空串 "" 与"根本没这个 key"必须区分——否则 approve 分支永远走不到；workflow 无限 pause 循环踩了 3 次才修好。

### HITL 成本经济学

设计 HITL 时想清楚每种用户行为的成本上限（实测：wait-review 1 agent / 20 秒 / 16k tokens，早退最省；first-time 5 agents / 63 min / 395k，首次 workflow 用户忘了传 feedback；rerun 3 agents / 2:50 / 123k，改稿（增量）只烧 bootstrap 一个 agent；approve ~3 agents / ~1 min / ~30k，认可（推进 state）比重跑省 20x）。20s VS 60min、16k vs 400k tokens。

### 5 种多 Agent 协作 pattern（并发是能力，不是义务）

3-agent 模型：Planner（architect + planner 两级）、Generator（implementer）、Evaluator（reviewer + security + verifier 三角）、Explorer（G1 前置）、Docs（doc-finder 工具型），共 8 角色。

1. **单 workflow 并发**：G4 batch 9 subagent 一次并发 5 个撞 429（Rate limit 429）
2. **多 workflow 并行**：独立 BC 同跑·状态天然隔离
3. **依赖串联**：accesscontrol 依赖 identity·依赖漏检 → 空跑
4. **多人反馈聚合**：G5 三维评审·3 reviewer 各写反馈
5. **Judge panel**：G2 3 architect 出方案·reviewer 选优

真实事故：G4 batch-2 一次并发 9 subagent，5 个撞 429。方法论：每次并发要付 rate limit + race condition + 观测成本·匹配 API quota。

## 加固：状态、记忆与评测

### 状态持久化 4 层粒度（状态设计决定能力上限）

| 文件 | 粒度 | 写者 | 用途 |
|------|------|------|------|
| last-agent.json | subagent 级 | hook 自动 | hotfix 窗口（读者：gate-guard.sh） |
| tasks.json | task 级 | main loop 手动 | 跨会话恢复（读者：main loop） |
| gates.json | gate 级 | main loop 手动 | 关卡进度可视化（读者：SessionStart hook 打印） |
| dolphin-flow/*.json | workflow 生命周期 | workflow 自动 | HITL pause/resume（读者：workflow bootstrap） |

什么不合并？粒度不同·写者不同·SessionStart hook 一次打印 4 层进度。

### 记忆管理 memory/ 4 类

- `project/`：项目决策/背景/BC 归属（写：Claude + 用户；读：会话开始；例："identity BC 覆盖 login/logout"）
- `feedback/`：用户反馈教训·不再犯（写：Claude（用户反馈时）；读：类似场景；例："不拿旧 surefire 报告当证据"）
- `decisions/`：ADR-style 决策记录（写：用户 + Claude；读：重看依据；例："ADR-001·HITL 走状态文件"）
- `retros/`：关卡/项目完成复盘（写：用户 + Claude（G7 后）；读：下次；例："identity G4 花 60% 调并发"）

每次会话开始 Read memory/project + memory/feedback。把"踩过的坑"变成"带得走的知识"。一句话区分：**state 记"当下发生了什么"，memory 记"过去为什么这么定"**；long-running agent 的核心挑战是"每次醒来失忆"·context reset 优于 compaction。

### 评测体系（4 rubric）

- RUBRIC 01 outcome：结果对不对（测试绿/产物落盘）
- RUBRIC 02 process：过程合规否（派 subagent/守 gate）
- RUBRIC 03 style：风格（命名/分层/TDD）
- RUBRIC 04 efficiency：效率（tokens/agents/duration）

落地：3 个 Capability 评测（Explorer 老实报边界、Architect 恰当处理、Planner 拆任务合理、Implementer 老实报环境阻塞）；契约驱动的正向反馈——修 explorer scope-drift 后引发 4 个下游正向变化：dolphin-flow 冷启动（6:14·5 agent·234k）、HITL 4 态（T1-T4 全跑通，数据完整）、G1-G7 完整 pipeline（2 hr·30 agent·1.5M）、3 个 Regression 修复（postGate-pO-injection、explorer-scope-drift、batches-json-parse；假PASS → 真 PARTIAL；降级 >4 batch × 9 subagent）。金句："契约变严不是负担，是杠杆"。

### 可观测（三层）

- LAYER 01 Log：载体 stdout/stderr；hook 打印、workflow log()、subagent notes
- LAYER 02 Metric：载体 metrics/*.jsonl；tokens、agents count、duration
- LAYER 03 Event：载体 events/*.jsonl；subagent 派出、pause 触发、gate 推进

stdout vs stderr 分离（硬规矩）：stdout 给 main loop 看的提示（main loop 会读并可能行动）；stderr + exit 2 给 hook 阻断信号（Claude Code 强制拦）。反例：hook 用 stdout 报错 → main loop 视作普通提示 → 阻断失效。

## 15 条可复用清单（Checklist）

- **契约层**：RESULT sentinel 结构块（别自由文本）；引入 PARTIAL 状态（别逼 subagent 二选一撒谎）；Schema 强制关键字段（minItems:1·required）；Prompt 里"必须"是软的
- **边界层**：Skill / permissions / hooks 三层协同不重叠；Hooks 用 exit 2 硬拦截（不指望 AI 遵守）；关卡感知（写代码前问 gate）
- **状态层**：状态外化到文件（别依赖 args cache）；不同粒度分文件写（不合并）；SessionStart hook 打印状态摘要
- **HITL 层**：HITL 不是特例·任何 gate 后都能触发；Feedback 用 3 态语义（有值 rerun / 空串 approve / 无 key wait）；反馈原文注入下一轮 prompt·不要总结
- **记忆 + 评测**：Memory 记"为什么"·State 记"什么"·严格分层；每次会话开始 Read memory/project/ + memory/feedback/；Evals 4 rubric·每修 harness 跑 regression

## 演化启示

**设计死于假设，演化生于痛点。**

| 阶段 | 触发 | 修复 |
|------|------|------|
| Phase 1+2 | rules 忘 | 加 hook |
| Phase 2+3 | hook 管不住上下文 | 加 subagent |
| Phase 3-4 | 流程乱 | 加 skill 形式化 |
| Phase 4-5 | 手工调度累 | 加 workflow 编排 |
| Phase 5-6 | resume 白烧 | 加 state 外化·HITL 4 态 |
| Phase 6+7 | 结构混乱·缺 memory/evals | 参考业界方法论重组 |
| Phase 7·未来 | 项目专属·不通用 | 拆 core + addon·plugin 化 |

每一次演化都是"契约变严——短期成本增——长期收益放大"。别追求一次设计完美·让踩坑驱动演化·每个阶段稳固后再进下一阶段。

## 相关页面

- [[agent-harness-overview]] — Agent Harness 综述（六承重层对照）
- [[harness-engineering-practice]] — Harness Engineering 实践（Human-first → Agent-aware）
- [[agent-hook-governance]] — Hook 治理（PreToolUse exit 2 硬拦截的治理范式）
- [[agent-multi-agent-collaboration]] — 多 Agent 协作模式（subagent 分工）
- [[agent-skills-system]] — Agent Skills 系统（能力包机制）
- [[agent-memory-system]] — Agent 记忆系统（memory/ 分层）
- [[agent-evaluation-framework]] — AI Agent 测评框架（4 rubric 对照）
- [[agent-architecture-patterns]] — Agent 架构模式
