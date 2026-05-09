---
ingested: 2026-05-09
wiki_pages: [ai-agent/progressive-spec-methodology]
---

## 2026年AI编程新范式："渐进式Spec"

**文章信息**：
- **原文标题**：2026年AI编程新范式："渐进式Spec"
- **来源**：https://www.toutiao.com/article/7624929990026592802
- **作者**：程序员清风
- **发表时间**：2026-04-05 09:00

---

**一段话总结**：

AI 编码效果翻车的根本原因不是模型不够聪明，而是"Vibe Coding"这种协作方式出了问题。"渐进式 Spec"是一套根据任务复杂度灵活调整规范粒度的编码方法论——简单需求直接走、中等需求加规格文档、复杂需求走完整流程。核心是三条铁律：先 Spec 后编码、Spec 覆盖所有场景、发现偏差先更新 Spec 再改代码。工作流为四步闭环：Propose（人主导需求澄清）→ Apply（AI 按任务清单执行）→ Review（双阶段 Sub Agent 质检）→ Archive（知识沉淀归档）。

![AI 编码翻车的四大致命问题](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/c0356e8b176247e38d63e197e291b69c~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667709&x-signature=27va7c1xowJmGkiEDc4xL1mtrU8%3D)

---

**作者判断**：

作者认为"Vibe Coding 是口述传话，Spec Coding 是书面合同"，明确推荐 Spec Coding 方法论。作者的核心观点是"流程是可选增强，不是强制前提"——渐进式的真谛在于简单的事别搞复杂，复杂的事不能马虎。作者同时指出 2026 年的 AI 编码早已不是"对话框里提需求"的时代，呼吁开发者从今天开始给需求写 Spec。作者还特别强调"知识飞轮效应"才是真正的护城河——今天踩的坑变成规则，本周的方案变成模板，时间越长越值钱。

---

**核心内容**：

**AI 编码四大致命问题**：
- 接口命名和项目规范对不上
- 边界情况全漏了
- 改了三遍还是不对
- 2026年顶级模型已能独立完成中等复杂度编码任务（Chatbot Arena 数据），但真正头疼的是以上协作问题

**渐进式 Spec 的核心思想**：

![渐进式 Spec 核心概念](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/e0fc678406b64fae9b33b4fa63ef2ebf~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667709&x-signature=j6ux27oBP1iNMl%2BCsuax8Dgv9Ys%3D)

不是所有需求都要写长篇文档。根据任务复杂度灵活调整——简单需求直接走、中等需求加规格文档、复杂需求走完整流程。

![按需求复杂度选择不同的流程粒度](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/dcab8a44a13f485fa8cae86f0823acb2~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667709&x-signature=YqZ76QJ%2B77ZYBFKLqQnt2XiwHR4%3D)

**三条铁律**：

![渐进式 Spec 三条铁律](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/b617b445815249d786c1499faa941832~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667709&x-signature=s9ODTFfkfOYcOKi%2Fz3059haAxg8%3D)

1. 先 Spec 后编码
2. Spec 覆盖所有场景（WHEN/THEN 句式）
3. **Reverse Sync**：发现实现和预期偏差时，必须先更新 Spec 文档再修改代码，保证文档是"唯一真相源"

**四步闭环工作流**：

![Propose → Apply → Review → Archive 四步闭环](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/9c3ffa6a9bf94bec9fd79ff897f70802~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667709&x-signature=zUvpRIdwRxg028Phtr84EHkI6As%3D)

- **Step 1 Propose**：人主导，AI 辅助。AI 自动 Research 代码现状，分段提问收敛模糊需求，生成 spec.md + tasks.md + log.md
- **Step 2 Apply**：AI 主导，人审查。按 tasks.md 逐步执行，每个 task 提供可验证证据，自动 commit（每个 task 一个 commit）
- **Step 3 Review**：两阶段 Sub Agent 质检——Spec Reviewer（检查实现是否符合 spec）+ Code Quality Reviewer（检查代码质量/安全/性能），发现进入 Fix 循环
- **Step 4 Archive**：经验教训写入 knowledge/ 知识库，变更记录归档，Delta Specs 合并到主规格文档

**2026 主流 AI 编码工具选型**：

![工具选型参考（截至2026年4月）](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/7d20324e9dab4ff39be28930348fbcd8~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667709&x-signature=noojM6s6PEXvIi7HD%2FxgwrvExtg%3D)

**模型梯队划分**：

![2026年3月 AI编码模型能力梯队](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/4603d4a8a0bf40ca918aba0f7f5c3404~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667709&x-signature=o7DGDpB2Sh9yNfY6x4bCktuMVQc%3D)

根据 Chatbot Arena 最新 ELO 排名和多轮交互稳定性，分为四个梯队。实用策略：编排层用 T0/T1 强模型做决策和 Spec 生成，执行层用 T1/T1.5 模型写代码。

**推荐的项目级 AI 编码框架目录结构**：
- rules/ — 工程记忆（技术栈、编码规范、安全红线）
- knowledge/ — 团队集体智慧（踩坑记录、架构决策）
- changes/templates/ — 标准化工件模板
- agents/ — 不同角色的 Prompt 配置

**知识飞轮效应**：

![知识飞轮——用得越久越值钱的复利系统](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/315af29fcbbe4199ba6e1dc96ce81820~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667709&x-signature=WSU3bJh7%2BH16WHdhLMhzq2E%2Bb%2B4%3D)

"今天踩的坑，明天变成规则；本周的方案，下月变成模板。"时间越长，人和 AI 配合越默契，别人抄不走。

**新手最易踩的 5 个坑**：
1. 把讨论和命令混在一起用——没想清楚就让 AI 写
2. 在不同阶段要求错误的产出——调研阶段就要代码
3. 不维护 knowledge 库——踩过的坑不记录
4. 迷信某个特定工具——工具会变，方法论不变
5. 忽视 Git 规范——禁止 master 直接推送，每个 task 一个 commit
