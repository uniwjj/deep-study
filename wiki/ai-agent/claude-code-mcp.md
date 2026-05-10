---
title: Claude Code MCP 协议实现
description: Claude Code 的 MCP 协议实现——8种传输、7个配置来源、4阶段工具包装、OAuth发现链、进程内传输、分层超时架构
aliases: [MCP, Model Context Protocol, MCP协议]
tags: [ai-agent, architecture, practice]
sources: [2026-05-10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code MCP 协议实现

MCP 定义了基于 JSON-RPC 2.0 的工具发现和调用协议。`tools/list` 发现功能，`tools/call` 执行功能。其余所有——传输选择、认证、配置加载、工具名标准化——是将规范投入生产的工程工作。

## 8 种传输类型

| 传输 | 特征 | 适用 |
|------|------|------|
| **stdio** | 本地子进程，默认值 | 本地工具 |
| SSE | 旧版遗留 | 向后兼容 |
| HTTP (Streamable) | 可流式 | 远程服务 |
| WebSocket | 双向 | 实时服务 |
| SDK | 编程式 | SDK 集成 |
| 两个 IDE 变体 | IDE 特定 | VS Code 等 |
| claudeai-proxy | 代理模式 | Claude.ai 桥 |

省略 type 时默认 stdio——向后兼容。本地工具用 stdio，远程服务用 HTTP（Streamable），SSE 是旧版遗留。

## 配置加载：7 个来源

按签名去重（非按名称）：

| 来源 | 位置 | 信任级别 |
|------|------|---------|
| local | 工作目录 `.mcp.json` | 需用户批准 |
| user | `~/.claude.json` 的 `mcpServers` 字段 | 用户管理 |
| project | 项目级配置 | 共享项目设置 |
| enterprise | 托管企业配置 | 组织预批准 |
| managed | 插件提供 | 自动发现 |
| claudeai | Claude.ai 网页界面 | 通过网页预授权 |
| dynamic | 运行时注入（SDK） | 编程式添加 |

`getMcpServerSignature()` 生成规范键：`stdio:["command","arg1"]` 用于本地，`url:https://...` 用于远程。

## 4 阶段工具包装

1. `normalizeNameForMCP()` — 替换非法字符，全限定名 = `mcp__{serverName}__{toolName}`
2. 描述截断至 2048 字符（OpenAPI 生成服务器曾将 15-60KB 输出到 tool.description）
3. inputSchema 加密透传
4. 注解映射：`readOnlyHint` → 并发安全，`destructiveHint` → 额外权限审查

## OAuth 完整 RFC 发现链

回退链：RFC 8414 → RFC 6749；`authServerMetadataUrl` 兜底方案。

跨应用访问（XAA）：`oauth.xaa: true` 可通过一个 IdP 进行联合令牌交换。

`normalizeOAuthErrorBody()` 处理不符合规范的服务器（如 Slack 返回 HTTP 200 但错误在 body 中）。

## 进程内传输

`InProcessTransport`（63 行）：
- `send()` 通过 `queueMicrotask()` 投递，防止同步请求/响应循环中的栈深度问题
- `close()` 级联到对端，防止半开状态
- Chrome MCP 服务器和 Computer Use MCP 服务器均使用此模式

## 连接管理

五种状态：connected、failed、needs-auth（15 分钟 TTL 缓存）、pending、disabled。

`isMcpSessionExpiredError()` 通过 HTTP 404 + 消息包含 `"code":-32001` 检测。检测后清除缓存并单次重试。

本地服务器一批连接 3 个，远程一批 20 个。

## 分层超时架构

| 层级 | 时长 | 防御 |
|------|------|------|
| Connection | 30s | 不可达或慢启动服务器 |
| Per-request | 60s（每请求独立） | 过期超时信号 bug |
| Tool call | ~27.8 小时 | 合法长时间操作 |
| Auth | 每 OAuth 请求 30s | 不可达 OAuth 服务器 |

每请求超时的修复：`wrapFetchWithTimeout()` 为每个请求创建独立超时信号。

## 设计模式

- 从 stdio 开始，后续按需增加复杂度
- 懒认证——服务器返回 401 前不尝试 OAuth
- 内置服务器使用进程内传输
- 尊重工具注解，对输出做安全处理

## 相关页面

- [[claude-code-tool-system]] — 工具系统（MCP 工具集成点）
- [[claude-code-remote-execution]] — 远程执行
- [[claude-code-extensibility]] — 扩展机制
