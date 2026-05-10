---
title: Claude Code 会话持久化
description: Claude Code 的会话持久化设计——只追加JSONL转录、恢复/分叉机制、权限不跨会话恢复的安全设计
aliases: [会话持久化, session persistence, transcript, resume, fork]
tags: [ai-agent, concept, architecture]
sources: [2026/05/10/Dive into Claude Code.txt]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 会话持久化

Claude Code 采用**只追加持久状态**原则，转录存储为可审计、可版本控制的 JSONL 文件。

## 三种持久化通道

| 通道 | 存储位置 | 内容 |
|------|---------|------|
| **会话转录** | 项目目录 `{sessionId}.jsonl` | 对话记录、工具结果、压缩标记 |
| **全局提示历史** | `~/.claude/history.jsonl` | 仅用户提示，支持上箭头和 ctrl+r |
| **子代理侧链** | 独立 `.jsonl` + `.meta.json` | 每个子代理的对话记录 |

## 会话转录模型

- **只追加 JSONL** 格式，清晰的例外是清理重写
- 路径计算：`join(projectDir, ${getSessionId()}.jsonl)`
- 存储多种事件：压缩标记、文件历史快照、归因快照、内容替换记录
- 每条事件**人类可读且可版本控制**

## 压缩与持久化的协作

压缩边界标记 `compact_boundary` 记录 `headUuid`、`anchorUuid`、`tailUuid`：
- 保留消息在磁盘上保持原始 `parentUuid` 
- 加载器使用边界元数据在读时正确链接
- **压缩从不修改或删除**之前写入的转录行

## Resume 和 Fork

- `--resume` 通过重放转录重建对话（`conversationRecovery.ts`）
- Fork 从现有会话创建新会话（`commands/branch/branch.ts`）
- 文件历史快照支持 `--rewind-files`

## 权限不恢复（安全关键设计）

Resume 和 Fork **不恢复会话级权限**：
- 用户必须在新会话中重新授予权限
- 会话被视作**独立信任域**
- 便利性换取安全性：信任始终在当前会话中建立

## Checkpoints

`~/.claude/file-history/<sessionId>/` 是**文件级快照**用于 `--rewind-files`，并非通用快照存储。

## 设计权衡

- 数据库后端的替代方案可实现更丰富的跨会话查询（"显示所有修改文件 X 的工具调用"），但会引入部署依赖并降低透明性
- 当前的只追加 JSONL 格式优先考虑**可审计性和简单性**而非查询能力

## 相关页面

- [[dive-into-claude-code]] — 论文总览
- [[claude-code-permission-system]] — 权限系统（Resume 不恢复权限）
- [[claude-code-context-compaction]] — 上下文压缩（压缩与持久化紧密耦合）
- [[claude-code-subagent]] — 子代理侧链转录
