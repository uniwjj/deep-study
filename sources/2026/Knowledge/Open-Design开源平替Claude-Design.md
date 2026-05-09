---
ingested: 2026-05-09
wiki_pages: [ai-agent/open-design]
---
## Open Design

**文章信息**：
- **原文标题**：最近爆火的 Open Design，居然是 Claude Design 开源平替？
- **来源**：https://www.toutiao.com/w/1864220706833419
- **作者**：互联眼界（广州极云智能科技有限公司法定代表人）
- **发表时间**：2026-05-04 09:50

---

**一段话总结**：

Open Design 是 Claude Design 的开源平替——输入需求，本地 Agent 即可生成网页原型/PPT/移动端界面/社交轮播/海报，沙盒预览后可导出 HTML/PDF/PPTX/ZIP。核心亮点：模型不绑死 Anthropic（支持 Claude Code、Codex、Cursor Agent、Gemini CLI、Qwen、Kimi 等）、Skills 按任务类型文件化（Web/SaaS/APP/Dashboard 各有专属 Skill）、通过 DESIGN.md 固化设计系统（颜色/字体/间距/组件/动效/品牌语气）、本地 daemon 架构直接操作真实项目文件、严格约束防"AI 味"生成。

---

**核心内容**：

**五大亮点**：

1. **不绑死 Anthropic**：本地已有的 Claude Code、Codex、Cursor Agent、Gemini CLI、Qwen、Kimi 都能当设计引擎

2. **Skills 文件化**：Web 原型/SaaS Landing/Dashboard/APP/社交轮播/PPT 各有专属 Skill，每个带规则、模板、参考和检查清单，模型按任务类型精准执行

3. **设计系统有规矩**：DESIGN.md 把颜色/字体/间距/组件/动效/品牌语气全写死，Agent 先读设计系统再动手，输出不乱飘

4. **本地 daemon 架构**：浏览器只是界面，核心是本地 daemon——负责扫 Agent、建项目目录、读写文件、管 SQLite、预览导出，Agent 在真实项目里写文件而非云端凭空生成

5. **认真治"AI 味"**：先问需求再动手，有品牌资料先提取，没真实数据就留空，绝不硬编"10x faster"这类虚的文案
