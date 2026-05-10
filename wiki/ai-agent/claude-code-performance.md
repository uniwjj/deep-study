---
title: Claude Code 性能优化
description: Claude Code 的性能优化体系——启动I/O并行化、输出槽位预留、令牌效率、位图预过滤器、推测工具执行、渲染自适应节流
aliases: [Performance, 性能优化, 令牌效率, 位图预过滤]
tags: [ai-agent, architecture, practice]
sources: [2026/05/10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 性能优化

Agent 系统的性能优化涵盖 5 个不同问题领域：启动延迟、令牌效率、API 成本、渲染吞吐量、搜索速度。所有优化由 50+ 启动 tracepoint 的数据驱动，内部 100% 采集、外部 0.5% 采集。

## 一、启动 I/O 并行化

| 技术 | 说明 |
|------|------|
| 模块级子进程 | `main.tsx` 在 import 评估期间启动 MDM + Keychain 子进程 |
| API 预连接 | 初始化期间发送 HEAD 请求，使 TCP+TLS 握手与用户输入重叠 |
| 快速路径分发 | `claude mcp` 不加载 React REPL，`claude daemon` 不加载工具系统 |
| Dynamic import | OpenTelemetry（~400KB+700KB gRPC）、事件日志等按需加载 |

## 二、输出槽位预留

**默认上限：8,000 tokens**（非 32K 或 64K）

- p99 输出为 4,911 tokens——SDK 默认预留多 8-16 倍
- 命中上限时（<1% 请求）获得干净 64K 重试
- 在 200K 窗口下提升可用上下文 12-28%

## 三、工具结果预算

| 限制项 | 值 |
|--------|-----|
| 每工具字符数 | 50,000（超出持久化到磁盘） |
| 每工具令牌数 | 100,000（约 400KB 文本上限） |
| 每消息聚合 | 200,000 字符（防止 N 个并行工具单轮超预算） |

## 四、Prompt Cache 体系

- 稳定部分在前，变化部分在后
- 全局缓存范围（同版本用户共享前缀缓存）
- 存在 MCP 工具时禁用全局缓存
- 5 个粘性锁存器防止 mid-session 缓存破坏
- `getSessionStartDate = memoize(getLocalISODate)` — 防止午夜日期变更破坏缓存
- `DANGEROUS_uncachedSystemPromptSection` 命名规范强制文档化

## 五、搜索三级优化

### 1. 位图预过滤器
每个路径 26 位位图表示包含哪些小写字母。`(charBits[i] & needleBitmap) !== needleBitmap` 是一次整数比较。含稀有字母的查询拒绝率达 90%+。成本：每路径 4 字节（~1MB/270K 路径）。

### 2. 分数上界拒绝
最优分数无法击败当前 top-K 阈值则跳过。

### 3. 融合扫描
使用 `String.indexOf()` ——在 JSC（Bun）和 V8（Node）均支持 SIMD 加速。

### 4. 异步索引
`loadFromFileListAsync()` 每 ~4ms 让出事件循环。返回 `queryable`（5-10ms 可搜索）和 `done`（完整索引）。让出检测使用 `(i & 0xff) === 0xff` 分摊 `performance.now()` 开销。

## 六、推测性工具执行

`StreamingToolExecutor` 在响应流式期间执行工具。`partitionToolCalls()` 将安全工具分批并发。结果始终按原始顺序输出。Bash 工具出错时同级 abort 控制器终止并行子进程。

## 七、渲染自适应节流

- 终端模糊时降至 30fps
- 滚动刷新帧以四分之一间隔实现最大滚动速度
- React 编译器自动记忆化
- `Object.freeze()` 消除 alt-screen 模式渲染路径分配

## 八、原始流式 API

使用原始流式 API 而非 SDK `BetaMessageStream`（对每个 `input_json_delta` 调用 `partialParse()` 导致工具输入 O(n²)）。90 秒空闲看门狗，故障时回退非流式调用。

## 相关页面

- [[claude-code-bootstrap-pipeline]] — 启动管道性能预算
- [[claude-code-terminal-ui]] — 终端渲染优化
- [[claude-code-concurrent-tools]] — 推测工具执行
- [[claude-code-api-layer]] — Prompt cache 体系
- [[claude-code-input-system]] — 输入优化
