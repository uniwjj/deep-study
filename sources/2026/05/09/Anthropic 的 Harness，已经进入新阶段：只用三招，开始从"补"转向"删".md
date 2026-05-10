---
ingested: 2026-05-09
wiki_pages: [ai-agent/harness-build-to-delete]
---
## Anthropic 的 Harness 三招

**文章信息**：
- **原文标题**：Anthropic 的 Harness，已经进入新阶段：只用三招，开始从"补"转向"删"
- **来源**：https://mp.weixin.qq.com/s/k8eoFVMRcCPXEQMSy9Dyqg
- **作者**：若飞（架构师公众号）
- **发表时间**：未知

---

**一段话总结**：

作者将 Anthropic 前后两篇 Harness 文章对着读，发现一条清晰变化：前一篇（长任务文章）讲"当模型自己还扛不住时，系统该补什么"——planner、evaluator、context reset、sprint contract 都是在替模型兜底；后一篇（Harnessing Claude's Intelligence）讲"当模型跨过某些门槛后，系统终于可以少做什么"。核心问题是 **What can I stop doing?** 新文落到工程里是三招：先用 Claude 已经熟悉的通用工具、把三类控制权（编排权、上下文管理权、记忆选择权）还给模型、把真正承重的边界（缓存/安全/可观测性）留在 Harness。作者提出一个关键工程节奏：每次模型升级后做一次 dead weight inventory，能删的就删。不是 Harness 越多越好，而是"Build to delete"——先搭出来，持续验证，最后敢于拆掉。

---

**作者判断**：

作者认为 Harness 正在进入第二阶段：第一阶段是拼命补（补能力缺口），第二阶段是持续删（删历史包袱）。原文原话："真正成熟的 Harness，不只是会补短板，还得会在短板消失之后，主动删掉曾经的补丁。" 作者最在意的不是三个新模式，而是背后的工程节奏。最终判断收束为："Build to delete。先搭出来。再持续验证。最后敢于拆掉。这可能才是 Agent 时代更像工程，而不是更像魔法的地方。"

---

**核心内容**：

**前后两篇文章的核心变化**：

前一篇《Harness design for long-running application development》（2026-03-24）主要解决：长任务为什么会越跑越偏、模型为什么总对自己的结果过于宽容。关键词是 planner/generator/evaluator、context reset、sprint contract、structured handoff。本质是"当模型自己还扛不住时，系统该补什么"。

后一篇《Harnessing Claude's Intelligence》（2026-04-02）不再讨论怎么把框架搭得更厚，而是开始问：编排是不是还必须由 Harness 接管？上下文是不是还需要大段预加载？记忆是不是一定要先做成模型外的检索基础设施？本质是"当模型已经跨过某些门槛时，系统终于可以少做什么"。

**关键判断**：很多 Harness 组件之所以存在，不是因为它们永远正确，而是因为它们曾经刚好填上了某一代模型的能力缺口。一旦缺口消失，组件就该重新评估。工程节奏：先承认模型有边界 → 用 Harness 把边界外移成系统能力 → 每次模型升级后重新检查哪些控制逻辑还在承重 → 能删就删。

