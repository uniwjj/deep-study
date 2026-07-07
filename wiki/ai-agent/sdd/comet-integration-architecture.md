---
title: Comet 整合架构：OpenSpec 与 Superpowers 的双星系统
description: Comet 将 OpenSpec（WHAT）和 Superpowers（HOW）视为双星系统，通过 .comet.yaml 状态机、五阶段工作流、三层门控和脚本体系实现无缝协作
aliases: [Comet, comet, Comet 整合, Comet 架构, 双星整合]
tags: [ai-agent, architecture, tool]
sources: [2026/07/07/Comet整合OpenSpec与Superpowers详解.md]
created: 2026-07-07
updated: 2026-07-07
---

# Comet 整合架构

Comet 是一套将 OpenSpec 和 Superpowers 整合为统一工作流的**状态机驱动 Skill 调度器 + 保障基础设施**。它不合并两套系统，而是通过桥接层让它们围绕同一目标独立协作。

## 核心哲学

```
OpenSpec 负责 WHAT  — 大纲、提案、spec 生命周期、归档
Superpowers 负责 HOW — 技术设计、计划、执行、收尾
```

Comet 把两者视为**双星系统**——各司其职、边界清晰，通过精心设计的桥接基础设施无缝协作。

## 五维整合

### 1. 双文件树 — 物理隔离，逻辑关联

```
openspec/changes/<name>/     ← OpenSpec 领地（WHAT）
├── .openspec.yaml
├── .comet.yaml              ★ 桥接文件（Comet 状态机）
├── proposal.md / design.md / tasks.md
├── specs/<capability>/spec.md
└── .comet/
    ├── handoff/              ★ 阶段交接包
    └── subagent-progress.md  ★ 子代理调度检查点

docs/superpowers/            ← Superpowers 领地（HOW）
├── specs/YYYY-MM-DD-<topic>-design.md
└── plans/YYYY-MM-DD-<feature>.md
```

`.comet.yaml` **物理上**存在于 OpenSpec 的 change 目录中，但**逻辑上**驱动整个 Superpowers 的执行流程。

### 2. `.comet.yaml` — 状态机桥接层

整合的**核心数据契约**，一条记录同时承载两套系统的信息：

| 类别 | 关键字段 | 说明 |
|------|---------|------|
| OpenSpec 侧 | `workflow`, `phase` | full/hotfix/tweak；open→design→build→verify→archive |
| Superpowers 侧 | `design_doc`, `plan`, `build_mode`, `tdd_mode`, `review_mode`, `isolation` | 指向 Superpowers 产物，控制执行方式 |
| 桥接 | `base_ref`, `handoff_context`, `handoff_hash`, `verify_result`, `archived` | 跨系统验证与追溯 |

**关键约束**：状态字段只能通过 `comet-state.sh` 修改；阶段推进只能通过 `comet-guard.sh --apply` 完成。

### 3. 五阶段分工与 Skill 调度

| 阶段 | 主导方 | 核心动作 |
|------|--------|---------|
| ① Open | OpenSpec | openspec-explore → openspec-new-change → comet-state init |
| ② Design | Superpowers | comet-handoff 生成交接包 → brainstorming → Design Doc |
| ③ Build | Superpowers | writing-plans → subagent-driven-dev / executing-plans → TDD + code review |
| ④ Verify | 双系统协作 | verification-before-completion + openspec-verify-change + finishing-branch |
| ⑤ Archive | OpenSpec | delta → main spec 合并 → Design Doc 标注 → change 移入 archive |

详见 [[comet-workflow-phases]]。

### 4. 阶段间自动衔接协议

每个子 skill 退出前执行两步：

```
1. comet-guard <name> <phase> --apply    # 验证 → 推进 phase
2. comet-state next <name>               # 解析下一步：auto / manual / done
```

`auto_transition` 只控制是否自动加载下一 skill，不影响 phase 推进。

**9 个决策阻塞点**：proposal 确认、设计方案确认、plan-ready + 工作方式选择、验证失败决策、分支处理、归档确认、hotfix/tweak 升级、范围扩张重设计、PRD 拆分确认。

### 5. 预设路径与升级机制

| | hotfix | tweak |
|---|---|---|
| 跳过 | brainstorming | brainstorming + plan |
| TDD 默认 | direct | direct |
| Review 默认 | off | off |
| 升级条件 | 3+ 文件 / 架构变更 / DB schema / 新 API | 5+ 文件 / 多模块 / 5+ 测试用例 / 新 capability |

升级触发后阻塞确认，补充产物后回到完整流程。

## 脚本体系

所有脚本位于 `comet/scripts/`，由 `comet-env.sh` 统一加载：

