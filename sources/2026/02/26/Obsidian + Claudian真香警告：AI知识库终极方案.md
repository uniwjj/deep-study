## Obsidian + Claudian真香警告：AI知识库终极方案

---
ingested: 2026-05-09
wiki_pages: [ai-agent/obsidian-claudian]
---

**文章信息**：
- **原文标题**：Obsidian + Claudian真香警告：AI知识库终极方案
- **来源**：https://www.toutiao.com/article/7611186164296352275/
- **作者**：电车博士张抗抗（清华大学汽车工程系博士）
- **发表时间**：2026-02-26

---

**一段话总结**：

这是一篇面向文字工作者的 Obsidian + Claudian（Claude Code + Obsidian 插件）部署实操指南。作者认为这套方案的核心优势是"几乎所有文档编辑工作都不需要自己完成，直接在 AI 工作区发指令"。文章梳理了四大优点：文件自主权（本地存储+云同步，不被任何平台绑定）、跨模型记忆（记忆存在本地，哪个模型便宜好用就换谁）、Skills 接入（从 Prompt 到 Agent 到 Skill，像雇人干活）、知识图谱（AI 自动整理 Obsidian 的标签和双向链接）。作者花费 4 天踩坑完成部署，将 15500 条微博和 1250 篇知乎文章导入 Obsidian，并给出了 10 步安装配置指南（含避坑提示）。

---

**作者判断**：

作者十分推荐所有文字工作者尽早部署。原文原话："Prompt 方式还是自己干活，只不过 AI 加快了效率；Skill 方式更像是雇人来干活了，谁的 Skill 好就用谁的。" 作者认为"对文字工作者来说，Obsidian + Claudian 可能是最佳实践方式"，并强调"现在 AI 可以帮助我们思考，但还无法帮助我们在物理世界中体验"——真实经历加入文章才会显得真实且不雷同。作者花了 4 天踩坑完成，写下指南是为了帮读者 1 天搞定。

---

**核心内容**：

**系统概述**：左侧为文档库（类似本地飞书云文档），中间为文档编辑区，右侧为 AI 工作区。核心能力：在 AI 工作区发指令即可完成选题、头脑风暴、边写边改、去 AI 味等全流程，每个 Skill 都可直接配置以符合个人品味。

**四大优点**：

1. **文件自主权**：NotebookLM 虽好用但 ChatGPT 无法调用，因为分属 Google 和 OpenAI。Obsidian 本地存储+云同步最符合个人利益。作者将 15500 条微博和 1250 篇知乎文章导入 Obsidian，"一辈子只需要建一次'数字分身'，值得投入"。

2. **跨模型记忆**：从 GPT 迁移到其他模型会丢失记忆。若不想被 AI 公司绑架，记忆最好存在本地。"谁便宜好用，咱们就换谁！" 作者目前用国产 Minimax2.5-HighSpeed，还想试智谱 GLM5。

3. **更友好的 Skills 接入**：从 Prompt（自己干活+AI 加速）→ Agent → Skill（雇人干活）。Skill 是 md 文档（标准 txt 格式），可根据需求自行优化，用不好的 Skill 可以"直接开除"。示例：热点收集 Skill、视频台词提取 Skill、概览图生成 Skill。目前应用最广泛的方式是 Claude Code，但命令行对多数人陌生，Obsidian 文档库窗口更友好。

4. **Obsidian 知识图谱**：文件越来越多后，检索和管理成问题——恰好是 Obsidian 的强项。标签和 wiki 双向链接功能将文档整理成知识图谱。以前手动整理，现在 AI 整理"效率腾飞 100 倍"。形成知识图谱后，能高效找回丢失的记忆（如作者让 AI 回忆与蔚来 ET5 的故事和换电经历），将真实经历加入文章显得真实不雷同。

**10 步部署指南（含避坑）**：

1. **安装 Obsidian**：免费开源软件
2. **安装 Claude Code**：`brew install node` → `npm install -g @anthropic-ai/claude-code`
3. **安装 Claudian 插件**：第三方公开插件
4. **接入大模型**：国产 GLM 或 Minimax。Minimax 配置：设置 `ANTHROPIC_BASE_URL`、`ANTHROPIC_API_KEY`、`ANTHROPIC_MODEL` 环境变量后启动 Obsidian
5. **导入微博**：油猴插件 weibo-achiever + python 脚本处理成 markdown
6. **导入知乎**：zhihu to markdown 插件逐个下载（只需一次）
7. **配置 PARA 架构**：一种文件夹分类管理方法
8. **配置记忆系统**：不配置真的没记忆（"一个只有 7 秒记忆的人说话，蠢得让我怀疑人生"），有现成 skill 可用
9. **配置改进系统**：self-improving skill
10. **思维方式进化**：重点在于 Skill。"如果你感觉 AI 太蠢了，那一定是没有配置合适的 Skill"——不要浪费时间自己琢磨，作者花了 4 天踩坑，指南可帮读者 1 天搞定

