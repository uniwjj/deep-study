---
title: OpenViking
description: 字节跳动开源的 Agent 知识操作系统——用文件系统范式替代扁平向量存储，实现 L0/L1/L2 三级上下文、目录递归检索与自我进化的 LLM Wiki 工程实践
aliases: [OpenViking, Agent 知识操作系统, 火山引擎知识库, Viking]
tags: [ai-agent, tool, concept]
sources: [2026/07/22/深度分析：OpenViking 中关于 LLM wiki 的理念的工程级实践.html]
created: 2026-07-22
updated: 2026-07-22
---

# OpenViking

OpenViking 是火山引擎（字节跳动）开源的 **Agent 知识操作系统**，把 Karpathy 的 [[llm-wiki-concept|LLM Wiki]] 理念做成了一套有文件系统、有分层索引、有递归检索、有自我进化的完整工程系统。

核心命题：**用文件系统范式替代扁平的向量存储，让 Agent 的上下文像操作文件一样被分层加载、按需检索、持续迭代。**

技术栈：Python（服务端 + AI 管线）+ Rust（底层文件系统 AGFS）+ C++（向量索引引擎），提供 FastAPI HTTP 服务。

## 从理念到工程

[[llm-wiki-concept|LLM Wiki]] 理念的核心洞察是：RAG 每次查询从零推导，什么都不积累；LLM Wiki 预编译、持续维护，知识随时间复利。

但从"用 LLM 维护 wiki"到"让 Agent 真正用上 wiki"，中间有四道工程鸿沟：

| 问题 | 表现 |
|------|------|
| **规模** | wiki 膨胀到数百页后，index.md 撑爆 context window |
| **精度** | grep + 正则只认字面命中，语义检索能力不足 |
| **效率** | Agent 每次都要读全量内容，Token 消耗失控 |
| **进化** | 知识不会自动更新、不会自动发现关联 |

OpenViking 在 LLM Wiki 范式基础上做了四件事：
1. **虚拟文件系统**：所有知识通过 `viking://` URI 寻址，有目录、权限、ACL
2. **L0/L1/L2 三级上下文**：每个目录自动生成 100 token 摘要和 2k token 概览，Agent 按需逐层展开
3. **目录递归检索**：从目录级定位到文件级精确定位，兼顾全局和局部
4. **自我进化**：会话结束后自动提取记忆、更新知识库、重新生成摘要

## L0/L1/L2：知识的三级分层

### 设计动机

类似 CPU 缓存机制：先看 L1 Cache（小而快），miss 了再看 L2 Cache（中而准），最后才访问 Main Memory（大而全）。

| 层级 | 载体 | 大小 | 作用 | 类比 |
|------|------|------|------|------|
| **L0 Abstract** | `.abstract.md` | ~100 tokens | 一句话说明"是什么" | CPU L1 Cache |
| **L1 Overview** | `.overview.md` | ~2k tokens | 核心信息 + 导航树 + 文件关系 | CPU L2 Cache |
| **L2 Detail** | 原始文件 | 不限 | 完整内容 | Main Memory |

每个目录自动维护三层内容，Agent 工作流变为：
1. 搜索 L0 → 快速判断哪些目录相关（~100 token/目录）
2. 读 L1 → 了解目录内部结构和文件关系（~2k token/目录）
3. 读 L2 → 只在确认需要时才读完整内容

**Token 消耗从全量降到按需。**

### 生成机制：自底向上的 DAG

L0/L1 不是手写的，而是 LLM 自动生成。关键在生成顺序：**自底向上**，先处理叶子文件，再逐层汇总到根目录。

OpenViking 使用 `SemanticDagExecutor`（DAG 调度器）编排整个过程：

```
                ┌─────────┐
                │  root/  │ ← 最后生成 overview
                └────┬────┘
               ┌────┴────┐
          ┌───┴──┐  ┌───┴──┐
          │docs/ │  │src/  │ ← 等子节点完成后生成 overview
          └───┬──┘  └───┬──┘
        ┌─────┼───┐ ┌───┼─────┐
        │     │   │ │   │     │
       a.md b.md c.md x.py y.py ← 最先生成 summary
```

