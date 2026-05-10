---
title: Claude Code 记忆系统
description: Claude Code 的文件级记忆系统——四类型分类法、可推导性标准、Sonnet 侧查询召回、双层级架构、过期系统、背景提取 Agent
aliases: [Memory System, 记忆系统, MEMORY.md, Sonnet side-query]
tags: [ai-agent, architecture, concept]
sources: [2026/05/10/Claude Code from Source.md, 2026/05/10/Claude-Code-Source-Analysis.pdf]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 记忆系统

记忆系统的核心赌注：**存储极简，检索智能**——磁盘上的 Markdown 文件（零基础设施）+ LLM 驱动的相关性查询（Sonnet 侧查询）。人类可读、可编辑、可版本控制、可调试。

## 四类型分类法

记忆被严格限制为 4 种类型，由单一**可推导性标准**控制：

| 类型 | 存储内容 | 判定依据 |
|------|---------|---------|
| `user` | 角色、目标、专业水平 | 无法从代码中推导 |
| `feedback` | 关于工作方式的纠正和确认 | 瞬态人类意图 |
| `project` | 进行中的工作上下文——谁、什么、为什么、何时 | 社会/组织上下文 |
| `reference` | 外部系统指针 | 文件系统不可见的外部指针 |

**关键约束**：任何可从当前项目状态推导的内容（代码模式、架构、文件结构）一律排除，防止膨胀。

## 文件结构

```yaml
---
name: <memory name>
description: <one-line description>  ← 负载字段
type: user | feedback | project | reference
---
```

`description` 是**相关性选择器**决定 surface/not 的唯一依据。

## 双层级召回

- **Tier 1（始终加载）**：`MEMORY.md` 索引在每次会话启动时加载，硬上限 **200 行、25,000 字节**
- **Tier 2（按需）**：单独记忆文件通过 LLM 相关性查询 surface

## Sonnet 侧查询

Manifest 作为侧查询发送给 Sonnet 模型。选择器**保守**：
- 仅选对当前查询有用的记忆
- 不确定时跳过
- 避免为已在使用的工具选择记忆

**为什么是 LLM 而非关键词/嵌入**：关键词匹配缺乏上下文理解；嵌入相似度对**否定**（"不要用 X" vs "用 X"）处理不佳。Sonnet 处理语义相关性、否定、零基础设施——以延迟换精度。

## 过期系统

- 今天/昨天：无警告
- 更旧："This memory is N days old. Code behavior claims or file:line citations may be outdated."
- 选择人类可读格式因为模型不擅长日期算术（"47 days ago" 触发比 ISO 时间戳更好的推理）
- 只警告，永不过期

## KAIROS 模式：只追加每日日志

长运行会话中，模型追加到日期命名的日志文件。通过 `/dream` 整理，四阶段：Orient → Gather → Consolidate → Prune。

## 记忆提取 Agent

一个 **forked agent**（共享父的 prompt cache）分析最近消息并写出主 agent 遗漏的记忆。**协作式**：主 agent 保存时，背景 agent 延迟。

## 团队记忆

子目录功能，feature flag 后面。三层深度防御：
1. 输入净化
2. 字符串级路径验证
3. Symlink 解析

所有验证失败产生 `PathTraversalError`。**Fail-closed**。

## 设计模式

- 双层级召回（轻量始终加载 + 重量按需）
- 可推导性标准作为存储的唯一门槛
- 检索用 LLM 而非基础设施
- 长运行会话的只追加日志
- 背景提取 Agent 作为安全网
- 团队记忆 fail-closed 安全

## 相关页面

- [[claude-code-subagent]] — 记忆提取使用 forked agent
- [[claude-code-fork-subagent]] — 背景 agent 共享 prompt cache
- [[claude-code-extensibility]] — Skills 和 Hooks 扩展机制
- [[claude-code-architecture]] — 六抽象之一
- [[claude-code-auto-dream]] — 自动梦境记忆整合

## 双向互斥

```javascript
if (hasMemoryWritesSince(messages, lastMemoryMessageUuid)) {
  return  // 主 agent 已写入，跳过背景提取
}
```

如果主 agent 在本轮直接保存了记忆，背景 fork agent 提取被跳过——防止双 agent 同时修改同一记忆文件。

## 重叠请求 Stash 模式

用户快速发送连续消息且前一次提取仍在进行时：
- 新请求被**暂存**（不启动）
- 当前提取完成后处理暂存请求
- 防止提取任务堆积

## Cache 共享设计

Fork agent 必须分享与主对话**完全相同的** system prompt、工具列表和模型以复用同一 prompt cache。权限控制通过 `canUseTool` 钩子函数实现，而非修改工具列表（否则改变 cache key）。结果：缓存效率 + 安全隔离同时满足。

## "记录成功不只记录失败"

Feedback 类型记忆不只是记录用户纠正（"不要做 X"），也要记录用户确认（"对，就这样"）。只记失败会让 AI 过度保守，畏手畏脚。记录成功告诉 AI 哪些做法被批准复用。

## "用户要求记也要追问"

当用户说"记住本周 PR 列表"，AI 不应直接记录 PR 列表（临时信息）。应追问："其中哪些是非预期或非显而易见的？" 只有值得记忆的才存储。防止记忆系统被当成记事本用。

## 时间规范化

`autoDream` 整理阶段要求：相对日期必须转换为绝对日期。"下周四" → 必须存为 `2026-04-10`。否则 2 周后召回时"下周四"歧义。

## 相关页面

- [[claude-code-extensibility]] — Skills 和 Hooks 扩展机制
- [[google-context-engineering]] — Google 上下文工程白皮书
- [[claude-code-auto-dream]] — 自动梦境记忆整合
- [[claude-code-fork-subagent]] — 背景 agent 共享 prompt cache
- [[claude-code-architecture]]
