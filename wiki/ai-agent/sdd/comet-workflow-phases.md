---
title: Comet 五阶段工作流与 Skill 调度
description: Comet 的 8 个 SKILL.md 文件驱动的五阶段工作流——open/design/build/verify/archive，含入口检测、阶段判定、预设路径与审查模式详解
aliases: [comet workflow, Comet 工作流, comet skill, Comet 阶段]
tags: [ai-agent, practice, tool]
sources: [2026/07/07/Comet整合OpenSpec与Superpowers详解.md]
created: 2026-07-07
updated: 2026-07-07
---

# Comet 五阶段工作流

Comet 由 8 个 SKILL.md 文件联合驱动，每个都是一个独立的"处理单元"。入口 skill（`/comet`）负责状态检测与分派，6 个子 skill 各管一个阶段，2 个预设 skill 提供简化路径。

## 整体流程

```
/comet 入口检测
  ├─ 预设命中 → /comet-hotfix 或 /comet-tweak
  └─ 读取 .comet.yaml phase → 分派到对应子 skill
       ├─ phase=open    → /comet-open
       ├─ phase=design  → /comet-design
       ├─ phase=build   → /comet-build
       ├─ phase=verify  → /comet-verify
       └─ phase=archive → /comet-archive
```

每个子 skill 退出前执行衔接协议：`comet-guard --apply` → `comet-state next`。

---

## 各阶段详解

### 阶段 0：入口与分派（comet/SKILL.md，269 行）

总调度器，负责：
- **Preset 检测**：命中 hotfix/tweak → 对应子 skill
- **状态发现**：`openspec list --json` → 读 `.comet.yaml`
- **断点恢复**：从文件重新判定阶段，不依赖对话历史（如 `build_pause: plan-ready`）
- **预设升级**：hotfix 涉及 3+ 文件 / tweak 涉及 5+ 文件 → 升级为 full workflow
- **9 个阻塞点**必须暂停等用户确认

### 阶段 1：开启 — `/comet-open`（220 行）

唯一创建 change 的入口，**铁律：禁止直接调用 `/opsx:new`**。

流程：
1. 加载 `openspec-explore` → 需求探索 → 形成澄清摘要
2. **PRD 拆分预检** [阻塞点]：大型需求候选拆分
3. **需求澄清确认** [阻塞点]：展示摘要等确认
4. **Change 名称确认** [阻塞点]：推荐 2-3 个 kebab-case 英文名
5. 加载 `openspec-new-change` → 创建 proposal/design/tasks
6. `comet-state init <name> full` → 初始化 `.comet.yaml`

### 阶段 2：深度设计 — `/comet-design`（264 行）

OpenSpec → Superpowers 的桥梁。

流程：
1. **生成交接包**：`comet-handoff.sh --write`
   - `design-context.json`（机器索引）+ `design-context.md`（人类可读）
   - 含 source path + line range + SHA256 哈希
   - 超出摘录预算标记 `[TRUNCATED]`
2. **Brainstorming**：加载 Superpowers `brainstorming` skill
   - 输入：交接包
   - 产出：2-3 个技术方案 + 测试策略
   - **用户确认设计方案** [阻塞点]
3. 产出 Design Doc → `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
   - Frontmatter 必含：`comet_change` + `role: technical-design` + `canonical_spec: openspec`

**交接包的核心价值**：不是省 token，而是防止上下文压缩导致信息丢失。Brainstorm 前读取的 proposal/design/tasks 在多轮对话后易被压缩掉，交接包凝聚成带 SHA256 校验的确定性快照——压缩丢不掉（最近读取优先保留）、丢失能找回（source path 精准定位）、篡改能发现（hash 比对）。

### 阶段 3：计划与构建 — `/comet-build`（317 行）

最复杂的阶段。

流程：
1. **Plan 创建（子代理离载）**：子代理加载 `writing-plans`，基于 Design Doc + tasks.md 创建实施计划
   - 为什么用子代理：writing-plans 的中间推理是一次性的，内联执行会永久占用 build 阶段上下文预算。子代理用完即焚，只把 plan 路径带回
2. **Plan-ready 暂停点** [阻塞点]：可选暂停切换模型（plan 用便宜模型，build 用最强模型）
3. **选择工作方式** [阻塞点]：
   - 隔离方式：worktree / branch
   - 执行方式：subagent-driven-development / executing-plans / direct
   - TDD 模式：tdd / direct
   - 审查模式：off / standard / thorough
4. **执行计划**：
   - `executing-plans`：按 plan 逐个执行，叠加 TDD + code review
   - `subagent-driven-development`：主会话只做协调，派发 implementer → spec reviewer → quality reviewer → 修复 agent 循环
     - 持久进度检查点：`subagent-progress.md`
     - 角色隔离：每个角色必须全新后台 agent
     - Task 勾选验证：协调者在双审查通过后才勾选 + `task-checkoff` 验证
5. **异常调试**：任何失败触发 `systematic-debugging`
6. **Spec 增量更新**：小改直接编辑 delta spec / 中改 brainstorming 更新 Design Doc / 大改创建独立 change

**tasks.md 更新机制**（不依赖 `openspec-apply-change`）：
```
Task 完成 → agent 直接编辑 tasks.md（改 `- [ ]` 为 `- [x]`）
         → comet-state task-checkoff 验证（防误勾/漏勾/重复勾）
         → comet-guard.sh build --apply 检查（全部勾选才放行）
