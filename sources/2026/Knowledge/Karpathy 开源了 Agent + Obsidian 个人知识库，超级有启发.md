---
ingested: 2026-05-09
wiki_pages: [ai-agent/llm-wiki-concept]
---
## Karpathy Agent + Obsidian

**文章信息**：
- **原文标题**：Karpathy 开源了 Agent + Obsidian 个人知识库， 超级有启发。
- **来源**：https://mp.weixin.qq.com/s/FG2gEdgZwpMkA3ApQMksnw
- **作者**：逛逛GitHub
- **发表时间**：未知

---

**一段话总结**：

Andrej Karpathy 提出用 LLM 持续维护一个结构化 Markdown Wiki 知识库，而非把 LLM 当搜索引擎用。核心区别在于：传统 RAG 每次从原始文档重新检索，答案用完即弃；Karpathy 的方案让 LLM 增量编译知识到 Wiki 中，知识像复利一样持续增长。系统分三层：原始资料（只读）、Wiki（LLM 维护）、Schema（组织规则）。三个核心操作是录入、提问和体检。中等规模下不需要向量数据库，靠索引文件就够用。

---

**作者判断**：

本文作者高度认同 Karpathy 的思路，认为其"最大的启发不是具体的技术实现，而是一种新的用 LLM 的思维方式"——把 LLM 当成不知疲倦的知识工程师，而非聊天机器人。作者自己从去年年底也开始用 LLM + Obsidian 做知识管理，已经不再用 NotebookLM 等工具。原文原话："大部分人现在用 LLM 还是把它当搜索引擎或者聊天机器人用，问一次答一次，答完就完了。Karpathy 的思路是把 LLM 当成一个不知疲倦的知识工程师。"

---

**核心内容**：

**Karpathy 的核心思路**：不要把 LLM 当搜索引擎用，让它像程序员写代码一样持续维护一个 Markdown 知识库。你负责找资料、提好问题，LLM 负责总结、交叉引用、分类整理、保持一致性。你在 Obsidian 里浏览，LLM 在后台编辑。

![Karpathy 推文截图，展示其用 LLM 管理个人知识库的思路](https://mmbiz.qpic.cn/mmbiz_jpg/M2ibDBMdECU1svE1ib7H8zy5Ssv8Sc3xpIWs84S0Z3M2Zgps5bHdwX2MVJOICjfOnlZMuSPySWhCzeHGViciaCJS50cUSrO2cHvOv1hjcMa0ZSQ/640)

Gist 地址：gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

**传统 RAG 方案的致命问题**：每次提问 LLM 都从原始文档重新检索拼接，问完答案就没了，知识从未真正沉淀。你问一个需要综合五篇文档的复杂问题，每次都得从头来。

![Karpathy 的 Gist 文档截图，展示 LLM Wiki 模式的核心概念](https://mmbiz.qpic.cn/mmbiz_png/M2ibDBMdECU03WiciabIkVq88Wghiax9DYcRp8EMft05vMV3PtZDs69ibHnzxcGxicxej4IrVoiaOM4CTcsuhSbMoesicvLFOd91ogVPDHsqia9L5ibws/640)

**Karpathy 的方案：知识复利增长**：让 LLM 增量式构建和维护一个 Wiki——结构化的、互相链接的 Markdown 文件集合。添加新资料时，LLM 提取关键信息整合进已有 Wiki：更新实体页面、修正主题摘要、标注新旧数据矛盾。交叉引用已建好，矛盾已标记，综合分析已反映所有内容。

**三层架构**：

1. **Raw Sources（原始资料层）**：论文、文章、图片、数据文件。LLM 只读不写，保持原始数据来源不变。
2. **The Wiki（知识库层）**：LLM 生成的 Markdown 文件目录，含摘要、实体页面、概念页面、对比分析、综述。完全由 LLM 拥有和维护。
3. **The Schema（规则文件）**：告诉 LLM Wiki 怎么组织、用什么约定。对 Claude Code 是 CLAUDE.md，对 Codex 是 AGENTS.md。

![Karpathy 推文截图，展示三层架构中各层的数据流动](https://mmbiz.qpic.cn/mmbiz_png/M2ibDBMdECU3tL94zs2uRGqF837ibOfImWCXI3MkuhJZIM8AMmsc51US3Mddav4L3JlgB6YzibdNMBF9fVIhgsGHyF0HO09ZibZpMaFOSVuf0xc/640)

**三个核心操作**：

1. **Ingest（录入）**：往原始资料目录丢新文件，LLM 读取、讨论要点、写摘要页、更新索引和相关页面。一个来源可能牵动 10-15 个 Wiki 页面更新。Karpathy 喜欢一个一个录入，边录边看。
2. **Query（提问）**：对着 Wiki 提问，LLM 搜索相关页面后综合回答。好的回答可回存为新页面，每次探索都在丰富知识库。
3. **Lint（体检）**：定期让 LLM 健康检查——找矛盾、过时信息、孤儿页面、缺失交叉引用，建议新研究方向。

![Query 操作演示：对着 Wiki 提问并生成回答](https://mmbiz.qpic.cn/mmbiz_png/M2ibDBMdECU11QnYjK6ParqiaSiaNWZLu2brnfntyk4icQdOGTqiaGicicGrTiaNgTiaoezG2pfPKtTibB0BMh0JKfic87xfnIR4pmMVGqoxMOibxojnHgg/640)

![Lint 操作演示：LLM 对 Wiki 做健康检查](https://mmbiz.qpic.cn/mmbiz_png/M2ibDBMdECU0AGZsjeyTAyUHgxgiaXOicaFV7tDgPhblr372XeNKKaWsibyYXevy5Y3DiavISNmicepET1ZJ7P5SbFjHPGSNOVBIMiblRCneSxLEhs/640)

**实际工作流**：一边开 Agent，一边开 Obsidian。LLM 编辑 Wiki，作者实时浏览、跟链接、看图谱。用法类比："Obsidian 是 IDE，LLM 是程序员，Wiki 是代码库。"辅助工具包括 Obsidian Web Clipper（网页转 Markdown）、index.md 做目录、log.md 做时间线日志。

![实际工作流：一边开 Agent 一边开 Obsidian，实时编辑和浏览](https://mmbiz.qpic.cn/sz_mmbiz_png/M2ibDBMdECU2BwnQhTI59ymZ7luh6XSMzTGTuhSPGDRAWm1DLmkFLO1OLqXiac6KBk9EaaFibdSs9G1e5TZOPlH6A8KS6uuH5mqhQ2pPwJ0px4/640)

**关键洞察**：中等规模（约 100 个来源、几百个页面）下，直接靠索引文件就够用，不需要向量数据库或复杂 RAG。LLM 先读索引定位页面，再深入阅读，效果很好。

**为什么管用**：人类维护 Wiki 的瓶颈是簿记负担——更新交叉引用、保持一致性，做着做着就烦了。LLM 不会厌倦，可以一次性修改 15 个文件，维护成本接近零。这跟 1945 年 Vannevar Bush 的 Memex 构想一脉相承，但 Bush 没解决"谁来维护"的问题，LLM 解决了。

**社区反响**：GitHub 上已有多个实现——Go 写的 sage-wiki 工具（支持增量编译、MCP Server）、Claude Code Skill（一行命令安装）、Thinking-Space IDE。

**上手方式**：需要两样东西——一个 Agent（Claude Code / Codex / OpenCode）和 Obsidian。最简单的方式是把 Gist 内容复制给 Agent 让它搭建。整个 Wiki 本质是 Git 仓库的 Markdown 文件，版本历史和协作为现成功能。


