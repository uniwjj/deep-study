---
ingested: 2026-05-09
wiki_pages: [ai-agent/openclaw]
---

## OpenClaw 用户集体"叛逃" Hermes时代来临！

**文章信息**：
- **原文标题**：OpenClaw 用户集体"叛逃" Hermes时代来临！
- **来源**：https://www.toutiao.com/article/7626657854161732106
- **作者**：木可
- **发表时间**：2026-04-09 15:29

---

**一段话总结**：

Hermes Agent 两个月内 GitHub 飙到 17K+ Stars（文章写作时），内置 `hermes claw migrate` 命令专门从 OpenClaw 迁移数据。它解决的核心痛点是 AI 的"金鱼记忆"——传统 AI 每次对话从零开始，而 Hermes 拥有三层记忆架构（短期上下文 + 长期知识库 + 用户画像），跨会话、跨平台保持连续性。相比 OpenClaw 追求系统控制权，Hermes 追求自我进化——不需要手动编写脚本，在使用过程中自己学习、自己总结、自己优化。

---

**作者判断**：

作者立场鲜明看好 Hermes，用"叛逃"一词形容用户从 OpenClaw 迁移的现象。核心观点是 AI 应从"被动响应的工具"向"主动成长的伙伴"演进。作者认为一年后积累了上百个量身定制技能的 Hermes Agent 将成为"真正懂你的智能助手"。同时也提醒 OpenClaw 正在收缩权限，网络安全机构建议只在专用设备上运行。

---

**核心内容**：

**核心痛点**：传统 AI 工具有"金鱼记忆"，每次打开都需要重新自我介绍，上周教会它的流程这周又忘记。致命缺陷是没有记忆、没有成长、每次对话从零开始。

**Hermes 的核心差异**：

1. **内置学习闭环**：第一次整理会议纪要认真完成，第二三次后会主动问"要不要保存成可复用技能"，同意后沉淀为自动化技能。下次只需说"整理今天的会议"，格式、重点、输出方式按用户习惯来。

2. **三层记忆架构**：

![Hermes 三层记忆架构图：短期记忆、长期记忆、用户画像，以及内置学习、多平台联动、安全进化等配套能力](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/f1c38728a11649dc80a4d829724ff8c5~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669884&x-signature=vehlXciZiAlMKKDK5ij5exOvmYY%3D)

- **短期记忆**：当前对话上下文
- **长期记忆**：跨会话知识库，记录项目、偏好、工作习惯
- **用户画像**：持续学习建立深度理解模型

定期"反思"过去对话，主动更新知识，确保记忆不因时间推移而"遗忘"。

3. **多平台无缝切换**：支持 12+ 个消息平台（Telegram、Discord、Slack、WhatsApp、Signal、邮件、命令行），早上命令行、中午 Telegram、晚上 Slack，全程记得用户是谁。

**OpenClaw vs Hermes 对比**：

![OpenClaw vs Hermes 对比图：系统控制 vs 自我进化，高权限 vs 沙箱隔离，手动维护 vs 自主学习](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/8f8fc357ab3740de8513d3df9f9ef892~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669884&x-signature=xsVJCEJdSB%2Fdo807tm2d%2F9Rf1qA%3D)

- **OpenClaw**：追求绝对控制权，能直接操作本地文件、修改定时任务、自动化控制浏览器，但需要花大量时间配置，且存在提示词注入风险，建议在专用设备或虚拟机上运行
- **Hermes**：追求自我进化，原生支持 5 种沙箱环境（Docker、SSH、Singularity、Modal 等），代码执行在隔离容器中，MIT 协议真正开源免费，支持 18+ 种模型提供商

**四大使用场景**：
1. **信息追踪与知识管理**：持续跟踪领域动态，逐渐成为"第二大脑"
2. **跨平台协作中枢**：多平台消息统一处理
3. **定时自动化任务**：内置自然语言 Cron 调度
4. **开发者智能助手**：记住代码风格、常用框架、项目结构

**安装**：一行命令 `curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash`，Windows 需要 WSL2。

