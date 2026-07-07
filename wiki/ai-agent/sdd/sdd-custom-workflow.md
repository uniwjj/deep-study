---
title: SDD 自定义工作流：OpenSpec + Superpowers 薄编排
description: 用 sdd-* action skills 统一入口，薄编排架构委托 OpenSpec/Superpowers，消除三套入口的选择困惑
aliases: [sdd-workflow, SDD 工作流, 薄编排, thin orchestration, Action Not Phases]
tags: [ai-agent, practice, architecture]
sources: [2026/05/17/SDD 实践：OpenSpec + Superpowers 整合创建自定义工作流.html]
created: 2026-05-17
updated: 2026-07-07
---

# SDD 自定义工作流：OpenSpec + Superpowers 薄编排

> 把 OpenSpec 和 Superpowers 的能力整合为一套连贯的 SDD 工作流，用户只认识 `sdd-*`，不知道 OpenSpec 或 Superpowers 的存在。

## 核心理念

### Action Not Phases

每个操作是**独立能力**，不是必须按顺序完成的阶段。依赖关系是 **enabler**（前置产物应存在），不是 **gate**（缺失则阻断）。

```
sdd-brainstorm  — 想深度探索设计
sdd-propose     — 想固化提案
sdd-ff          — 想快进生成所有规划文档
sdd-plan        — 想细化实施计划
sdd-code        — 想按 TDD 实施
sdd-review-spec — 想审查 spec 质量
sdd-review-code — 想审查代码
sdd-verify      — 想全面验证
sdd-ship        — 想归档合并
```

大特性走完整流程，小修复可以跳过 brainstorm 直接 `sdd-propose`——不是违规跳阶段，而是根据需要选择能力组合。

### 产物接力

每个 action 的输出是下一个 action 的输入，**所有中间状态持久化为文件**：

```
brainstorm.md → proposal.md → specs/ → design.md → tasks.md → plan.md → [代码] → [归档]
```

核心价值：任意步骤之间可以安全 `/clear`，状态在文件系统中，不在对话历史里。每个 action 是"无状态"的——只读取文件，不依赖之前的对话记忆。

## 薄编排架构

SDD 只做编排，核心工作委托给底层 skill：

```
用户 ──→ sdd-* skills（唯一入口）
             │
             ├── invoke → Superpowers skills（brainstorming / TDD / debugging...）
             ├── invoke → OpenSpec skills（continue-change / ff-change / verify...）
             └── SDD 自有逻辑（前置检查 / review 循环 / 产物校验）
```

**为什么是"薄"编排**：不复制 OpenSpec 或 Superpowers 的任何逻辑。底层工具可以独立升级，SDD 自动跟进。修 bug 只需改一处。

### 三层架构

```
SDD Action Skills（编排层）
  sdd-brainstorm  sdd-propose  sdd-ff  sdd-plan  sdd-code
  sdd-continue  sdd-review-spec  sdd-review-code
  sdd-verify  sdd-ship  sdd-doctor
        │                      │
┌───────▼───────────┐  ┌───────▼──────────────────────┐
│ Superpowers（纪律层）│  │   OpenSpec（规格层）          │
│ brainstorming      │  │ Schema / 模板系统             │
│ writing-plans      │  │ continue-change / ff-change  │
│ TDD                │  │ verify-change / archive      │
│ debugging          │  │ Delta Spec 格式              │
│ code-review        │  │                              │
│ verification       │  │                              │
└────────────────────┘  └──────────────────────────────┘
```

### 每个 Skill 的三段式结构

| 阶段 | 执行方 | 职责 |
|------|--------|------|
| 前置逻辑 | SDD 自有 | 定位 change 目录、读取已有 artifact、检查前置条件 |
| 核心执行 | invoke 底层 skill | 核心工作交给 OpenSpec 或 Superpowers |
| 后置逻辑 | SDD 自有 | review 循环、产物格式校验、下一步引导 |

### Override 机制

部分 Superpowers skills 有内置的自动转场（如 brainstorming 完成后自动进入 writing-plans），这会在 SDD 中打断编排节奏。通过显式 Override 解决：

| 被调用 Skill | Override | 保留 |
|-------------|----------|------|
| brainstorming | 输出位置、模板格式、禁止转入 writing-plans、跳过内置 reviewer | 苏格拉底式提问、方案探索、分段确认 |
| writing-plans | 输出位置、模板格式、禁止转入 executing-plans、跳过内置 reviewer | 2-5 分钟粒度、TDD 步骤结构 |
| using-git-worktrees | 分支命名为 `sdd/<change-name>` | 安全验证、基线测试 |

其余 skills（TDD、debugging、code-review、verification 等）的核心纪律全部保留，无需 Override。

## 委托关系全景

| SDD Action | 委托给 | SDD 自有逻辑 |
|------------|--------|-------------|
| sdd-doctor | 无 | 检查环境、skill 完整性、change 状态 |
| sdd-brainstorm | superpowers:brainstorming | 前置：定位 change dir / 后置：brainstorm-reviewer 循环 |
| sdd-propose | openspec-continue-change | 前置：读 brainstorm / 后置：决策追溯检查 |
| sdd-continue | openspec-continue-change | 识别下一个缺失 artifact、格式校验 |
| sdd-ff | openspec-ff-change | 识别所有缺失 artifact、批量校验 |
| sdd-plan | superpowers:writing-plans | 前置：读 tasks+specs+design / 后置：plan-reviewer 循环 |
| sdd-code | superpowers:TDD + git-worktrees + debugging | 前置：读 plan 定位批次 / 后置：更新 tasks.md |
| sdd-review-code | Phase 2: superpowers:requesting-code-review | Phase 1: spec-compliance 审查 |
| sdd-verify | superpowers:verification + openspec-verify-change | 逐条 scenario 覆盖率统计 |
| sdd-ship | openspec-sync + openspec-archive + superpowers:finishing-branch | 前置：最终验证 / 编排：三步顺序执行 |

