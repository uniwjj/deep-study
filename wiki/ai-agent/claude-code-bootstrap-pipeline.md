---
title: Claude Code 启动管道
description: Claude Code 的5阶段启动管道——300ms预算下通过模块级I/O并行、快速路径分发、信任边界隔离实现瞬时启动
aliases: [Bootstrap Pipeline, 启动流程, bootstrap pipeline]
tags: [ai-agent, architecture, practice]
sources: [2026-05-10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 启动管道

Claude Code 的启动管道必须在 **300ms** 内完成（人类感知“瞬时”的阈值）。它使用 5 阶段管道配合激进的并行策略和早期退出快速路径来保证预算。

## 5 阶段管道

5 个文件顺序执行，每个阶段缩小需要处理的范围：

| 阶段 | 文件 | 职责 |
|------|------|------|
| Phase 0 | `cli.tsx` | 快速路径分发——检查 argv 是否需要完整启动 |
| Phase 1 | `main.tsx` | 模块级 I/O 并行——边导入边发起子进程 |
| Phase 2 | `init.ts` | 解析参数、解析配置、信任边界 |
| Phase 3 | `setup.ts` | 注册命令、agent、hooks、插件 |
| Phase 4 | `replLauncher.ts` | 分发到入口点，挂载 UI |

## 三种并行策略

### 1. 模块级子进程分发
`main.tsx` 的模块级副作用在 import 评估期间触发 I/O：

```javascript
const mdmPromise = startMDMSubprocess()
const keychainPromise = readKeychainCredentials()
```

在 JS 引擎解析约 138ms 的传递依赖期间，这两个 I/O 密集型子进程已在飞行中。

### 2. 初始化阶段的 Promise 并行
`setup.ts` 中 socket 绑定、hook 快照、命令加载、agent 定义加载全部并发执行。

### 3. 渲染后延迟预取
用户看到提示符后，才加载 git 状态、模型能力、AWS 凭证。

### 4. 动态导入延迟
12+ 处 `await import('./module.js')` 延迟模块评估。OpenTelemetry（400KB + 700KB gRPC）仅在遥测初始化时加载。React 组件仅在渲染时加载。

## 快速路径分发

`cli.tsx` 在加载全量系统前检查 ~12 个快速路径：

- `claude --version` → 动态导入 `version.js` → 打印 → 退出
- `claude --help` → 动态导入 help 处理器 → 退出
- `claude mcp list` → 只加载 MCP 命令 → 退出

原则：**通过了解意图来做更少的事**。argv 数组揭示了用户的意图——如果意图狭窄，执行路径也应该是狭窄的。

## 信任边界

信任边界**不是用户信任 Claude Code**，而是 **Claude Code 信任环境**。恶意的 `.bashrc` 可以设置 `LD_PRELOAD` 注入代码。

- **边界前**：仅安全操作（TLS 配置、主题偏好、遥测退出）
- **边界后**：读取危险环境变量（PATH、LD_PRELOAD、NODE_OPTIONS）、执行 git 命令
- 10 个独立的信任敏感操作

## 性能时间线

| 阶段 | 耗时 | 说明 |
|------|------|------|
| 快速路径检查 | ~5ms | 检查 argv，可能提前退出 |
| 模块评估 | ~138ms | Import 树 + 并行 I/O |
| Commander 解析 | ~3ms | 解析标志和子命令 |
| init() | ~14ms | 配置解析、信任边界 |
| setup() | ~35ms | 命令、agent、hooks、插件 |
| 启动 + 首渲染 | ~25ms | 选路径、挂载 React、首帧 |
| **总计** | **~240ms** | 300ms 预算下有 60ms 余量 |

## 设计模式

- **漏斗模式**：每个阶段缩小范围
- **I/O 与初始化重叠**：在模块评估时发起慢 I/O，使用时 `await`
- **尽早缩小范围**：一旦知道工作不需要就跳过
- **Memoize init()**：多入口点安全调用
- **Hook 快照安全**：启动时冻结 hook 配置，防止运行时注入攻击
- **Schema 迁移**：启动时自动运行，幂等，版本号函数，失败时继续（可用性 > 严格一致性）

## 七个启动路径

所有最终都调用 `query()`：

- 交互式 REPL
- print 模式（`--print`）
- SDK 模式
- resume（`--resume`）
- continue（`--continue`）
- pipe 模式
- headless

启动路径决定呈现方式（交互终端、单次、SDK 协议），不影响核心行为。

## 相关页面

- [[claude-code-architecture]] — 六抽象整体架构
- [[claude-code-state-architecture]] — 两层状态架构
- [[claude-code-agent-loop]] — async generator 查询循环
- [[claude-code-tool-system]] — 14 步工具执行管道
