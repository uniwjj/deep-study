---
title: Anthropic Cookbook
description: Anthropic 官方 Claude API 示例库——Jupyter Notebooks 覆盖分类/RAG/工具调用/多模态，经官方测试的最佳实践集合
aliases: [Cookbook, Anthropic Cookbook, Claude食谱]
tags: [ai-agent, tool, practice]
sources: [2026/04/07/Anthropic Cookbook食谱库.md]
created: 2026-05-09
updated: 2026-05-09
---

# Anthropic Cookbook

## 解决三个痛点

1. 官方文档太散 → 每个食谱独立完整的 Notebook
2. 网上代码跑不通 → 全部经官方测试与最新 API 同步
3. prompt 写法低效 → 展示官方推荐写法

## 量化收益

- 搭建 RAG 系统：2-3 天 → 2 小时
- 工具调用：反复调试 → 直接复制示例
- 多模态：研究半天 → 5 分钟上手

## 快速上手

```
git clone https://github.com/anthropics/anthropic-cookbook.git
pip install anthropic jupyter notebook
jupyter notebook
```

## 推荐示例

| 示例 | 时间 | 内容 |
|------|------|------|
| Hello World | 5min | 基础 API 调用 |
| 工具调用 | 15min | Claude 调用计算器 |
| RAG | 30min | Pinecone 向量检索 + Claude 生成 |

## 追加资源

docs.anthropic.com / Discord 社区 / github.com/anthropics/courses

## 相关页面

- [[claude-code]] — Claude Code
- [[agent-mcp-protocol]] — MCP 协议
