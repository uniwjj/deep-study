---
title: Claude Code 权限系统
description: Claude Code 的7种权限模式、拒绝优先规则引擎、ML自动分类器和Shell沙箱的详细分析
aliases: [权限系统, permission system, deny-first]
tags: [ai-agent, concept, architecture]
sources: [2026/05/10/Dive into Claude Code.txt, 2026/05/10/Claude-Code-Source-Analysis.pdf]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 权限系统

Claude Code 采用 **拒绝优先（deny-first）+ 人工升级** 的安全姿态，结合**分层防御**和**可逆性加权风险评估**。

## 七种权限模式

| 模式 | 行为 |
|------|------|
| `plan` | 模型必须先创建计划，用户批准后才执行 |
| `default` | 标准交互模式，大多数操作需审批 |
| `acceptEdits` | 工作目录内的编辑和某些文件系统命令自动批准 |
| `auto` | ML 分类器评估未通过快速检查的请求（Feature Flag 控制） |
| `dontAsk` | 不提示，但拒绝规则仍生效 |
| `bypassPermissions` | 跳过大多数提示，安全关键检查和免疫规则仍适用 |
| `bubble` | 内部专用，子代理权限升级到父终端 |

5 个外部可见模式 (`plan`, `default`, `acceptEdits`, `dontAsk`, `bypassPermissions`) 构成一个**渐进式自主梯度**。

## 拒绝优先规则引擎

权限规则按**拒绝优先**顺序评估（`permissions.ts`）：

- **拒绝规则永远优先于允许规则**，即使允许规则更具体
- 宽泛的拒绝（"拒绝所有 shell 命令"）不能被窄化允许（"允许 npm test"）覆盖
- 支持工具级匹配（按工具名）和**内容级匹配**（如 `Bash(prefix:npm)`）

## 七层安全防御

一次性操作必须通过所有适用层，任何单独一层都可以阻止：

1. **工具预过滤** — 完全禁止的工具在模型视角中移除（`tools.ts`）
2. **拒绝优先规则评估** — 拒绝规则先于允许规则
3. **权限模式约束** — 活跃模式决定无匹配规则请求的基线处理
4. **Auto-Mode 分类器** — ML 分类器评估工具安全性（Feature Flag 控制）
5. **Shell 沙箱** — 已批准的命令仍可在沙箱内执行
6. **Resume 时不恢复权限** — 会话级权限不跨会话携带
7. **钩子拦截** — PreToolUse 钩子可修改权限决定

## Auto-Mode 分类器

ML 分类器（`yoloClassifier.ts`）在启用时参与权限决定：

- 两阶段：**快速过滤** + **链式思维安全评估**
- 加载三种提示资源：基础系统提示、外部权限模板、Anthropic 内部模板
- 产生 `allow` / `deny` / 需人工审批 三种结果

用户约批准 93% 的权限提示（Anthropic 内部分析），审批疲劳使交互确认变得不可靠，因此系统不依赖人类警觉。

## Shell 沙箱

`shouldUseSandbox()` 检查全局启用状态、调用是否 opt-out、命令是否匹配排除模式。沙箱提供独立于应用级权限模型的**文件系统和网络隔离**。

## 授权管道流程

```
工具请求 → 预过滤 → PreToolUse钩子 → 规则评估 → 权限处理器 →
  ├── Coordinator路径：自动解析优先
  ├── Swarm Worker路径：独立解析逻辑
  ├── 投机分类器路径：BashTool 竞速分类
  └── 交互路径：标准用户审批对话框
```

## 初始化时序安全漏洞

独立安全研究（Check Point Research）发现：**扩展加载 → 信任对话框 → 权限执行** 的初始化顺序在权限管道完全激活前创造了一个**预信任执行窗口**（CVE-2025-59536 CVSS 8.7, CVE-2026-21852 CVSS 5.3）。

## YOLO 分类器：两阶段判断（64→4096 token）

**Stage 1**：64 token 预算，输出 `<block>yes/no</block>` ——明显安全操作的快速通道。

**Stage 2**：4096 token 预算，输出 `<block>yes/no</block><reason>...</reason>` ——仅 Stage 1 标记为潜在危险时触发。

- 两阶段均 temperature=0，保证确定性安全判断
- CLAUDE.md 注入分类器上下文——用户规则同时影响主 AI 和安全分类器
- 内外部使用不同权限模板（`permissions_external.txt` vs `permissions_anthropic.txt`）

## 熔断机制

```js
DENIAL_LIMITS = { maxConsecutive: 3, maxTotal: 20 }
```

- **3 次连续拒绝**或**20 次累计拒绝** → Auto 模式自动降级为手动确认
- 成功操作重置连续计数器（不重置累计计数器）

## 危险权限剥夺（Auto 模式）

进入 Auto 模式时**自动剥离**宽泛权限规则（`Bash(*)`、`Bash(python:*)`、`Bash(node:*)`），退出时 `restoreDangerousPermissions()` 恢复。

原因：Auto 模式的 Rule Matching（Layer 1）绕过 ML Classifier（Layer 4）。宽泛规则在 Auto 模式下 = 分类器绕过。

## Zig 层 DRM

API 请求中 `cch=00000` 占位符。请求离开进程前，Zig 代码（编译进 Bun runtime）将 5 个零替换为加密哈希。服务端验证此哈希确认请求来自真实 Claude Code 二进制。

**为什么是 Zig 而非 JS**：JavaScript 可在运行时 monkey-patch；编译进 Bun 的 Zig 代码无法在不重新编译整个 runtime 的情况下修改。

## Bash 23 项安全检查

`bashSecurity.ts` 包含 23 项编号检查：
- Zsh 等号展开防御（`=curl` 在 Zsh 中执行 curl）
- Unicode 零宽空格注入
- IFS 空字节注入
- 18 个被阻止的 Zsh 内建命令
- HackerOne 审查中发现的畸形 token 绕过

## 风险评估器结构化输出

```json
{
  "explanation": "What this command does (1-2 sentences)",
  "reasoning": "Why YOU are running this. Start with 'I'",
  "risk": "What could go wrong, under 15 words",
  "riskLevel": "LOW | MEDIUM | HIGH"
}
```

- `reasoning` 强制第一人称（"I"）
- `risk` 上限 15 词，强制简洁
- 上下文注入：最近 3 条 AI 消息（max 1000 chars）
- 可通过 `permissionExplainerEnabled` 配置禁用
- 工具定义使用 `tool_choice` 强制执行结构化输出

## 相关页面

- [[dive-into-claude-code]] — 论文总览
- [[claude-code-architecture]] — 源码架构分析
- [[claude-code-harness]] — Harness 编排系统
- [[claude-code-context-compaction]] — 上下文压缩系统
- [[claude-code-extensibility]] — 可扩展性机制
