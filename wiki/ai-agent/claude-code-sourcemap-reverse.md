---
title: Claude Code Sourcemap 逆向还原
description: 通过 npm 包 sourcemap 还原 Claude Code 1,884 个 TypeScript 源文件——七类核心问题、三层降级策略、缺失模块分析
aliases: [Claude Code源码, sourcemap逆向, Claude Code逆向工程, CC, Claude CLI]
tags: [ai-agent, tool, practice]
sources: [2026/03/31/Claude Code sourcemap还原实战.md]
created: 2026-05-09
updated: 2026-05-09
---

# Claude Code Sourcemap 逆向还原

## 还原概况

Anthropic 发布 npm 包时意外携带 `cli.js.map`，通过 Node.js source-map 库解析后还原：
- **1,884 个 TypeScript 源文件**，约 51 万行代码
- 覆盖：对话引擎、45+ 工具、权限系统、MCP 集成、多 Agent 协作、React TUI
- 整体还原率约 **95%**
- 开源仓库：https://github.com/zxdxjtu/claude-code-sourcemap

## 七类核心问题

| # | 问题 | 详情 |
|---|------|------|
| 1 | 无构建配置 | 无 package.json/tsconfig.json，需手动分析 84 个 npm 依赖 |
| 2 | 编译时 API 缺失 | Bun `bun:bundle` 的 `feature()` 函数运行时不存在 |
| 3 | 321 个 import 路径断裂 | `from 'src/services/...'` 绝对路径对不上还原后的目录结构 |
| 4 | 25 个内部源文件缺失 | 被 DCE 移除或为内部模块 |
| 5 | 8 个内部 npm 包不公开 | sandbox-runtime、chrome-mcp、computer-use-mcp 等 |
| 6 | 2 个 C++ 原生模块 | color-diff-napi、modifiers-napi 源码不在 sourcemap |
| 7 | React 版本兼容 | 必须 React 19 canary + 499 处 `useEffectEvent` 调用 |

## 三层降级策略

### 第一层：完整恢复
- `@anthropic-ai/bedrock-sdk`、`vertex-sdk`、`foundry-sdk` 源码在 sourcemap 中，直接复制。

### 第二层：运行时 Stub
- 25 个缺失源文件 → 最小空实现（`isEnabled()` 返回 false）
- 上下文压缩服务缺失 → 处理函数原样返回
- 8 个内部 npm 包 → 统一重定向到 `native-stubs.ts`，`SandboxManager.isSupportedPlatform()` 返回 false

### 第三层：Feature Flag 禁用
78 个编译时 feature flag 用运行时 `Set.has()` 替代：
- **启用**（~50 个）：FORK_SUBAGENT、MCP_TOOL、PROMPT_CACHE
- **禁用**（~28 个）：KAIROS、COORDINATOR_MODE、DAEMON、BUDDY

构建：4 个 Bun Bundler 插件 + 21.6 MB 单文件 `dist/cli.js`

## 缺失的 5%：三大退化

| 缺失模块 | 影响 |
|----------|------|
| **Context Collapse** | 旧对话无法自动折叠为摘要释放 token |
| **Snip Compact** | 历史无法自动裁剪，token 累积更快 |
| **Cached Microcompact** | API prompt cache 优化失效 |
| **沙箱隔离** | shell 命令直接在宿主机执行 |

三者叠加 → 长对话（50+ 轮）体验退化，短对话基本无感。

## React 版本：最大的坑

构建成功但屏幕空白——表现极具迷惑性：编译通过、进程在跑、什么都不显示。最终定位到 React 19 canary（`19.3.0-canary-74568e86-20260328`）才能正确渲染 Ink TUI。

## 架构洞察

还原揭示了 Claude Code 的源码结构：
- 对话引擎、45+ 工具、Ink TUI、权限系统、MCP 集成完整
- Coordinator Mode 源码完整（370 行）但被 feature flag 禁用
- 对原始源码仅 2 处修改即可运行

## 相关页面

- [[claude-code]] — Claude Code 开发工具
- [[claude-code-architecture]] — Claude Code 源码架构
- [[claude-code-auto-dream]] — 自动梦境记忆机制
- claude-code-claudemd — CLAUDE.md 配置
