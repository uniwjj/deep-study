---
title: Agent 设计范式
description: ReAct、Plan-and-Execute、Reflection 三种 Agent 设计范式的核心区别、实现原理、优劣势与选型指南
aliases: [Agent Paradigms, Agent范式, ReAct, Plan-and-Execute, Reflection, 设计范式]
tags: [ai-agent, concept, comparison]
sources: [2026/05/10/Agent范式.md]
created: 2026-05-10
updated: 2026-05-10
---

# Agent 设计范式

三种主流 Agent 设计范式的核心区别在于「决策和执行的关系」。

## 概念区分

| 概念 | 定义 | 类比 |
|------|------|------|
| **设计范式** | 顶层做事流程框架，定系统从头到尾按什么大逻辑跑 | 公司管理制度 |
| **推理模式** | 每步干活时脑子里具体怎么思考 | 员工干活方法 |

设计范式和推理模式一一对应。

## 三种范式对比

| 维度 | ReAct | Plan-and-Execute | Reflection |
|------|-------|-----------------|------------|
| **核心逻辑** | 思考→行动→观察→再思考 | 先定完整计划→按计划分步执行 | 生成→评估→改进 |
| **规划执行关系** | 边想边干，混在一起 | 完全解耦，先规后执 | 不改变原本流程，只加检查层 |
| **定位** | 基础款，最原始 | 复杂任务增强款 | 质量增强 buff，叠加使用 |
| **角色** | 外卖骑手：接单→取餐→导航→送达 | 项目经理：需求→设计→开发→测试→上线 | 做考后检查：检查→发现错→改→交卷 |
| **独立完整流程** | 是 | 是 | **否**（可叠加于前两者） |

## ReAct（单步迭代）

**循环**：`Think → Act → Observe → Think → ...`

核心特征：每一步的行动完全基于上一步的结果实时调整，没有提前定死的完整计划。

| 维度 | 说明 |
|------|------|
| 优势 | 实现最简单、灵活度最高、逻辑透明、新手入门零门槛 |
| 短板 | 长流程易跑偏、易陷入无效循环、浪费 token |
| 适用场景 | 流程不固定、需实时调整的简单/中等复杂度任务（信息搜索、客服、问答、单轮工具调用） |
| Claude Code 实现 | [[claude-code-agent-loop]] — async generator，10 终止 + 7 继续状态 |

## Plan-and-Execute（规划执行）

**流程**：`Plan → Execute Step 1 → Step 2 → ... → Aggregate`

核心特征：把 ReAct 中混在一起的「规划推理」和「执行推理」**完全解耦**——专门用一个 LLM 负责做规划，另一个负责按清单执行。

| 维度 | 说明 |
|------|------|
| 优势 | 完美解决长任务跑偏、结构清晰、执行链路可控、可并行 |
| 短板 | 灵活度不如 ReAct、实现复杂度高、token 消耗多 |
| 适用场景 | 流程长、复杂度高的任务（竞品分析报告、项目开发、行业调研、多步骤数据分析） |
| Claude Code 实现 | [[claude-code-coordinator-mode]] — Research→Synthesis→Implementation→Verification 四阶段 |

## Reflection（反思迭代）

**循环**：`Generate → Evaluate → Improve → Generate → ...`

**不是独立完整流程**，而是可叠加在 ReAct 或 Plan-and-Execute 之上的增强机制。

| 维度 | 说明 |
|------|------|
| 优势 | 大幅提升输出质量、降低幻觉和逻辑错误 |
| 短板 | 至少多 1 次 LLM 调用，token 和延迟线性增加，需设最大轮次防死循环 |
| 适用场景 | 输出质量要求极高（生产代码、商业报告、法律文书） |
| 叠加方式 | ReAct 加步骤级 Reflection；Plan-and-Execute 加任务级 Reflection |

## 选型指南

> 「够用就好，别搞过度工程化。」

```
任务简单、流程灵活 ─────────────────→ ReAct
任务复杂、怕跑偏 ───────────────────→ Plan-and-Execute
输出质量要求高 ───→ 在前两者基础上加 Reflection
```

新手入门首选 ReAct，复杂场景升级 Plan-and-Execute，高要求场景叠加 Reflection。三个全堆一起会导致系统又慢又复杂。

## 相关页面

- [[claude-code-agent-loop]] — Claude Code 的 ReAct 实现（async generator）
- [[claude-code-coordinator-mode]] — Plan-and-Execute 在 Claude Code 中的应用
- [[claude-code-internal-mechanisms]] — Verification Agent（Reflection 的工程实践）
- [[sub-agent-vs-team]] — 多 Agent 协作模式对比