**Step 1：文件级摘要** — 根据文件类型使用不同 prompt 模板：

| 文件类型 | Prompt 模板 | 摘要长度 | 特殊处理 |
|----------|------------|----------|---------|
| 普通文件 | `semantic.file_summary` | 50-150 词 | 直接读内容 |
| 文档文件 | `semantic.document_summary` | 60-180 词 | 关注章节结构 |
| 代码文件 | `semantic.code_summary` | 80-200 词 | 关注类/函数/依赖 |
| 大代码文件 | `semantic.code_ast_summary` | 80-200 词 | AST 提取骨架再总结 |

代码文件的优化：超过 100 行时，先用 AST 提取结构骨架（import、class、function 签名），再用 LLM 总结骨架而非全文。

**Step 2：目录级概览** — 子文件全部处理后，DAG 触发目录 overview 生成。Prompt 输入包括所有子文件的编号摘要和子目录的 abstract。输出是结构化的 L1 overview，其中第一段文字自动提取为 L0 abstract。**一次 LLM 调用产出两层结果。**

**Step 3：写入 + 向量化** — `.abstract.md` 和 `.overview.md` 作为 sidecar 写入目录，送入 `EmbeddingQueue` 向量化，每条产出对应层级的向量记录。

### 增量更新

DAG 调度器内置增量检测：
- **文件级**：内容未变 → 复用已有摘要
- **目录级**：子节点无变更 → 复用已有 overview
- **变更传播**：只有真正变化的路径才触发重新向量化

## 目录递归检索

### 问题

传统 RAG 检索是**扁平的**：所有 chunk 平等躺在向量库，查询时 top-k 返回。丢失了层级信息，且全局搜索可能命中不相关目录的零散 chunk。

### 五步检索流程

```
Step 1: 意图分析（可选）
   ↓
Step 2: 全局定位 — 在 L0/L1 向量中搜索 top-10 目录
   ↓
Step 3: 选择递归入口 — 高分目录 + 预设根目录
   ↓
Step 4: 递归搜索 — 优先队列驱动，并行展开子目录
   ↓
Step 5: 结果聚合 — 去重、分数融合、热度加权、排序
```

### 核心算法：优先队列驱动的并行递归

`HierarchicalRetriever._recursive_search` 用**最大堆**驱动目录展开顺序。每轮取最高分的 4 个目录并行展开，子节点中 L2 文件作为终端命中，L0/L1 目录重新入队等待展开。

**为什么不用 BFS/DFS？**
- BFS 平等对待所有目录，浪费算力在低分目录上
- DFS 可能在一个分支钻太深，错过全局更优解
- 优先队列保证每轮展开当前全局最高分目录 —— **best-first search 变体**

### 分数传播

```
final_score = α × child_score + (1-α) × parent_score
```

子节点的最终得分不仅取决于自身相似度，还受父目录相关性加权。直觉：一个和查询高度相关的目录下，即使某个文件向量分数不是最高，也可能比无关目录下的高分文件更值得返回。α 通过 `retrieval.score_propagation_alpha` 配置。

### 收敛检测

三重机制控制停止：
- **top-k 收敛**：连续 3 轮 top-k 不变 → 停止
- **池大小停滞**：连续 3 轮无新结果 → 停止
- **已访问去重**：visited set 保证每个目录只展开一次

实测大多数查询在 2-4 轮内收敛。

### 热度加权

最终排序融合热度分，借鉴 Hacker News 排序算法：

```
final_score = (1 - α) × semantic_score + α × hotness_score
hotness_score = sigmoid(log1p(active_count)) × exp(-ln(2) / 7天 × age_days)
```

- **频率**：sigmoid 压缩访问次数
- **新鲜度**：指数衰减，**7 天半衰期**（一周前的知识热度减半）

### Rerank 精排

