---
title: Claude Code Swarm 多Agent协作
description: Claude Code Swarm 系统——对等 Teammate 协作、Mailbox 文件级通信、Auto-Resume、Coordinator 模式扩展细节
aliases: [Swarm, Teammate, Mailbox, 多Agent团队, 对等协作]
tags: [ai-agent, architecture, concept]
sources: [2026/05/10/Claude Code from Source.md, 2026/05/10/Claude-Code-Source-Analysis.pdf]

---
- [[claude-code-remote-execution]] — 远程执行（Bridge 消息路由）

## Team 销毁顺序

```
1. Kill Panes → 2. Destroy Worktrees → 3. Delete Team Dir → 4. Cleanup Tasks Dir
```

**顺序至关重要**："Kill panes first — on SIGINT the teammate processes are still running; deleting directories alone would orphan them in open tmux/iTerm2 panes."

两种清理场景：
- **正常删除**：teammates 已优雅退出 → 跳过 kill 步骤
- **异常清理**：Leader 收到 SIGINT/SIGTERM → force kill

Team 生命周期通过 `registerTeamForSessionCleanup()` / `unregisterTeamForSessionCleanup()` 管理——即使 `TeamDelete` 从未被调用也能防御资源泄漏。

## Continue vs Spawn 决策矩阵

| 场景 | 决策 | 原因 |
|------|------|------|
| Research 探索了要编辑的文件 | **Continue** | Worker 已有文件在上下文中 |
| Research 广泛但实现窄 | **Spawn fresh** | 避免探索噪音干扰精确实现 |
| 第一次实现完全错误 | **Spawn fresh** | 错误方法上下文会锚定重试 |
| 验证另一个 worker 的代码 | **Spawn fresh** | 验证者需全新视角 |

## 权限同步文件锁

```
~/.claude/teams/{teamName}/permissions/
  pending/          ← 待处理权限请求
    .lock           ← 文件级互斥
    perm-*.json
  resolved/         ← 已处理请求
    perm-*.json
```

- `.lock` 通过 `lockfile.lock()` 管理并发写入互斥
- `cleanupOldResolutions()` 清理 `resolved/` 中超过 1 小时的条目（`maxAgeMs = 3600000`）
- 新版 mailbox 方案逐步替代文件目录方案

## .worktreeinclude 文件

解决 gitignore 文件（`.env`、本地密钥等）在隔离 worktree 中不存在的问题。列出需从主仓库复制到 worktree 的文件。

## 反偷懒规则

Coordinator prompt 明确禁止：
> "Never write 'based on your findings' or 'based on the research.' These phrases delegate understanding to the worker instead of doing it yourself."

验证标准："Verification means proving the code works, not confirming it exists"——必须跑测试、检查类型错误、测边界情况，不能扫一眼代码说"看起来没问题"。
