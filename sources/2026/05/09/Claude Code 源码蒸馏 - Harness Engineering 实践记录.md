---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-harness]
---

## Claude Code 源码蒸馏 - Harness Engineering 实践记录

**文章信息**：
- **原文标题**：Claude Code 源码蒸馏 - Harness Engineering 实践记录
- **来源**：https://zhuanlan.zhihu.com/p/2023168552018921120
- **作者**：LastWhisper（北京大学 计算机应用技术硕士）
- **发表时间**：2026-04-02

---

**一段话总结**：

本文记录了作者在 Claude Code 源码泄露后，用多 Agent 协作方式将其 51.2 万行 TypeScript 源码蒸馏为 Agent Harness Patterns Skill 的完整过程。这不是一篇源码分析，而是一次 Harness Engineering 的实践记录——作者设计了一套"审查与执行分离"的协作架构：Codex（GPT 5.4 xhigh）负责源码事实核验和架构审查，Claude Code（Opus 4.6 max）负责源码探索和文档起草，两者通过文件系统协调（handoff 文档充当 Agent 间的 API），每轮 review-action 循环都在全新 session 中进行。作者将自己的 Context Engineering 博客作为"品味注入"（taste injection）喂给 Agent，用 select/write/compress/isolate 四轴框架和"Do the simple thing that works"偏好作为"基向量"，让 Agent 从高维代码空间中做 PCA 式投影，提取低秩的设计原则。整个过程经历 7 个 Phase（搭脚手架→探索→核验纠错→转向→8 Agent 并行重写→Review 收敛→最终优化），从 v0.1 的 6 个错误收敛到 v0.5 通过。作者的核心洞察：好的蒸馏需要明确的立场（基向量），客观的模式提取反而最没用，因为它没有视角就没有优先级；Harness 设计的核心理念是"先建协调机制，再开始干活"。

---

**作者判断**：

作者的核心观点有三个层次。第一，关于蒸馏方法："一个客观的模式提取可能反而是最没用的，因为它没有视角，也就没有优先级（无法归约 agent action space）。好的蒸馏需要一个明确的立场，然后诚实地标注这个立场是什么。" 第二，关于 Harness Engineering："先建协调机制，再开始干活。如果直接让 Agent 开始提取模式，没有 task-board 追踪进度、没有 review-checklist 定义质量标准，后面的多 Agent 协作就无从协调。" 第三，关于 Agent 使用："如果你给 Agent 一个近乎无限的 Action Space，Agent 会直接失败。你需要先 Laying the foundation by hand 亲手奠定基础。" 作者还提出了验证方法论：后续对 Codex CLI 和 Gemini CLI 用同一组基向量分析，如果一个模式在三套独立实现中都沿同一方向出现，它大概率反映问题域本身的结构而非某个团队的设计偏好。

---

**核心内容**：

### 核心框架：PCA 类比

作者将蒸馏过程类比为 PCA（主成分分析）：通过注入自己的博客和偏好构建一组"基向量"，让 Agent 从复杂的高维代码空间中做投影，提取出几个最主要的设计原则。代码是高维的，但有价值的设计模式是低秩的。

