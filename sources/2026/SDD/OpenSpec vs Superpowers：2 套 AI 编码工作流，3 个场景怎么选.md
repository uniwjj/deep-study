---
ingested: 2026-05-09
wiki_pages: [ai-agent/sdd-openspec-superpowers]
---

## OpenSpec vs Superpowers：2 套 AI 编码工作流，3 个场景怎么选？总结

**文章信息**
- **原文标题**：OpenSpec vs Superpowers：2 套 AI 编码工作流，3 个场景怎么选？
- **来源**：<https://www.toutiao.com/article/7623687623630782991>
- **作者**：热闹星星G5QW2y5（术哥）
- **发表时间**：未知

**一段话总结**：文章深入对比了两套热门 AI 编码工作流方案——OpenSpec（规格驱动开发框架）和 Superpowers（纯 Markdown 行为约束系统）。OpenSpec 由 Fission AI 创建，核心是增量规格系统 + DAG 依赖图 + 验证引擎，解决"AI 不知道该做什么"的问题；Superpowers 由 Jesse Vincent 创建，GitHub 125K Star，核心是 14 个行为技能 + 反合理化设计 + TDD 强制，解决"AI 不知道该怎么做"的问题。作者通过三个真实场景（大型企业项目、个人快速原型、团队协作）给出选型建议：两者不是竞品关系，而是不同层面的互补工具。

---

**核心事件**：作者系统拆解 OpenSpec 和 Superpowers 两套 AI 编码工作流的机制差异，指出它们分别管理"规格层"和"行为层"，并给出三场景选型建议。

![OpenSpec vs Superpowers 信息图封面](https://p26-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/bdff0a7310204a73bf147d83d10a7937~tplv-tt-origin-web:gif.jpeg)

---

### OpenSpec：AI 原生规范驱动开发框架

**核心定位**：让 AI 编码工具按照结构化规格文档干活，而非随心所欲写代码。

**增量规格系统（Delta-Based Specs）**

这是 OpenSpec 的核心创新。变更以增量形式表达：
- **ADDED**：新行为追加到主规格
- **MODIFIED**：替换现有需求块
- **REMOVED**：删除需求块
- **RENAMED**：用 FROM:/TO: 格式改标题

对棕地项目（已有代码库）特别友好，不需要一次性写全所有需求，发现遗漏直接追加 ADDED 即可。

**DAG 工件依赖图**

内部使用 DAG（有向无环图）管理依赖关系，采用 Kahn 算法拓扑排序。默认流程：先有提案（proposal）→ 才能写规格（specs）和设计（design）→ 都完成才能拆任务（tasks）→ 拆完才能动手实现（apply）。支持三层 Schema 自定义（项目本地、用户全局、包内置）。

**验证引擎**

内置验证引擎检查规格完整性和一致性：重复检测、跨节冲突检测、SHALL/MUST 关键字检查（RFC 2119 标准）、场景覆盖检查。

**CLI 设计**

专为 AI Agent 设计 JSON 输出模式，目前支持 22 个 AI 编码工具（Claude Code、Cursor、Windsurf、Copilot、Gemini CLI 等）。

**OpenSpec 短板**：需要 Node.js >= 20.19.0；目前仅 1 个内置 Schema；无内置实现验证；仅支持 Markdown；无 Web UI。

![OpenSpec 架构图 - DAG 工件依赖关系与增量规格系统](https://p26-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/7d872ddd54634f96bb9a423869ad8882~tplv-tt-origin-web:gif.jpeg)

---

### Superpowers：纯 Markdown 行为约束系统

**核心定位**：零运行时依赖，纯 Markdown + YAML 文档作为 prompt 上下文注入 AI Agent，制定行为规范让 AI 按最佳实践工作。

**14 个技能组成的工作流管道**

包括：using-superpowers（启动检查）、brainstorming（Socratic 设计）、using-git-worktrees（隔离变更）、writing-plans（任务拆分）、subagent-driven-development（子 Agent 驱动 + 两阶段审查）、test-driven-development（RED-GREEN-REFACTOR 循环）、systematic-debugging（4 阶段根因调查）、verification-before-completion（验证收尾）、代码审查工作流、合并/PR/清理、并行 Agent 派遣等。

**subagent-driven-development** 最有意思：每个任务派遣独立子 Agent（新鲜上下文），完成后经两阶段审查（规格合规性 + 代码质量），子 Agent 必须给出明确状态：DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT。

**反合理化设计（Anti-Rationalization）**

AI 编码工具有"偷懒通病"——说好的严格 TDD，该跳过测试时还是会跳过，还会给出看似合理的借口。Superpowers 在每个技能文档里列出常见合理化借口和对应反驳。基于 Cialdini 说服原则设计，在 N=28,000 次 AI 对话中验证，合规率从 33% 提升到 72%。

支持 5 个平台：Claude Code、Cursor、Codex、OpenCode、Gemini CLI。

**Superpowers 短板**：无实际代码执行（纯 prompt）；Token 消耗较大（14 个技能全量注入）；过于有主见（严格 TDD 对快速原型太重）；无程序化强制执行；子 Agent 功能依赖平台支持。

![Superpowers 14 个技能的工作流管道](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/8cc28951d71444668774fd52b43d739a~tplv-tt-origin-web:gif.jpeg)

---

### 三场景选型

**场景 A：大型企业项目的需求变更管理**（50+ 模块，8 人团队）→ **推荐 OpenSpec**

理由：增量规格系统应对频繁变更；DAG 依赖图自动排执行顺序；验证引擎防止需求冲突；22 个 AI 工具适配团队多工具环境。Superpowers 不合适——大型项目核心挑战是规格管理，光有行为纪律不够。

**场景 B：个人开发者的快速原型迭代**（独立开发者 SaaS 产品）→ **推荐 Superpowers**

理由：零配置开箱即用，不需要写规格文档；TDD 自动化防止"先跑起来再说"的坑；4 阶段系统化调试；验证前置不能 AI 说"搞定了"就真搞定。OpenSpec 太重——个人项目需求变化快，写规格成本和收益不成正比。

**场景 C：团队协作的规范化开发流程**（5 人中型 Web 应用）→ **推荐 OpenSpec + Superpowers 组合**

- **OpenSpec 管规格层**：需求文档转成规格、DAG 拆分任务、验证引擎覆盖场景、增量管理变更
- **Superpowers 管行为层**：统一加载技能文档、TDD 确保测试覆盖、代码审查统一质量标准、子 Agent 处理复杂任务

组合注意事项：Token 消耗叠加（推荐只选 5 个核心技能而非全量 14 个）、两套工具维护成本、团队学习曲线。

![三种场景选型决策树](https://p26-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/993683a7a3ee4ee59371723f4229bf9e~tplv-tt-origin-web:gif.jpeg)

---

**一句话选型建议**：
- 需求复杂、变更频繁、多人协作 → OpenSpec
- 一个人快速迭代、需要代码质量保障 → Superpowers
- 中型团队、从零开始、有明确需求 → 组合方案

**作者判断**：OpenSpec 和 Superpowers 不是竞品，而是互补工具——OpenSpec 解决"AI 不知道该做什么"，Superpowers 解决"AI 不知道该怎么做"。工具选型没有标准答案，基本原则是先想清楚核心问题再选工具，"而不是哪个 Star 多就用哪个"。作者还测试了智谱 GLM-5.1 和 MiniMax M2.7 两个模型，认为 GLM-5.1 综合实力略强但限购涨价，MiniMax M2.7 日常够用且性价比更高。
