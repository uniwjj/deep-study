---
title: Loop Engineering
description: 把反馈循环放进工程现场——从手工提示到可自主运行的控制循环，六组件拆解、闭环vs开环、验证准入、预算约束、状态记忆与试点方案；补充 WorkBuddy 四层工程体系与 Goal≠Loop 区分
aliases: [Loop Engineering, 循环工程, Agent Loop, 反馈循环]
tags: [ai-agent, concept, practice]
sources: [2026/06/15/loop-engineering详解.md, 2026/07/10/从模型到Harness：WorkBuddy如何把Agent做成可用产品.html]
created: 2026-06-15
updated: 2026-07-10
---

# Loop Engineering

**Loop Engineering** 是将 Agent 从"单次提示→单次响应"升级为"持续发现问题→处理问题→留下证据"的工程方法。它不是替代 Prompt Engineering，而是把提示词放进一个有状态、可验证、可停止的反馈系统中。

## 核心判断

- Loop 不只是 cron — 它更接近一个**带反馈的控制循环**
- Prompt 没有消失，只是从聊天框里的手工输入，变成了循环系统里的一个**组件**
- 一个可用的 loop 至少要有：**自动触发、隔离工作区、过程资产、外部连接、独立验证**，再加**状态记忆**
- Loop 的价值取决于**验证是否便宜**。验证很贵/边界模糊/后果很重时，loop 会把问题放大

## 为什么需要 Loop

传统的手工提示协作模式存在系统性缺陷：

| 问题 | 根因 |
|------|------|
| 人会忘记上一轮边界 | 状态只存在聊天记录中 |
| Agent 会提前收工 | 没有外部验证压力 |
| 同一上下文自写自审，标准容易松 | 缺少独立复核 |
| 验证结果散落在聊天/终端/截图/脑子里 | 证据不可追溯 |
| 第二天要重新解释 | 上下文丢失 |

Loop 把这些来回动作变成一个有状态的系统：自动扫描→拆任务→隔离执行→独立复核→生成证据→进入人工队列 OR 继续。

## 六组件拆解

把 Addy Osmani 的分解翻译为六个工程问题：

| 组件 | 对应问题 | 解决的风险 |
|------|----------|-----------|
| **自动触发** | 什么时候启动？ | 靠人想起来才做 |
| **隔离工作区** | 在哪里改？ | 并行互相覆盖 |
| **Skills/规则模板** | 按什么规则做？ | 每轮重新解释业务 |
| **MCP/插件/CLI** | 能连到哪里？ | 只能看本地文件 |
| **Sub-agent/Reviewer** | 谁来复核？ | 自写自审过于宽松 |
| **状态文件/看板** | 怎么接上下一轮？ | 上下文中断和目标漂移 |

### 四个架构端口

1. **触发入口** — 谁能启动，多久一次，输入从哪里来
2. **执行沙箱** — 哪个 worktree/分支/临时环境里跑，能不能回滚
3. **验收出口** — 用哪些测试/日志/规则/人工复核判定结果
4. **状态账本** — 本轮尝试、证据、失败原因、下一步写到哪里

## 四层工程体系（WorkBuddy 视角）

> 来源：[[workbuddy-agent-product-design|WorkBuddy Agent 产品设计]]

WorkBuddy 将 Agent 工程分为四层，Loop Engineering 是最外层：

| 层次 | 核心问题 | 简单例子 |
|------|---------|---------|
| Prompt Engineering | 本次请求应如何表达？ | 写清目标、格式和约束 |
| Context Engineering | 这一次决策前，模型该看什么？ | 加载相关文件、工具、历史和记忆 |
| Harness Engineering | Agent 如何被引导、约束、观测、验证和纠正？ | 规则、沙箱、审批、测试、日志、编排 |
| Loop Engineering | 任务如何被触发、流转、验收、继续与停止？ | 定时任务、工作树、子 Agent、记忆、反馈闭环 |

**Goal ≠ Loop**：Goal 只定义"要去哪里"和"还剩什么"；Loop 还需要触发器、执行环境、工具、验证信号和停止条件。一个只会保存目标的功能，是 Loop 的状态组件，不是完整循环。

一个可用的 Loop 至少需要八个组件：**触发器、独立执行环境、Skills、Tools/Connectors/MCP、Sub-agents、Memory/Durable Artifacts、Sensors/Evals、Stop Conditions/Budget**。

## 与 Harness 的关系

> Harness 管的是"这一次任务怎么跑"，Loop 管的是"这类任务怎么持续发生"

```
Prompt → Skill → Sub-agent → Harness → Loop
（单次）  （复用）  （分工）   （可控）   （持续）
```

详见 [[harness-engineering-practice]] 和 [[harness-build-to-delete]]。

## 闭环 vs 开环

