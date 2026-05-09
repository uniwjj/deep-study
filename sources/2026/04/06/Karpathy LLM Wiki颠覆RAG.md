---
ingested: 2026-05-09
wiki_pages: [ai-agent/llm-wiki-concept]
---

## Karpathy 新发的LLM Wiki，可能彻底颠覆RAG知识库

**文章信息**：
- **原文标题**：Karpathy 新发的LLM Wiki，可能彻底颠覆RAG知识库
- **来源**：https://www.toutiao.com/article/7625520330618044968
- **作者**：王鹏LLM
- **发表时间**：2026-04-06 13:58

---

**一段话总结**：

Andrej Karpathy 于 2026 年 4 月 4 日在 GitHub Gist 发布 LLM Wiki 方案，48 小时内 star 突破 5000。核心思想是让 LLM 增量构建持久的知识 wiki，而非每次从原始文档重新检索——知识被"编译"一次后持续更新，每次提问的答案也可回填。三层架构（原始文档只读、LLM 拥有的 wiki 层、共同维护的 Schema 配置）配合三个核心操作（Ingest 摄入、Query 查询、Lint 检查），解决了知识库维护负担随规模增长而难以持续的问题。该方案在精神上延续了 Vannevar Bush 1945 年 Memex 的愿景，但通过 LLM 解决了"谁来维护"这个 80 年未解的难题。

---

**作者判断**：

作者认为这是知识管理的一次范式转移——从"检索"到"积累"，从"每次重新推导"到"持续编译"。作者明确指出：AGENTS.md、MEMORY.md、memory/ 目录本质上就是这个模式的雏形，"我们已经在做了，只是没有系统化"。工具链已经够用（Claude Code、Codex、Obsidian、MCP tools），"没有借口了，可以开始了。"

---

**核心内容**：

**RAG 的根本困境**：每次对话 LLM 都从零开始检索，知识从未沉淀。上传文件问完问题，下次再问类似问题又得重新翻。NotebookLM、ChatGPT 文件上传、大多数 RAG 系统都是这个模式。

**LLM Wiki 的核心方案**：让 LLM 增量构建和维护一个持久的 wiki。加入新来源时，LLM 不只是索引，而是读取内容、提取关键信息、整合进已有 wiki——更新实体页面、修订主题摘要、标注新旧数据矛盾。一个来源可能触发 10-15 个 wiki 页面的更新。

**三层架构**：

1. **Raw Sources（原始文档）**：文章、论文、图片、数据文件。只读，LLM 从中读取但永远不修改。Source of truth。
2. **The Wiki（LLM 生成的知识库）**：LLM 完全拥有这一层——创建、更新页面、维护交叉引用、保持一致性。你只管读，LLM 负责写。
3. **The Schema（配置文档）**：CLAUDE.md 或 AGENTS.md 之类的文件，告诉 LLM wiki 怎么组织、遵循什么约定、摄入来源时用什么工作流。你跟 LLM 共同维护的"建筑图纸"。

**三个核心操作**：

- **Ingest（摄入）**：新来源扔进 raw 目录 → LLM 读取 → 讨论关键点 → 写摘要页 → 更新 index → 更新相关实体页和概念页 → log 追加记录。Karpathy 偏好逐个摄入，保持参与。
- **Query（查询）**：LLM 搜索相关页面、阅读、综合答案。关键洞察：好的答案可以回填进 wiki 作为新页面——你的探索也像摄入的来源一样复利。
- **Lint（检查）**：定期让 LLM 体检——页面间矛盾、过时声明、孤儿页面、缺失交叉引用、可通过 web 搜索补上的数据空白。

**两个特殊文件**：

- **index.md**：内容目录，每个页面的链接 + 一句话摘要 + 元数据。Karpathy 说在中等规模（~100 个来源、几百个页面）下出奇地好用，不需要 embedding-based RAG 基础设施。
- **log.md**：append-only 时间日志，每条记录格式统一（`# [日期] 操作 | 标题`），可用 unix 工具解析。

**为什么能工作**：维护知识库最累的不是阅读或思考，而是记账——更新交叉引用、保持摘要最新、标注矛盾、在几十个页面间保持一致性。人类放弃 wiki 是因为维护负担增长得比价值快。LLM 不会无聊、不会忘记、可以一次触及 15 个文件，维护成本接近零。

![LLM Wiki 概念示意](https://p26-sign.toutiaoimg.com/tos-cn-i-axegupay5k/8bf89e894a254f40a1715a7e8ecaa3a1~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776650935&x-signature=vdBbLqxROJ%2B%2FzGhH5Opte3Az7II%3D)

**人与 LLM 的分工**：人策展来源、引导分析、问好问题、思考意义。LLM 负责其他一切。

**与 Memex 的精神连接**：该方案延续 Vannevar Bush 1945 年 Memex 的愿景——私有的、主动策划的知识存储，文档间联想路径与文档本身一样有价值。Bush 没能解决的部分是"谁来维护"，现在 LLM 解决了。

![三层架构示意](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/669a8877f34c457bbfa58f1cd1005455~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776650935&x-signature=cIRnQQMZMf%2F2e3VoVzSFeR1IbhI%3D)

**评论区有价值的延伸**：

- 已有人做成 OpenClaw skill：`npx clawhub@latest install karpathy-llm-wiki`
- 有人建议加 reflect 步骤——不只记录什么变了，还记录为什么变，决策记录应是一等公民
- @Paul-Kyle 已在用类似架构：git-versioned markdown + 17 个 MCP 工具 + 混合搜索（BM25 + vector via SQLite-vec），每个操作有 git commit，每个事实有来源追溯
- @xoai 实践经验：编译器应是 pipeline 而非 prompt；概念去重是最难的部分

**来源**：karpathy/llm-wiki.md，发布于 2026 年 4 月 4 日，作者 Andrej Karpathy
