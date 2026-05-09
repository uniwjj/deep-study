---
title: Google Scion
description: Google 开源多 Agent 沙箱项目——每个 AI 助手跑在独立容器中，支持并行代码审计/测试/文档生成，30 分钟缩至 10 分钟
aliases: [Scion, Google AI虚拟机]
tags: [ai-agent, tool]
sources: [2026/04/11/Google悄悄开源Scion.md]
created: 2026-05-09
updated: 2026-05-09
---

# Google Scion

## 核心能力

每个 AI 助手跑在独立容器中互不干扰，支持多 Agent 并行执行任务。

## 已适配

Claude Code、Gemini CLI、Codex、OpenCode。

## 示例

多 Agent 解谜游戏《Athenaeum遗迹》：AI 分饰侦探/工程师/分析师，共享工作区+消息协作，全程自动化。

## 局限

早期项目，需要会 Docker 和命令行；Hub 多机器协作 ~80% 可用；K8s 尚有兼容性问题。

## 趋势

> "单 Agent 的天花板正在被多 Agent 的编排层打穿。"

## 相关页面

- [[claude-code-vs-opencode]] — 框架对比
- [[agent-multi-agent-collaboration]] — 多 Agent 协作
- [[claude-managed-agents]] — 托管 Agent 基础设施
