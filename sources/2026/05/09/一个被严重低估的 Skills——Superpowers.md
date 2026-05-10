---
ingested: 2026-05-09
wiki_pages: [ai-agent/superpowers-framework]
---

## 一个被严重低估的 Skills——Superpowers总结

**文章信息**
- **原文标题**：一个被严重低估的 Skills——Superpowers
- **来源**：<https://www.toutiao.com/article/7624175963751957001>
- **作者**：硅基观察室
- **发表时间**：未知

**一段话总结**：Superpowers 是一套给 AI 编程 Agent（Claude Code、Codex、Cursor等）用的技能框架和软件开发方法论，核心目标是让AI从"冲动型码农"变成有纪律、有方法的高级工程师。它不追求让AI"能做某件事"，而是把优秀软件工程习惯——需求澄清、设计先行、任务拆分、TDD、子代理驱动开发、Code Review——变成AI的默认行为。通过 bite-sized tasks 拆分、subagent-driven-development 流程、RED-GREEN-REFACTOR 的测试驱动开发等机制，将AI编程从"随便写写"推进到"系统化工程开发"。

---

**核心事件**：Superpowers 项目将软件工程纪律（需求澄清→设计→拆任务→TDD实现→Review→收尾）系统性地注入AI编程Agent的工作方式，解决的核心问题是"AI写代码的过程太随意"。

**AI编程工具的痛点**

用过Claude Code、Codex、Cursor等工具的人都有同一种感受：AI写代码确实快，但也确实太急了。用户说"帮我搞个功能"，AI下一秒就直接开写——不问需求、不聊约束、不做设计、不写测试。写完之后用户得自己接手Review、修bug、补测试。说白了，像一个手很快但不太稳的初级工程师。Superpowers要解决的恰恰就是这个问题。

**它不只是另一个技能合集**

很多人第一次看Superpowers容易误会它是另一个"技能合集"。但真正值得关注的点不在它有多少skill，而在于它补了一层更关键的东西：让AI更像一个真正的工程同事，知道什么时候该先问清楚、什么时候该先出设计、什么时候该拆任务、什么时候该写测试、什么时候该Review、什么时候不能急着宣布"搞定了"。

**核心工作流**

**第一步：先问清楚再动手** — 正常AI编程工具你一提需求立刻生成代码，但Superpowers的第一反应是先退一步问：你到底要实现什么？功能边界在哪？有没有约束条件？有没有更简单的做法？有没有必要先出设计？然后把设计文档分段给你看，控制在"你真的会看"的范围内。

**第二步：拆成bite-sized tasks** — 设计确认后不一口气往下冲，而是把工作拆成很小的执行单元，通常控制在几分钟就能完成的粒度。每个任务具体到要改哪些文件、需要什么输入、预期什么输出、前后依赖是什么。

**第三步：Subagent-driven Development** — 每个任务交给新的子Agent去做，做完之后再检查、再Review、再决定是否继续下一个。不让AI一口气"自由发挥几个小时"，而是让它在有边界、有检查点的流水线上工作。

**第四步：测试驱动开发（TDD）** — 这是Superpowers最核心也最"工程师"的一步。标准RED-GREEN-REFACTOR流程：先写一个失败测试 → 再写刚好让测试通过的代码 → 再做重构优化。要解决的正是AI最常见的坏习惯：先写一堆代码 → 再假装补测试 → 最后说"应该没问题"。

**完整技能体系**

覆盖从需求→设计→拆任务→实现→测试→Review→收尾的完整链路：
- test-driven-development
- verification-before-completion
- systematic-debugging
- brainstorming
- writing-plans / executing-plans
- dispatching-parallel-agents
- subagent-driven-development
- requesting-code-review / receiving-code-review
- using-git-worktrees
- finishing-a-development-branch
- writing-skills / using-superpowers

**它到底在训练什么**

Superpowers在训练的不是"能力"，而是"纪律"：
- 需求没清楚，不准直接写
- 没做计划，不准直接冲
- 没写测试，不算真正开始
- 没验证通过，不准宣布完成
- 没Review，不算真正交付

**适合谁**

- **重度AI编程用户**：已经在用Claude Code等工具，但总要后面补很多工作、经常得到"八成对"的答案
- **重视流程胜过工具的开发者**：不缺生成能力，缺的是让AI按靠谱流程做事
- **研究AI如何进入系统化工程开发的人**

**边界与局限**

- 只改一行字、修个超小bug时，完整流程可能"有点重"
- 不同平台体验深度可能有差异
- 流程再好也要看底层模型能不能理解并遵守，换模型效果可能不同
- 它不是魔法，更像一个把成功率抬高的工作流框架

**安装方式**

Claude Code：`claude mcp add -s user superpowers -- npx -y @anthropic/superpowers`
Codex / OpenCode：按仓库INSTALL.md说明接入
官方仓库：<https://github.com/obra/superpowers>

**作者判断**：作者认为Superpowers是最容易被低估但最重要的AI编程项目之一。很多AI coding项目做的是更强的自动补全、更快的代码生成、更多工具调用，但Superpowers在做另一件事——把优秀的软件工程习惯变成AI的默认行为。真正拉开差距的不只是模型有多强，而是它有没有一套稳定、可复用、可验证的工程工作流。给AI多一点纪律，往往比给AI多一点自由更有用。

[原文配图：文章配图3张，内容未知]

