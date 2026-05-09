---
title: 日志诊断 Skill
description: 得物技术实战——Skill + MCP 组合将调BUG流程（查日志→定位→修代码）闭环为一条命令，AI 自动拉取日志、还原链路、定位问题
aliases: [log-diagnosis, Skill+MCP, AI日志诊断]
tags: [ai-agent, practice, tool]
sources: [2026/04/02/日志诊断Skill.md]
created: 2026-05-09
updated: 2026-05-09
---

# 日志诊断 Skill

## 核心方案

**MCP + Skill** 组合，缺一不可：
- 只有 MCP → AI 能查日志但不知道怎么系统地分析
- 只有 Skill → AI 有流程但没有数据来源

组合后，一条命令闭合查日志→找关键信息→扫描代码→定位问题的完整链路。

## 架构

```
用户: /log-diagnosis {环境} {代码分支} {诉求}
   ↓
Claude Code + SKILL.md（行为规范）
   ↓
MCP SSE 长连接 → 日志平台（查询/分析）
   ↓
代码联动（切换到指定分支，日志关键词检索代码）
   ↓
诊断报告（飞书文档 / 本地 Markdown）
```

## Skill 关键约束

- **禁止只查第一页**：必须分页拉取全量日志（最多 20 页）
- **Token 自动刷新**：secretKey → acquireToken → 1 小时有效，过期自动续
- **代码分支切换**：自动 checkout 到目标分支检索代码
- **跨服务分析**：支持多服务日志关联

## 实战案例：隐蔽 SQL BUG

AI 仅凭一条 traceId，自动：
1. 从 traceId 第 9-16 位推算时间范围
2. 分 2 页拉取 73 条日志
3. 还原完整调用链路
4. 提取实际执行 SQL，发现 `customer_tag` 字段只判了 `IS NULL`，遗漏 `= ''` 的空字符串情况
5. 定位到 MyBatis Mapper XML 并给出修复方案

**BUG 隐蔽性**：SQL 语法正确、"看起来"没问题，只有横向对比其他字段写法才能发现。AI 没有"先入为主"的经验偏见，对所有字段同等审查，比人工更擅长发现这类横向差异。

## 适合 Skill + MCP 自动化的工作特征

- 步骤固定
- 信息来源明确
- 输出格式可预期

可延伸场景：代码审查、性能分析报告生成、告警巡检。

## 核心观点

> "AI 时代，工程师的核心竞争力不只是'能写代码'，更是'能把自己的经验和流程转化成可复用的 AI 能力'。"

## 相关页面

- [[agent-mcp-protocol]] — MCP 协议与工具集成
- [[agent-skills-system]] — Agent Skills 能力体系
- [[agent-tdd-workflow]] — Agent TDD 开发流程
- [[superpowers-framework]] — Superpowers 工程纪律框架
