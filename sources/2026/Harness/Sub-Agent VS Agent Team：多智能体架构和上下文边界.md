---
ingested: 2026-05-09
wiki_pages: [ai-agent/agent-harness-overview]
---

## Sub-Agent VS Agent Team：多智能体架构和上下文边界

**文章信息**：
- **原文标题**：Sub-Agent VS Agent Team：多智能体架构和上下文边界
- **来源**：https://mp.weixin.qq.com/s/LNkT_xRhdh2iCxBQcVKpUQ
- **作者**：架构师（JiaGouX）
- **发表时间**：未知

---

**一段话总结**：

多智能体架构最先该判断的不是"要拆几个"，而是子任务之间是否共享同一段上下文。能干净切开的用 Sub-Agent，必须共享状态的才上 Agent Team。大多数多 Agent 系统搭歪了，是因为按"岗位"（Planner→Developer→Tester）拆而非按"上下文边界"拆，导致每次交接信息都在变薄。Sub-Agent 的价值是隔离（不污染父上下文）、压缩（只返回结论不返回过程）、并行（互不通信可并发）。Agent Team 适合持续协作和共享状态，但成本远高于 Sub-Agent。生产级多 Agent 系统真正用到的就五种原语：Prompt Chaining、Routing、Parallelization、Orchestrator-Worker、Evaluator-Optimizer。核心原则：围绕上下文边界设计，而不是围绕角色设计；从简单结构开始，只在确定需要时再加复杂度。

---

**作者判断**：

作者明确反对按角色拆 Agent 的默认套路，认为这是多 Agent 系统搭歪的根源。核心立场："围绕上下文边界设计，而不是围绕角色设计；从简单结构开始，只在确定需要时再加复杂度。"作者认为大多数多 Agent 系统是被"岗位思维"搭歪的，单个 Agent 能干完的事就别拆，多 Agent 不是产品形态而是一组可组合的工作流原语。作者也承认 Agent Team 的成本远高于 Sub-Agent，需要共享状态层、通信协议、Lead Agent 仲裁，调试链路也更长。

---

**核心内容**：

**核心判断框架**：

多智能体架构最先该判断的不是"要拆几个"，而是这些子任务之间**是否共享同一段上下文**。能干净切开的用 Sub-Agent，必须共享状态的才上 Agent Team。

判断标准：
- 这两件事，需不需要看到对方的中间过程？
- 这两件事，会不会因为对方做完了某一步就影响自己下一步？
- 如果交给同一个 Agent 一次性做完，会不会更省心？

如果答案都偏向"是"，那它们本来就该在一个 Agent 里，强行拆只是把成本转嫁给沟通层。

