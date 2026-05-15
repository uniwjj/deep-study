---
title: AI 编程三剑客对比
description: Spec-Kit、OpenSpec、Superpowers 三个 AI 编程辅助工具的全面对比分析——定位差异、工作流对比、场景选择、协同方案
aliases: [AI编程三剑客, Spec-Kit vs OpenSpec vs Superpowers, AI编程工具对比]
tags: [ai-agent, comparison, concept]
sources: [2026/05/15/AI 编程三剑客：Spec-Kit、OpenSpec、Superpowers 深度对比与实战指南.html]
created: 2026-05-15
updated: 2026-05-15
---

# AI 编程三剑客对比

2024-2026 年 AI 编程工具爆发式增长，三个工具从不同角度解决了"如何让 AI 按预期工作"的问题。

## 工具定位

每个工具解决一个不同的核心问题：

| 工具 | 核心问题 | 类比 |
|------|---------|------|
| **[[spec-kit]]** | "按什么规矩干" | 建筑规范手册 |
| **[[openspec-sdd-practice]]** | "改了什么" | 施工变更单 |
| **[[superpowers-framework]]** | "怎么干" | 施工队工作手册 |

## 核心对比

| 维度 | Spec-Kit | OpenSpec | Superpowers |
|------|----------|----------|-------------|
| **维护方** | GitHub 官方 | Fission-AI | obra (社区) |
| **Stars** | 69.1k | 23.7k | 50k |
| **技术栈** | Python (uv) | TypeScript (npm) | Markdown + JS Plugin |
| **工具类型** | 🔵 规范管理（SDD） | 🔵 规范管理（SDD） | 🟢 执行方法论 |
| **核心理念** | 分阶段规范驱动 | 统一真相源 + 增量变更 | TDD + 代码审查 |
| **规范结构** | 分散式（每功能独立文件） | 统一式（单一文档） | 无规范管理 |
| **适用项目** | 新项目 / 复杂系统 | 存量项目 / 快速迭代 | 所有项目 |
| **与其他工具关系** | ⚔️ 与 OpenSpec 竞争 | ⚔️ 与 Spec-Kit 竞争 | ✅ 与两者互补 |

## ⚠️ 关键认知：二选一 + 一必选

Spec-Kit 和 OpenSpec 都是"规范驱动开发"（SDD）工具，解决同一个问题——防止 AI "Vibe Coding" 导致的实现漂移。**两者是竞争关系，应二选一**。

Superpowers 则是**执行方法论**工具，与两者互补——无论选哪个 SDD 工具，都可以（也建议）搭配 Superpowers。

## SDD 工具二选一：Spec-Kit vs OpenSpec

| 维度 | Spec-Kit | OpenSpec |
|------|----------|----------|
| 规范结构 | 分散式（每功能独立目录） | 统一式（单一 spec.md） |
| 阶段控制 | 严格顺序（必须逐阶段推进） | 灵活跳转（可随时切换） |
| 适合场景 | Greenfield（从零开始） | Brownfield（存量改造） |
| 迭代速度 | 较慢（完整流程保障质量） | 较快（轻量级快速循环） |
| 团队规模 | 大团队（需要流程规范） | 小团队/个人（灵活优先） |
| 启动成本 | 较高（需建立 constitution） | 较低（直接 /opsx:new） |
| 文档产出 | 丰富（多层文档） | 精简（单一 spec.md） |

### 选 Spec-Kit 的场景

- 🏗️ 新项目从零开始 — 分阶段流程确保基础设计不跳过
- 🏦 金融/医疗/合规项目 — 严格阶段文档满足审计要求
- 👥 大型团队协作 — 阶段门控防止各自为战
- 📋 复杂系统架构 — constitution.md 保证全局一致性
- 🎯 需要完整设计文档 — 自动生成 specify/plan/tasks 文档链

### 选 OpenSpec 的场景

- 🔧 存量项目改造 — 无需从头建立完整规范体系
- 🚀 初创公司快速迭代 — 轻量级循环不拖慢节奏
- 🧪 原型/MVP 开发 — 快速验证想法
- 👤 个人/小团队项目 — 单一 spec.md 便于维护
- 🔄 频繁需求变更 — 灵活跳转适应变化

## 协同方案

### 方案 A：Spec-Kit + Superpowers（新项目/复杂系统）
严格规范 + TDD 执行 = 企业级质量保障

### 方案 B：OpenSpec + Superpowers（存量项目/快速迭代）
灵活规范 + TDD 执行 = 敏捷高质量交付

### 仅 Superpowers（简单任务）
零配置，即插即用，TDD 保证代码质量

## 不推荐的组合

| 组合 | 原因 |
|------|------|
| Spec-Kit + OpenSpec | 功能重叠，两套规范系统会造成混乱 |
| 仅 Spec-Kit 或仅 OpenSpec | 缺少执行方法论，AI 可能"乱写代码" |
| 三者都不用 | "Vibe Coding" 模式，小项目可行，复杂项目必翻车 |

## 决策树

```
你的项目是...

├─ 🆕 新项目（Greenfield）
│   ├─ 大型/复杂系统？ → Spec-Kit + Superpowers
│   └─ 小型/简单项目？ → OpenSpec + Superpowers
│
├─ 🔧 存量项目（Brownfield）
│   └─ OpenSpec + Superpowers
│
└─ ⚡ 简单任务/一次性脚本
    └─ 仅 Superpowers
```

## 学习路径

1. **Day 1**：安装 Superpowers，零配置体验 TDD 和代码审查
2. **Week 1**：选择一个 SDD 工具，在真实项目中完整走一遍流程
3. **Month 1**：建立个人/团队工作流，配置 Claude Rules 自动化

## 核心洞察

- **Spec-Kit / OpenSpec** 解决"实现什么"（WHAT）—— 防止 AI "Vibe Coding"，提供可追溯的决策文档
- **Superpowers** 解决"怎么高质量实现"（HOW）—— 强制 TDD 确保代码可测试，代码审查防止低级错误

让 AI 成为可靠的工程伙伴，而非需要时刻看管的实习生。

## 相关页面

- [[spec-kit]] — GitHub 官方 SDD 框架
- [[openspec-sdd-practice]] — OpenSpec SDD 实战
- [[superpowers-framework]] — Superpowers 执行方法论
- [[sdd-openspec-superpowers]] — SDD 双框架原始对比
- [[superpowers-vs-openspec]] — 个体效率 vs 团队一致性哲学对决
- [[progressive-spec-methodology]] — 渐进式 Spec 方法论
