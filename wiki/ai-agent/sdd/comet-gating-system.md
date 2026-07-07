---
title: Comet 三层门控体系
description: Comet 的纵深防御门控——硬门控（PreToolUse Hook 文件系统级拦截）、阶段守卫（agent 显式调用的批量校验脚本）、软门控（Rule 注入每轮行为引导）
aliases: [comet gating, Comet 门控, 三层门控, 硬门控, 阶段守卫, 软门控]
tags: [ai-agent, architecture, practice]
sources: [2026/07/07/Comet整合OpenSpec与Superpowers详解.md]
created: 2026-07-07
updated: 2026-07-07
---

# Comet 三层门控体系

Comet 的门控体系由三种不同层次、不同时机的门控组成，形成**纵深防御**。核心区别在于**由谁触发、在何时执行**。

## 三层总览

```
硬门控 comet-hook-guard.sh  ← Hook 自动触发（PreToolUse，每次 Write/Edit 前）
         ↓ 放行后
阶段守卫 comet-guard.sh     ← agent 显式调用（Bash 工具，按 skill 指令执行）
         ↓ 参考
软门控 comet-phase-guard.md ← 每轮自动注入（Rule 注入机制，系统提示通道）
```

| 维度 | 硬门控 (Hook) | 阶段守卫 (Guard) | 软门控 (Rule) |
|------|--------------|-----------------|-------------|
| **实现** | bash 脚本 | bash 脚本 | Markdown 文件 |
| **触发方** | Claude Code 平台 | LLM agent | Claude Code 平台 |
| **触发机制** | PreToolUse Hook 自动拦截 | Agent 按 skill 指令显式调用 | Rule 注入每轮对话 |
| **触发时机** | 每次 Write/Edit 前 | 子 skill 工作完成后 | 每轮对话开始时 |
| **作用** | 事后拦截（文件写入前） | 阶段收尾批量校验 | 事前引导（行为层面） |
| **失败后果** | 写入被 OS 级阻止 | phase 不推进 | agent 自我纠正 |
| **agent 可控** | ❌ 不可控 | ✅ 主动执行 | ❌ 自动注入 |

## 关键设计原则

1. **软门控是主力** — 从 skill 调用到决策点到预设升级，全链路行为引导
2. **阶段守卫是质量关口** — 批量校验 handoff hash、构建通过、tasks 勾选
3. **硬门控是兜底安全网** — 只做最简单拦截："phase 不对，不准写源码"
4. **能力递减，拦截力递增** — 从广泛引导收敛到精准硬拦截的漏斗
5. **触发方分离** — Rule 平台注入（不可控但无强制力）、Guard agent 调用（按 skill 执行）、Hook 平台拦截（不可控且强制）

---

## 一、硬门控：`comet-hook-guard.sh`

由 Claude Code 平台的 PreToolUse Hook 自动触发，agent 感知不到。每次 Write/Edit 前被 harness 拦截执行。

### 五步执行流程

1. **提取目标文件路径** — 从 `FILE_PATH` 环境变量或 stdin JSON 解析
2. **路径规范化** — 处理跨平台差异（反斜杠、JSON 转义、macOS `/var` 符号链接）
3. **确定管辖的 Comet change 和阶段** — "谁的地盘谁说了算"，精确匹配优先，非全局一刀切
4. **白名单路径放行** — Phase 感知白名单：

| Phase | 允许写入的路径 |
|-------|---------------|
| `open` | `*/proposal.md`, `*/design.md`, `*/tasks.md`, `*/.comet.yaml`, `*/.comet/*`, `*/specs/*` |
| `design` | 上述 + `*/specs/*`（delta spec） |
| `build` | `*/specs/*`, `*/tasks.md`, `*/.comet.yaml` |
| `verify` | `*/tasks.md`, `*/.comet.yaml` |
| `archive` | 仅 `*/.comet.yaml` |

`docs/superpowers/*` 仅在 `design`/`build`/`verify` 阶段放行。

5. **阶段强制拦截** — open/design/archive 阶段拦截源码写入，显示 ASCII 横幅告知原因

### 设计亮点

- **精确的 change 隔离**：每个 change 独立判断阶段，多 change 互不干扰
- **已归档豁免**：归档后的 change 目录可自由修改
- **非法空跳检测**：验证产物链完整性（如 `workflow: full` 但 `design_doc` 为空 → 拦截）
- **安全侧设计**：提取不到路径时放行，不影响非 Comet 工具调用

---

## 二、阶段守卫：`comet-guard.sh`

由 LLM agent 按 SKILL.md 指令显式调用：