```

### 阶段 4：验证与收尾 — `/comet-verify`（232 行）

唯一同时调用 OpenSpec 和 Superpowers skill 的阶段。

流程：
1. **改动规模评估**：`comet-state scale <name>` 自动判定 light/full
2. **执行验证**：
   - `verification-before-completion`（Superpowers）：light（构建/测试/tasks 全勾）或 full（+ spec 全覆盖 + 代码审查）
   - `openspec-verify-change`（OpenSpec）：验证实现 vs proposal/design/tasks/spec
3. **验证失败决策** [阻塞点]：汇总三方发现 → 定严重程度 → 全部修复或逐项处理（CRITICAL 必修）
4. **分支处理**：`finishing-a-development-branch` [阻塞点] → `branch_status: handled`

### 阶段 5：归档 — `/comet-archive`（100 行）

最简短的 skill，不加载任何外部 Skill。唯一的 CLI 调用（`openspec archive --yes`）在 `comet-archive.sh` 内部执行。

流程：
1. **归档前最终确认** [阻塞点]：展示 change 名称 + 验证结论 + 不可逆动作
2. `comet-archive.sh <name>` 一键归档：Design Doc 标注 → openspec archive → comet-state transition archived

---

## 代码审查模式

`review_mode` 在 build 阶段选择，控制整个执行过程的审查策略：

| | off | standard | thorough |
|------|------|------|------|
| 审查次数 | 0 | 1 次（最终） | N 批次 + 1 次最终 |
| 审查范围 | — | 正确性/安全/边界 | spec 合规 + 代码质量 |
| 最大修复轮次 | 0 | 1 轮 | 2 轮（每批次+最终） |
| 失败上限 | — | 1 轮不通过→BLOCKED | 2 轮不通过→BLOCKED |
| 适用场景 | 文档/配置 | 大多数普通改动 | 高风险/多模块/架构 |

设计思想：**审查成本与风险成正比**。

---

## 预设路径

### hotfix（200 行）

跳过 brainstorming → open → build → verify → archive。
- TDD 默认 direct，Review 默认 off，Isolation 默认 branch
- 升级条件：3+ 文件 / 架构变更 / DB schema / 新 public API / 超出单一函数

### tweak（176 行）

跳过 brainstorming + 完整 plan → open → lightweight build → light verify → archive。
- TDD 默认 direct，Review 默认 off
- 升级条件：5+ 文件 / 多模块 / 5+ 测试用例 / 新 capability / 需要 delta spec

---

## 衔接机制

所有子 skill 退出前执行相同的两步协议，实现无中心控制器的自动流转：

```
子 skill 完成阶段工作
    │
    ├─ ① comet-guard.sh <name> <phase> --apply
    │     逐项检查 → ALL CHECKS PASSED → 推进 phase
    │     任一 FAIL → BLOCKED，不推进
    │
    └─ ② comet-state.sh next <name>
          ├─ NEXT: auto → Skill 工具自动加载下一 skill
          ├─ NEXT: manual → 提示用户手动运行
          └─ NEXT: done → 流程完成
```

`auto_transition` 只控制步骤②是否自动加载下一 skill——步骤①的 phase 推进**始终发生**。

## 相关页面

- [[comet-integration-architecture]] — 整体架构与 .comet.yaml 状态机
- [[comet-gating-system]] — 三层门控（硬门控/阶段守卫/软门控）
- [[comet-codegraph-integration]] — CodeGraph 在 build/design 阶段的使用
- [[sdd-custom-workflow]] — 薄编排方案（Action Not Phases 替代思路）
- [[superpowers-framework]] — Superpowers 7 步工作流
- [[openspec-sdd-practice]] — OpenSpec 七步工作流