THINKING 模式下，每轮递归结果经过 rerank 模型精排。Rerank 失败时自动 fallback 到原始向量分数。

## 自迭代：会话后的自动知识提取

### Session Memory 提取流程

```
会话消息
  ↓
归档摘要生成（compression summary）
  ↓
ExtractLoop（ReAct 循环，最多 3 轮）
  ├── LLM 看到：对话历史 + 现有记忆概览 + 记忆 schema
  ├── LLM 决策：用 read 工具查看更多细节？直接输出操作？
  └── 输出：upsert/delete 操作列表
  ↓
MemoryUpdater 执行操作
  ├── 写入/更新/删除 viking:// 中的记忆文件
  ├── 重新向量化
  └── 重新生成目录的 L0/L1 摘要
```

### ExtractLoop：ReAct 式记忆更新

不是简单"LLM 提取 → 写入"管线，而是有工具调用能力的 ReAct 循环。LLM 有 `read` 工具可以在循环中主动查阅现有知识，避免重复或矛盾写入。

支持的操作类型：

| 操作 | 合并策略 | 说明 |
|------|---------|------|
| upsert + patch | 增量合并 | 保留旧内容，追加/修改 |
| upsert + replace | 整体替换 | 完全覆盖 |
| upsert + immutable | 不覆盖 | 仅文件不存在时创建 |
| delete | — | 清理过时记忆 |
| link | — | 建立记忆间双向链接 |

### 闭环

记忆更新 → 文件写入 → 目录 L0/L1 重新生成 → 向量索引更新。下次查询时新知识已经"编译"好了。这就是 LLM Wiki 的工程闭环：**每次交互都在丰富知识库，知识随时间复利。**

## 与 LLM Wiki 范式的对比

| 维度 | LLM Wiki 原版 | OpenViking |
|------|--------------|------------|
| 知识表示 | Markdown + wikilink | `viking://` URI + L0/L1/L2 三层 |
| 检索方式 | index.md + grep | 目录递归检索 + rerank |
| 规模承载 | ~100 页 | 数千页（向量索引 + 分层过滤） |
| 知识更新 | 手动 ingest | 会话后自动提取 + 增量 DAG |
| 权限控制 | 无 | 多租户 + ACL + 路径级权限 |
| 可观测性 | 无 | OpenTelemetry + Prometheus + 事件总线 |
| 自进化 | 无 | ReAct 记忆提取 + RL 训练管线 |

OpenViking 本质上是 **LLM Wiki（范式二）+ GraphRAG（范式四的检索能力）+ 企业级基础设施** 的工程融合，核心哲学仍然是 Karpathy 的：**知识要被编译、沉淀下来，而不是每次从零检索。** L0/L1 就是"编译后的知识"。

## 局限

| 代价 | 表现 |
|------|------|
| LLM 调用成本高 | 每个文件都要 LLM 摘要，大规模摄入时 Token 消耗巨大 |
| 延迟 | 递归检索多轮向量搜索 + 可能 rerank，比单次向量检索慢 |
| 系统复杂度 | Python + Rust + C++ 三语言栈，AGFS + 向量数据库 + 消息队列，部署运维门槛高 |
| 摘要质量依赖 LLM | L0/L1 质量完全取决于 LLM 总结能力，弱模型导致摘要失真 |

企业场景下可接受（有运维团队和预算），个人用户仍以 [[llm-wiki-concept|LLM Wiki 原版]] 轻量方案更务实。

## 相关页面

- [[llm-wiki-concept]] — LLM Wiki 核心概念与三层架构
- [[llm-wiki-implementations]] — LLM Wiki 开源实现对比
- [[knowledge-retrieval-paradigms]] — 知识检索三代范式
- [[code-knowledge-base-pyramid]] — 企业微信代码知识库的 L1/L2/L3 分层方案（相似的分层思路）
- [[rag-vs-llm-wiki]] — RAG vs LLM Wiki 对比
- [[mem0]] — 用户记忆层
