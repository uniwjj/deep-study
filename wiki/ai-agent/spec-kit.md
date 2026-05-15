---
title: Spec-Kit
description: GitHub 官方规范驱动开发（SDD）框架——先写规范再写代码，通过 5 阶段流程（constitution→specify→plan→tasks→implement）防止 AI "Vibe Coding"
aliases: [SpecKit, Speckit, spec-kit, GitHub Spec-Kit, 规范驱动开发框架]
tags: [ai-agent, tool, practice]
sources: [2026/05/15/AI 编程三剑客：Spec-Kit、OpenSpec、Superpowers 深度对比与实战指南.html]
created: 2026-05-15
updated: 2026-05-15
---

# Spec-Kit

**Spec-Kit** 是 GitHub 官方在 2025 年初推出的开源工具包，核心理念是**先写规范，再写代码**（Spec-Driven Development）。

## 基本信息

| 维度 | 详情 |
|------|------|
| 维护方 | GitHub 官方 |
| Stars | 69.1k ⭐ |
| 技术栈 | Python (uv 包管理器) |
| 适用 AI | Claude Code、Copilot Agent、Cursor、Gemini CLI、Windsurf、Codex 等 20+ 工具 |
| 仓库 | [github.com/github/spec-kit](https://github.com/github/spec-kit) |

## 核心概念：类比"建筑规范手册"

Spec-Kit 解决的核心问题：**"按什么规矩干"** —— 给 AI 设定明确的建设标准，防止无约束的自由发挥。

类似建筑行业：项目宪法是总建筑规范，功能规范是楼层设计图，技术计划是施工方案。

## 5 阶段工作流

```
/speckit.constitution → /speckit.specify → /speckit.plan → /speckit.tasks → /speckit.implement
```

### 1. constitution（项目宪法）
定义全项目级别的治理原则，一次性建立，所有功能共享：
- 代码质量标准
- 测试规范
- 用户体验一致性要求
- 性能要求

产出：`.specify/memory/constitution.md`

### 2. specify（功能规范）
描述具体功能的 What 和 Why，**不涉及技术栈**：
- 用户故事
- 功能需求
- 约束条件

产出：`.specify/功能名/spec.md`

### 3. plan（技术计划）
技术实现方案，涉及 How 和 Tech：
- 技术栈选择
- 架构设计
- API 契约
- 数据模型

产出：`.specify/功能名/plan.md`

### 4. tasks（任务分解）
从 plan 中提取可执行的任务清单：
- 具体实现步骤
- 任务依赖关系

产出：`.specify/功能名/tasks.md`

### 5. implement（执行实现）
按任务清单逐步构建功能。

**可选命令**：

| 命令 | 用途 |
|------|------|
| `/speckit.clarify` | 澄清规范中不明确的地方（推荐在 plan 前使用） |
| `/speckit.analyze` | 跨工件一致性和覆盖率分析（tasks 后、implement 前） |
| `/speckit.checklist` | 生成自定义质量检查清单 |

## 安装

### 前置条件
- Python 3.11+
- [uv](https://github.com/astral-sh/uv) 包管理器
- Git
- 支持的 AI 编码助手

### 安装步骤

```bash
# 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 安装 Specify CLI
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# 验证
specify check
```

### 初始化项目

```bash
# 创建新项目
specify init my-project --ai claude

# 当前目录初始化
specify init . --ai claude
```

初始化后目录结构：

```
your-project/
├── .specify/
│   ├── memory/
│   │   └── constitution.md    # 项目宪法
│   ├── scripts/               # 内置脚本
│   ├── specs/                 # 功能规范目录
│   └── templates/             # 模板文件
└── CLAUDE.md                  # AI 助手配置
```

## 核心理念：阶段门控

Spec-Kit 采用严格的阶段流程，每个阶段必须完成才能进入下一阶段。这种设计适合：
- **新项目**从零开始建设
- **金融/医疗/合规**等需要完整审计文档的领域
- **大型团队**协作，阶段门控防止各自为战
- **复杂系统架构**，constitution.md 保证全局一致性

## 与 OpenSpec 的关键区别

两者都是规范驱动开发（SDD）工具，解决同一个问题（防止 AI "Vibe Coding"），是**竞争关系，应二选一**。

| 维度 | Spec-Kit | OpenSpec |
|------|----------|----------|
| 规范结构 | 分散式（每功能独立目录） | 统一式（单一 spec.md） |
| 阶段控制 | 严格顺序 | 灵活跳转 |
| 适合场景 | Greenfield（从零开始） | Brownfield（存量改造） |
| 迭代速度 | 较慢（完整流程） | 较快（轻量级循环） |
| 文档产出 | 丰富（多层文档） | 精简（统一文档） |

## 协同方案

Spec-Kit 与 [[superpowers-framework]] 互补：
- **Spec-Kit** 负责规范管理层（定义"实现什么"）
- **Superpowers** 负责执行方法层（保障"怎么高质量实现"，TDD + 代码审查）

详细对比参见 [[ai-programming-tools-comparison]]。

## 相关页面

- [[ai-programming-tools-comparison]] — Spec-Kit、OpenSpec、Superpowers 三剑客深度对比
- [[openspec-sdd-practice]] — OpenSpec SDD 实战（竞争对手）
- [[superpowers-framework]] — Superpowers 执行方法论（互补工具）
- [[sdd-openspec-superpowers]] — SDD 规范驱动开发
- [[progressive-spec-methodology]] — 渐进式 Spec 方法论
