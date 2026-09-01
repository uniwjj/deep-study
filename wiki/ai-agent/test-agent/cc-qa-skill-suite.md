---
title: ccc-qa Skill 家族
description: 雷火 AI 测试链路的五个 skill 串联——用例编写、接口测试（模式A/B、9类参数分类、6层断言）、Playwright UI 铁律、年报测试、多语言功能/文案分离与报告发布
aliases: [ccc-qa, 雷火测试skill, api-test-skill, i18n-testing, playwright-mcp-testing]
tags: [ai-agent, tool, practice]
sources: [2026/09/01/游戏数据年报类活动AI测试实践分享.html]
created: 2026-09-01
updated: 2026-09-01
---

# ccc-qa Skill 家族：雷火 AI 测试链路

雷火沉淀的 AI 测试 skill 体系。主线依次串联：**编写用例 → 接口测试 → UI 测试 → 年报测试 → 多语言测试**，各阶段成果向下传递，横切一条测试报告发布线。年报是本次新建 skill，其余四个均为已有 skill 改造更新——**五个 skill 各司其职，层层委托复用，不重复造轮子**。

| Skill | 定位 | 输出 |
|-------|------|------|
| `ccc-qa-testcase-create` | 编写用例（已有，改造） | 用例基线 |
| `ccc-qa-api-test-skill` | 接口测试（已有，改造） | 接口链路验证 |
| `ccc-qa-playwright-mcp-testing` | UI 测试（已有，改造） | 通用 UI 能力 + 铁律 |
| `ccc-qa-annual-report-testing` | 年报测试（**本次新建**） | 中文年报全量覆盖（见 [[config-driven-data-testing]]） |
| `ccc-qa-i18n-testing` | 多语言测试（已有，改造） | 繁英泰验收 |
| `ccc-qa-html-report-publish-skill` | 报告发布（横切，已有，改造） | 在线测试报告一键发布 |

## 一、用例编写（ccc-qa-testcase-create）

产出两份用例：

- **UI 用例**：AUTO 通用操作 + BIZ 业务场景 + DATA 配表分支数据断言。AUTO 覆盖所有账号一致的通用页面操作（封面入口、通用交互、翻页浏览、导航操作、页面元素）；BIZ 覆盖任务流程、分享链路、门槛校验、数据埋点、SEO、反馈等业务场景（D90 实例共 24 条）；DATA 配表分支数据断言由年报 skill 阶段 3 单独处理。
- **接口用例**：多玩家类型串联链路 + 异常路径（D90 实例 CHAIN-001~021 共 19 条，覆盖 6 个接口全部调用路径 + 3 个玩家类型链路差异：看自己年报领奖、分享领奖、他人年报领奖、活动时间边界、玩家类型门槛、安全与越权、并发与缓存、基础接口）。

知识库沉淀：年报易错模式（三态混淆 AR-003、混合分支 AR-002、极值排版等）入 testcase-knowledge-base（新增 `09-annual-report.md`），写用例和断言时直接引用。

## 二、接口测试（ccc-qa-api-test-skill）

双模式架构，共用预处理：`qa-account-mcp → get_account → get_cookie → 写入 project.yaml`。

### 模式 A（文档驱动，Agent 大脑层）

输入接口文档（Swagger/OpenAPI/自定义文件/灵感链接），AI 按知识库自动生成用例。核心机制：

- 13 维知识库智能调度（skills/01~13）、测试策略自动生成、门槛判断、业务推理（跨接口关键链路）
- **9 类参数分类法**（Skill 02）：字段自动归类为 ID/内容/枚举/分页/时间/URL/数值/格式/布尔/数组/游标；不同类型 → 不同测试矩阵（ID 类只测 3 项，内容类注入+超长+敏感词）；优先级裁决：ID > 分页 > 枚举 > 时间 > URL > 数值 > 格式 > 内容
- 门槛① 接口收敛确认后启动；**Pre-check 前置自检**（真实数据激活？正常场景优先？列表排序/筛选？CRUD 联动？）；逐接口先正常再异常边界
- **6 层断言体系**（每个请求）：L1 HTTP 状态 → L2 业务码 → L3 Message 语义 → L4 数据结构 → L5 安全头 → L6 响应时间
- **Post-check 后置自检**（每个接口测完后）：17 项 checklist 强制过检，未全 ✅/N/A 禁止测下一接口
- `cascade_test.py` 并发串联（状态变更接口）：删除/下线后 ThreadPoolExecutor 并行验证 详情 404 + 列表不含 + 交互被拒 + 计数 -1 + 角色色不可见
- 门槛② 覆盖率 100% 验证；**错误码一致性矩阵**（覆盖率通过后全局分析）：联合所有错误码检测「同码不同义」+「同义不同码」，零额外请求
- 数据验证：真值 vs 模型，禁止编造