| | 闭环 (Closed Loop) | 开环 (Open Loop) |
|---|---|---|
| **目标** | 写清楚，边界明确 | Agent 自行发现和扩展 |
| **动作范围** | 受限 | 可扩展 |
| **反馈** | 客观可验证 | 依赖人工判断 |
| **停止条件** | 可自动验证 | 模糊，容易漂移 |
| **风险** | 低（可回滚） | 高（预算/目标/验证都可能失控） |
| **适用场景** | CI分流、链接检查、事实核验、依赖预检 | 开放探索、复杂决策 |

**实践建议**：先让闭环跑稳，再谈更大的自动化。开环先放一放。

## 验证准入表

把任务放进 loop 前，检查五个条件：

| 检查项 | 可以试 | 暂缓 |
|--------|--------|------|
| **输入** | 日志/issue/测试报告/文档（结构化） | 靠口头描述，边界每天变 |
| **输出** | 分类表/候选PR/证据清单/风险说明 | 直接改核心逻辑 |
| **验证** | 有测试/lint/链接检查/可复现命令 | 只能靠人读一遍感觉对不对 |
| **权限** | 默认只读，写入走隔离分支 | 直接改主分支或生产配置 |
| **停止** | 有预算/时间/证据不足等停止条件 | 只写"继续优化"，没有退出点 |

五项里有两项落在右边，先补测试/状态/边界，再考虑自动 loop。

## 预算约束

Loop 的每一轮都可能重新读上下文、调用工具、生成计划、做验证。叠加 sub-agent 后，成本从"多几次对话"变成"一个小型后台系统"。

**任务卡模板**：

```
循环名称：每日 CI 分流
触发频率：每天 09:30
输入范围：过去 24h 失败 job、最近 20 个 commit
最大运行：30 分钟
最大分支：一次最多处理 5 个失败簇
权限：默认只读；修复必须进入独立 worktree
验证：只运行相关测试，不跑全量回归
停止条件：无新失败 / 证据不足 / 预算到达 / 需要人工决策
交付物：失败分类表、可复现命令、候选 PR、剩余风险
```

## 状态记忆

Loop 的记忆不能只放在对话里。上下文会压缩、截断、被新信息冲淡。

需要对话之外的**状态载体**（plan.md / issue / 看板 / Markdown）：

```markdown
## 当前目标
修复订单导出最近 24 小时的 CI 失败。

## 已尝试
- 读取 job 1324 日志，失败点在 export timeout。
- 检查最近 20 个 commit，疑似与批量查询改动有关。
- 尝试增加超时，不通过，已回滚。

## 已验证
- 单测 OrderExportTest.test_large_batch 复现失败。
- 小批量导出正常。

## 禁止事项
- 不改权限模型。不改导出文件格式。

## 下一步
- 对比批量查询前后 SQL。
- 如无法定位，交给人工看慢查询日志。
```

## 人在场

Loop 越强，人的判断越要提前出现。以前在每轮 prompt 里临时判断的事（下一步、可信度、禁区、停止时机），现在要提前变成**规则、模板、权限、预算和停止条件**。

五条保守原则：

1. 先只读，后写入
2. 先低风险，后核心路径
3. 先小频率，后高频率
4. 先人工确认，后自动合并
5. **先写停止条件，再写继续条件**

一个成熟的 loop 应该能诚实地说：*"我没有足够证据继续。"*

## 试点路径（7天）

| 天 | 动作 | 要点 |
|----|------|------|
| 1 | 选场景 | 低风险：CI分流/文档检查/事实核验，输入稳定 |
| 2 | 写任务卡 | 输入范围、权限、预算、停止条件、交付物 |
| 3 | 做 Skill | 项目规则、常见误区、验证命令、输出格式 |
| 4 | 接状态记忆 | plan.md/issue/看板记录已尝试/已验证/阻塞/下一步 |
| 5 | 跑手动 loop | 人手动触发每一步，确认哪些要自动化，哪些要人工 |
| 6 | 加自动触发 | 固定频率+时间上限+预算上限，输出先进人工 inbox |
| 7 | 复盘 | 省了多少重复排查、误报多少、人工复核是否更轻 |

### 复盘验收指标

| 指标 | 看什么 | 标准 |
|------|--------|------|
| 命中率 | loop 发现的问题有多少值得处理 | 人工复核后仍有稳定收益 |
| 误报率 | 多少输出浪费人的时间 | 不影响日常节奏 |
| 回滚率 | Agent 写入后有多少需要撤回 | 写入先控制在候选分支 |
| 成本 | token/时间/工具调用次数 | 账单和收益能对上 |
| 证据 | 每个结论有无来源/命令/日志 | 人能在5分钟内复核一轮 |

## 试点场景优先级

| 场景 | 为什么适合 |
|------|-----------|
| 技术稿事实核验 | 读多、改少、证据可列 |
| CI 失败分流 | 输入结构化，验证路径明确 |
| 文档链接和配置漂移检查 | 可自动扫描，可人工确认 |
| 重复故障归类 | 需要并行阅读大量日志和 issue |
| 陈旧 feature flag/实验配置清理 | 天然可拆，结果可先只读输出 |
| 依赖升级预检查 | 先生成影响范围和测试建议 |

