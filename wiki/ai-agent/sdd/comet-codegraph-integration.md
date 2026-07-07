---
title: CodeGraph 在 Comet 中的代码智能引擎角色
description: CodeGraph 作为 MCP 代码智能工具，以 SQLite 知识图为 Comet 的 design/build/verify 阶段提供亚毫秒级代码理解，消除 agent "幻觉式"实现
aliases: [CodeGraph, codegraph, Comet CodeGraph, 代码知识图]
tags: [ai-agent, tool, concept]
sources: [2026/07/07/Comet整合OpenSpec与Superpowers详解.md]
created: 2026-07-07
updated: 2026-07-07
---

# CodeGraph 在 Comet 中的角色

CodeGraph 是独立于 Comet 的 MCP（Model Context Protocol）代码智能工具，以 SQLite 知识图的形式为工作区中每个符号、调用边和文件建立索引。在 Comet 工作流的 build 和 design 阶段作为 agent 的"理解引擎"。

## 与 Comet 的关系

CodeGraph 不是 Comet 的一部分，而是与 Comet **同层协作**的基础设施：

```
Comet 工作流层 → 门控层 → 工具层（CodeGraph + Superpowers + OpenSpec）
                              ↓
                         数据层（SQLite 知识图 + .comet.yaml + spec artifacts）
```

CodeGraph 是"预建索引"而非"即时搜索"——agent 在需要理解代码时直接查询，而非启动文件搜索子代理或 grep+read 循环。

## 生命周期

### 初始化（一次性，手动）

```bash
codegraph init     # 全量扫描项目，创建 .codegraph/，构建 SQLite 索引
codegraph status   # 查看索引状态
```

### 增量更新（全自动）

初始化后无需手动操作。MCP server 的 file watcher 持续监听文件变更，约 1 秒延迟后增量更新索引。

### 启动链

```
Claude Code 启动 → spawn CodeGraph MCP server
  → 检测 .codegraph/ 目录
    ├─ 有 → 加载 SQLite 索引 → file watcher 增量更新
    └─ 没有 → 不索引，codegraph_* 工具调用直接报错
```

## 工具矩阵（8 个 MCP 工具）

| 意图 | 工具 | 说明 |
|------|------|------|
| **探索理解** | `codegraph_explore` | **主力工具**——自然语言问答或符号集合，一次返回相关符号完整源码 |
| | `codegraph_search` | 按名称查找符号（仅返回位置） |
| | `codegraph_node` | 获取单个符号完整信息（位置/签名/调用链/源码体） |
| **调用追踪** | `codegraph_callers` | 列出调用某符号的函数 |
| | `codegraph_callees` | 列出某符号调用的函数 |
| | `codegraph_impact` | 分析修改某符号的影响范围（重构前评估） |
| **文件导航** | `codegraph_files` | 索引化文件树（比 Glob 更快） |
| **健康检查** | `codegraph_status` | 索引健康状态（调试用） |

## 在 Comet 各阶段的典型使用

### Design 阶段

Brainstorming 过程中 agent 使用 CodeGraph 理解现有架构：

- `codegraph_explore("用户认证 auth login UserService token")` → 查看现有 auth 模块所有符号源码
- `codegraph_callers("UserService.authenticate")` → 评估改动影响面
- `codegraph_impact("UserService", depth=2)` → 重构前影响分析

### Build 阶段

Implementer agent 持续使用 CodeGraph：

- `codegraph_explore("用户注册接口 register controller service dto")` → 理解现有流程，匹配代码风格
- `codegraph_search("PasswordEncoder", kind="class")` → 查找可复用工具类
- `codegraph_explore("registerUser saveToDatabase sendVerificationEmail")` → 追踪完整调用链

### Verify 阶段

验证 agent 使用 CodeGraph 确认实现完整性：

- `codegraph_files("src/main/java/com/example/auth", pattern="*.java")` → 列出模块所有文件与 delta spec 比对
- `codegraph_callers("OAuth2LoginService")` → 确认新模块被正确集成调用

## 对门控体系的补充

CodeGraph 在门控体系中承担**隐性校验**角色：

| 门控层 | CodeGraph 作用 |
|--------|---------------|
| 硬门控 | 无直接作用（门控关心"能不能写"，不是"写什么"） |
| 阶段守卫 | 无直接作用（守卫校验产物完整性和字段合规） |
| 软门控 | Rule 中隐含指导——"先理解再编码" |
| **agent 执行质量** | **核心价值**——基于准确代码理解实施方案，减少"幻觉式"实现 |

CodeGraph 解决了 LLM agent 在编码中最常见的失败模式：**对代码库的认知过时或错误**。配合 Comet 门控体系：
- 硬/软门控保证 agent **不会在错误的阶段写代码**
- CodeGraph 保证 agent **写的代码基于对现有代码的准确理解**
- 阶段守卫保证 agent **写完了该写的所有东西**

三者构成 **"流程正确 + 理解正确 + 产物完整"** 的质量铁三角。

## 相关页面

- [[comet-integration-architecture]] — Comet 整体架构
- [[comet-gating-system]] — 三层门控体系
- [[comet-workflow-phases]] — 五阶段工作流（design/build 阶段使用 CodeGraph）
- [[superpowers-framework]] — Superpowers 框架（子代理调度中同样使用代码理解）
