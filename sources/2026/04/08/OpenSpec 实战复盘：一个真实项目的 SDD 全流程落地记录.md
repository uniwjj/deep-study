---
ingested: 2026-05-09
wiki_pages: [ai-agent/openspec-sdd-practice]
---

## OpenSpec 实战复盘：一个真实项目的 SDD 全流程落地记录

**文章信息**：
- **原文标题**：OpenSpec 实战复盘：一个真实项目的 SDD 全流程落地记录
- **来源**：https://www.toutiao.com/article/7626241869428671014
- **作者**：吃瓜便利店
- **发表时间**：2026-04-08 12:35

---

**一段话总结**：

这是在真实企业项目中用 OpenSpec 跑通 SDD 全流程的落地复盘，作者团队是一个跑了几年的老系统。选 OpenSpec 的核心原因是其 Brownfield-first（增量描述变更）设计哲学适合已有系统。团队通过 MCP 连接器对接了 Azure DevOps、Confluence、Figma，定制了七步工作流：Explore → New → Implement → Code Review → Verify → Testing → Archive。跑完 2-3 个迭代后，代码质量和一致性显著提升，需求前置对齐减少了返工，Spec 归档后自动成为活的系统说明书。

![OpenSpec 实战工作流](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/a41a04e38f7c4c6b93a9f814a011d845~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667714&x-signature=tS%2Bk9mxPyXIOQ0oPNBanbINnm9A%3D)

---

**作者判断**：

作者以一线团队视角真实记录了 SDD 的落地过程，立场务实。作者认为"先花一天写 Spec"前期确实增加了时间成本，但随着落地推广，大家发现代码质量提升、测试 bug 减少，抵触逐渐消退。作者强调 SDD 不是从零开始的专属，Brownfield 项目同样适用，且"Spec 永远是 source of truth，代码改了 Spec 也必须跟着改"。最后作者引用 Thoughtworks 技术雷达和 EPAM 报告，认为 SDD 正从小众实践走向主流，本质是"Harness Engineering"——让 AI 跑在正确的马路上。

---

**核心内容**：

**选 OpenSpec 的三个原因**：
1. **Brownfield-first**：原生支持"增量描述变更"，不用把整个系统从头写一遍 Spec
2. **Fluid not rigid**：没有死板的流程步骤，随时回头改之前步骤的产出
3. **工具无关**：不挑 IDE、不绑模型，支持 20+ 种 AI 编码助手

团队额外通过 MCP 连接器对接 Azure DevOps（拉取 User Story）、Confluence（补充业务背景）、Figma（获取 UI 设计）。

**七步工作流详解**：

```
Explore → New → Implement → Code Review → Verify → Testing → Archive
```

**Step 1 Explore**：执行 `/opsx:explore`，AI 自动拉取 Azure DevOps 的 User Story，结合 Confluence 背景，多回合对话澄清需求模糊点。产出是一份"事前对齐清单"——把隐性假设变成显性问题。价值：逼团队把口头共识变成白纸黑字，提前对齐需求方与开发部门的理解。

**Step 2 New**：采用 `/opsx:new` + `/opsx:continue` 逐件创建产出物，每件人工 review：
- proposal.md → review Intent 和 Scope
- spec.md → review Delta Spec 的验收场景是否覆盖边界条件
- design.md → review 技术方案是否符合架构约束
- tasks.md → review 任务拆分是否合理、粒度是否够细

Delta Spec 格式示例（GIVEN-WHEN-THEN 句式）：
```
## ADDED Requirements
### Requirement: 审批流配置
The system SHALL allow admin to configure approval workflows.
#### Scenario: 创建新审批流
- GIVEN an admin user on the workflow config page
- WHEN the admin defines a new 3-step approval workflow
- THEN the workflow is saved and immediately available
- AND an audit log entry is created
```

**Step 3 Implement**：执行 `/opsx:apply`，AI 按 tasks.md 逐项实现，每完成一项自动打勾。有 Spec 时 AI 知道 MUST 做什么、哪些 Scenario 必须覆盖，产出质量显著提升。类比：没有 Spec 的 AI 像"天才实习生"，有 Spec 后像"有规范意识的工程师"。

**Step 4 Code Review**（团队定制的人工门禁）：定制内部 Code Review Skill，让 Copilot 基于 Spec + 团队规范审查 GIT 提交内容。检查维度：命名和分层约束、接口向后兼容性、BL Number 两种格式兼容、潜在安全问题。逻辑是"AI 审 AI"——人只需 review 发现的问题。

**Step 5 Verify**：执行 `/opsx:verify`，三个维度自动检查：
- Completeness：tasks 是否全部完成，specs 里每个 Requirement 是否有对应实现
- Correctness：实现是否符合 Spec 意图，边界场景是否覆盖
- Coherence：代码结构是否和 design.md 的架构决策一致

**Step 6 Testing**：开发人工验收，跑真实场景确认系统行为符合预期，"纯大师手作"。

**Step 7 Archive**：执行 `/opsx:archive`，两件事：Delta Spec 自动合并进主 Spec（`openspec/specs/`），变更文件夹归档到 `changes/archive/`。归档后 `openspec/specs/` 就是活的系统说明书，Archive 对抗 Thoughtworks 提出的"认知泄漏（Cognitive Leakage）"。

**踩过的四个坑**：

1. **微服务架构下的 SDD**：在 IDE workspace 中集成多个微服务，每个服务配独立 Spec 和 schema，通过 sub-agent 跨服务协作
2. **实现到一半发现 Spec 漏细节**：两种策略——完整流程（让 AI 同时改 Spec 和代码）或"悄咪咪改"（小错误手动改代码并同步更新 Spec），核心原则是 Spec 永远是 source of truth
3. **团队抵触"又多了一道流程"**：随着推广，大家看到代码质量提升、bug 减少，抵触消退
4. **工具迭代太快**：紧跟社区发展，定期 `openspec update` 刷新 agent instructions

**SDD 带来的真实改变（2-3 个迭代后）**：
- 更高的质量和一致性：AI 按结构化 Spec 实现完整 User Story
- 需求前置对齐，减少返工：Spec 阶段暴露歧义
- 嵌入式文档：Spec 即活文档，归档后成为系统说明书和测试基础

**挑战**：
- 学习曲线：从"写代码优先"到"写 Spec 优先"是范式转变
- Spec 维护成本：保持代码和 Spec 同步需要团队纪律性
- 工具成熟度：仍在快速迭代，偶尔踩坑

SDD 正从小众实践走向主流——Thoughtworks 技术雷达第 33 期已将 SDD 列为值得关注的趋势，EPAM《2026年软件工程趋势报告》也视其为关键方向。SDD 本质是"Harness Engineering"：真正稀缺的不是"让 AI 跑起来"，而是让 AI 跑在正确的马路上，并由人类牵着这匹马。

