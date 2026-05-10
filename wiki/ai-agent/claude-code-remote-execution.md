---
title: Claude Code 远程执行
description: Claude Code 的远程控制与云端执行系统——Bridge v1/v2、Direct Connect、上游代理凭证注入、非对称读写通道、FlushGate排序
aliases: [Remote Execution, 远程执行, Bridge, CCR]
tags: [ai-agent, architecture, practice]
sources: [2026-05-10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 远程执行

当控制来自浏览器、agent 在云端容器中运行或作为服务暴露时，本地执行假设不再成立。四个系统解决不同拓扑，共享设计哲学：读写不对称、自动重连、优雅降级。

## Bridge v1（轮询、分发、生成进程）

- 通过 Environments API 注册，长轮询工作项
- 预检查清单：feature flag、OAuth 令牌验证、组织策略检查、死令牌检测（同一过期令牌连续失败 3 次后跨进程退避）、主动令牌刷新
- 每个会话以 NDJSON 通过 stdin/stdout 生成子 Claude Code 进程
- 权限请求通过 bridge 传输在网页界面审批/拒绝（10-14 秒往返时限）

## Bridge v2（直接会话 + SSE）

- 完全消除 Environments API 层
- 三步骤生命周期：创建会话 → 连接 bridge（返回 worker_jwt、api_base_url、worker_epoch）→ 打开传输（SSE 读取，CCRClient 写入）
- 每次 `/bridge` 调用递增 epoch——它本身就是注册
- `ReplBridgeTransport` 统一 v1 和 v2 接口

## FlushGate 排序

解决排序问题——历史刷新期间实时写入入队并通过 flushGate 保护。刷新 POST 完成后按序排出，防止消息乱序。

## 令牌刷新与 epoch 管理

主动刷新 worker JWT。epoch 不匹配（409 响应）时两个连接均关闭并抛出异常，防止脑裂。

## 消息路由与去重

`handleIngressMessage()` 中央路由器。UUID 去重：`recentPostedUUIDs`（echo 防护）+ `recentInboundUUIDs`（重传防护）。

`BoundedUUIDSet`：FIFO 有界集合，环形缓冲区 O(1) 查找，容量 2000。两个实例并行运行，无计时器、无 TTL。

## 非对称设计：持久读取，HTTP POST 写入

读取是高频服务器发起的流（每秒数百条消息），写入是低频客户端发起的 RPC（每分钟级），需要确认。

统一 WebSocket 会产生耦合——掉线时需要区分“未发送”与“已发送但确认丢失”。

## 远程会话管理

| 错误 | 策略 |
|------|------|
| 4003（未授权） | 立即停止，不重试 |
| 4001（会话未找到） | 最多 3 次，线性退避（压缩期间瞬时错误） |
| 其他瞬时错误 | 指数退避，最多 5 次 |

## Direct Connect：本地服务器

最简单拓扑——Claude Code 作为服务器运行，客户端通过 WebSocket 连接。五种会话状态（starting、running、detached、stopping、stopped），元数据持久化到 `~/.claude/server-sessions.json`。`cc://` URL 方案。

## 上游代理（CCR 容器凭证注入）

6 步序列：
1. 从 `/run/ccr/session_token` 读取令牌
2. 通过 Bun FFI 设置 `prctl(PR_SET_DUMPABLE, 0)` 阻止同 UID ptrace
3. 下载 CA 证书并拼接
4. 启动本地 CONNECT 到 WebSocket 中继
5. 删除令牌文件
6. 导出环境变量

每一步失败时**开放**——错误禁用代理但不终止会话。手动编码 Protobuf `UpstreamProxyChunk`（10 行替代完整 protobuf 运行时）避免供应链风险。

## 设计模式

- 分离读写通道（高频流 vs 低频 RPC）
- `BoundedUUIDSet` 限定去重内存
- 重连策略与错误信号成正比
- 对抗环境仅堆上保存密钥
- 辅助系统故障时开放

## 相关页面

- [[claude-code-mcp]] — MCP 协议
- [[claude-code-agent-loop]] — 查询循环（远程会话入口）
- [[claude-code-subagent]] — 远程 agent task 类型