![PCA 类比：从高维代码空间到低秩模式空间的投影](https://pic1.zhimg.com/v2-b2e80edab82cd2057f6e9dd5d50991a1.jpg?source=7e7ef6e2&needBackground=1)

三个基向量：select/write/compress/isolate（Context Engineering 四轴框架）、"Do the simple thing that works"（简洁偏好）、portable principles（可移植原则判断标准）。

### 1. Harness Design: Split by Role, Coordinate via Filesystem

核心设计：将 review 与 execution 分离到两个不同的模型家族，每轮 review-action 循环都在全新 session 中进行。

| 角色 | Agent | 职责 |
|------|-------|------|
| 审查与架构 | Codex（GPT 5.4 xhigh） | 源码事实核验、定位决策、抽象层级判断、UX 审计 |
| 构建与执行 | Claude Code（Opus 4.6 max） | 源码探索、参考文档起草、并行子 Agent 编排、编辑执行 |

分开的原因：和不让实现者审查自己代码的逻辑一样。两个 Agent 互不可见对方 session，通过文件系统协调——通过文件名 + Read Tool，低成本且不丢信息。

![Harness 文件系统交互架构](https://pica.zhimg.com/v2-26291a493f1b66e68866326d11037336_r.jpg)

Harness 基础设施文件按职能分三组：

**角色定义（Role Briefs）**：clean-agent-brief.md（Builder Agent 入场说明）+ review-agent-brief.md（Reviewer Agent 入场说明）

**协调层（Coordination）**：context-map.md（50+ 源文件按 harness 层分组）、pattern-notes.md（探索笔记）、task-board.md（35 个任务含依赖和退出标准）、progress-log.md（活动日志+决策日志，最终 13 项关键决策）、handoff_v0_*.md（行动简报）、codex_review_v0_*.md（审查输出含严重度分级）

**质量门控（Quality Gates）**：review-checklist.md（阻塞性问题 vs 质量检查）、execution-strategy.md（多 Agent 执行策略）、output-format.md（产出物格式规范）

Handoff 文档是 Agent 之间的 API——恰好包含下一个 Agent 执行所需的信息（索引，告诉新 agent 需要的 context 大致范围）。接收方在全新 session 中启动，读取最新 handoff 后执行。

每次 review/execution，agent 有三类 context：agent role、agent task handoff（task specific）、repo filesystem（"give Codex a map, not a 1,000-page instruction manual"）。

### 2. Human-in-the-Loop: Taste Injection

**品味注入**：作者将之前写的 Context Engineering 博客作为 instruction 喂给 Claude Code。Agent 带着作者对 select/write/compress/isolate 四轴框架的理解、对简洁的偏好、对抽象层级的判断标准来提取模式。蒸馏结果不是源码的"客观映射"，而是经过偏好基向量投影后的结果。

"如果你给 Agent 一个近乎无限的 Action Space，Agent 会直接失败。你需要先 Laying the foundation by hand 亲手奠定基础。"

**架构决策——三个关键时刻**：
1. **定位选择**：Codex 第一轮 review 指出 skill 在回答错误的问题（"Claude Code 有哪些子系统？" vs "构建者正在解决什么问题？"）。重新定位的决策是 Codex 建议的，但选择接受并执行是作者做的判断
2. **抽象边界**："去掉 Claude Code 这个名字，这条原则是否仍然有价值？"——这个判断标准在多轮 review 中逐渐确定
3. **用户视角补位**：5 轮专注内容准确性后，作者意识到缺少用户视角审查，hardcode 了要求引入读者/使用者角度的规则

### 3. 过程概览（7 个 Phase）

![蒸馏过程的 7 个 Phase 时间线](https://pic2.zhimg.com/v2-e93238fde03f9c9a08778fa98bc447c7_r.jpg)

**Phase 0：搭脚手架** — 和 Codex 对齐项目范围和命名，生成全套 harness 文件。

**Phase 1-2：探索与并行起草** — 4 个并行 Explore Agent 扫描约 1,900 个文件产出上下文地图，7 个并行子 Agent 各领一个 harness 层写深度分析。

**Phase 3-4：核验→纠错→转向** — Codex 第一轮 review 发现 6 个事实错误（记忆系统未覆盖、并发分类搞错字段、权限来源数量写错等）。修正后 Codex 指出方向问题——应该提取设计原则而非解释代码。

**Phase 5：模板标准化—8 个 Agent 并行重写** — 确定"原则优先"后，8 个子 Agent 按统一模板重写：问题（通用）→ 黄金法则（可移植）→ 适用场景 → 权衡 → 实现模式（无代码）→ 踩坑指南 → Claude Code 实证（自然语言，不出现源码路径/函数名/代码片段）。

**Phase 6：Review 收敛** — v0.4（1P1+6P2+1P3）→ v0.5（3P3 通过）→ UX 审计（1P1+2P2+2P3，发现技能列表 250 字符硬限截断触发词）。严重度在下降说明收敛在发生，但 UX 审计引入新 P1——内容收敛后展示层面还有关键问题。

**Phase 7：最终优化** — 缩短技能描述（248→116 字符），添加"从这里开始"操作指引，声明目标读者。

### 4. Reflection

**关于"客观"提取**：客观的模式提取可能反而是最没用的，因为它没有视角就没有优先级（无法归约 agent action space）。好的蒸馏需要明确的立场，然后诚实地标注这个立场是什么。

**关于 Roadmap**：后续对 Codex CLI 和 Gemini CLI 用同一组基向量分析。如果一个模式在三套独立实现中都沿同一方向出现，它大概率反映问题域本身的结构，而非某个团队的设计偏好。

**核心洞察**：
1. **先建协调机制再干活**——task-board 追踪进度、review-checklist 定义质量标准是多 Agent 协作的前提
2. **品味注入是必要的**——没有基向量的蒸馏会产生"客观映射"但缺乏优先级
3. **Handoff 是 Agent 间的 API**——恰好包含下一个 Agent 所需的信息，渐进式披露
4. **好的 Harness 让 Agent 运行 30-60 分钟得到高质量产出**——在 coding 任务上效果尤为明显
5. **审查与执行分离**——不让实现者审查自己的代码，两个 Agent 互不可见通过文件系统协调
