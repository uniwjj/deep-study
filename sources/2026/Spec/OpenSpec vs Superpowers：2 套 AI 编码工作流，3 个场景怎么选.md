---
ingested: 2026-05-09
wiki_pages: [ai-agent/sdd-openspec-superpowers]
---

## OpenSpec vs Superpowers：2 套 AI 编码工作流，3 个场景怎么选？

**文章信息**：
- **原文标题**：OpenSpec vs Superpowers：2 套 AI 编码工作流，3 个场景怎么选？
- **来源**：https://www.toutiao.com/article/7623687623630782991
- **作者**：热闹星星G5QW2y5（术哥无界 | ShugeX）
- **发表时间**：2026-04-01 15:23

---

**一段话总结**：

文章对比了两套 AI 编码工作流方案：OpenSpec（Fission AI 出品，`@fission-ai/openspec` v1.2.0）管"做什么"，Superpowers（Jesse Vincent 出品，v5.0.6，GitHub 125K Star）管"怎么做"。OpenSpec 核心是增量规格系统（Delta-Based Specs）+ DAG 依赖图 + 验证引擎，解决需求管理和变更追踪问题；Superpowers 核心是 14 个 Markdown 行为技能 + 反合理化设计 + TDD 强制，解决代码质量和开发纪律问题。三个场景选型建议：大型企业棕地项目选 OpenSpec，个人开发者快速迭代选 Superpowers，中型团队从零开始选两者组合。

---

**作者判断**：

作者认为两者"不是竞品关系，更像是不同层面的互补工具"。OpenSpec 解决"AI 不知道该做什么"的问题，Superpowers 解决"AI 不知道该怎么做"的问题。选型原则：**先想清楚核心问题是什么，再选对应的工具，而不是哪个 Star 多就用哪个**。模型适配方面，作者实测认为智谱 GLM-5.1 综合实力更强，MiniMax M2.7 性价比更高。

---

**核心内容**：

**OpenSpec 核心机制**：

Fission AI 创建的 AI 原生规范驱动开发框架，让 AI 按结构化规格文档干活。

- **增量规格系统（Delta-Based Specs）**：需求变更以增量形式表达，不重写整个文档。四种变更类型：ADDED（追加新行为）、MODIFIED（替换现有需求块）、REMOVED（删除）、RENAMED（FROM:/TO: 格式改标题）。对棕地项目特别友好。
- **DAG 工件依赖图**：内部用 Kahn 算法拓扑排序管理依赖：proposal → specs + design → tasks → apply。支持三层 Schema 自定义（项目本地 / 用户全局 / 包内置）。
- **验证引擎**：自动检查重复、跨节冲突、RFC 2119 SHALL/MUST 关键字、场景覆盖。
- **Agent 友好 CLI**：所有命令支持 `--json` 输出，支持 22 个 AI 编码工具（Claude Code、Cursor、Windsurf、Copilot、Gemini CLI、Codex 等）。

OpenSpec 短板：需要 Node.js >= 20.19.0；只有 1 个内置 Schema；无内置实现验证；无 Web UI；无并行变更冲突解决。

![OpenSpec 架构图](https://p26-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/7d872ddd54634f96bb9a423869ad8882~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776607448&x-signature=GDcyTBR9Qd2rdu8hYk1mHorfq%2Fg%3D)

**Superpowers 核心机制**：

纯 Markdown + YAML 文档，零运行时依赖，作为 prompt 上下文注入 AI Agent。

- **14 个技能工作流管道**：using-superpowers（启动检查）、brainstorming（Socratic 设计）、using-git-worktrees（隔离变更）、writing-plans（任务拆分）、subagent-driven-development（子 Agent 派遣 + 两阶段审查）、test-driven-development（RED-GREEN-REFACTOR）、systematic-debugging（4 阶段根因调查）、verification-before-completion（声明成功前提供证据）、requesting/receiving-code-review、finishing-a-development-branch、dispatching-parallel-agents 等。
- **反合理化设计**：基于 Cialdini 说服原则，列出常见 AI 偷懒借口和反驳。在 N=28,000 次 AI 对话中验证，合规率从 33% 提升到 72%。
- **子 Agent 驱动开发**：每个任务派遣独立子 Agent（新鲜上下文），完成后两阶段审查（规格合规性 + 代码质量），状态为 DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT。

Superpowers 短板：纯 prompt 工程无代码执行；token 消耗较大（14 个技能全量注入）；过于有主见对快速原型太重；子 Agent 功能依赖平台支持；仅支持 5 个平台（Claude Code、Cursor、Codex、OpenCode、Gemini CLI）。

![Superpowers 技能管道流程图](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/8cc28951d71444668774fd52b43d739a~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776607448&x-signature=aw7rfNK9QZ0IDgkguRjMB5%2FGbS8%3D)

**三个场景选型**：

**场景 A：大型企业项目需求变更管理**（50+ 模块，8 人团队）→ **推荐 OpenSpec**
- 增量规格系统对付频繁变更，DAG 保证执行顺序，验证引擎防冲突，22 个 AI 工具适配团队混合使用

**场景 B：个人开发者快速迭代**（独立开发者，Claude Code 主力）→ **推荐 Superpowers**
- 零配置开箱即用，TDD 自动化防"先跑起来再说"，4 阶段根因调查，验证前置

**场景 C：中型团队规范化开发**（5 人，Claude Code + Cursor 混合）→ **OpenSpec + Superpowers 组合**
- OpenSpec 管规格层（产品需求 → 规格文档，DAG 拆任务，验证引擎检查覆盖，增量变更审计）
- Superpowers 管行为层（统一 TDD、代码审查、子 Agent 驱动）
- 组合流程：OpenSpec 定义"做什么"，Superpowers 定义"怎么做"

![选型决策树](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/993683a7a3ee4ee59371723f4229bf9e~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776607448&x-signature=H%2Fz9RJ9rvhnAm6lD0zLLJeL5bf0%3D)

**组合实战方案**：

- **第一层**：OpenSpec 做项目骨架（`npm install -g @fission-ai/openspec` → `openspec init` → `openspec propose` → `openspec spec`）
- **第二层**：Superpowers 做行为约束（技能文档放 `.claude/` 或 `.cursor/` 目录，推荐 5 个核心技能：test-driven-development、writing-plans、systematic-debugging、verification-before-completion、requesting-code-review，比全量 14 个省 token）
- **第三层**：两者协作（OpenSpec 规格 → Superpowers writing-plans 输入 → 拆任务对照规格 → TDD 写测试再写实现 → OpenSpec 验证引擎检查 → Superpowers 验证技能提供证据）

组合注意事项：token 消耗叠加、维护成本高、学习曲线陡。

![组合方案架构图](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/09d656cc428a4a92852072766893d3f6~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776607448&x-signature=LZb%2F2iuMAhmLUAuICUypR9Yplow%3D)

**快速选型**：
- 需求复杂、变更频繁、多人协作 → OpenSpec
- 一个人快速迭代、需要代码质量保障 → Superpowers
- 中型团队、从零开始、有明确需求 → 组合方案

