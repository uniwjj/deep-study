---
title: AI Agent 安全
description: AI Agent 面临的安全风险与防护策略——指令注入、权限控制、数据隔离
aliases: [ai-agent-security, AI Agent Security, Agent安全]
tags: [ai-agent, concept, draft]
sources: []
created: 2026-05-14
updated: 2026-05-14
status: draft
---

# AI Agent 安全

AI Agent 在获得工具调用与自主决策能力后，面临独特的安全挑战。

## 主要风险

- **指令注入**：恶意 prompt 劫持 Agent 行为
- **权限失控**：Agent 越权访问敏感资源
- **数据泄漏**：Agent 将内部信息暴露给外部
- **工具滥用**：Agent 对工具能力的不当使用

## 防护策略

- 最小权限原则：Agent 仅获得完成任务必需的工具与数据访问
- 人类审批节点：关键操作（部署、删除）要求人工确认
- 沙箱隔离：Agent 操作在受限环境中执行

## 相关页面

- [[openclaw]] — OpenClaw 自托管方案的安全考量
- [[claude-code-permission-system]] — Claude Code 权限系统
