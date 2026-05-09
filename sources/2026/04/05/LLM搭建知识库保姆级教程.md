---
ingested: 2026-05-09
wiki_pages: [ai-agent/knowledge-base-karpathy-guide]
---

## 最近很火的用LLM搭建知识库的保姆级教程

**文章信息**：
- **原文标题**：最近很火的用LLM搭建知识库的保姆级教程
- **来源**：https://mp.weixin.qq.com/s/MpQAESTZgOMfDwtA6qpO2Q
- **作者**：未知
- **发表时间**：2026-04-05 15:21

---

**一段话总结**：

该教程基于 Andrej Karpathy 分享的知识库搭建理念，分四步教用户用 LLM 搭建个人或领域知识库。核心工具是 Obsidian（存储和阅读 Markdown）+ Web Clipper 插件（采集网页内容）。流程是：用 Obsidian 创建 vault 并设置 raw/wiki/output/images 四个文件夹，通过 Web Clipper 收集原始资料到 raw 目录，然后让 LLM（Claude Code / Cursor 等）将 raw 中的内容编译为结构化 wiki，用户可提问获得 output 输出，最后每周定期让 LLM 做知识库健康检查和增量更新。核心理念是用 AI 办公时代，知识库的丰富程度决定你和别人的差距。

---

**作者判断**：

作者认为 LLM 搭建知识库的理念来自 Andrej Karpathy，方法是明确的，但"每个人理解不一样，基于此能搭建到什么程度又不一样"。作者强调"通过搭建知识库，可以把你的资料积累起来，把你的经验固化"，并指出"在用 AI 办公的年代，你和别人用 AI 的程度，就看你知识库的丰富程度"。对最终效果，作者态度是："方法已经教给大家，修行就看个人自己了。能建成什么样的知识库来，也要看各位自己的水平了。" 本文为操作教程，作者观点集中在"知识库的重要性"层面。

---

**核心内容**：

**理念来源**：Andrej Karpathy 分享的方法，总共四步：收集原始资料→编译成 wiki→提问 & 输出→定期检查更新。

**知识库类型**：个人知识库（记录、经验）、领域知识库（网上收集的专业内容）、项目/调研知识库。

**工具选择**：LLM 可用 Claude Code / codex / Cursor 等；存储工具标配 Obsidian。

**第一步：收集原始资料**

1. 下载安装 Obsidian：https://obsidian.md/download

![Obsidian 下载安装界面](https://mmbiz.qpic.cn/sz_mmbiz_png/tsSmGjxQJnxDgkYDLicZdu4YzlS5yUxDGv7uibwTHXibcecibAXtkHSEd1Nv9KcwAwT4WAicsHf7kggXw7abf4GwMf7UAQhyIlgEurj45aLnLAK8/640)

2. 安装后创建仓库

![创建 Obsidian 仓库](https://mmbiz.qpic.cn/sz_mmbiz_png/tsSmGjxQJnyYzRWAP4XwybhCibdRMoEGL0picI8DRPWf2UMggAkZazWMN3QbP144JSkQZarmSmY3WBZBSp3BI0EqqQed1O0OkJtheoTA0zsjI/640)

3. 创建知识库文件夹结构：

```
你的知识库文件夹 (Vault)
├── raw/          ← 放所有原始文件
├── wiki/         ← LLM 自动生成结构化文章（不要手动改）
├── output/       ← 提问后生成的 Markdown、幻灯片等
├── images/       ← 图片
└── Claude/       ← 知识库搭建规则
```

4. 安装 Obsidian Web Clipper 插件，用于保存浏览的网页内容

![Web Clipper 插件](https://mmbiz.qpic.cn/sz_mmbiz_png/tsSmGjxQJnzJqXMia9ZfSplljsslnflSkOCNxcGYokDtFLkadiaEj2XC48kh6g82CicYibnpMSd6zericsbiccW0x9UkRc47Y5ibpqk3mmu3TkNtKY/640)

5. 插件设置中将保存地址设为 raw 文件夹地址（必须设置，否则影响后续流程）

![Web Clipper 设置界面](https://mmbiz.qpic.cn/sz_mmbiz_png/tsSmGjxQJnxkTDUibB3Sj3wfFHBClhHRcgnfcK6T6Ushl67ibNrw6GCL3kTJ6VMTydFrpMWB33NJZxibwDeQWL97pWeMlia5tC9omTfvNtE5dIQ/640)

**第二步：编译成 wiki**

让 LLM（Agent）将 raw 文件夹中的原始内容编译为结构化的 wiki Markdown 文件。wiki 类似百科，把知识点的所有内容及相关知识链接放到一个 Markdown 文件中。

**第三步：提问 & 输出**

基于编译好的 wiki 知识库，向 LLM 提问，生成 Markdown、幻灯片等输出内容到 output 文件夹。

**第四步：定期检查更新**

每周让 LLM 对 wiki 文件夹做健康检查，提示词模板：

1. 找出可能重复或矛盾的概念
2. 发现应该互相链接但还没连的概念
3. 建议新增或需要补充的概念页面
4. 检查并更新 INDEX.md
5. 如果 raw/ 有新内容，进行增量编译

**最终体验**：用 Obsidian 打开 wiki 文件夹阅读，核心亮点功能是"关系图谱"，直观看到知识之间的连接。建议日常记录习惯存为 md 文档，方便后续投喂智能体进行知识库更新。

