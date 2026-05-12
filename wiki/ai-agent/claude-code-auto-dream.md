---
title: Claude Code 自动梦境（Auto Dream）
description: Claude Code 记忆整合机制——四阶段流程解决 AI Agent 记忆衰减，类比人类 REM 睡眠的自动记忆维护
aliases: [Auto Dream, 自动梦境, Claude Code记忆机制, REM睡眠记忆, CC, Claude CLI]
tags: [ai-agent, tool, concept]
sources: [2026/03/29/claude-code-auto-dream-guide.md]
created: 2026-05-09
updated: 2026-05-09
---

# Claude Code 自动梦境（Auto Dream）

## 核心问题

Auto Memory 在 20+ 会话后出现记忆衰减：
- 冲突条目堆积，新旧信息矛盾
- 相对日期（"昨天"）失去意义
- 过时解决方案引用已删除的文件
- 信号噪声比持续下降

## 命名隐喻：REM 睡眠

| Auto Memory（白天大脑） | Auto Dream（REM 睡眠） |
|------------------------|---------------------|
| 工作时做笔记 | 回放当天事件 |
| 新增条目不断堆积 | 强化重要连接，丢弃无用信息 |
| 不清理、不整合 | 组织成长期记忆 |

> "没有 Auto Dream 的 Claude Code 本质上是睡眠不足的。"

## 四阶段流程

```
定向(Orientation) → 收集信号(Gather Signal) → 整合(Consolidation) → 修剪索引(Prune & Index)
```

### 阶段 1：定向
读取 MEMORY.md，扫描主题文件列表，构建当前记忆状态的心理地图。
回答："我已经知道什么，它是如何组织的？"

### 阶段 2：收集信号
使用 **grep 风格搜索**（非全量阅读）扫描会话 JSONL 记录，寻找：
- 用户纠正（告诉 Claude 它错了的时刻）
- 显式保存（"记住这个" / "保存到记忆"）
- 重复主题（跨会话出现的模式）
- 重要决策（架构选择、工具选择、工作流变更）

### 阶段 3：整合（核心阶段）
- 相对日期 → 绝对日期（"昨天" → "2026-03-15"）
- 删除矛盾事实（Express → Fastify，旧条目移除）
- 修剪过时记忆（已删除文件的调试笔记）
- 合并重叠条目（同一问题 3 个会话的记录 → 1 个干净条目）

### 阶段 4：修剪和索引
- MEMORY.md 保持在 **200 行以下**（启动加载截止点）
- 移除过期指针，添加新链接
- 按相关性和新近度重新排序
- **外科手术式**：只修改需要变更的文件

## 触发条件（双重门）

必须同时满足：
1. 自上次整合 ≥ **24 小时**
2. 自上次整合 ≥ **5 次会话**

手动触发：告诉 Claude "运行自动梦境周期" 或 "整合我的记忆文件"

**性能数据**：整合 913 次会话约需 8-9 分钟，后台运行不阻塞。

## 安全约束

- **只读模式**：只能写入记忆文件，不能修改源码/配置/测试
- **锁文件**：防止两台实例并发运行
- **后台执行**：不阻塞用户会话

## 四种记忆机制全景

| 机制 | 写入者 | 目的 | 存储 |
|------|--------|------|------|
| **CLAUDE.md** | 用户 | 权威规则与指令 | `./CLAUDE.md` |
| **Auto Memory** | Claude | 会话中捕获学习 | `~/.claude/projects/<p>/memory/` |
| **Session Memory** | Claude | 会话连续性 | 同目录 session-memory/ |
| **Auto Dream** | Claude | 定期整合清理 | 同 Auto Memory |

> "Auto Memory 没有 Auto Dream 是从不整理笔记本的记录者。Auto Dream 没有 Auto Memory 没有什么可整合。两者都需要运行。"

## 最佳实践

- 日常开发：让自动梦境自动运行
- 重大重构后：手动触发梦境周期
- 偶尔审查 MEMORY.md 输出，可以手动编辑
- 结合 CLAUDE.md 使用：干净的记忆 = 更少的启动噪音 = 更多上下文窗口

## 相关页面

- [[claude-code]] — Claude Code 开发工具
- [[agent-memory-system]] — Agent 记忆系统设计
- claude-code-claudemd — CLAUDE.md 配置最佳实践
- [[google-context-engineering]] — 上下文工程
