---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-harness]
---

## 7张图搞懂Claude Code的Harness架构设计 总结

**文章信息**
- **原文标题**：7张图搞懂Claude Code的Harness架构设计
- **来源**：https://mp.weixin.qq.com/s/MsWfAil7l8388WmYrdvXyw
- **作者**：FloraCat
- **发表时间**：2026-04-03 21:41

**一段话总结**：文章基于 Claude Code（57MB）泄露源码，系统拆解其作为"通用终端 Agent"的架构设计，涵盖 Harness 三层编排（入口分流→会话编排→Runtime 执行）、TAOR 循环的 Agent Loop（Think-Act-Observe-Repeat）、三层状态管理（会话运行态+全局会话态+持久化层）、LLM 驱动的文件级记忆选择（区别于传统 RAG 向量检索）、三层工具能力供给链，以及代码中暴露的远端协作、会话资产化等隐藏探索方向。作者认为 Claude Code 提供的是一个「下限可控、上限可拓」的基础框架，核心竞争力在于细节打磨。

---

**核心内容**：

**整体概述**

Claude Code 不仅仅是 Coding 产品，更是一个**通用的终端 Agent**：能循环思考、调度工具、治理权限、恢复上下文、稳定长会话。57MB 源码既呈现了企业级通用 Agent 架构，又蕴藏了 Coding Agent 外溢的方向布局。

研读路径：先让 AI 梳理**目录架构**和**模块职责**，对工程架构建立整体认知，再深入各模块的设计哲学。