![Sub-Agent vs Agent Team 架构对比图](https://mmbiz.qpic.cn/mmbiz_png/Fnx2G2wYdELguf3N7Dgonhp6fK0PojLjYEp4easoibFRJiaiaJyyJHibqicIhqtVB3xdjFe2yp5mF6TYFicIw3rUt9ucw4iadB4Vs3jIXLfMIYEj7o/640)

**大多数多 Agent 系统搭歪的原因**：

按"岗位"拆 Agent（Planner→Developer→Tester→Reviewer）是最常见的错误。问题在于：Planner 知道的上下文没传到 Developer，Developer 的临时取舍没沉淀到 Tester，Tester 拿到的是干瘪的描述。**每一次交接，信息都在变薄。** LLM 没有"茶水间记忆"，它能拿到什么上下文就只能基于什么上下文做事。

原文核心观点：Design around context boundaries, not roles.

![上下文边界设计原则](https://mmbiz.qpic.cn/mmbiz_png/Fnx2G2wYdEJLWZAfFXLkLgvdxxQ8hRDaKMbuZuYYW1EqTGsV46G9kQhHNbfYtLAbB03Oxst9O1bMliagibsM5c20tfYxZY9L3qxN4Ydq0ibDic8/640)

**Sub-Agent：解决隔离 + 压缩 + 并行**：

Sub-Agent 的本质是父 Agent 把一段定义清楚的工作扔出去，子 Agent 在独立上下文里跑完，把**结论**（不是推理过程）回收回来。

硬约束：
- 子 Agent 之间**不能直接通信**
- 子 Agent **不能再生新 Agent**
- 所有流量必须经过父 Agent
- 跑完只返回最终输出，不带中间思考

三个核心价值：
1. **隔离**：子任务在自己的上下文里跑，不会污染父上下文窗口
2. **压缩**：返回结论而非过程，把乱糟糟的探索压成干净的信号
3. **并行**：子 Agent 之间互不通信，可放心并发跑

![Sub-Agent 架构示意图](https://mmbiz.qpic.cn/mmbiz_png/Fnx2G2wYdELVPkHeVX9FicfkIS7fuvKzZciaSDdI4UJyRBuE0icbgHueteTnrv3ysicnIrBmhREGqdxtZs89DygeHwFo00wPHZ6gQQ6GTGf2Wwo/640)

**Agent Team：解决持续协作**：

Agent Team 像一个长期在一起干活的小组：有 Lead、有队员、有共享的任务板。

关键差异：
- 上下文是**共享**的，不是各管各的
- Agent 之间可以**直接对话**，不用都经过父级中转
- 任务有持续的状态层，进度、依赖、阻塞点都在上面挂着
- 一个 Agent 改了什么，会影响另一个 Agent 的下一步动作

适用场景：做着做着会发现问题，然后需要互相调头的任务（如软件项目：前端改了接口契约，后端要立刻知道）。

成本远高于 Sub-Agent：需要共享状态层（能处理冲突、可见性、版本化）、节点间通信协议、Lead Agent 仲裁分歧。

![Agent Team 架构示意图](https://mmbiz.qpic.cn/sz_mmbiz_png/Fnx2G2wYdEKldbJMoOXiblt3rhh9icPGiaIia13IPpIX6Y87VS33ia1HVPNLyhxnR4IvmtqQRtU5JVBaicghYalW5Db62J8QvicaT2b8zrA5ylVspY/640)

**五种生产级编排原语**：

1. **Prompt Chaining（顺序串）**：A 做完给 B，B 做完给 C。简单线性任务最常用。
2. **Routing（路由）**：根据任务特征派给最合适的 Agent。客服系统"先识别意图再分流"本质上是这个。
3. **Parallelization（并行）**：互不依赖的任务一起跑，最后汇总。代码审查、文档多维度分析。
4. **Orchestrator-Worker（调度-执行）**：一个 orchestrator 拆任务、派任务、收结果，workers 各自闷头干。Sub-Agent 的标准形态。
5. **Evaluator-Optimizer（评估-优化）**：先生成、再评估、再迭代。高质量产出场景。

**多 Agent 不是一个产品形态，它是一组可组合的工作流原语。**

![五种编排原语](https://mmbiz.qpic.cn/sz_mmbiz_png/Fnx2G2wYdELqdnwUpk35DEJKy330ak7AZtZLxm0vVJKVCeykibelJ2NPGNDmW2N0ibleLja1AaKOosvFpU0wic2tKd94t8viccYxib1gpDBHkOUQ/640)

**选型判断表**：

- 这个任务能不能一个 Agent 干完？→ 能就先这样，别提前优化
- 子任务之间需不需要看到彼此的中间过程？→ 不需要 Sub-Agent；需要 Agent Team
- 子任务跑的时候要不要互相影响？→ 不要并行 Sub-Agent；要 Team
- 是不是只是想"看起来更高级"？→ 退回单 Agent，先把任务模型搞清楚
- 每一步要不要严格按业务规则走？→ 加确定性中间层，不要硬塞给 team

![选型判断框架](https://mmbiz.qpic.cn/mmbiz_png/Fnx2G2wYdEItlSFtCpicML9U28JhRySxIeh3ufBTQbTzrWU6U8DvjrrnPYFVicreyMdA9lok8vRrXz6e5WgCAPSz6SK8ibGe0pbo8YYZDbJWsI/640)

**什么时候根本不需要多 Agent**：

单个 Agent 能干完且体感不差就别折腾。多 Agent 带来的隐藏成本：
- 编排逻辑要写、要维护、要监控
- Agent 之间的契约要定义、要版本化
- 调试链路变长，问题定位成本上升
- 上下文在多个 Agent 间一致流转的挑战
- 治理成本（审计、回滚、计费）翻倍

**当任务高度依赖、协调成本远大于收益，或者上下文压根没法切干净的时候，单 Agent 反而是最稳的选择。**

---

**相关推文**：
- Agent Harness 综述
- MCP 退潮后，CLI 又成了王？
- 再看 Hermes Skills：Agent 如何自我进化？
- Agent 的下一步不是更长记忆，而是会维护过程资产
- 如何为 Agent 设计产品？
