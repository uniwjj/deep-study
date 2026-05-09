---
ingested: 2026-05-09
wiki_pages: [ai-agent/log-diagnosis-skill]
---

## 日志诊断 Skill：用 AI + MCP 一键解决BUG｜得物技术

**文章信息**：
- **原文标题**：日志诊断 Skill：用 AI + MCP 一键解决BUG｜得物技术
- **来源**：https://www.toutiao.com/article/7623715964521087494
- **作者**：得物技术（阿程）
- **发表时间**：2026-04-02 10:36

---

**一段话总结**：

得物技术团队开发了一个 `/log-diagnosis` Skill，将后端调 BUG 的固定流程——查日志→找关键信息→扫描代码→定位问题——自动化为一条命令完成的闭环。核心组合是 MCP + Skill：MCP 让 Claude Code 能通过 SSE 连接直接调用日志平台查询能力，Skill 给 AI 一份完整的行为规范（分页拉取、Token 自动刷新、跨服务分析、代码联动）。实战案例中，AI 仅凭一条 traceId，自动拉取 73 条日志、还原调用链路、从实际执行的 SQL 中发现了一个隐蔽的字段空字符串判断遗漏 BUG，并直接定位代码给出修复方案。文章认为，凡是"步骤固定、信息来源明确、输出格式可预期"的工作，都值得用 Skill + MCP 自动化。

---

**作者判断**：

作者的核心判断是：调 BUG 的过程逻辑固定、步骤繁琐但不需太多创造性思维，恰好是 AI 最擅长接管的工作。实现闭环靠两个关键组合——MCP 突破了"AI 只能处理静态上下文"的限制，Skill 把"一次性对话"变成"可复用的工程化能力"。原文原话："两者缺一不可。只有 MCP，AI 能查日志但不知道怎么系统地分析；只有 Skill，AI 有流程但没有数据来源。" 作者还发现 AI 在处理"同类字段逻辑不一致"类问题时比人工更好，因为 AI 没有"先入为主"的经验偏见。最后判断："AI 时代，工程师的核心竞争力不只是'能写代码'，更是'能把自己的经验和流程转化成可复用的 AI 能力'。"

---

**核心内容**：

**问题背景**

后端调 BUG 有一个固定的耗时流程：打开日志平台搜 traceId → 从几十上百条日志找关键条 → 复制类名方法名去 IDE 找代码 → 结合代码逻辑判断问题 → 找不准就重复。光在日志平台和 IDE 之间来回切换就消耗大半时间。

**方案演进**

最初考虑 Cursor + MCP + 代码知识库，但日志查询是"动态的"（依赖环境、应用、时间范围），无法静态预置。后来用 Claude Code 接触到 Skill 概念——可以在项目里定义自定义命令描述 AI 执行步骤，思路才清晰：日志平台有 MCP，Claude Code 有 Skill，两者结合形成闭环。

**MCP 机制**

日志平台基于 MCP 协议提供日志查询服务，Claude Code 通过 SSE 长连接与 MCP Server 通信。鉴权流程：`secretKey（后管申请）→ acquireTokenTool → accessToken（1小时有效，最多同时5个）→ 携带 accessToken 调用 logsQuery/logSqlQuery/countLogTool 等`。

![核心 MCP 工具列表](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/596b59faf7df4fd3bcbac24e59156fcb~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776615120&x-signature=LS9leH4xNzqt%2BXv2wRBoexkToUk%3D)

**/log-diagnosis Skill 核心能力**

用户输入 `/log-diagnosis {环境} {代码分支} {诉求}`，AI 自动执行完整链路：
1. 加载 SKILL.md → 读取环境配置 → 检查 accessToken（过期自动刷新）
2. 从 traceId 推算日志时间范围（取第 9-16 位 16 进制时间戳）
3. 调用日志平台 MCP **分页拉取全量日志（最多 20 页，禁止只查第一页就下结论）**
4. 切换到指定代码分支，结合日志关键词检索代码
5. 综合分析 → 生成诊断报告（飞书文档 or 本地 Markdown）→ 恢复原始分支

四个核心能力：Token 自动管理、分页全量拉取、跨服务分析、代码联动。

**安装配置**

- MCP 安装：Claude Code 执行 `claude mcp add --transport sse {name} {url}`，按环境分别添加
- Skill 安装：将 `log-diagnosis` 目录放到 `.claude/skills/` 或 `.cursor/skills/` 下
- 唯一需人工填写的字段：`secretKey`（日志平台后管申请），其余自动管理

**实战案例：隐蔽的 SQL BUG**

某搜索接口测试环境无返回数据，执行命令：
```
/log-diagnosis T1 feature/your-branch trace_id: "your-trace" 为什么最终没有返回数据
```

AI 自动完成：
1. 从 traceId 推算时间范围 → accessToken 过期自动刷新 → 分 2 页拉取 73 条完整日志
2. 还原调用链路，定位关键节点：`resultList is empty`，SQL 查询返回空结果——问题在 DB 层

![AI 自动还原完整调用链路](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/9d80b68135384a95978298f516fbfccf~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776615120&x-signature=abwduSNby5THRm1%2B2CvN2zj5rNE%3D)

3. AI 从日志提取实际执行的 SQL，发现根因：其他字段（`order_types`、`delivery_modes`、`ticket_sources`）都处理了 `IS NULL` 和 `= ''`（空字符串=不限制）两种情况，唯独 `customer_tag` 只判断了 `IS NULL`，遗漏了空字符串。DB 中该字段全存的空字符串，本应匹配所有请求，却被全部过滤掉。

4. 定位到 MyBatis Mapper XML 中的代码，给出修复：
```xml
<!-- 修复前 -->
<if test="customerTag != null">
  and (a.customer_tag IS NULL OR a.customer_tag = #{customerTag})
</if>
<!-- 修复后：加 a.customer_tag = '' -->
<if test="customerTag != null">
  and (a.customer_tag IS NULL OR a.customer_tag = '' OR a.customer_tag = #{customerTag})
</if>
```

BUG 的隐蔽性在于 SQL 语法正确、逻辑"看起来"没问题，只有对比其他字段写法才能发现。AI 没有"先入为主"的经验偏见，会对所有字段做同等审查，反而比人工更擅长发现这类横向对比差异。

![效率对比：人工 vs AI](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/cf9a9f3ebb9546b6aa330736fc24e92e~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776615120&x-signature=LUbmKk3FFd%2F3%2BnPuIeA%2FVETVs8s%3D)

**值得借鉴的思路**

1. 识别"固定流程"是自动化的起点——步骤固定、信息来源明确、输出格式可预期的工作都适合 Skill + MCP 自动化
2. Skill 的本质是"给 AI 写操作手册"——约束越明确（如"禁止只查第一页""必须分页拉完"），AI 执行质量越稳定
3. 类似场景可延伸：代码审查、性能分析报告生成、告警巡检等