特长：检测隐藏缺陷、稳健性验证。

### 模式 B（用例驱动，机械执行层）

输入 YAML 用例文件（由编写用例 skill 生成），无知识库调度、无自主判断，严格按用例预期断言。特长：用例忠实执行（回归路线）。

- B.0 解析 YAML，按 P0→P1→P2 排序
- **三种参数数据源**：project-yaml（固定）/ qa-account-mcp（动态租借）/ auto（YAML 优先 + MCP 补充）；支持多角色 primary + secondary_1~N + extra
- **变量传递链**：`{step_N.data.xxx}`（顺序步骤响应）、`{ctx.xxx}`（用例声明的上下文变量）、`{base_url}`/`{primary_roleId}`（project.yaml）
- B.2 逐步执行：解析 `{role=xxx}` → 选凭证 → 变量替换 → requests 发请求 → 按用例期望断言 → 保存 ctx 变量 → 写 stepN.json
- 失败处理策略：断言 ✗ FAIL（模块执行，易被链路影响）/ 请求异常 / 变量提取失败 / 凭证缺失 → BLOCKED（跳过依赖步骤）
- **步骤数一致性校验**：实际执行步骤数 = 用例声明步骤数（少了回退补执行，多了删除自行添加的步骤）
- B.3 反哺优化建议：字段名差异（msg vs message）→ 建议更新用例；计数不匹配 → 区分是用例过时还是 BUG；每步 stepN.json 完整归档到 artifacts/

共用输出：Markdown 测试报告 | Python 回归脚本 | 缺陷追踪 | 用例报告。执行框架：Python requests，前端 `responseJson['code']`，pytest/unittest 断言。

## 三、UI 测试（ccc-qa-playwright-mcp-testing）

翻页切换、任务领取、分享链路、他人年报查看等所有账号一致的通用水路，按 UI 用例（AUTO + BIZ）走标准流程执行。**铁律**（供下游继承，不重复建设）：100% 覆盖不抽样、严格校验杜绝假通过、翻页依赖渲染就绪信号不固定 sleep、双端适配、失败自愈。本次改造：DATA 模式、is-active 翻页、逐屏验收、造数缓存检查、双端地址、新增 cookie-generators.md。

断言纪律（跨部门会讨论确认）：Playwright 仅负责执行动作，**断言逻辑完全交由用例及需求文档判定**——由 AI 将 PRD 中的预期结果「读」出来并代为判断（如：拉人进队后队伍列表必须出现该人信息），不让 AI 自由发挥，实现断言结果固定化。

## 四、多语言测试（ccc-qa-i18n-testing）

核心思路：**功能测试和文案测试彻底分离**——功能测试（点击流程、交互逻辑、接口）只在基准语言做一次，其他语言只做文案层四件事：

1. 文案匹配（页面实际文案 vs 配表该语言列逐条核对）
2. 溢出截断检测（翻译变长后是否撑破容器）
3. 翻译完整性校验（缺译漏译、占位符丢失）
4. 跨语言逐 key 对比（同一文案 key 各语言并排）

关键前提：**同代码换文案，DOM 选择器跨语言不变**——基准语言跑测试时生成的交互场景脚本（scenes.json，记录弹窗、Tab 等真实触发成功的选择器）直接复用到所有其他语言。执行方式：单语言 `--lang`，多语言批量 `--langs` 一次跑完并输出跨语言对比表；登录态按语言分别通过 account-mcp 生成 Cookie 注入；批量模式额外做时区/时间检测（基准比对法核对海外版本时区转换）。翻译文案来源是可动态扩展语言列的配表（POPO 在线文档，`text`/`text_en`/`text_cht`…，新增语言不改代码）；测试结果（通过/失败原因/未测试原因）回填本地 Excel 副本，**在线原文档只读，永不覆盖**。效果：多语言测试边际成本从「再来一遍完整功能测试」降到「跑一遍文案比对脚本」。

## 五、报告发布（ccc-qa-html-report-publish-skill）

统一一键发布在线测试报告。发布前过 `publish_precheck.py` 自检，**0 断链才放行**；截图压 JPEG q75、文件名纯 ASCII、链接正斜杠。年报测试额外产出：每账号汇总图（grid 平铺）+ 截图画廊（按用例分组 + lightbox）；统一 rsync 上传。

## 相关页面

- [[config-driven-data-testing]] — 年报测试 skill 的五阶段设计
- [[test-agent-harness-paradigm]] — 渲染就绪信号、确定性/概率分工的范式来源
- [[yanxuan-api-test-agent]] — 严选的接口自动化流水线（对照）
- [[ai-test-cross-team-review]] — 该 skill 家族在跨部门分享中的位置
