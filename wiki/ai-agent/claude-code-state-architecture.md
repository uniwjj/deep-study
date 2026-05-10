---
title: Claude Code 两层状态架构
description: Claude Code 使用双轨状态——可变进程单例（基础设施状态）和最小化响应式 store（UI 状态），通过集中式副作用系统 onChangeAppState 连接
aliases: [Two-Tier State, 状态架构, AppState, bootstrap state]
tags: [ai-agent, architecture, practice]
sources: [2026/05/10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 两层状态架构

Claude Code 使用 2 层状态架构，分离驱动是**依赖约束**：基础设施模块不能 import React。

## 第 1 层：Bootstrap State — 进程单例

**位置**：`bootstrap/state.ts`

```javascript
const STATE: State = getInitialState()  // 模块作用域可变对象
```

源码注释：“DO NOT ADD MORE STATE HERE — BE JUDICIOUS WITH GLOBAL STATE”

~80 个字段分类：

| 类别 | 字段 | 说明 |
|------|------|------|
| 身份/路径 | `originalCwd`, `projectRoot`, `sessionId` | `originalCwd` 启动时通过 `realpathSync` + NFC 归一化，永不改变 |
| 成本/指标 | `totalCostUSD`, `totalAPIDuration`, `totalLinesAdded` | 单调累加，退出时持久化到磁盘 |
| 遥测 | `meter`, `sessionCounter`, `tokenCounter` | 全 nullable（遥测初始化前为 null） |
| 模型配置 | `mainLoopModelOverride`, `initialMainLoopModel` | 用户中途切换模型时更新 |
| 会话标志 | `isInteractive`, `kairosActive`, `sessionTrustAccepted` | 会话级状态 |
| 缓存优化 | `promptCache1hAllowlist`, `systemPromptSectionCache` | 粘性状态防止缓存失效 |

**Getter/Setter 模式**：`STATE` 从不直接导出。~100 个独立的 `getX()`/`setX()` 函数。所有路径 setter 执行 NFC 归一化防止 macOS 上的 Unicode 不匹配。

**Signal 模式**：`createSignal` 最小化 pub/sub 原语。Bootstrap 是 DAG 叶子节点（不 import 任何东西），`sessionSwitched` signal 让调用者订阅而 bootstrap 不知道他们是谁。

### 五个粘性锁存器（Sticky Latches）

三态类型 `boolean | null`（null = 未评估，true = 已锁定）：

| 锁存器 | 用途 |
|--------|------|
| `afkModeHeaderLatched` | AFK beta header 防止切换破坏缓存 |
| `fastModeHeaderLatched` | Fast mode 进/出 cooldown 切换 |
| `cacheEditingHeaderLatched` | 远程 feature flag 变更防止破坏缓存 |
| `thinkingClearLatched` | 空闲 >1h 后防止重新启用 thinking block 破坏缓存 |
| `pendingPostCompaction` | 区分 compaction 触发的缓存失效和 TTL 过期 |

## 第 2 层：AppState — 响应式 Store

**位置**：`state/store.ts`

**实现**：**34 行代码** — 闭包包裹可变的 `current`、`Set` 订阅者、`Object.is` 相等检查、同步监听器通知、`onChange` 副作用回调。

关键设计：
- 仅支持 updater 函数模式 `setState((prev) => next)` ——禁止 `setState(newValue)`，消除状态过期 bug
- `Object.is` 守卫：updater 返回相同引用即 no-op
- `onChange` 在监听器**之前**触发：同步接收旧+新状态
- React 集成：`useSyncExternalStore` + selector 模式

**AppState 类型**：~452 行，包裹在 `DeepImmutable<>` 中，明确排除含函数类型的字段。

## 两层通信

```
Bootstrap State  →  AppState
     │                   │
     │  getDefaultAppState()  读取磁盘设置、feature flags
     │                   │
     │              onChangeAppState 副作用:
     │  ← setMainLoopModelOverride()
     │  ← 清除凭证缓存
     └── 永不共享引用（ESLint 强制执行）
```

## 成本追踪

- 每次 API 响应 → `addToTotalSessionCost()` → 累加 per-model 使用量 → 更新 bootstrap state → 上报 OpenTelemetry
- 进程重启后通过项目配置文件恢复
- Session ID 守卫：仅当持久化的 session ID 匹配时才恢复
- 直方图使用 **reservoir sampling (Algorithm R)** — 1024 条目 reservoir 产生 p50/p95/p99

## 架构对比

| 属性 | Bootstrap State | AppState |
|------|----------------|----------|
| 位置 | 模块作用域单例 | React context |
| 可变性 | 通过 setter 可变 | 通过 updater 的不可变快照 |
| 订阅者 | Signal (pub/sub) | `useSyncExternalStore` |
| 可用时机 | import 时（React 之前） | Provider 挂载后 |
| 依赖 | DAG 叶子（不 import 任何东西） | 从整个代码库 import 类型 |

## 相关页面

- [[claude-code-bootstrap-pipeline]] — 启动管道（状态初始化时机）
- [[claude-code-architecture]] — 六抽象整体架构
- [[claude-code-agent-loop]] — 查询循环（状态消费者）
- [[claude-code-api-layer]] — API 层（Prompt cache 锁存器的使用者）
