---
title: Claude Code 源码架构
description: Claude Code 架构全景——六抽象模型、黄金路径、10大设计模式、14+可迁移架构原则
aliases: [Claude Code架构, Claude Code源码解析, six abstractions]
tags: [ai-agent, tool, architecture]
sources: [2026/04/01/Claude Code 源码架构解析.md, 2026-05-10/Claude Code from Source.md]
created: 2026-05-09
updated: 2026-05-10
---

# Claude Code 源码架构

Claude Code 是 Anthropic 的生产级 TypeScript 单体（~2000 文件），核心是一个 `while` 循环（调用模型 → 执行工具 → 重复），但 **98.4% 的代码**是围绕这个循环的操作基础设施。

## 六个核心抽象

| 抽象 | 文件 | 职责 |
|------|------|------|
| **查询循环** | `query.ts` (~1,730行) | async generator，全系统心跳。所有交互都经过它 |
| **工具系统** | `Tool.ts`, `tools.ts` | 40+ 工具，自我描述接口，14 步执行管道 |
| **任务系统** | `Task.ts`, `tasks/` | 后台工作单元，状态机 pending→running→completed/failed/killed |
| **状态** | `bootstrap/state.ts`, `state/store.ts` | 两层：进程单例（~80 字段）+ 响应式 store（34 行） |
| **记忆** | `memdir/` | 文件级记忆，LLM 侧查询召回 |
| **Hooks** | `hooks/` | 27 个生命周期事件，4 种执行类型 |

## 黄金路径：从按键到输出

```
用户输入 → REPL → query() generator
  → 模型流式输出
  → StreamingToolExecutor 在流式期间启动工具（推测执行）
  → 工具结果追加到消息历史
  → 再次调用模型（同一循环）
  → 模型不再产生工具调用时停止
```

三个关键洞察：
1. 查询循环是 generator——背压自然（消费者 pull），取消干净（`.return()`）
2. 工具执行与模型流式输出重叠——Read 调用在模型还在生成时就能完成
3. 整个循环是可重入的——没有单独的“处理工具结果”阶段

## 10 大设计模式

| # | 模式 | 说明 |
|---|------|------|
| 1 | **AsyncGenerator 作为 agent 循环** | 背压 + 取消 + 类型化终止状态 |
| 2 | **自我描述工具接口** | 每个工具声明自己的并发安全、权限、渲染 |
| 3 | **按访问模式分离状态** | 基础设施状态（单例） vs UI 状态（响应式） |
| 4 | **权限模式而非分散检查** | 7 种命名模式，统一解析链 |
| 5 | **递归 agent 架构** | 子 agent 是新实例化的相同循环 |
| 6 | **漏斗式启动** | 早退出快速路径，I/O 与初始化重叠 |
| 7 | **粘性锁存器缓存稳定** | `boolean | null` 三态锁定，防止 mid-session 缓存破坏 |
| 8 | **`DANGEROUS_` 命名规范** | 使破坏缓存的代码在 review 中可见 |
| 9 | **流式推测执行** | API 响应还在到达时就开始工具 |
| 10 | **按安全分区的并发批处理** | per-invocation 分类，保持顺序 |

## 系统架构图

```
用户 → 界面层 → Agent Loop → 权限系统 → 工具 → 执行环境
                    ↕
              状态与持久化
```

五层子系统：
```
表层（入口与渲染）→ 核心层（Agent Loop + 压缩管道）
→ 安全/操作层（权限+钩子+工具+沙箱+子代理）
→ 状态层（上下文装配+运行时+持久化+记忆+侧链）
→ 后端层（执行后端+外部资源）
```

## 可迁移的架构原则

1. **将复杂性推向边界** — 每个边界吸收混沌并输出秩序
2. **生成器循环优于回调** — 开发者拥有循环控制权
3. **文件存储优于数据库** — 透明、可审计、零基础设施
4. **工具自描述优于中央编排** — 添加工具 N+1 不需要修改现有代码
5. **Fork 代理实现缓存共享** — 成本优化本身就是能力
6. **Hook 优于插件** — 进程隔离，API contract 自 1971 年稳定

## 相关页面

- [[claude-code-bootstrap-pipeline]] — 5 阶段启动管道（300ms 预算）
- [[claude-code-state-architecture]] — 两层状态架构详细
- [[claude-code-api-layer]] — 多云提供商 + prompt cache
- [[claude-code-agent-loop]] — async generator 查询循环
- [[claude-code-tool-system]] — 14 步工具执行管道
- [[claude-code-concurrent-tools]] — 并发工具执行
- [[claude-code-memory-system]] — 文件级记忆系统
- [[claude-code-extensibility]] — Skills/Hooks 扩展机制
- [[claude-code-terminal-ui]] — 终端 UI 渲染架构
- [[claude-code-mcp]] — MCP 协议实现
- [[claude-code-performance]] — 性能优化体系
- [[dive-into-claude-code]] — MBZUAI 学术论文分析
- [[claude-code-permission-system]] — 7 种权限模式
- [[claude-code-context-compaction]] — 5 层压缩管道
- [[claude-code-subagent]] — 子代理委托
- [[claude-code-session-persistence]] — JSONL 会话持久化
- [[claude-code-sourcemap-reverse]] — Sourcemap 逆向还原
