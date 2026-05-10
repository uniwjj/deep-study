---
title: Claude Code 输入系统
description: Claude Code 的输入与交互系统——5种键盘协议、Chord和弦状态机、Vim 模式12变体区分联合类型、虚拟消息列表
aliases: [Input System, 输入系统, 键盘协议, Vim mode]
tags: [ai-agent, architecture, practice]
sources: [2026-05-10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 输入系统

终端多样性导致组合爆炸——输入系统必须从所有终端类型中产生正确的 ParsedKey 对象。设计哲学：渐进增强，优雅降级。

## 按键解析管线

分词器维护状态机，缓存部分转义序列并发出完整 token。对不完整序列使用 50ms 超时。

同一 `read()` 调用中的所有解析按键在一个 `reconciler.discreteUpdates()` 调用中处理——粘贴 100 个字符只产生一次重渲染。

## 5 种并行协议

| 协议 | 格式 | 备注 |
|------|------|------|
| CSI u (Kitty) | `ESC [ codepoint [; modifier] u` | 现代标准，无歧义 |
| xterm modifyOtherKeys | `ESC [ 27 ; modifier ; keycode ~` | Ghostty over SSH 回退 |
| Legacy sequences | `ESC O`/`ESC [` 模式 | VT100/VT220/xterm 变体 |
| SGR mouse | `ESC [ < button ; col ; row M/m` | 点击、拖拽、滚轮 |
| Bracketed paste | `ESC [200~ ... ESC [201~` | 识别粘贴内容 |

通过 XTVERSION 查询识别终端类型，可穿透 SSH 连接。

## 按键绑定系统

分离三个关注点：
- **绑定**（数据）
- **处理器**（代码）
- **上下文**（激活哪些绑定）

最后匹配的绑定胜出（用户覆盖优先于默认值）。16 个上下文：Global、Chat、Autocomplete、Confirmation、Scroll、Task、Help 等。

## Chord 和弦支持

`ctrl+x ctrl+k` 使用 1000ms 超时的状态机。`ChordInterceptor` 处理边界情况：不匹配的字符正常输入（只丢弃和弦前缀）。

## Vim 模式

12 变体区分联合类型状态机：

```typescript
type CommandState =
  | { type: 'idle' }
  | { type: 'count'; digits: string }
  | { type: 'operator'; op: Operator; count: number }
  | { type: 'operatorCount'; op: Operator; count: number; digits: string }
  | { type: 'operatorFind'; ... }
  | { type: 'operatorTextObj'; ... }
  | { type: 'find'; ... }
  | { type: 'g'; ... }
  | { type: 'operatorG'; ... }
  | { type: 'replace'; ... }
  | { type: 'indent'; dir: '>' | '<'; count: number }
```

`transition()` 函数基于当前状态类型分发，返回 TransitionResult（可选 next state + 可选 execute 闭包）。副作用被返回而不执行——调用者决定何时运行。

## 虚拟消息列表

`VirtualMessageList` 只渲染可见消息 + 缓冲区。每条消息高度缓存，搜索导航跳转句柄，粘性提示跟踪。滚动变更直接操作 DOM 节点属性，绕过 React 重协调器。

## 设计模式

- 绑定与处理器分离
- 上下文提升为一等概念
- 和弦状态作为带超时的显式状态机
- 让类型系统强制执行状态机
- 将复杂性推向边界，保持内部清晰

## 相关页面

- [[claude-code-terminal-ui]] — 终端 UI 渲染引擎
- [[claude-code-performance]] — 性能优化
- [[claude-code-architecture]] — 整体架构
