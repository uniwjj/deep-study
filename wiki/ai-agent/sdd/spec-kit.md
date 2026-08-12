---
title: Spec-Kit
description: GitHub 官方规范驱动开发（SDD）框架——先写规范再写代码，通过 7 步流程（constitution→specify→clarify→plan→analyze→tasks→implement）防止 AI "Vibe Coding"
aliases: [SpecKit, Speckit, spec-kit, GitHub Spec-Kit, 规范驱动开发框架, Specify CLI, specify-cli]
tags: [ai-agent, tool, practice]
sources: [2026/05/15/AI 编程三剑客：Spec-Kit、OpenSpec、Superpowers 深度对比与实战指南.html, 2026/08/12/从零理解 GitHub Spec Kit 开发者必看的入门指南.md]
created: 2026-05-15
updated: 2026-08-12
---

# Spec-Kit

**Spec-Kit** 是 GitHub 官方在 2025 年初推出的开源工具包（有一篇 2026-08 知乎文章称其"由微软开发"，与"GitHub 官方"并不矛盾——GitHub 隶属微软），核心理念是**先写规范，再写代码**（Spec-Driven Development，规格驱动开发）。

## 基本信息

| 维度 | 详情 |
|------|------|
| 维护方 | GitHub 官方（微软） |
| Stars | 69.1k ⭐ |
| 技术栈 | Python (uv 包管理器) |
| 适用 AI | Claude Code、Copilot Agent、Cursor、Gemini CLI、Windsurf、Codex 等 20+ 工具 |
| 仓库 | [github.com/github/spec-kit](https://github.com/github/spec-kit) |

## 核心概念：类比"建筑规范手册"

Spec-Kit 解决的核心问题：**"按什么规矩干"** —— 给 AI 设定明确的建设标准，防止无约束的自由发挥。

类似建筑行业：项目宪法是总建筑规范，功能规范是楼层设计图，技术计划是施工方案。

SDD 的精髓：**先想清楚「做什么」和「为什么做」，再动手「怎么做」**——强制回归软件工程的本源，让 AI 和开发者在同一份清晰蓝图下工作。协作模式为：人类定义「规格」和「意图」，AI 作为高效的「实现者」。

## SDD 四大关键词

1. **Intent first** — 明确要「做什么/为什么」，再谈实现细节
2. **Rich specs** — 用结构化的规格与检查清单约束 AI
3. **Multi-step refinement** — 多阶段收敛替代一次性大 Prompt
4. **Model-agnostic control** — 与多种代理协同但不绑定技术栈

## SDD 六大核心原则

1. **规格是主要产物**：代码只是规格的一种实现——扭转「代码为王」的传统观念
2. **可执行的规范**：规范必须足够精确，能够生成可工作的系统，杜绝模棱两可
3. **持续细化**：开发是持续验证和调整的过程，而不是一次性活动
4. **研究驱动的背景**：动手前通过专门的「研究代理」收集足够的技术与业务背景信息
5. **双向反馈**：生产环境的实际表现（性能指标、用户行为）反向驱动规格的演进
6. **分支探索**：支持从同一份规格探索多种技术实现路径（如追求性能、追求低成本），便于最优决策

## 7 阶段工作流

完整流程为七步（clarify 与 analyze 为可选步骤）：

```
/speckit.constitution → /speckit.specify → /speckit.clarify → /speckit.plan → /speckit.analyze → /speckit.tasks → /speckit.implement
```

官方配图将流程概括为三个阶段：**Initialize**（`specify init`）→ **Setup Project Guidelines**（constitution）→ **Create Spec and Tasks**（specify→clarify→plan→tasks→implement），参与者为 Project Owner 与 Developer 双方。

> ⚠️ **analyze 的时序在两来源间存在矛盾**：知乎文章正文把 `/speckit.analyze` 列为第 5 步（plan 后、tasks 前，"分派任务前沙盘推演"），但其配图流程图未含 analyze；而官方 `specify init` 输出与 [[ai-programming-tools-comparison]] 来源均说明 analyze 在 **tasks 后、implement 前**。以官方 init 输出为准，文章顺序存疑。

### 1. constitution（项目宪法）
定义全项目级别的治理原则，一次性建立，所有功能共享：
- 代码质量标准
- 测试规范
- 用户体验一致性要求
- 性能要求

产出：`.specify/memory/constitution.md`

实践贴士：把团队技术栈偏好、部署环境要求、第三方库选用原则都写入宪法，AI 后续规划技术方案时不会天马行空。

### 2. specify（功能规范）
描述具体功能的 What 和 Why，**不涉及技术栈**：
- 用户故事
- 功能需求
- 验收标准

产出：`specs/功能名/spec.md`（顶层目录；见下方目录结构说明）

spec.md 的典型结构（2026-08 实战示例）：用户场景（按优先级 P1/P2）、功能需求、成功标准（可衡量指标）、边界情况处理、验证结果（质量检查通过徽章）、下一步建议。

实践贴士：把自己当成产品经理，描述越具体越好。例如「用户点击完成按钮后，任务项应变为灰色，并移动到列表底部」，而不是「用户可以完成任务」。

### 3. clarify（澄清需求）（可选）
在制定技术方案前消除模糊地带：
- 让 AI 角色扮演挑剔的测试人员/产品经理，对 `spec.md` 提出问题，发现描述不清或有歧义的地方
- 产出：对 `spec.md` 的补充和修正

实践贴士：强制在编码前想清楚所有边界情况和异常流程，能有效减少后期返工。

### 4. plan（技术计划）
AI 从「产品经理」切换为「架构师」角色，基于 spec.md 和 constitution.md 设计 How 和 Tech：
- 技术栈选择
- 架构设计
- API 契约
- 数据模型

产出（`specs/功能名/` 下的一组文档）：
- `plan.md` — 实施计划
- `research.md` — 研究文档（技术选型决策与替代方案分析、性能/安全/部署策略、风险评估——体现「研究驱动的背景」原则）
- `data-model.md` — 数据模型（Prisma schema、验证规则、业务逻辑）
- `contracts/api.yaml` — API 契约（OpenAPI 规范）
- `quickstart.md` — 快速开始指南

功能目录按分支命名，如 `001-todo-manager`。

实践贴士：AI 方案不符合预期可以直接让它修改——你始终拥有最终决策权。

### 5. analyze（分析计划）（可选）
分派任务前的「沙盘推演」：
- 检查所有已生成工件（spec、plan 等）是否存在矛盾、遗漏或不一致
- 产出：分析报告，指出潜在问题

实践贴士：类似 Code Review，能在早期发现许多高成本的错误——虽非强制，但强烈建议不要跳过。

### 6. tasks（任务分解）
从 plan 中提取可执行的任务清单：
- 具体实现步骤
- 任务依赖关系

产出：`specs/功能名/tasks.md`，任务格式模板：`- [ ] T[ID] [P?] [Story?] Description`（任务编号+优先级+用户故事标识）。

实践贴士：`tasks.md` 是与 AI 协作的核心界面——可随时介入调整任务优先级，或标记某些任务为自己完成。

### 7. implement（执行实现）
按任务清单逐步构建功能，产出可运行的应用程序代码。

实践贴士：复杂任务建议一次执行一个任务，随时检查每步产出，确保一切尽在掌握。

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
specify init --here
```

`specify init` 会完成：检查依赖 → 选择 AI 助手（claude 等）→ 选择脚本类型（ps/sh）→ 下载模板（`spec-kit-template-claude-ps-v0.0.90.zip` 等）→ 解压 → 初始化 git → 完成。

⚠️ **Agent Folder Security**：init 输出会提示——代理可能在项目内的 agent 文件夹（`.claude/`）中存储凭据、auth token 等私密工件，**建议将 `.claude/`（或其中部分）加入 `.gitignore` 防止意外泄露凭据**。

初始化后目录结构（v0.0.90 claude-ps 模板，据 2026-08 实战截图）：

```
claude-todo-app/
├── .claude/          # AI 助手配置（可能含凭据，建议 gitignore）
├── .specify/
│   └── memory/
│       └── constitution.md    # 项目宪法
├── specs/            # 功能规范目录（顶层）
│   └── 001-todo-manager/      # 每功能一个编号目录
│       ├── spec.md
│       ├── plan.md
│       ├── research.md
│       ├── data-model.md
│       ├── contracts/api.yaml
│       ├── quickstart.md
│       └── tasks.md
├── scripts/          # 内置脚本
├── backend/          # 后端代码（实战示例）
├── frontend/         # 前端代码（实战示例）
├── docs/             # 文档
├── CLAUDE.md         # AI 助手配置
└── package.json
```

> 注：2026-05 的早期来源把 scripts/specs/templates 描述为集中在 `.specify/` 下；2026-08 截图显示 specs/、scripts/、docs/ 为顶层目录——疑似模板版本差异（截图基于 v0.0.90），以顶层结构为准。

## Claude Code 实战示例（任务管理器）

`specify init claude-todo-app` 并选择 Claude Code 后，启动 `claude` 即可看到 spec-kit 斜杠命令。开发流程：

1. `/speckit.constitution` — 制定以代码质量、测试标准、用户体验一致性及性能要求为核心的原则
2. `/speckit.specify` — 只谈功能：创建任务（标题+描述）、删除任务、查看任务、查看任务列表、标记完成
3. `/speckit.plan` — 示例技术栈：Node.js + Express + SQLite（ORM）、React + Vite、Jest/Supertest/React Testing Library、ESLint + Prettier
4. `/speckit.tasks` — 自动分析 plan 文件生成可执行任务列表
5. `/speckit.implement` — 按任务清单逐一实现

### 实用技巧

- **每一步都可以反复修改**：spec 不对直接让 Claude 改；技术栈想换重新跑 `/speckit.plan`
- **`/speckit.clarify` 在 plan 之前运行**：让 Claude 主动提问，把需求细节弄清楚，避免返工
- **检查 Review Checklist**：每个 spec.md 里都有 `Review Checklist`，用来检查 AI 交付物是否满足预期
- **随时中断并调整**：Claude 不会一次性跑完所有任务，过程中可随时介入修正方向

### 增强命令（可选，来自 `specify init` 输出）

| 命令 | 用途 | 建议时机 |
|------|------|----------|
| `/speckit.clarify` | 结构化提问，消除规划前的模糊风险 | plan 之前（若使用） |
| `/speckit.analyze` | 跨工件一致性与对齐报告 | tasks 之后、implement 之前 |
| `/speckit.checklist` | 生成质量检查清单，验证需求完整性/清晰度/一致性 | plan 之后 |

### 核心要点

- 不要一上来就让 Claude 写代码，先走 spec-kit 流程
- Specify 阶段不谈技术，只谈业务需求
- Plan 阶段再定技术栈，让决策有理有据
- Implement 前确保 Spec 和 Plan 都已确认无误

## 适用场景

**SDD 特别适用于：**

- 从零开始的全新项目（需要从头设计架构）
- 复杂功能开发（多子模块/组件协同）
- 团队协作项目（对齐需求，避免高沟通成本与返工）
- 需要最终落地为生产代码的快速原型
- 遗留系统现代化改造（记录设计意图，避免功能回归和不可控风险）

**可以不使用 SDD 的情况：**

- 简单且显而易见的 bug 修复
- 小范围的 UI 微调
- 真正的探索式开发（仍在摸索需求、方向未明时）

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
- [[vibops-spec-first-toolchain]] — spec-first 企业级 AI CodingAgent 工具链