```bash
"$COMET_BASH" "$COMET_GUARD" <change-name> <phase> --apply
```

核心职责：批量验证当前阶段的所有前置条件，**全部通过才推进 phase 字段**。

### 工作流程

```
preflight（前置校验）
  ├── 验证 change 目录存在
  ├── 验证 .comet.yaml 存在
  └── 运行 comet-yaml-validate.sh schema 校验

guard_<phase>（阶段专项检查）
  ├── 逐项 check() → BLOCK 计数
  ├── 全部 PASS → apply_state_update()
  └── 任一 FAIL → BLOCKED，exit 1

apply_state_update()
  └── 委托 comet-state.sh transition 推进 phase
```

### 各阶段检查项

#### guard_open → design

1. `proposal.md` 存在且非空
2. `design.md` 存在且非空（仅 full workflow）
3. `tasks.md` 存在且非空
4. `tasks.md` 至少有一个任务

#### guard_design → build

1. OpenSpec 产物完整性（proposal/design/tasks）
2. handoff context 存在且有效
3. handoff hash 格式合法（64 位 hex SHA256）
4. **handoff hash 与源文件当前内容匹配**（防篡改）
5. handoff markdown 可追溯（含 Generated-by、Source、SHA256）
6. `design_doc` 已记录（full workflow）
7. Design Doc frontmatter 含 `comet_change`
8. Design Doc frontmatter 声明 `role: technical-design`
9. Design Doc frontmatter 声明 `canonical_spec: openspec`

#### guard_build → verify

1. `isolation` 已选（branch/worktree）
2. `build_mode` 已选
3. `build_mode` 与 workflow 兼容（direct 仅限 hotfix/tweak）
4. `subagent_dispatch: confirmed`（若 subagent 模式）
5. `tdd_mode` 已选
6. `review_mode` 已选
7. **`tasks.md` 全部勾选**
8. **Superpowers plan 全部勾选**
9. **构建通过**（优先 `build_command`，回退自动检测）

#### guard_verify → archive

1. `tasks.md` 全部勾选
2. 验证命令通过
3. `verification_report` 存在
4. `branch_status: handled`

---

## 三、软门控：`comet-phase-guard.md`

通过 `.claude/rules/` 机制每轮对话自动注入到系统提示，**不受上下文压缩影响**。

### 为什么规则不会被压缩掉

系统提示和对话历史是 LLM API 调用的**两个独立通道**。上下文压缩只压缩对话历史，系统提示里的规则每次 API 调用都原样携带——它是"铁打的营盘"，对话历史才是"流水的兵"。

### 七大模块

1. **阶段感知表** — agent 自查当前 phase 对应的允许/禁止操作
2. **阶段进入自洽性校验** — 不仅看 `phase` 字段，还要验证"是如何到达这个阶段的"（检测空跳）
3. **Skill 调用强制** — 特定场景必须通过 Skill 工具加载，不可用普通对话替代
4. **脚本执行不可跳过** — guard、state、handoff 必须脚本执行，禁止手工编辑
5. **用户确认不可自动跳过** — 9 个决策点必须暂停等待用户明确选择
6. **上下文压缩恢复** — `comet-state check <name> <phase> --recover` 从文件恢复运行状态
7. **阶段退出后自动过渡** — `comet-state next` 按 auto/manual/done 决定下一步

### 压缩恢复机制

上下文压缩的真正威胁**不是规则丢失**（规则在系统提示中永不被压缩），而是**运行状态丢失**：
- 压缩前 agent 知道当前 change、task、进度
- 压缩后 agent 只知道阶段规则，不知道具体在哪个 change、哪个 task

恢复：`comet-state check <name> <phase> --recover` 从 `.comet.yaml` 重新读出状态字段。对于 subagent 模式，还需读取 `subagent-progress.md` 恢复精确 task 进度。

---

## 三层协作流程

```
Agent 发起 Write/Edit
    │
    ▼
硬门控: 解析路径 → 白名单匹配 → phase 判定
    ├─ 命中白名单 → 放行
    ├─ build/verify → 放行
    └─ open/design/archive + 未命中 → exit 2 阻止
    │
    ▼ (放行后)
文件写入成功 → agent 继续工作
    │
    ▼
软门控: 每轮注入规则 → agent 自查允许/禁止表 → 自洽性校验
    │
    ▼
阶段工作完成 → 阶段守卫: 逐项 check → ALL PASS → 推进 phase
```

## 相关页面

- [[comet-integration-architecture]] — Comet 整体架构
- [[comet-workflow-phases]] — 五阶段工作流
- [[sdd-custom-workflow]] — 薄编排方案中的前置/后置逻辑检查
