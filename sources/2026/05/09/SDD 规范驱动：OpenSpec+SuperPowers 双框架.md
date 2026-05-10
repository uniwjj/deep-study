---
ingested: 2026-05-09
wiki_pages: [ai-agent/sdd-openspec-superpowers]
---

## SDD 规范驱动，OpenSpec+SuperPowers 双框架，让 AI 高效无债可追溯总结

**文章信息**
- **原文标题**：SDD 规范驱动，OpenSpec+SuperPowers 双框架，让 AI 高效无债可追溯
- **来源**：<https://www.toutiao.com/article/7626258875386167859>
- **作者**：AI码韵匠道
- **发表时间**：未知

**一段话总结**：文章系统性介绍了规范驱动开发（SDD，Spec-Driven Development）的理念——"先想清楚再动手"，在让 AI 写代码前先明确需求规范、设计标准和验收条件，用规范文档约束 AI 行为。文章详细拆解了两个落地 SDD 的核心框架：OpenSpec 侧重"变更追溯+知识积累"，通过 proposal→design→spec→tasks→apply→verify→archive 的 Artifact 依赖链实现全流程可追溯，并有主 spec+delta spec 体系实现知识长期积累；SuperPowers 侧重"执行质量+自动化审查"，通过子代理驱动开发（Controller+Implementer+Spec Reviewer+Code Quality Reviewer）、强制 TDD（RED-GREEN-REFACTOR 循环）、两阶段审查确保代码质量。两者互补，可结合使用。

---

**核心事件**：文章提出 SDD 规范驱动开发理念并详细对比两个落地框架 OpenSpec 和 SuperPowers，解决 AI 编码"不规范"痛点。

**传统 AI 辅助编码的三个致命问题**：
1. **需求传递偏差**：用户描述碎片化，AI 根据字面猜测，代码偏离实际需求
2. **缺乏可追溯性**：代码出问题无法追溯设计意图，只能逐行排查
3. **技术债积累**：AI 生成代码可能冗余、不规范，长期积累形成难以清理的技术债

**SDD 的闭环工作流**：用户描述需求→生成规范文档→AI 按规范实现代码→按规范验证代码。核心是"用文档约束行为，用验证确保正确"。

---

### OpenSpec 框架

OpenSpec 是基于 Node.js 的结构化开发工作流框架，核心思想是"每次变更都有完整设计文档可追溯"。安装只需 `npm install -g @anthropic/openspec` + `openspec init`。

**Artifact 依赖链**：proposal.md → design.md → specs/*.md → tasks.md → 代码实现 → 归档

- **proposal.md**：变更起点，阐述"为什么做"（原因、目标与非目标、影响范围）
- **design.md**：明确"怎么做"（技术方案、替代方案对比、风险评估）
- **specs/*.md**：核心规范文件，采用 Requirements + Scenarios 格式，Scenarios 用 WHEN/THEN 句式描述预期行为
- **tasks.md**：需求拆解为可执行 checkbox 任务

**核心命令**：
- `/opsx:explore`：探索模式，自由讨论需求和技术方案
- `/opsx:propose`：生成 proposal.md 并规划后续 Artifact
- `/opsx:ff`：快速通道，一次性生成所有 Artifact（适合简单需求）
- `/opsx:apply`：按 tasks.md 逐任务实现代码并自动验证
- `/opsx:verify`：对照所有 Artifact 检查代码合规性
- `/opsx:archive`：归档变更到 openspec/changes/archive 目录

**主 spec + delta spec 体系**：主 spec 位于 openspec/specs 目录，是项目级规格知识库；delta spec 在各变更的 specs 目录，只含本次变更相关规范。归档时自动将 delta spec 同步到主 spec，实现知识长期积累。

---

### SuperPowers 框架

SuperPowers 由 Jesse Vincent 开发，核心是"Skills 组合"——一组可组合的 Markdown 指令文件，定义 AI 在各开发阶段的行为。支持 Claude Code、Cursor、Codex、Gemini CLI 等平台。

**核心 Skills 分类**：
- **brainstorming**：苏格拉底式对话梳理需求，输出设计文档
- **writing-plans**：拆分为 2-5 分钟可完成的 bite-sized tasks
- **subagent-driven-development**：每个 task 派遣独立子代理实现和自审查
- **test-driven-development**：强制 RED-GREEN-REFACTOR 循环
- **requesting-code-review**：两阶段审查（Spec 合规性→代码质量）
- **using-git-worktrees**：创建隔离工作空间
- **finishing-a-development-branch**：分支合并、创建 PR

**子代理驱动开发流程**：Controller 派遣 Implementer 子代理（代码实现+TDD+自审查）→Controller 派遣 Spec Reviewer 子代理（需求覆盖检查）→Controller 派遣 Code Quality Reviewer 子代理（代码规范/架构/安全/测试覆盖审查，分 Critical/Important/Minor 三级）。审查者与实现者分离，无心理包袱，独立上下文避免逻辑漂移。

**强制 TDD**：Plan 阶段就把 TDD 步骤写进每个 task。Step 1 写失败测试（RED）→Step 2 运行确认失败→Step 3 写最小实现→Step 4 运行确认通过（GREEN）→Step 5 提交。如果 Implementer 没先写测试就开始编码，Spec Reviewer 直接拒绝审查。

**两阶段审查**：第一阶段 Spec Reviewer 检查"做对了吗"（需求覆盖、场景遗漏、过度实现）；第二阶段 Code Quality Reviewer 检查"做好了吗"（代码规范、架构、安全、测试覆盖）。

---

### OpenSpec vs SuperPowers 对比

**核心定位差异**：
- OpenSpec：规格驱动的变更管理框架，侧重决策追溯+知识积累
- SuperPowers：多代理协作的完整开发工作流，侧重执行质量+自动化审查

**关键差异**：
- **代理模式**：OpenSpec 单代理全程负责；SuperPowers Controller+Implementer+Reviewers 分工明确
- **Spec 管理**：OpenSpec 有独立 spec 体系（主 spec+delta spec 自动同步）；SuperPowers 无独立 spec，融合在 design doc 和 plan 中
- **变更追溯**：OpenSpec archive 按日期归档含完整 Artifact，追溯性强；SuperPowers 依赖 Git 历史
- **测试策略**：SuperPowers 强制 TDD；OpenSpec 不强制，以编译通过和行为验证为准
- **平台依赖**：OpenSpec 任何 AI 编码助手可用；SuperPowers 需要支持子代理（如 Claude Code）

**适用场景推荐**：
- 全新项目/绿地开发→SuperPowers（TDD+子代理协作端到端自动化）
- 大型企业项目改动→OpenSpec（spec 追溯+变更归档适合团队 review）
- 需要长期维护 specs→OpenSpec（知识库长期积累）
- 有严格测试要求→SuperPowers（内置强制 TDD）
- 不支持子代理的平台→OpenSpec（天然单代理）
- 多代理协作环境→SuperPowers（分工明确）

**作者判断**：OpenSpec 和 SuperPowers 不是竞争关系而是互补关系。实际开发中可结合使用——用 OpenSpec 做 spec 管理和变更追溯，从 SuperPowers 提取审查和 git worktree 能力。SDD 的终极目标不是否定 AI 的效率，而是用规范给 AI"立规矩"，让 AI 的高效发挥在正确的轨道上，让 AI 写的每一行代码都有据可查、有规可循。