![Claude Code 项目目录架构与模块职责](https://mmbiz.qpic.cn/sz_mmbiz_png/3p6PfyrqhYFcX7Yl9ZamZHxElxYCYbA29TuBiaNLGAZaPJJgvrjaI0lIchpYZOYS1d3ia3ib17Q80DliaiambYy250GOKUiarYn0ZHgsicsO0QuibnQ/640)

**Harness 架构设计**

Harness 是一套「**入口适配 + 会话成型 + 运行桥接**」编排，设计本质是：**把多入口、多模式、多运行位置，收敛成一套统一的 agent turn 执行模型**。架构拆成三层：

- **第一层：入口与分流** — 接住用户不同的使用方式（命令行、交互界面、SDK 调用、assistant 模式、远端链接等），经过 `main.tsx` 做参数解析、模式判断和路由分发
- **第二层：Harness 会话编排** — 整个架构核心，标准化处理层。把三种不同形态（交互会话、无界面会话、远端接入）的请求整理成**统一的 turn 契约**，同时接入工具能力、扩展、状态与持久化
- **第三层：Runtime 与支撑层** — 本地路径进入本地 Runtime 执行完整 Agent Loop；远端路径转入远端会话宿主，由 remote/bridge/server 机制承载执行

![Harness 三层架构：入口分流 → 会话编排 → Runtime 执行](https://mmbiz.qpic.cn/mmbiz_png/3p6PfyrqhYFJdOHqC4QkRZUYot6ZAVl3ceEA7QSicyMZr8grnL4MxL7WNH4fW32aLvUTq9L1x1nbicHJ1YDbNGlicQFWxRNThWbvY4ibGTvgcnQ/640)

**Agent Loop 设计**

Agent Loop 是一种 **TAOR 循环**：Think-Act-Observe-Repeat。背后哲学是「**将智能下沉给模型，释放智能自主权**」。一次 turn 拆成稳定步骤：

> 预处理上下文 → 流式采样 thinking → 执行工具 → 拼接 toolResult → 判断是否回流到下一轮

![Agent Loop TAOR 循环流程图](https://mmbiz.qpic.cn/sz_mmbiz_png/3p6PfyrqhYFjI0r6Q3jVBylNI0GNeiaNGkOyz15LDNS8WYWkWt6ibGxppFQ7ECOKzNzgnSYZ42uZYIviaXSkp1M3iaIUlVuk9DEiccftTFfXt80w/640)

模型负责生成意图，工具负责和外部世界交互，系统把结果补回上下文决定是否进入下一轮。这不是简单的问答回合，而是一套**可持续推进任务的执行循环**，是 Claude Code 支撑复杂长任务的核心基础。

**状态管理**

采用「**会话运行态 + 全局会话态 + 持久化层**」分层协同管理：

- **AppState 会话态** — 当前会话运行实时状态（REPL/TUI 任务、MCP、插件、权限、通知、远端连接），服务于交互体验和当前 turn 运行
- **bootstrap/state 全局态** — 全局运行信息（项目位置、会话标识、成本与预算、模型与功能开关、通道与遥测），是 runtime 公共配置底座
- **sessionStorage 持久化层** — 会话记录和恢复信息写入磁盘（对话轨迹、续接数据、历史快照），供 resume、历史读取和会话恢复

三层由 **ToolUseContext** 串联：把当前会话状态暴露给 query/tools/tasks，也把 turn 内上下文带进 Agent Loop。

![状态管理三层架构：会话态 + 全局态 + 持久化层](https://mmbiz.qpic.cn/mmbiz_png/3p6PfyrqhYHOugw0YXd1M189VPvSMHQgyMfM7lNgGxUScaR5AYOY6WUIGq2FqgEzHaGVSKzLBZ3nQIy7tXWQBGCT8QVzLrdzMGcSBEZDw18/640)

**记忆管理**

有三套机制构成「**策略约束 + 指令注入 + 分层记忆召回**」的组合式上下文系统：

- **策略层** — 企业或组织下发的开关与限制，约束系统能做什么
- **指令层** — CLAUDE.md、用户级和项目级规则文件，告诉模型如何工作
- **记忆层** — 当前会话沉淀、项目与团队范围记忆、Agent 专属记忆、每轮按相关性动态注入的动态记忆

![记忆管理策略层、指令层与记忆层](https://mmbiz.qpic.cn/sz_mmbiz_png/3p6PfyrqhYEliazRzKm70klwr3XNczIoFadhbpeLicPZkkIxm1b3txIPibTWibP2tJIJOH5cfL3Wjs2DbiaGF18ndBBzxm6DQ8SIaZtOPjpDVSx8/640)

记忆加载链路是：**记忆文件 → 相关性筛选 → 附件注入 → 进入下一轮 turn**。与 OpenClaw 典型的 RAG 语义检索路线不同，Claude Code 的记忆筛选机制是 **LLM 驱动的文件级记忆选择**：

- 候选集来自磁盘记忆文件
- 选择依据是文件名、描述、query 语义匹配
- **选择器是模型，不是 embedding similarity**
- 结果是挑文件，不是查向量库 top-k chunk

![记忆加载链路：筛选、注入与回写流程](https://mmbiz.qpic.cn/sz_mmbiz_png/3p6PfyrqhYFtvn900xHpe4m1DoYJ4rfhuR7PB91AD8TnFu9ZXOyyqnOhyLjJ0HYICTLv9rUP4kso8ibeQIKGEqo5DdLX863MOmQJ9ZpKb7Xk/640)

记忆系统实现「按作用域分层、按时机写入、按相关性召回、按预算约束展示」，让用户感知到系统越来越懂上下文，而非"强行回忆"。

**工具系统**

能力供给分三层：**能力定义 → 能力装配 → 能力执行**。先定义统一的能力契约，再按场景动态装配可用能力，以运行时上下文形式交给 agent loop。内建工具、MCP 外部能力、skills/plugins 扩展能力在运行时都表现为**统一能力**，体验上能力多但使用方式一致。

![工具能力供给链：定义 → 装配 → 执行](https://mmbiz.qpic.cn/mmbiz_png/3p6PfyrqhYF5YoNMz64XUavicDVzOkDLB2L1FiaUG9XYPqmzRahu66XCt62D6bfzT8MQ6dyl3KSGubt8mF1GaBCpfUkicwRfTofMMFt2HxoHq0/640)

**隐藏功能与演进方向**

代码中未完成的部分显示，Anthropic 内部在探索将 Coding Agent 外溢到各行各业。演进方向是打造一个：**兼具主动协作、长时记忆、远端运行与会话资产管理能力的 Agent 工作平台**。

Claude Code 重视恢复能力、权限模型、上下文压缩和长会话稳定性，持续补齐远端协作、记忆沉淀、会话资产化等更强自主性的 Agent 能力。

![隐藏功能与 Coding Agent 外溢方向](https://mmbiz.qpic.cn/mmbiz_png/3p6PfyrqhYEdlIFdYxDo1dMYz6icNdMfFIr3txz5el2ONSxtXoicliaM9Fba1WlCDpPw51wBWAh4kNNiaVS5s5dST9B5JusjUvsF3NwEF1zw2xw/640)

**作者判断**：

技术栈和架构形态也许可以相似，但**真正拉开差距的，往往是细节是否被打磨到了足够扎实**。Claude Code 这套 Harness 范式提供的是一个「**下限可控、上限可拓**」的基础框架，如何做到卓越，很多 track 依然要从代码细节中挖掘，再结合业务打造差异化。