## 关键设计决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 架构模式 | 薄编排（invoke 委托） | 消除逻辑复制和版本漂移，底层工具可独立升级 |
| 用户入口 | 只有 `sdd-*` | 消除三套入口（SDD / OpenSpec / Superpowers）的选择困惑 |
| Schema 职责 | 只管内容约束 | 执行者无法执行流程编排指令，职责分离 |
| design.md | 可选化 | 简单变更不需要技术设计文档 |
| ff 不生成 plan | apply 依赖 tasks 而非 plan | plan 单独由 writing-plans 纪律保证质量 |
| tasks vs plan | 分开（what vs how） | 合并会导致要么 tasks 过于详细，要么 plan 过于抽象 |
| 代码审查 | 两阶段：spec 合规 → 代码质量 | 先确认"做对了"再讨论"做好了" |
| Review 形态 | 内嵌（brainstorm/plan）+ 独立（spec/code） | 内嵌是质量保证，独立是可选能力 |

## 信息丢失防护

长文档链路中， brainstorm 的关键约束可能在 plan 生成时已不可见（中间经过多次 `/clear`）。

**两层防护机制**：

1. **"决策追溯"必填节** — proposal.md 和 design.md 模板要求显式引用 brainstorm 中的关键决策：
   - `选择 [X] 而非 [Y]：[原因]（见 brainstorm.md §关键决策）`
   - `约束 [Z]：[来源]（见 brainstorm.md §约束分析）`

2. **后置逻辑自动检查** — sdd-propose 的后置逻辑验证 proposal 是否引用了 brainstorm 中的所有关键决策

信息沿依赖链向下传递：`brainstorm → proposal（决策追溯）→ specs（需求场景）→ tasks（spec 链接）→ plan（保留 spec 链接）`。每一环都有显式引用，不靠对话记忆。

## 上下文卫生

核心习惯：**每个 action 完成后 `/clear`**。

- 每个 action 完成时输出：`"本 action 已完成，产物已持久化至 <path>。如需释放上下文，可安全 /clear。"`
- 三个 action 包含交互式流程，不适合中途清空：sdd-brainstorm、sdd-plan、sdd-code
- 不要在 action 中途 `/clear`——每个 action 设计为原子操作

## 完整工作流

### 大特性标准路径

```
sdd-brainstorm → /clear
sdd-ff → /clear
sdd-review-spec → /clear
sdd-plan → /clear
sdd-code → /clear     # 批次一
sdd-review-code → /clear
sdd-code → /clear     # 批次二
sdd-review-code → /clear
sdd-verify → /clear
sdd-ship
```

### 小修复最短路径

```
sdd-propose → /clear → sdd-ff → /clear → sdd-plan → /clear → sdd-code → /clear → sdd-ship
```

跳过 brainstorm（需求已明确）、跳过 spec review（改动小）、跳过独立 verify（ship 内置验证）。

### Next Action 引导

每个 action 完成时给出明确的下一步建议。例如：
- `sdd-brainstorm` 逐步确认 → `sdd-propose`；需求充分 → `sdd-ff`
- `sdd-verify` Pass → `sdd-ship`；Fail → `sdd-code` 补充

## 渐进采用策略

| 阶段 | 使用 Action | 建立习惯 |
|------|-----------|---------|
| 第一阶段 | sdd-propose → sdd-ff → sdd-plan → sdd-code → sdd-ship | spec 驱动 + TDD |
| 第二阶段 | + sdd-review-spec + sdd-review-code | 审查纪律 |
| 第三阶段 | + sdd-brainstorm + sdd-verify | 完整工程纪律 |

每个阶段都能独立提供价值，不需要一次性使用全部 11 个 action。

## 适用场景

- **复杂特性**：brainstorm → spec → TDD 流程在需求复杂、边界条件多时价值最大
- **长期项目**：归档机制将变更历史转化为制度记忆
- **团队协作**：artifact 体系提供共同语言，review 让代码审查更有焦点
- 对于简单的一次性脚本或快速验证原型，完整流程的开销大于收益

> 核心哲学：用结构消除歧义，用纪律保证质量，用归档积累智慧。

## 相关页面

- [[openspec-schema-mechanism]] — Schema 机制深度解析
- [[openspec-skill-mechanism]] — SKILL 文件机制与 invoke 委托
- [[superpowers-framework]] — Superpowers 框架详解
- [[openspec-sdd-practice]] — OpenSpec 七步工作流实战
- [[superpowers-openspec-pitfalls]] — 双框架组合的 7 个坑及衔接 Skill
- [[superpowers-openspec-legacy-workflow]] — 老旧项目四阶段实战
- [[sdd-openspec-superpowers]] — SDD 双框架对比
- [[comet-integration-architecture]] — Comet 状态机驱动整合（替代/互补方案）
- [[comet-gating-system]] — Comet 三层门控体系
