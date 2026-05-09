---
ingested: 2026-05-09
wiki_pages: [ai-agent/openclaw]
---

## 狂揽4万星，换掉OpenClaw太爽了，5美元就能养个AI打工人

**文章信息**：
- **原文标题**：狂揽4万星，换掉OpenClaw太爽了，5美元就能养个AI打工人
- **来源**：https://www.toutiao.com/article/7626724484761829942
- **作者**：36氪（新智元）
- **发表时间**：2026-04-09 19:46

---

**一段话总结**：

Hermes Agent 是 Nous Research 今年 2 月推出的开源 Agent，上线不到两个月 GitHub 超 4 万星，v0.8.0 平均不到一周一个大版本。它是一个"会跟着你成长的 Agent"，核心是内置学习闭环——记忆沉淀技能，技能反哺训练，训练提升模型能力。三层闭环：第一层记忆（MEMORY.md + FTS5 跨会话检索），第二层技能（Agent 完成 5+ 次工具调用后自动生成结构化 skill 文件），第三层训练数据（批量轨迹生成 + Atropos 强化学习环境）。可在每月 5 美元的 VPS 上运行，一行命令安装。

---

**作者判断**：

作者立场高度看好 Hermes Agent，判断"私有AI的自进化时刻，可能真的来了"。作者认为 Nous Research 押注的路线是：Agent 不该只是一次性调用接口，而应该是私有的、常驻的、会积累的、最终能反哺训练的，这站在了云端 Agent 服务的反面。同时指出目前的"成长"仅发生在技能层和记忆层，模型本身的天花板取决于接入的大模型。

---

**核心内容**：

**项目概况**：Nous Research 出品，2026 年 2 月底上线，v0.8.0，平均不到一周一个大版本，贡献者超 240 人，合并 PR 达 1400 个。有用户反馈"切到 Hermes 太爽了，比 OpenClaw 响应速度快了太多倍"。

![Hermes Agent 官网介绍页："the agent that grows with you"，强调部署在用户服务器上、保留所学知识、运行越久越强大](https://p3-sign.toutiaoimg.com/tos-cn-i-tjoges91tu/7700187bc67aeac0dc5f23c1c3f699ae~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669594&x-signature=paj3qKLu%2Fn4BORvy%2BpFtw9EUmP4%3D)

**六大核心特性**：与你同在、越用越强、定时自动化、委派与并行、沙盒隔离、全网页与浏览器控制。可在每月 5 美元的 VPS 上运行，闲置时几乎不花钱。

**消息平台支持**：Telegram、Discord、Slack、WhatsApp、Signal、SMS、飞书、企业微信，一个 gateway 进程连通所有入口。

![Hermes Agent 六大核心功能特性展示图：多平台适配、学习成长、定时自动化、委派并行、沙盒隔离、浏览器控制](https://p26-sign.toutiaoimg.com/tos-cn-i-tjoges91tu/f4c2aae5166f760e128ecc5b2028c452~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669594&x-signature=WJaoaUMSdmNXGrlpjs2ANG5F34w%3D)

**三层闭环**：

**第一层：记忆**。内置 MEMORY.md（环境信息和历史教训）和 USER.md（用户偏好和工作习惯），支持基于 FTS5 的跨会话检索与 LLM 摘要。

**第二层：技能**。Agent 完成复杂任务（通常 5+ 次工具调用）后自动写成结构化 skill 文件（操作步骤、常见陷阱、验证方法）。下次直接调用 skill，发现更好做法会自动更新。有 Reddit 用户报告两小时内创建 3 个 skill 后执行效率明显提升。

**第三层：训练数据**。内置批量轨迹生成和 Atropos 强化学习环境，日常工具调用记录可直接训练下一代模型。记忆→技能→训练→模型能力→回到 Agent，这是 Nous Research 真正想跑通的链路。

**生态系统**：agentskills.io 是开放技能标准，第三方社区已长出 HermesHub（带安全扫描的技能市场）、hermes-workspace（网页端 GUI）、mission-control（多 Agent 管理面板）。

![Hermes Agent 技能中心界面：从 4 个注册表中发现、搜索和安装 643 个技能](https://p3-sign.toutiaoimg.com/tos-cn-i-tjoges91tu/a1e99e12b1cc608385005568bcc439c3~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669594&x-signature=k0XvmyHPuAoZGTZyBbjY0ryA17Y%3D)

**模型支持**：Nous Portal、OpenRouter、OpenAI、Anthropic、Google Gemini、xAI、z.ai、Kimi、MiniMax 及本地 Ollama，通过 `hermes model` 随时切换，不锁定任何厂商。

**版本演进路线**：v0.5.0（安全加固：50+ 项安全修复）→ v0.7.0（长期运行能力：可插拔记忆架构、凭证池轮换）→ v0.8.0（智能：后台任务自动通知、模型实时切换、MCP OAuth 2.1）。从安全→稳定→智能的路径反映了 Nous 对 Agent 产品形态的真实判断。

![Hermes Agent v0.8.0 "The intelligence release" 版本更新内容](https://p26-sign.toutiaoimg.com/tos-cn-i-tjoges91tu/5e3c087a54a4fe8864b42fbb305f8df5~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669594&x-signature=HsV87%2BeKWlEBX%2FSht6gGy4hWXQQ%3D)

**安装**：`curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash`。从 OpenClaw 迁移：`hermes claw migrate` 一键导入。

**团队**：Nous Research 2023 年成立，约 20 人，累计融资 6500 万美元（5000 万美元 A 轮由 Paradigm 领投）。四位创始人此前代表作是 Hermes、Nomos、Psyche 三个开源模型家族。训模型的人亲自做 Agent，Agent 数据回流训练——这很可能是一种设计。
