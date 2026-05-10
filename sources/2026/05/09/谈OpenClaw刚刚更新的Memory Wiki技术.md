---
ingested: 2026-05-09
wiki_pages: [ai-agent/openclaw]
---

# 谈OpenClaw的Memory Wiki

**来源**：智能生活观测员  
**时间**：2026-04-10  
**链接**：https://www.toutiao.com/article/7626983954985304585

## 一段话总结

智能体表现不佳的关键因素在于记忆系统搭建，作者梳理了智能体记忆架构的演进路线：从RAG时代到Markdown时代，再到分类总结模式，最终OpenClaw最新版本将"做梦机制"和LLM Wiki两条思路合并，正式发布Memory Wiki方案。

## 作者判断

作者从实际团队搭建智能体的经验出发，认为记忆系统是智能体表现的核心瓶颈。对OpenClaw Memory Wiki持肯定态度，认为它融合了两个最值得参考的方向——Claude Code的"做梦模式"（后台整理沉淀记忆）和Andrej Karpathy的LLM Wiki架构。

## 核心内容

**智能体记忆架构演进四阶段**：

1. **RAG时代**（去年）：主要用RAG做智能体记忆检索
2. **Markdown时代**（春节前后）：OpenClaw出来后，大家意识到Markdown文件是对AI更友好的记忆载体——简单、可读、可编辑，符合Agent工作方式
3. **分类和总结模式**：对Markdown文件分类管理，原始文档和总结分开存放
   - Claude Code源码泄露的"做梦模式"：让智能体在后台整理资料、沉淀记忆
   - Andrej Karpathy提到的LLM Wiki架构（已开源）
4. **Memory Wiki**（最新）：OpenClaw将"做梦机制"和LLM Wiki合并，正式发布Memory Wiki方案

**参考项目**：Andrej Karpathy的LLM Wiki方案已开源，适合搭Agent的开发者参考。