## 与 OpenClaw 的对照

OpenClaw 是 Loop Engineering 理念的一个超前实现——六组件齐备，且在记忆系统和多模态处理上远超 Loop Engineering 描述的最低标准。但两者有结构性差异：

### 六组件逐项对照

| 组件 | Loop Engineering 的要求 | OpenClaw 怎么做 | 对齐程度 |
|------|----------------------|----------------|---------|
| **自动触发** | 定时/事件驱动，写死频率和输入范围 | AgentManager 服务编排层，任务调度+事件触发 | ✅ 完全覆盖 |
| **隔离工作区** | worktree/临时分支，可回滚 | Sandbox 运行环境层，安全隔离执行 | ✅ 完全覆盖 |
| **Skills/规则** | 项目规则、模板、验证命令写成 Skill | Skills 透出 + 开源 ACP 协议透出，标准交付模式 | ✅ 完全覆盖 |
| **外部连接** | MCP/插件/CLI/连接器 | 多工具协同：联网搜索+爬虫+BrowserUse+知识库+DataAgent+IPython沙箱 | ✅ 覆盖且更丰富 |
| **独立复核** | sub-agent/reviewer 独立验证，自写自审不算 | AgentScope 多 Agent 执行引擎，多工具并行+多源交叉验证 | ✅ 覆盖 |
| **状态记忆** | plan.md/issue/看板级别的简单状态文件 | Agentic Memory 五步流水线（Extract→Embed→Store→Retrieve→Fuse），ElasticSearch VectorStore + BM25+KNN 混合召回，In-session/Cross-session/Self-evolution 三种模式 | 🔷 OpenClaw 远更复杂 |

### 四个结构性差异

**1. 验证哲学**：Loop Engineering 把验证便宜当作 loop 的准入门槛——验证太贵的任务就不该自动跑。OpenClaw 的验证内建在执行流程（多源交叉验证→反馈→修正），但它不强调"验证太贵就不要跑"这个克制判断。一个是"我能做"，一个是"你该不该做"。

**2. 预算模型**：Loop Engineering 把 token 预算当一等设计约束——任务卡先写死"最多 30 分钟 / 最多 5 个失败簇"。OpenClaw 是企业级平台，有降级限流机制，但不把 token 上限作为任务级停止条件。假设不同：个人开发者 vs 企业预算是两种经济。

**3. 记忆系统复杂度（刻意差异）**：OpenClaw 的记忆是工业级（ES、BM25+KNN、冲突判别、自进化）。Loop Engineering 刻意保持极简（plan.md 记录已尝试/已验证/禁止/下一步）。论点：验证不稳、边界不清的阶段，复杂记忆是过早投资。

**4. 覆盖域**：Loop Engineering 面向 Agentic Coding（修 bug、CI 分流、配置清理），侧重写入行为的隔离和回滚。OpenClaw 面向企业研究（多模态搜索、长文档分析、跨会话知识复用），侧重检索和分析。

### 关系

```
OpenClaw = Loop Engineering 理念的企业级实现
Loop Engineering = OpenClaw 使用说明书里被省略的"何时用/何时停"决策框架
```

两者不矛盾。即使团队用 OpenClaw，验证准入、停止条件设计、预算上限这些工程判断仍然要自己做——OpenClaw 给了能力，没替你做决策。

详见 [[openclaw-agentic-search-memory]] 和 [[openclaw]]。

## 参考来源

- Addy Osmani: [Loop Engineering](https://addyosmani.com/blog/loop-engineering/)
- OpenAI: [Using Goals in Codex](https://developers.openai.com/cookbook/examples/codex/using_goals_in_codex)
- Anthropic: [Enabling Claude Code to work more autonomously](https://www.anthropic.com/news/enabling-claude-code-to-work-more-autonomously)
- Blake Crosley: [Loops Win Where Verification Is Cheap](https://blakecrosley.com/blog/loops-win-where-verification-is-cheap)
- AlphaSignal: [Most Developers Do Not Need Agent Loops Yet](https://alphasignalai.substack.com/p/most-developers-do-not-need-agent)

## 相关页面

- [[harness-engineering-practice]] — Harness 是 Loop 的基座，管"单次任务怎么跑"
- [[harness-build-to-delete]] — 从补约束到删约束，与 Loop 的克制理念一致
- [[harness-design-long-running]] — 长任务 Agent 的设计模式
- [[agent-harness]] — Agent Harness 基础概念
- [[agent-vs-framework-vs-harness]] — 三层抽象：Loop 在 Harness 之上再加一层
- [[workbuddy-agent-product-design]] — WorkBuddy 四层工程体系与 Goal≠Loop 区分
