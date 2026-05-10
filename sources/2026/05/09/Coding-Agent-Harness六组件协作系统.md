---
ingested: 2026-05-09
wiki_pages: [ai-agent/agent-harness-overview]
---

## Coding Agent Harness：模型差距在缩小，Harness 差距在放大

**文章信息**：
- **原文标题**：为什么同样是强模型，进了 Codex、Claude Code 之后会显得更像一个能协作的同事（原标题较长，此为提炼）
- **来源**：https://mp.weixin.qq.com/s/L0fGwMIh4HOXrg-Xi_BcWA
- **作者**：架构师（公众号 JiaGouX，主理人若飞）
- **发表时间**：约 2026 年 5 月初

---

**一段话总结**：

文章基于 Sebastian Raschka 的《Components of a Coding Agent》，系统拆解 Coding Agent 六层架构：Live Repo Context（避免模型盲启动）、Prompt Shape & Cache（稳定前缀与动态内容分离）、Structured Tools（工具可验证可约束）、Context Management（压缩去重防止上下文膨胀）、Session Memory（完整记录 vs 工作记忆分层）、Delegation & Subagents（有边界的分工委托）。核心论点是 LLM/Reasoning Model/Agent/Harness 四层需要先拆开放对位置——很多讨论之所以混乱就是把这几层混为一谈，真正的体感差距更多来自 Harness 系统层而非模型层。

---

**核心内容**：

**一、四层概念拆开**

| 层级 | 定义 | 类比 |
|------|------|------|
| LLM | 预测下一个 token 的引擎 | 会写代码的人 |
| Reasoning Model | 花更多 test-time compute 做推理和校验的 LLM | 更愿意思考的人 |
| Agent Loop | 观察→检查→选择→执行的循环 | 处理任务的工作节奏 |
| Harness | 外围工程系统：上下文/工具/状态/权限/执行/恢复 | 工位、规范、工具链、权限和验收机制 |

核心认知：你在聊天框看到的是模型裸输出，在 Codex/Claude Code 里看到的是模型+Harness。同款模型进了不同产品，体感会像换了一套工作方式。

**二、六个核心组件**

**1. Live Repo Context：先拿稳定事实再开工**
- 模型不能"盲启动"，必须先知道：Git 仓库状态、当前分支、目录结构、README、AGENTS.md 规则、标准入口脚本
- 成熟 Agent 会先收集稳定事实，整理成工作区摘要，每轮任务都带着出发
- 像新同事入组：先看文档、摸清目录、知道团队规则，不是一坐下就改文件

**2. Prompt Shape & Cache：动静分离**
- 稳定前缀（系统指令/工具定义/仓库摘要）变化少，更容易命中 Prompt Cache
- 动态部分（最新请求/近期对话/工具结果/短期记忆）往后放
- 长期规则和当前 bug 揉在一起会两头误事

**3. Structured Tools：边界比数量重要**
- Harness 负责校验参数、检查路径、判断是否需要批准、执行、结果回传
- 最危险的不是工具少，而是边界不清——一个能在错误方向上高速乱跑的 Agent 比慢但受控的更危险
- 工具设计核心：白名单、描述清晰、输入可验证、错误可明确、路径有边界

**4. Context Management：长任务难点在"别吃撑"**
- 三种压缩：Clipping（截断超长输出）、Summarization（压成近期摘要）、Deduplication（重复文件不重复喂）
- 原则：近的事保留更多细节，远的事压得更狠
- 金句："很多表面上的模型质量，其实是上下文质量"

**5. Session Memory：完整记录 ≠ 工作记忆**
- Full Transcript：完整记录，可恢复、可审计
- Working Memory：人工维护的精炼备忘录（当前任务、关键文件、最近动作、待办事项）
- 很多长任务不是模型变笨，而是系统没把"完整历史"和"当前重点"分开

**6. Delegation & Subagents：关键在怎么绑住**
- 适合拆的支线任务：符号在哪定义、测试为什么挂、配置写了什么
- 危险点：上下文不够什么都做不了；边界太松会把系统搞乱
- Claude Code 用任务范围、上下文大小和深度控制边界，不单靠"只读"

**三、自查清单**
按实用排查顺序：模型是否盲启动 → 长期规则和当前请求是否混一起 → 工具是否太多太散 → 日志是否在吞掉注意力 → 工作记忆和完整记录是否混一坨 → 子智能体是否比主智能体还难控。先补稳单 Agent 基础设施，通常比多 Agent 更划算。

**参考**：Sebastian Raschka 原文及 Mini Coding Agent 实现，code.claudecn.com 源码分析地图
