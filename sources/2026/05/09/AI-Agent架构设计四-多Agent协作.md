---
ingested: 2026-05-09
wiki_pages: [ai-agent/sub-agent-vs-team]
---

## AI Agent 架构设计（四）：多 Agent 协作

**文章信息**：
- **原文标题**：AI Agent 架构设计（四）：多 Agent 协作（OpenClaw、Claude Code、Hermes Agent 对比）
- **来源**：https://www.toutiao.com/article/7628998729999172139
- **作者**：人工智能科普站
- **发表时间**：2026-04-15 22:52

---

**一段话总结**：

文章从四个架构问题（角色分离、上下文隔离、通信协调、结果汇总）切入，对比 OpenClaw、Claude Code、Hermes Agent 三个框架的多 Agent 设计哲学。OpenClaw 分 SubAgent（非阻塞派发，文件协调）和 Routed Agent（Gateway 层隔离）两层；Claude Code 分 Subagents（会话内派发）和 Agent Teams（P2P 直接通信，Git Worktree 解决写冲突）；Hermes 通过完全隔离的 Subagent + 共享 Skills 知识层（PLUR 插件实现经验双向传播）实现协作。最后给出三个工程取舍：AI 自主分工 vs 人定规则、实时消息 vs 文件传话、子 Agent 继承上下文 vs 完全隔离。

---

**核心内容**：

**为什么需要多 Agent？**
- 单个 Agent 上下文窗口有限，复杂项目很快撑满，"Lost in the Middle"加剧
- 单 Agent 必须串行独立子任务，多 Agent 可并行

**四个架构问题**：角色怎么分离、上下文怎么隔离、怎么通信、结果怎么汇总

**三个框架对比**：

| 维度 | OpenClaw | Claude Code | Hermes Agent |
|------|---------|-------------|-------------|
| SubAgent 通信 | 只向主 Agent 汇报 | 只向主 Agent 汇报 | 完全隔离，文件和 Skills 协调 |
| 高级模式 | Routed Agent（Gateway 层） | Agent Teams（P2P） | Skills 共享知识层 |
| 上下文隔离 | 各自 workspace + MEMORY.md | Git Worktree 独立分支 | 独立对话线程 + 独立终端 |
| 协调层 | 文件系统 | 文件系统 | 文件系统 + Skills |
| 独特之处 | 非阻塞派发 | P2P 直接通信 | PLUR 经验双向传播 |

**Claude Code 选择标准**：Subagents 用于快速聚焦只需汇报的任务；Agent Teams 用于需跨层协调、相互验证的并行任务；单 Session 用于顺序任务、同一文件、强依赖任务。

**三个核心取舍**：
1. AI 自主分工 vs 人定规则：任务越固定越该人定，越开放越该 AI 判断
2. 实时消息 vs 文件传话：需来回确认用实时消息，只传递结果用文件
3. 继承上下文 vs 完全隔离：需要执行时多给背景，需要审查时完全隔离更易发现问题

**Hermes 的独特设计**：Agent 完成复杂任务后自动写"工作笔记"（Skill），存到共享目录 `~/.hermes/skills/`，所有 Agent 启动时加载。PLUR 插件让纠正自动同步给同项目所有 Agent——不用实时通话，靠积累共同经验协作。