| 脚本 | 职责 | 触发方式 |
|------|------|---------|
| `comet-hook-guard.sh` | 硬门控：文件系统级拦截越权写入 | PreToolUse Hook 自动触发 |
| `comet-guard.sh` | 阶段守卫：校验前置条件 → 推进 phase | agent 按 SKILL.md 显式调用 |
| `comet-state.sh` | 状态机操作：init/check/set/transition/scale/next | agent 按需调用 |
| `comet-handoff.sh` | 交接包生成：OpenSpec 产物 → 确定性快照 | agent 在 design 阶段调用 |
| `comet-archive.sh` | 归档执行：spec 合并 + 标注 + 移动 | agent 在 archive 阶段调用 |
| `comet-yaml-validate.sh` | YAML 校验 | 被 guard/state 内部调用 |

详见 [[comet-gating-system]]。

## 外部调用方式

Comet 调用 OpenSpec 和 Superpowers 只有两种方式：

- **Skill 工具加载**：agent 按 SKILL.md 文本指令加载（如 `brainstorming`、`writing-plans`）
- **Bash 执行 CLI/脚本**：agent 执行 `openspec list --json` 等命令

唯一的例外是 `comet-archive.sh` 内部执行的 `openspec archive --yes`——其余所有调用都是 agent 读 SKILL.md 后主动执行，没有钩子、回调或自动触发。

### Comet 不用的 OpenSpec 能力

| 能力 | 替代方案 |
|------|---------|
| `openspec-propose` | 由 `/comet-open` 分步执行 |
| `openspec-apply-change` | 由 Superpowers 执行体系替代 |
| `openspec-archive-change` | 由 `comet-archive.sh` 替代 |
| `openspec-sync-specs` | 归档时由 `comet-archive.sh` 处理 |

## 与现有 SDD 自定义工作流的关系

Comet 与 [[sdd-custom-workflow]]（薄编排）是**同一问题的两种解决方案**：

| 维度 | 薄编排 (sdd-*) | Comet |
|------|---------------|-------|
| 架构 | 薄编排，invoke 委托 | 状态机驱动，桥接层 |
| 状态管理 | 产物文件接力 | `.comet.yaml` 集中状态机 |
| 门控 | 前置/后置逻辑检查 | 三层纵深防御（硬/软/守卫） |
| 上下文防护 | /clear + 决策追溯 | handoff 交接包 + SHA256 校验 |
| 阶段衔接 | Next Action 引导 | guard --apply + auto_transition |
| 灵活性 | Action Not Phases（独立能力） | 五阶段线性流程（预设路径简化） |

两者可互补：薄编排强调"每个 action 是独立能力"，Comet 强调"流程不该被跳过"。实际选择取决于项目对流程纪律的要求程度。

## SDD 设计启示

1. **整合复杂度不在"调用"，在"保障"** — 把精力花在状态持久化、防信息丢失、产物校验、兜底拦截上
2. **"调用"的本质是让 agent 读到正确的指令** — 所有外部调用都是 SKILL.md 中的文本指令，不需要回调/RPC/事件总线
3. **状态必须存文件，不能靠对话历史** — 上下文压缩会丢、会话会断、模型会换
4. **门控本质是 agent 自律 + 兜底硬拦截** — 软门控和阶段守卫依赖 agent 遵守，只有硬门控是平台级强制
5. **交接包解决上下文压缩导致的信息丢失** — 脚本确定性提取 + SHA256 校验，压缩丢不掉、丢失能找回、篡改能发现
6. **子代理离载是上下文预算管理** — 一次性推理用完即焚，只把结果带回主会话
7. **审查成本应与风险成正比** — off/standard/thorough 三级分级投入
8. **进度分三层可见** — OpenSpec 管"做什么"、Superpowers 管"做到哪了"、Comet 管"质量是什么状态"

### 核心原则

1. **不合并，不重复** — 各自独立演进，Comet 只做桥接
2. **文件是唯一可靠的状态载体** — `.comet.yaml` 跨会话、跨模型、跨压缩存活
3. **脚本生成 > agent 总结** — 交接包、状态更新、归档全部脚本化
4. **漏斗式门控** — 软门控广泛引导 → 阶段守卫批量校验 → 硬门控精准兜底
5. **触发方分离** — Rule 平台注入、Guard agent 调用、Hook 平台拦截
6. **指令即调用** — 没有魔法，没有回调
7. **子代理离载 = 上下文预算管理**
8. **审查成本与风险成正比**
9. **知识驱动** — CodeGraph 消除"幻觉式"实现

## 相关页面

- [[comet-gating-system]] — 三层门控体系（硬门控/阶段守卫/软门控）
- [[comet-workflow-phases]] — 五阶段工作流与 8 个 SKILL 详解
- [[comet-codegraph-integration]] — CodeGraph MCP 代码智能引擎
- [[sdd-custom-workflow]] — 薄编排方案（Comet 的替代/互补方案）
- [[sdd-openspec-superpowers]] — OpenSpec + Superpowers 双框架对比
- [[superpowers-framework]] — Superpowers 框架详解
- [[openspec-sdd-practice]] — OpenSpec 实战指南
