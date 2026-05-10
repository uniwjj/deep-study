---
title: Claude Code 终端 UI
description: Claude Code 的自定义终端渲染引擎——7元素DOM、7阶段渲染管线、驻留池内存管理、双缓冲、流式Markdown优化
aliases: [Terminal UI, 终端UI, 渲染架构, Ink fork]
tags: [ai-agent, architecture, practice]
sources: [2026/05/10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 终端 UI

终端不是浏览器——没有 DOM、没有 CSS、没有合成器。Claude Code 最初使用 Ink，但将其 fork 到无法辨识的程度，因为原有“每个单元格一个 JS 对象”的渲染方式在流式输出 60fps 时会因 GC 压力崩溃。

## 自定义 DOM：7 种元素类型

| 元素 | 用途 |
|------|------|
| ink-root | 文档根节点 |
| ink-box | Flexbox 容器 |
| ink-text | 带 Yoga 度量的文本节点 |
| ink-virtual-text | 嵌套样式文本 |
| ink-link | 通过 OSC 8 转义序列的超链接 |
| ink-progress | 进度指示器 |
| ink-raw-ansi | 预渲染 ANSI 内容（初始高亮后零开销） |

DOMElement 携带：`yogaNode`、`style`、`attributes`、`childNodes`、`dirty` 标志、`_eventHandlers`（分离存储，避免触发 dirty）、`scrollTop`、`stickyScroll`。`markDirty()` 沿祖先链向上传播——兄弟子树保持干净。

## 7 阶段渲染管线

1. React 提交 + Yoga 布局（解析 flexbox 树）
2. DOM 到屏幕（遍历 DOM 树，向 Screen 缓冲区写入字符）
3. 叠加层（文本选择和搜索高亮原地修改缓冲区）
4. 差异计算（逐单元格比较前一帧——每个单元格两次整数比较）
5. 优化（合并相邻补丁，消除冗余光标移动）
6. 写入（在单个 `write()` 调用中输出 ANSI 转义序列）
7. 帧计时（通过 FrameEvent 上报各阶段耗时）

## 基于池的内存管理

200×120 终端有 24,000 个单元格。60fps 下不驻留则每秒分配 576 万个对象。

### 单元格布局
连续 Int32Array 中的两个 Int32 字：
- `word0: charId`（CharPool 索引）
- `word1: styleId | hyperlinkId | width`

并行 BigInt64Array 视图使行级清除可用单个 `fill()` 调用完成。

### 三个驻留池

| 池 | 实现 | 用途 |
|----|------|------|
| CharPool | 128 入口 Int32Array ASCII 快速路径 | 字符到 ID |
| StylePool | ANSI 样式码到整数 ID | 缓存预序列化过渡字符串 |
| HyperlinkPool | OSC 8 URI | 超链接池 |

所有池在前后帧间共享——驻留 ID 跨帧有效，无需重新驻留。

## 双缓冲渲染

两个帧缓冲区（`frontFrame`、`backFrame`）在每次渲染后交换——零分配、纯指针赋值。使用 lodash throttle 进行 16ms 渲染调度。滚动操作使用独立的 4ms setTimeout，完全绕过 React。

## 流式 Markdown 优化

- 模块级 LRU 缓存（500 条目）存储 `marked.lexer()` 结果
- `hasMarkdownSyntax()` 检查前 500 个字符；纯文本直接绕过解析器
- React Suspense 延迟代码块语法高亮加载
- `StreamingMarkdown` 组件处理不完整代码围栏、部分粗体标记、截断列表

## 设计模式

- 单元格级差异化——基于压缩类型数组
- 热路径与 React 分离——滚动变更绕过重协调器
- 定期清理——5 分钟池重置周期限制增长
- 自适应节流——终端模糊时降至 30fps

## 相关页面

- [[claude-code-input-system]] — 输入与交互系统
- [[claude-code-performance]] — 性能优化体系
- [[claude-code-architecture]] — 整体架构
