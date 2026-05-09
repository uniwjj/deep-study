---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-harness]
---

## 7张图搞懂Claude Code的Harness架构设计

**文章信息**：
- **原文标题**：7张图搞懂Claude Code的Harness架构设计
- **来源**：https://mp.weixin.qq.com/s/MsWfAil7l8388WmYrdvXyw
- **作者**：诗与沅方
- **发表时间**：2026-04-03 21:41

---

**一段话总结**：

Claude Code 源码泄露后，作者与 AI 协作解析了 57MB 源码，提炼出企业级通用 Agent 架构设计要领。其 Harness 是一套「入口适配 + 会话成型 + 运行桥接」的编排系统，将 CLI、SDK、远端等多入口收敛为统一的 agent turn 执行模型。架构分三层：入口分流层、Harness 会话编排层、Runtime 与支撑层。Agent Loop 负责核心运行链路，状态管理采用会话运行态 + 全局会话态 + 持久化层分层架构。记忆系统按项目/团队/Agent 专属/动态四层管理，工具系统将内置、MCP 外部、skills 扩展能力统一为一致调用方式。整体强调「下限可控、上限可拓」的设计哲学。

---

**作者判断**：

作者认为 Claude Code 不仅是 Coding 产品，更是一个通用终端 Agent，其架构比 OpenClaw 更成熟，值得每个 Agent 团队研究。作者明确指出 Anthropic 内部在做多方面探索，将 Coding Agent 外溢到各行各业。Claude Code 的演进方向是打造兼具主动协作、长时记忆、远端运行与会话资产管理能力的 Agent 工作平台。作者总结："技术栈和架构形态也许可以相似，但真正拉开差距的，往往是细节是否被打磨到了足够扎实。"

---

**核心内容**：

**整体概述**：Claude Code 是一个通用终端 Agent，能循环思考、调度工具、治理权限、恢复上下文、稳定长会话。57MB 泄露源码涵盖 Harness 架构、Agent Loop、状态管理、记忆系统、工具系统等模块。

项目工程架构全景图如下：

![Claude Code 项目工程架构全景图](https://mmbiz.qpic.cn/sz_mmbiz_png/3p6PfyrqhYFcX7Yl9ZamZHxElxYCYbA29TuBiaNLGAZaPJJgvrjaI0lIchpYZOYS1d3ia3ib17Q80DliaiambYy250GOKUiarYn0ZHgsicsO0QuibnQ/640)

**Harness 架构设计**：Harness 是一套「入口适配 + 会话成型 + 运行桥接」编排，本质是把多入口、多模式、多运行位置收敛成统一的 agent turn 执行模型。

三层架构：
- **第一层：入口与分流**：接住用户不同使用方式（CLI、交互界面、SDK 调用、assistant 模式、远端链接），经 main.tsx 做参数解析、模式判断和路由分发
- **第二层：Harness 会话编排**：核心层，将三种形态（交互会话、无界面会话、远端接入）整理成统一 turn 契约，同时接入口工具能力、扩展接入、状态与持久化
- **第三层：Runtime 与支撑层**：本地路径进入本地 Runtime 执行完整 Agent Loop；远端路径转入 remote / bridge / server 机制承载

Harness 三层架构图解：

![Harness 三层架构图](https://mmbiz.qpic.cn/mmbiz_png/3p6PfyrqhYFJdOHqC4QkRZUYot6ZAVl3ceEA7QSicyMZr8grnL4MxL7WNH4fW32aLvUTq9L1x1nbicHJ1YDbNGlicQFWxRNThWbvY4ibGTvgcnQ/640)

**Agent Loop 设计**：Harness 解决请求从哪来、交给谁；Agent Loop 解答链路如何运行。核心设计理念是「将智能下沉给模型，释放智能自主权」。

Agent Loop 运行流程图：

![Agent Loop 运行流程](https://mmbiz.qpic.cn/sz_mmbiz_png/3p6PfyrqhYFjI0r6Q3jVBylNI0GNeiaNGkOyz15LDNS8WYWkWt6ibGxppFQ7ECOKzNzgnSYZ42uZYIviaXSkp1M3iaIUlVuk9DEiccftTFfXt80w/640)

**状态管理**：采用「会话运行态 + 全局会话态 + 持久化层」分层协同管理架构，确保长会话稳定性与上下文恢复能力。

状态管理分层架构：

![状态管理分层架构](https://mmbiz.qpic.cn/mmbiz_png/3p6PfyrqhYHOugw0YXd1M189VPvSMHQgyMfM7lNgGxUScaR5AYOY6WUIGq2FqgEzHaGVSKzLBZ3nQIy7tXWQBGCT8QVzLrdzMGcSBEZDw18/640)

**记忆系统**：按四层管理——当前会话沉淀、项目与团队范围记忆、Agent 专属记忆、每轮按相关性动态注入的动态记忆。

记忆系统架构图：

![记忆系统架构图](https://mmbiz.qpic.cn/sz_mmbiz_png/3p6PfyrqhYEliazRzKm70klwr3XNczIoFadhbpeLicPZkkIxm1b3txIPibTWibP2tJIJOH5cfL3Wjs2DbiaGF18ndBBzxm6DQ8SIaZtOPjpDVSx8/640)

记忆层在 runtime 中的筛选、注入和回写流程：

![记忆层注入流程图](https://mmbiz.qpic.cn/sz_mmbiz_png/3p6PfyrqhYFtvn900xHpe4m1DoYJ4rfhuR7PB91AD8TnFu9ZXOyyqnOhyLjJ0HYICTLv9rUP4kso8ibeQIKGEqo5DdLX863MOmQJ9ZpKb7Xk/640)

**工具系统**：内置工具、MCP 外部能力、skills / plugins 扩展能力在运行时统一表现为一致能力接口，使用方式一致。

**隐藏功能**：代码中大量未完成的代码表明 Anthropic 内部组织分化在做多方面探索，将 Coding Agent 外溢到各行各业。演进方向是打造兼具主动协作、长时记忆、远端运行与会话资产管理能力的 Agent 工作平台。

