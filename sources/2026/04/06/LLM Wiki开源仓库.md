---
ingested: 2026-05-09
wiki_pages: [ai-agent/llm-wiki-implementations]
---

## 看看实现Karpathy LLM-wiki理念的开源仓库，总有一款适合你

**文章信息**：
- **原文标题**：看看实现Karpathy LLM-wiki理念的开源仓库，总有一款适合你
- **来源**：https://www.toutiao.com/article/7625637860418257449/
- **作者**：VibeCoder
- **发表时间**：2026-04-06 21:30

---

**一段话总结**：

Karpathy 在 2026 年 4 月提出 LLM Wiki 概念：让 LLM 将原始材料增量编译成结构化 Markdown 知识库，实现知识复利而非每次从头检索。核心方案是三层架构——raw/ 存原始材料（不可变）、wiki/ 由 LLM 编译生成、schema/ 用自然语言定义组织规则，即"廉价本体论"。社区迅速出现三个开源实现：sage-wiki（Go，编译器范式，搜索和知识图谱最强）、llm-wiki（Python，CLI 与 LLM 完全分离，20+ 格式输入，中文原生支持）、Thinking-Space（Electron 桌面应用，功能最全但与编译思路关系较松）。文章作者认为该构想最有价值之处在于把知识管理从"每次从头检索"推到"编译一次持续积累"，但幻觉污染和认知外包问题是真实风险。

---

**作者判断**：

作者认为 Karpathy 的编译式知识管理思路"确实解决了一个真实痛点：你跟 AI 聊了很多有价值的东西，但对话关了就没了"。同时认同 HN 用户 qaadika 的观点：深度理解领域别把所有整理工作都扔给 AI，交叉引用和索引可以交给工具，但概念关联和判断还是自己来。原文原话："工具终归是工具，用得好能省很多时间，用得不好就是在给自己挖坑。"

---

**核心内容**：

**Karpathy 的核心主张**：传统 RAG 是无状态的，每次查询都从原始文档重新检索，知识在对话结束后消失。LLM Wiki 让 LLM 将原始材料增量编译为结构化 Markdown，每次提问的答案被回写 Wiki，形成知识复利。

以下是 Karpathy 在 X 上发布的原始帖子截图：

![Karpathy 在 X 上发布的 LLM Wiki 构想帖](https://p26-sign.toutiaoimg.com/tos-cn-i-axegupay5k/f81ac95509984d8a96986fb6acaeb8f0~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776666800&x-signature=tnU7d7vbcd2%2FAKoA%2BifIaxhxwIs%3D)

**"廉价本体论"**：传统知识本体（OWL、SPARQL）需要专业本体工程师，年薪十几万美金。Karpathy 用自然语言写一份 CLAUDE.md 即可定义 Wiki 结构和规则，几个 Markdown 文件替代传统企业花一两千万美金搭建的知识图谱。

**四个核心操作**：
- **Ingest**：摄入新材料，LLM 读完后同时更新十几个 Wiki 页面
- **Query**：提问，答案回写到 Wiki
- **Lint**：审计，检查矛盾和孤立页面
- **Future**：合成 QA 生成

人类负责选材料、问好问题，LLM 负责其余所有事。

**三层架构**：

以下是三层架构示意图：

![三层架构示意图：raw/wiki/schema](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/961f2658b28140849a0aa652c41b6319~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776666800&x-signature=W2B72Bmrg8ZVB6jK51xUXJv32uU%3D)

- **raw/**：原始源材料（PDF、论文、文章），不可变，LLM 只读不写，确保可回头验证
- **wiki/**：LLM 编译生成，按概念、实体、源摘要、对比等维度组织。典型规模 100 篇文章约 40 万词，可一次性塞进 200 万 token 上下文窗口，无需向量数据库
- **schema**：自然语言配置文件（CLAUDE.md、AGENTS.md），定义 Wiki 组织规则

能跑起来的前提：上下文窗口从 2020 年 2K token 扩展到 2025 年 2M token，1000 倍增长。

**三个开源实现对比**：

**sage-wiki（编译器范式）**：Go 语言，与 Karpathy 构想对齐度最高。编译管道 5 个 Pass：diff 检测→并行生成摘要→批量提取概念去重→生成 Wiki 文章和本体关系→处理图片。支持断点续传。搜索用 BM25 + 向量余弦 + RRF 融合。知识图谱为类型化实体-关系图，8 种语义关系，支持 BFS 遍历和环检测。提供 MCP 服务器（14 个工具），单一二进制零依赖，有 Watch 模式。许可证 MIT。

**llm-wiki（CLI 与 LLM 分离）**：Python 语言，独特设计：CLI 处理所有确定性操作（解析、建索引、搜索、校验），一行 LLM 调用都没有，通过 SKILL.md 委托外部 Agent。可接任何 LLM（Claude Code、GPT、本地模型）。格式支持 20+ 种（PDF、DOCX、PPTX、HTML、图片、音频、ZIP）。中文用 jieba 分词 + CJK 停用词过滤。Clean Architecture，162 个测试，1600 行代码。许可证 MIT。

**Thinking-Space（全功能工作站）**：TypeScript + Electron 桌面应用，跨平台（Mac、Windows、iOS）。55 种以上 Agent 能力操作，多 AI 提供商，有 Excalidraw 绘图、思维导图、PDF 转换、RSS 阅读器。Live Source Mode 可热更新自身源代码。但与 Karpathy 编译思路关系较松，更像 AI 增强笔记工具。许可证 AGPL-3.0，商用需单独授权。

**选型建议**：
- 大量论文文章要处理、想要自动编译出 Wiki → sage-wiki
- 文档格式杂、含中文、想自由切换 Agent → llm-wiki
- 不习惯命令行、想要图形界面工作站 → Thinking-Space
- 百万级文档库 → 这三个都不行，还是传统 RAG

**社区争论**：
- **支持方**：认为是人机协作的正经延伸，类比 Licklider 1960 年人机共生论文。Lex Fridman 确认在用类似设置。企业家 Vamshi Reddy："每个企业都有一个 raw/ 目录，没有人编译过它，这就是产品。"
- **反对方**：核心质疑是认识论完整性（epistemic integrity）——LLM 把错误信息写进 Wiki 会污染后续所有查询且难以发现。认知外包风险：整理知识的苦活本身就是洞察产生的过程。规模限制：Karpathy 自己说甜蜜点是 100 篇文章/40 万词，超过可能撑不住。


