---
title: Claude Code 并发工具执行
description: Claude Code 的并发工具执行机制——按安全性分区的两阶段系统、流式推测执行、结果顺序保持、最多10个并行工具的批量管理
aliases: [Concurrent Tool Execution, 工具并发, StreamingToolExecutor, partitionToolCalls]
tags: [ai-agent, architecture, practice]
sources: [2026/05/10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 并发工具执行

工具并发是 **per-invocation** 的，不是 per-tool-type。两层系统——批量编排（分区 + 执行）和推测执行（流式期间启动）——消除大部分墙钟等待时间。

核心洞察：`Bash("ls -la")` 是可并行的，`Bash("rm -rf build/")` 不是。

## 分区算法

`partitionToolCalls()` 从左到右遍历数组，生成每组要么“全部并发安全”要么“单个串行工具”的批次。

```
[Read, Read, Grep, Edit, Read, Read]
     ↓
[Read, Read, Grep] 并发
[Edit]              串行
[Read, Read]        并发
```

每个工具调用的决策：
1. 查找工具定义
2. Zod `safeParse()` 解析输入（失败 → 保守地视为不并发安全）
3. 调用 `isConcurrencySafe(parsedInput)` ——per-input 分类
4. 如果安全且最近的批次也是安全的 → 追加。否则 → 开始新批次

## 并发批量执行

`runToolsConcurrently()` — `boundedAll()` 限制活跃 generator 数量为 `MAX_CONCURRENCY`（默认：10）。

**上下文修饰器排队**：工具并发运行时，修饰器（transform `ToolUseContext` 的函数）被收集并在整个批次完成后应用，按**工具顺序**（非完成顺序）——保证确定性的上下文演进。

## 推测执行：StreamingToolExecutor

模型流式输出时，每个完整解析的 `tool_use` 块立即交给执行器。到响应结束时，多个工具可能已经完成。

四种工具状态：
- `queued` — 已解析注册，等待并发条件
- `executing` — `call()` 运行中
- `completed` — 执行完成，结果就绪
- `yielded` — 结果已发出，终结

准入检查：工具开始执行 iff：
- 当前没有正在执行的工具，OR
- 新工具和所有当前执行中的工具都是并发安全的

**顺序保持**：结果按工具被接收的顺序（a-b-c）yield，不是按完成顺序。模型按此顺序发出——对话历史必须与模型预期匹配。

## 工具并发属性表

| 工具 | 并发安全 | 条件 |
|------|---------|------|
| Read | 始终 | 纯读 |
| Grep | 始终 | 纯读 |
| Glob | 始终 | 纯读 |
| Fetch | 始终 | HTTP GET，无本地副作用 |
| WebSearch | 始终 | API 调用搜索提供商 |
| Bash | 有时 | 仅只读命令 |
| Edit | 从不 | 修改文件，两个并发编辑会损坏 |
| Write | 从不 | 创建/覆盖文件 |

## 设计模式

- **按安全性分区，不按类型** — `isConcurrencySafe(input)` 接收解析后的输入
- **在 I/O 等待期间推测执行** — API 响应还在到达时就开始工具
- **保持提交顺序** — 缓冲已完成结果，按请求顺序释放

## 性能数据

- 典型回合：3-5 次工具调用
- 串行 @ 200ms 每次：~1,000ms
- 并行：~200ms（5:1 改善）
- 默认最大并发：10
- 多工具响应流式：2-3 秒完整到达，第一个工具调用解析在 ~500ms 后

## 相关页面

- [[claude-code-tool-system]] — 完整工具执行管道
- [[claude-code-agent-loop]] — 查询循环
- [[claude-code-api-layer]] — 流式 API 处理