![Anthropic Harness 第二阶段核心框架：三招从"补"转向"删"](https://mmbiz.qpic.cn/mmbiz_png/Fnx2G2wYdEKwTx7DKjKD5oOLwMBjvLeRq14pWux2PXWaVyWfHm35bufyUGRwSgokfMte7lvZSEOCAa6pXkdKadnwSuNUueWHtgEyiaVqLLmA/640)

**模式一：先用 Claude 已经会的东西**

回顾：2024 年底 Claude 3.5 Sonnet 只靠 bash + text editor 就在 SWE-bench Verified 拿到 49%（当时 SOTA），Opus 4.5 同样两个工具做到 80.9%。Claude Code 也是基于它们构建的。很多看似独立的能力（programmatic tool calling、Skills、memory）本质上都是从 bash + text editor 上逐步长出来的组合模式。

核心判断：如果一个问题可以先用 Claude 已经熟悉的通用工具解决，就别急着把它过早固化成框架逻辑。过早专用化是在替模型做决策，而模型能力一旦继续上涨，提前写死的分工、过滤和流程迟早会过时。专用工具真正值得的场景：动作高风险、难回滚、必须可追踪（安全门禁、结构化参数、审计回放）。

**模式二：把三类控制权还给模型**

![三类控制权归还 vs 三类边界保留](https://mmbiz.qpic.cn/sz_mmbiz_png/Fnx2G2wYdEKg79ByVokfMTn41aRwusKyJnzKiaDnZJbxJicVAPRibTrFaX7M68na1VPfTic2LfG0bM4TAibKibZsEsKtgBOjvUiafl3GAQxdMNIuoU/640)

1. **编排权**：传统做法是每次工具调用结果都回模型上下文再决定下一步——又慢又贵且很多时候没必要。Anthropic 的解法：给 Claude bash/REPL 代码执行能力，让它自己写代码串联工具调用，中间结果在代码里处理，只有真正需要的输出才回上下文。效果：Opus 4.6 在 BrowseComp 上准确率从 45.3% 提升到 61.6%。核心判断：很多所谓"工具编排"不是 Harness 必须接管的系统职责，而是 Claude 自己就更适合做的局部决策。

2. **上下文管理权**：常见做法是把大量任务指令提前写进 system prompt——任务越多预加载越重，注意力预算越紧，很多说明当前任务根本用不到。Anthropic 的答案：Skills + progressive disclosure（先给目录，需要时再读全文）。同时结合 context editing（删掉过时上下文）和 subagents（开更干净的上下文窗口隔离子任务）。Opus 4.6 使用 subagents 后在 BrowseComp 上比最佳单 Agent 结果再提升 2.8%。

3. **记忆选择权**：传统做法把"记忆"理解成模型外的检索系统。新思路：先给模型足够简单的持久化手段，让它自己学会选什么该记。两条路径：compaction（总结过去上下文保持长任务连续性）和 memory folder（给模型可写可读可整理的地方）。关键数字：compaction 在不同模型上表现差异巨大——Sonnet 4.5 卡在 43%，Opus 4.5 到 68%，Opus 4.6 到 84%。说明很多时候不是记忆机制本身不工作，而是模型终于更会决定"什么值得留下来了"。宝可梦例子：Sonnet 3.5 写流水账什么都记，Opus 4.6 开始提炼战术笔记和目录结构。

**模式三：能放权不等于没边界**

![从"补短板"到"删负担"的完整工程节奏](https://mmbiz.qpic.cn/sz_mmbiz_png/Fnx2G2wYdEJQAmAG2FXh4RCAOKcL6aNIfZFYk55y2EM1W2guXRD2kZ1W4LibER3pB8zibfkR6Z1x3DVHtjAsxYek8ibPkc1iabqz8lIicBBJUIIs/640)

必须留在 Harness 的三类边界：

- **成本边界**：Messages API 无状态，缓存命中率直接影响账单与延迟。原则：静态内容放前面、动态放后面；更新尽量追加消息不改 prompt 本体；同会话别频繁切模型；多轮应用持续更新 breakpoint。"缓存 token 成本只有基础输入的 10%"——缓存命中率不是小优化，是 Harness 该主动负责的预算纪律。

- **安全边界**：reversibility（可逆性）标准——越难回滚的动作越值得做成独立工具。bash 很强但 Harness 只看到命令字符串，删文件和调外部 API 看到的形状没区别但风险完全不同。Claude Code 的 auto-mode 用"第二个 Claude"审查 bash 命令是否安全——"用模型审查模型"适用于用户已有足够信任的场景。

- **可观测性边界**：类型化工具让 Harness 能记录参数、追踪链路、做回放。demo 阶段常被低估，但进入真实生产环境后越来越重要。

**Harness 体检 5 问**：
1. 现在有哪些流程只是为补偿旧模型短板而存在？
2. 哪些工具过滤、任务编排、上下文拼装已在替模型做它本可以自己做的决定？
3. 哪些专用工具真正承重，哪些只是历史上"先做出来了"但今天收益已不明显？
4. 哪些动作涉及安全、确认、审计、回滚，必须留在 Harness 控制面？
5. 每次模型升级后，有没有系统地做过一次 dead weight inventory？

**核心结论**：Harness 进入第二阶段。第一阶段拼命补（补能力缺口），第二阶段持续删（删历史包袱）。一个从来不删的 Harness 最后会变成"对模型太重、对系统太复杂、对成本太贵、对团队太难维护"。

